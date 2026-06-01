# License Manager

The chango runtime ships every managed component on disk, but four of them
are paid products and require an explicit license before their admin UI page
becomes usable. v1 is intentionally minimal — license activation toggles UI
access on the per-page level; install/start is **not** gated yet.

## Licensed components

| Component | License required | Notes |
|---|---|---|
| ontul, kiok | No | shipped by default |
| **shannonstore** | Yes | per-customer |
| **neorunbase** | Yes | per-customer |
| **mium** | Yes | per-customer |
| **itdastream** | Yes | per-customer |
| postgres, polaris, trino, trino-gateway, kafka, schema-registry, spark, flink, ui-proxy | No | upstream / open source |

All four paid components keep their **sidebar menu item visible** so the
operator knows the page exists. Clicking the menu before activating a
license shows a *"License Required"* screen with a single link to **Settings
→ License Manager**.

## License types

| Type | `expiresAt` | UX |
|---|---|---|
| `eval` | required, ≤ 30 days from `issuedAt` | row turns `status:expired` automatically once `now ≥ expiresAt`; page gate closes again. |
| `full` | optional (omit / `0` → no expiry) | row stays `status:active` forever (until operator deactivates). |

`status:expired` rows stay in the table for visibility but are treated as
inactive by the page gate.

## Token shape

```
<base64url(payload-json)>.<base64url(RSA-SHA256(payload))>
```

Payload (compact JSON):

```json
{
  "component":  "shannonstore",
  "type":       "eval",
  "customer":   "Acme Corp",
  "issuedAt":   1717200000000,
  "expiresAt":  1719792000000
}
```

The signature is computed over the raw payload bytes with the cloudcheflabs
RSA private key. chango verifies it against the public key embedded inside
the chango jar at `chango-common/src/main/resources/license/license-pubkey.pem`.

## Operator flow

1. cloudcheflabs sends you a single-line token (`<base64>.<base64>`).
2. In chango admin UI, **Settings → License Manager**.
3. Paste the token into the textarea and click **Activate**.
4. The table shows the new row (component / type / customer / expires / status).
5. Open the matching component's sidebar menu — the page should now render
   normally instead of the *License Required* screen.

To revoke a license, click **Deactivate** on the row. The component's
sidebar page reverts to the *License Required* screen immediately on the
next render.

## API

| Method | Path | Body | Notes |
|---|---|---|---|
| `GET` | `/admin/api/license` | — | Returns every stored license with `status:"active"` or `"expired"`. |
| `POST` | `/admin/api/license` | `{"token":"..."}` | Verifies + stores. Replaces any existing license for the same component. **admin-only**. |
| `DELETE` | `/admin/api/license/<component>` | — | Removes the license for the component. **admin-only**. |

Reads are local on every node (the snapshot is replicated cluster-wide on
every mutation and pulled at startup); writes are leader-forwarded
automatically.

## Cluster-wide replication

The leader's `LicenseManager` is the sole writer to its KMS-encrypted RocksDB
under `<chango.base.data.dir>/master/license/`. Every activate/deactivate
broadcasts `LICENSE_SYNC_PUSH` (opcode 241) to every follower + NM; the
periodic leader-pull also calls `LICENSE_FETCH_REQ` (242) at startup and on
the configured interval, so a follower restart never loses the cache.

KMS key id: `license` (separate from `iam` / `metadata-encryption`).

## What v1 does NOT do

- **install/start gate** — operators *with API access* can still
  `POST /admin/api/<comp>/install` and bring the component up without a
  license. v2 is expected to enforce in the cluster-service layer.
- **cluster-id binding** — one token works on any chango cluster that has
  the matching public key. v2 will likely embed a customer-binding field
  the master verifies against its own master-key fingerprint.
- **phone-home / online revocation** — chango is air-gapped friendly; the
  only way to invalidate a token mid-life is `DELETE /admin/api/license/<comp>`.
- **expiry warning banner** — v2 will surface "7 days left" notices in the
  admin UI top bar.

## Issuer side (cloudcheflabs internal)

The `tools/license-issuer/chango-license-sign.sh` script in the chango repo
signs payloads with the RSA private key. **It is not packaged into the
chango distribution** — `package.sh` does not look at `tools/`, and the
private key is gitignored. Keep the key off shared infrastructure until
cloudcheflabs ships a dedicated issuance service.

```bash
export CHANGO_LICENSE_PRIVATE_KEY=/path/to/license-private.pem
chango-license-sign.sh \
  --component shannonstore --type eval --customer "Acme Corp" \
  --expires-at "2026-07-01T00:00:00Z"
```
