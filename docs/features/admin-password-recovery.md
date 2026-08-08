# Admin Password Recovery

Chango ships with a built-in recovery channel that lets an operator reset the
`admin` user's password **without stopping the master**, even when no one
remembers the current password.

Recovery uses a local Unix domain socket — there is no HTTP back-door, no
network endpoint, no recovery URL. Authentication is performed by the
operating system: only a process that already shares the master's filesystem
identity can open the socket.

## When to use

- The admin password was forgotten or rotated out of the password manager.
- An automation script needs to provision a known admin password during
  first-time setup, without going through the web UI.
- A new operator is onboarded and you need to hand them an admin credential.

For day-to-day password changes (the user remembers the old password and
wants a new one), use the Admin UI's **Change Password** screen instead —
that path requires the old password and does not flag the user for forced
rotation.

## How it works

```
┌────────────────────┐   JSON over UDS    ┌────────────────────┐
│  chango-cli        │ ─────────────────▶ │  Master (running)  │
│  iam:reset-password│                    │   AuthManager      │
└────────────────────┘                    │   .adminResetPwd() │
         ▲                                │                    │
         │ stdout: new password           │  saveToDb()        │
         │ (one-time)                     │  cluster sync push │
                                          │  audit log append  │
                                          └────────────────────┘
```

| Property | Value |
|---|---|
| **Socket path** | `<chango.base.data.dir>/master/admin.sock` (mode `600`) — for example, `/opt/chango/data/master/admin.sock` in the default install layout |
| **Authentication** | OS file permission — same user as the master process |
| **Network surface** | none — Unix domain socket only |
| **Downtime** | none — applied in-process on the live master |
| **Cluster sync** | automatic — leader pushes the new IAM snapshot to follower masters and node managers |
| **Audit log** | `<chango.base.data.dir>/master/iam-audit/reset.log` (mode `600`, append-only) |
| **Post-reset state** | `requirePasswordChange = true` (forced rotation on next login) |

## Quick start

The simplest invocation lets the master generate a strong 20-character
password and print it to stdout. The new password must be changed on the
admin's next login (the `requirePasswordChange` flag is set automatically).

Run as the OS user that owns the master process (typically `chango`) on
the master host. `JAVA_HOME` must point at Java 17, and `CHANGO_ADMIN_SOCKET`
must be an absolute path to the live master's socket:

```bash
export JAVA_HOME=/opt/openlogic-openjdk-17.0.7+7-linux-x64
export CHANGO_ADMIN_SOCKET=/opt/chango/data/master/admin.sock
cd /opt/chango
bin/chango-cli.sh iam:reset-password
```

!!! note "Why the explicit socket path"
    The default discovery resolves `<chango.base.data.dir>/master/admin.sock`
    relative to the master's working directory. When the CLI is launched
    from a different working directory the relative path will not resolve;
    setting `CHANGO_ADMIN_SOCKET` to the absolute path avoids that. If you
    see an error like
    `admin socket not found at /opt/chango/${chango.base.data.dir}/master/admin.sock`,
    the property placeholder was not expanded — fall back to the explicit
    `CHANGO_ADMIN_SOCKET` or `--socket` override shown here.

Sample output (TTY):

```
  ┌────────────────────────────────────────────────────────────┐
  │  Password reset for user: admin                            │
  │                                                            │
  │  New temporary password: K3p9WvTx7qLm8zXa#-Bd              │
  │                                                            │
  │  Must be changed on next login (requirePasswordChange).    │
  └────────────────────────────────────────────────────────────┘
```

## Input modes

| Mode | Command | When to use |
|---|---|---|
| **Master-generated** | `chango-cli.sh iam:reset-password` | Default. Strong random password printed once on stdout. |
| **Explicit** | `chango-cli.sh iam:reset-password --new-password 'My!Pass'` | Automation that knows the desired value. Beware: argv may show up in `ps`. |
| **Stdin** | `echo 'My!Pass' \| chango-cli.sh iam:reset-password --new-password -` | Automation that wants to avoid argv exposure. |
| **Interactive** | `chango-cli.sh iam:reset-password --interactive` | Operator at a TTY. Prompts for password twice with no echo. |

Resetting a different user is also supported:

```bash
bin/chango-cli.sh iam:reset-password --user some-user --new-password 'NewPass123'
```

## Configuration

These are the shipped defaults in `conf/chango.properties` — the recovery socket
is **on** out of the box:

```properties
# conf/chango.properties
chango.admin.socket.enabled = true
chango.admin.socket.path    = ${chango.base.data.dir}/master/admin.sock
chango.iam.audit.dir        = ${chango.base.data.dir}/master/iam-audit
```

To remove the local recovery path entirely (for example in a hardened
production deployment), set the one key:

```properties
chango.admin.socket.enabled = false
```

