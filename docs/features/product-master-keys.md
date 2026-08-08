# Product master keys

Every managed product — ontul, kiok, shannonstore, neorunbase, itdastream, mium — encrypts its **own** state under its **own** master key: its IAM store, saved connections, per-bucket DEKs, job history. Chango installs those products, so chango is where that key is entered, stored and retrieved.

This page covers the whole lifecycle: where the key lives, how to read it back, what happens to it when you delete a cluster, and why a reinstall over surviving data must present the same key.

Not to be confused with chango's *own* master key (`$CHANGO_MASTER_KEY`), which protects chango's control plane — see [Encryption & Key Management](kms.md).

## Where the key lives

At install time the key is a required field (minimum 32 characters):

```json
POST /admin/api/ontul
{
  "clusterId": "ontul-prod",
  "masterKey": "…at least 32 chars…",
  "zkNodes": ["nm-…"], "masterNodes": ["nm-…"], "workerNodes": ["nm-…"]
}
```

Chango then:

1. Stores it in the cluster record as `settings.masterKey`, inside the metadata RocksDB — which is itself envelope-encrypted under chango's own KMS (see [KMS](kms.md)). Reading the RocksDB files without `$CHANGO_MASTER_KEY` yields nothing.
2. Passes it to every instance of that cluster as an environment variable (`ONTUL_MASTER_KEY`, `KIOK_MASTER_KEY`, …) in the component's launch config, so the product derives its own keys from it.
3. Replicates the record to follower masters over the normal metadata sync.

**The key is never returned over REST.** Cluster GETs filter `settings.masterKey` out, alongside `pgPassword`, `clientSecret`, `s3SecretKey` and friends. The admin UI never sees it after the install form is submitted.

## Reading a key back

The only path is the master's local Unix domain socket (mode 600), and it requires **two** factors:

1. **OS-level access** — you must be able to connect to the master's admin socket
   (`<base.data.dir>/master/admin.sock` by default), i.e. run as the master's own
   filesystem identity. You do not have to know that path: the running master
   publishes the one it actually bound to into `<chango.home>/bin/master.socket`
   and the CLI reads it from there, so a relocated data dir needs no extra flags.
   See [Admin Password Recovery](admin-password-recovery.md#how-the-cli-finds-the-socket)
   for the full resolution order.
2. **Proof of chango's master key** — the CLI sends the *fingerprint* of `$CHANGO_MASTER_KEY` (never the raw key) and the master matches it against its own. Export it before running any `master-key` subcommand; without it the master rejects the request.

Note `sudo -E`: it preserves `CHANGO_MASTER_KEY` across the user switch. Plain
`sudo -u chango` drops the variable and the command fails on factor 2.

```bash
$ export CHANGO_MASTER_KEY=<from your secret store>
$ sudo -u chango -E /opt/chango/bin/chango-cli.sh master-key list
CLUSTER-ID                   PRODUCT        STATUS     VERSION    MASTER-KEY
ontul-prod                   ontul          RUNNING    1.0.0      yes
kiok-prod                    kiok           RUNNING    1.0.0      yes

$ sudo -u chango -E /opt/chango/bin/chango-cli.sh master-key show --cluster ontul-prod
ontul-prod (ontul) master key:
…the key…
```

On a TTY the output is labelled; piped or redirected it prints the key alone, so `show … > key.txt` is safe to script.

!!! note "Run it on the leader"
    The socket serves that master's **local** metadata store. A follower trails the leader by one snapshot broadcast, so a cluster installed or deleted seconds ago may not be visible yet. Reads on a follower print a warning naming the leader; mutating commands (`rotate`, `purge`) are refused there outright.

## Deleting a cluster retires the key

`DELETE /admin/api/<product>/<clusterId>` stops every instance and removes their install directories. It does **not** remove state kept outside those directories:

| Product | Durable state outside the install dir |
|---|---|
| neorunbase, mium | Their catalog in an **external PostgreSQL**, which delete never drops — so their state *always* survives |
| ontul, kiok, shannonstore, itdastream | Only if `<product>.base.data.dir` was overridden to an absolute path. The default `./data` sits inside the install dir and is wiped with it — but pointing it at a separate data disk, as production ShannonStore does, makes the data outlive the cluster |

Surviving ciphertext plus a destroyed key is unrecoverable. So delete **retires** the key instead of destroying it, recording the fingerprint, the deletion time, and *what the key was protecting*:

```bash
$ sudo -u chango -E /opt/chango/bin/chango-cli.sh master-key retired
DELETED-CLUSTER              PRODUCT        KEY-FINGERPRINT    DELETED      STILL-PROTECTS
nb-prod                      neorunbase     51e8dbe099c0c9db   2026-08-08   PostgreSQL 10.0.1.5:5432 database 'neorunbase'
kiok-old                     kiok           988fd73d505dbffc   2026-08-08   data directory /data/kiok on [nm-…-19998]
```

`master-key show --cluster <deleted-cluster>` keeps working after the delete — it falls back to the retired vault — so "back the key up before deleting" is no longer something you have to remember in advance.

Retired keys are also listed over REST, **fingerprints only, never key material**:

```bash
curl -H "Authorization: Bearer $TOK" "$BASE/admin/api/retired-keys?componentType=ontul"
```

```json
[{
  "clusterId": "ontul-old",
  "componentType": "ontul",
  "fingerprint": "a3f91c04d5e6b7c8",
  "deletedAt": 1786150178485,
  "retainUntil": 2101510178485,
  "retainElapsed": false,
  "boundTo": "data directory /data/ontul on [nm-…-19998]"
}]
```

## Reinstalling over surviving data

If an install would sit on state a retired key protects, chango refuses it — **before touching any node** — rather than producing a cluster that cannot read its own data:

```
HTTP 400
This install reuses PostgreSQL 10.0.1.5:5432 database 'neorunbase', which still holds
state encrypted by deleted neorunbase cluster 'nb-prod' (master key fp=51e8dbe099c0c9db).
The supplied key (fp=84187e1a992e2c53) cannot decrypt it. Either reuse that key — pass
"retiredKeyRef":"nb-prod" instead of a raw masterKey — or, if the surviving state is
meant to be discarded, drop it first (the PostgreSQL database / the data directory) and
purge the retired key with 'chango-cli master-key purge --cluster nb-prod'.
```

Two bindings count as "the same state", and only these two, because they are the ones that genuinely survive:

- **The same external PostgreSQL endpoint *and* database.**
- **The same absolute `base.data.dir` on a host the old cluster used.**

An install base dir alone does *not* count — every cluster gets its own `<installBaseDir>/<instanceId>` subtree, so reusing the base dir reuses nothing. Installing with a fresh key at a *different* data dir is allowed and untouched.

### Reusing the retired key

Pass `retiredKeyRef` — the deleted cluster's id — instead of `masterKey`. The master substitutes the key server-side, so raw key material never crosses the REST boundary:

```json
POST /admin/api/neorunbase
{
  "clusterId": "nb-prod-2",
  "retiredKeyRef": "nb-prod",
  "zkNodes": ["nm-…"], "coordinatorNodes": ["nm-…"], "datanodeNodes": ["nm-…"],
  "pgHost": "10.0.1.5", "pgDatabase": "neorunbase", "pgUser": "postgres", "pgPassword": "…"
}
```

The admin UI does this for you: when retired keys exist for the product you are installing, the create form lists them by fingerprint and what each still protects, with "use a new master key" as the default. Selecting one sends `retiredKeyRef`.

### Discarding the data instead

If the surviving state is genuinely meant to go away, drop it and purge the key — in that order:

```bash
# 1. Drop the data the key protects
psql -h 10.0.1.5 -U postgres -c 'DROP DATABASE neorunbase;'
#    …or remove the absolute data directory on each host

# 2. Then destroy the key (leader only, irreversible)
sudo -u chango -E /opt/chango/bin/chango-cli.sh master-key purge --cluster nb-prod
```

`purge` asks for confirmation on a TTY (`--yes` to skip) and is **refused on a follower**: a follower's local delete neither replicates nor survives the next leader snapshot, so it would report a key destroyed that still exists. The REST equivalent, `DELETE /admin/api/retired-keys/{clusterId}`, is leader-only for the same reason.

## Retention

**Nothing expires a retired key automatically.** `retainUntil` is advisory only — it appears in the REST listing so a UI can flag old entries, and nothing acts on it.

That is deliberate. Silently destroying a key on a timer would recreate exactly the failure this mechanism exists to prevent, and the key may still be the only thing that can open a backup archive.

The hint is computed as:

| Situation | `retainUntil` |
|---|---|
| `backup.retention.days > 0` | `max(backup.retention.days, chango.retired.key.min.retention.days)` from the deletion date |
| `backup.retention.days = 0` (archives kept forever) | `chango.retired.key.max.retention.days` (default 3650) |

```properties
# chango.properties
chango.retired.key.min.retention.days = 30      # floor even with backups disabled
chango.retired.key.max.retention.days = 3650    # used when archives never expire
```

The tie to [backup retention](backup.md#retention) is the point: an archive is worthless without the key that opens it, so a key must not be considered stale while archives sealed under it still exist.

## Storage and replication

Retired keys are ordinary config rows (`retired-key:<clusterId>`) in the metadata store, so they inherit its envelope encryption and leader → follower replication. Deleting one on the leader propagates; a follower restart does not resurrect it.

They are included in the [backup archive](backup.md) along with the rest of the metadata store — which means a restored control plane still knows how to open the data of clusters that were deleted before the backup was taken.

## Product key vs chango key — quick reference

| | Product master key | chango master key |
|---|---|---|
| Protects | That product's own state (IAM, connections, per-bucket DEKs) | chango's control plane (KMS keystore → metadata + IAM) |
| Supplied | Install form / `masterKey` in the install request | `$CHANGO_MASTER_KEY` env on every chango process |
| Stored by chango | Yes — `settings.masterKey`, envelope-encrypted | **No** — process memory only |
| Read back with | `chango-cli master-key show --cluster <id>` | Never; compare `master-key fingerprint` instead |
| Rotation | Product-specific; chango does not rotate it for you | `chango-cli master-key rotate` — see [KMS](kms.md#rotating-the-master-key) |
| On cluster delete | Retired, not destroyed | Unaffected |