### How the CLI finds the socket

Re-deriving the socket path from `conf/chango.properties` is not reliable on its
own: `chango.base.data.dir` can be overridden with `-D` at launch or edited after
startup, and the file records nothing about which value the live process used. So
the master **publishes the absolute path it actually bound to** into
`<chango.home>/bin/master.socket` when the socket comes up, and deletes that file
on shutdown. `bin/chango-cli.sh` prefers it.

Resolution order, highest priority first:

1. `--socket /path/to/admin.sock` — read by the Java CLI, always wins.
2. `$CHANGO_ADMIN_SOCKET` — if exported in the caller's shell. Must be absolute.
3. `<chango.home>/bin/master.socket` — the path published by the running master.
   Used only when the file exists *and* the path inside it is a live socket.
4. `chango.admin.socket.path` from `conf/chango.properties`, with both
   `${chango.base.data.dir}` and `${chango.home}` expanded. A value still holding
   a `${...}` placeholder is rejected rather than used literally.
5. `<chango.base.data.dir>/master/admin.sock`, then `<package>/data/master/admin.sock`.

Step 3 is what makes a relocated data dir work without extra flags: start the
master with `-Dchango.base.data.dir=/var/lib/chango/data` and the socket moves to
`/var/lib/chango/data/master/admin.sock` while `chango.properties` still reads
`./data` — only the marker knows where the live master bound.

```bash
cat /opt/chango/bin/master.socket
# /var/lib/chango/data/master/admin.sock
bin/chango-cli.sh ping
# pong
```

Unlike ontul/kiok/mium/itdastream/neorunbase/shannonstore, chango has **no**
`admin.socket.marker.file` key — the marker name is fixed at `master.socket`.

It is also no longer necessary to pass `--socket` just because the CLI is invoked
from another directory: the marker holds an absolute path, so `cwd` does not
matter. Use `--socket` only when the master is stopped (no marker) or you are
reaching a socket the marker does not describe.

### When the master key is needed

`ping` and `iam:reset-password` do **not** need `CHANGO_MASTER_KEY` — the running
master holds the unsealed key and does the work.

The key **is** required for the subcommands the master gates on it server-side
(`master-key show/rotate`, `kms rewrap`, `backup s3-config`): the CLI sends a
fingerprint of `$CHANGO_MASTER_KEY`, never the raw key, and the master rejects the
request if it does not match its own active key.

```bash
export CHANGO_MASTER_KEY='…the cluster master key…'
bin/chango-cli.sh master-key status
```

## Security model

**1. The socket is OS-gated.**
At startup the master creates `<chango.base.data.dir>/master/admin.sock`
with mode `600` (owner read/write only). Even other unprivileged users on
the same host cannot connect. There is no token, no shared secret, no
network listener.

**2. The audit log records every reset.**
Every successful reset appends a JSON line to
`<chango.base.data.dir>/master/iam-audit/reset.log` (mode `600`). The
plaintext password is **never** logged — only the first 8 characters of
its hash, the user, whether it was master-generated, and the OS user that
invoked the CLI.

```json
{"ts":"2026-05-23T16:00:46.922Z","event":"iam.reset-password","user":"admin","generated":true,"hashFp":"OYV/Ojf/","invokedAs":"root"}
```

**3. The new password is exposed exactly once.**
For master-generated passwords, the plaintext is returned only on the
single CLI invocation that triggered the reset. It is not written to the
audit log, not stored in RocksDB, and not retransmitted. Treat scrollback
and shell history accordingly — or use stdin input mode to avoid argv
exposure entirely.

**4. Forced rotation on next login.**
After reset, the user is flagged `requirePasswordChange = true`. The next
successful login forces the user through the change-password flow, so a
temporary password used by the operator is immediately replaced by a
password only the user knows.

**5. The master must be running.**
Because the recovery channel is in-process, the master must be alive for
the CLI to connect. This is intentional: RocksDB requires an exclusive
lock, so an offline edit would either conflict with a running master or
need a complex stale-lock recovery. With this design, the only way to
reset is to be on the host *and* have the master running *and* share its
filesystem identity.

## Limitations

- **Master must be running.** If the master is down (e.g. KMS unseal
  failure during boot), this CLI cannot help. The recovery path in that
  case is to inspect the boot failure, fix the underlying issue, and let
  the master come up — then run the CLI.
- **No knowledge factor.** Any process that shares the master's filesystem
  identity can invoke the CLI. In multi-tenant or shared-shell
  environments, restrict shell access to the master accordingly. A future
  enhancement may add an opt-in "recovery key" requirement for an
  additional knowledge factor.

## Related

- [Identity & Access Management](iam.md) — users, groups, policies
- [Encryption & Key Management](kms.md) — how IAM is encrypted at rest
