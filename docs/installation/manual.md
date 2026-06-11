# Manual Install

This page brings up a chango cluster by running shell commands directly on each host — no ansible, no automation. Use it when you want full visibility of every step, when you are evaluating chango on a single host, or when ansible is not available on the controller (truly air-gapped, restricted network policy, etc.).

The end state is identical to the [automated install](automated.md): masters running on the chosen master hosts, a bundled ZooKeeper quorum running on the chosen ZK hosts (master and ZK groups are independent — master count is decoupled from ZK count), a node manager on every host, no systemd units, the cluster master key held only in operator-controlled secret storage.

## What you will install on each host

A host can play more than one role — in the HA default, host1/host2 are master + ZK + node manager, host3 is ZK + node manager. The columns below are the three roles, not three machines.

| Step | Master | ZooKeeper | Node manager |
|---|---|---|---|
| Rocky 9 prep (SELinux / ulimit / sysctl) | ✓ | ✓ | ✓ |
| Java 11 (for Spark + Livy) | ✓ | ✓ | ✓ |
| Java 17 | ✓ | ✓ | ✓ |
| Java 25 (for Trino) | ✓ | ✓ | ✓ |
| chango distribution under `/opt/chango` | ✓ | ✓ | ✓ |
| bundled ZooKeeper first start (`start-zk.sh`) | — | ✓ | — |
| chango master first start (`start-master.sh`) | ✓ | — | — |
| chango node manager first start (`start-node-manager.sh`) | — | — | ✓ |

The chango tarball is identical on every host. A host's role is decided by which start script you run there, not by which package you install — so install the same distribution everywhere and start the right processes per role.

The HA-default end state is **2 masters + 3-node ZooKeeper quorum + 3 node managers**, matching the [automated install](automated.md). For a quick single-host eval, run all three roles on one box.

## 1. Rocky 9 host prep

On **every** host (master + node managers), follow [Node Preparation](node-preparation.md) — SELinux disabled, ulimit and sysctl drop-ins, dedicated `/opt` mount. Quick recap of the manual steps:

```bash
# SELinux
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config

# ulimit
sudo tee /etc/security/limits.d/chango.conf > /dev/null <<'EOF'
*  soft  nofile  131072
*  hard  nofile  131072
*  soft  nproc   128000
*  hard  nproc   128000
EOF

# sysctl
sudo tee /etc/sysctl.d/99-chango.conf > /dev/null <<'EOF'
vm.max_map_count = 262144
net.core.somaxconn = 4096
fs.file-max = 2000000
EOF
sudo sysctl --system
```

## 2. Install Java 11, Java 17, and Java 25

Chango itself runs on **Java 17**. **Spark and Livy** run on **Java 11** (Livy 0.8 was built against Java 8/11 and breaks on JDK 17's module system; Spark 3.5 standalone is most stable on 11 too). Trino — deployed later via the admin UI on the node managers — needs **Java 25**. Install all three on every host so any host can run any component.

```bash
# Java 11 — Spark + Livy
curl -L -O https://github.com/cloudcheflabs/chango-libs/releases/download/chango-comps/openlogic-openjdk-11.0.27+6-linux-x64.tar.gz
sudo tar -xzf openlogic-openjdk-11.0.27+6-linux-x64.tar.gz -C /opt

# Java 17 — chango master / NM / most components
curl -L -O https://github.com/cloudcheflabs/chango-libs/releases/download/chango-comps/openlogic-openjdk-17.0.7+7-linux-x64.tar.gz
sudo tar -xzf openlogic-openjdk-17.0.7+7-linux-x64.tar.gz -C /opt

# Java 25 — Trino
curl -L -O https://github.com/cloudcheflabs/chango-libs/releases/download/chango-comps/openlogic-openjdk-25.0.3+9-linux-x64.tar.gz
sudo tar -xzf openlogic-openjdk-25.0.3+9-linux-x64.tar.gz -C /opt

# Verify
/opt/openlogic-openjdk-11.0.27+6-linux-x64/bin/java -version
/opt/openlogic-openjdk-17.0.7+7-linux-x64/bin/java -version
/opt/openlogic-openjdk-25.0.3+9-linux-x64/bin/java -version
```

All three JDKs land under `/opt/`. Chango does **not** want them on `PATH` or set as the system `JAVA_HOME` — the start scripts take `JAVA_HOME` from the environment of the shell that launches them, and the node manager picks the right JDK per component from `chango.nodemanager.component.java.homes`.

## 3. Create the `chango` user and base directories

On every host:

```bash
sudo groupadd -r chango || true
sudo useradd -r -g chango -s /bin/bash -d /opt/chango -M chango || true
sudo install -d -o chango -g chango /opt/chango /var/lib/chango /var/log/chango
```

Grant passwordless sudo for the `chango` user — chango needs it to deploy components (port allocator, host file sync, component install):

```bash
echo 'chango ALL=(ALL) NOPASSWD:ALL' | sudo tee /etc/sudoers.d/chango
sudo chmod 0440 /etc/sudoers.d/chango
```

## 4. Download and extract chango

On every host:

```bash
curl -L -O https://github.com/cloudcheflabs/chango-pack/releases/download/chango-archive/chango-3.0.0.tar.gz
sudo tar -xzf chango-3.0.0.tar.gz -C /opt/chango --strip-components=1
sudo chown -R chango:chango /opt/chango
```

This is the **lean** distribution — chango master + node-manager binaries, bundled ZooKeeper, admin UI. Component packages (Trino, Spark, Flink, ShannonStore, Ontul, NeoRunBase, ItdaStream, kiok, Mium, …) are pulled separately by the master at component-install time.

If you want to pre-stage component packages on the master host (faster first install, useful in air-gapped environments), run:

```bash
sudo -u chango bash /opt/chango/build-with-comps/download.sh
```

This downloads every component tarball into `/opt/chango/components/`. The master reads from there when an operator clicks **Install** in the admin UI.

## 5. Generate the cluster master key

Once, on the master host:

```bash
export CHANGO_MASTER_KEY=$(openssl rand -base64 48 | head -c 48)
```

Treat this value the same way you would treat a root credential — it is the root of trust for every secret chango stores. Keep it in a real secret manager (Vault, AWS Secrets Manager, a hardware-backed password manager). You must re-supply this **exact** value:

- on every chango master restart,
- on every node-manager restart,
- when adding any new host later.

## 6. Start the bundled ZooKeeper quorum

Chango ships its own ZooKeeper — you do not install ZK separately. For the HA default (3-node quorum), do this on **every ZK host**; for a single-host eval, do it once on the one host.

### Configure the quorum

On each ZK host, write `conf/zk/zoo.cfg` listing **every** quorum member, and write that host's own id into `myid`. The two extra ports are the ZK peer (`2888`) and leader-election (`3888`) ports.

```bash
# /opt/chango/conf/zk/zoo.cfg — identical on every ZK host
sudo -u chango tee /opt/chango/conf/zk/zoo.cfg > /dev/null <<'EOF'
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/var/lib/chango/zookeeper
clientPort=2181
4lw.commands.whitelist=*
admin.enableServer=false
maxClientCnxns=200

server.1=<zk-host1>:2888:3888
server.2=<zk-host2>:2888:3888
server.3=<zk-host3>:2888:3888
EOF

# myid — UNIQUE per host: 1 on zk-host1, 2 on zk-host2, 3 on zk-host3
sudo -u chango install -d /var/lib/chango/zookeeper
echo 1 | sudo -u chango tee /var/lib/chango/zookeeper/myid > /dev/null
```

For a **single-host eval**, the quorum is one line — `server.1=<host>:2888:3888` — and `myid` is `1`.

### Start ZK on each host

```bash
export CHANGO_MASTER_KEY=<from your secret manager>
export JAVA_HOME=/opt/openlogic-openjdk-17.0.7+7-linux-x64

sudo -u chango -E /opt/chango/bin/start-zk.sh
```

Start all ZK hosts, then confirm the quorum formed (`echo stat | nc <zk-host> 2181` reports one `leader` and the rest `follower`). ZK binds `:2181` (client), `:2888` (peer), `:3888` (leader election).

## 7. Start the chango master(s)

The ZK server list every master and node manager connects to is the **comma-separated client endpoints of all ZK hosts**. Define it once:

```bash
ZK_LIST=<zk-host1>:2181,<zk-host2>:2181,<zk-host3>:2181   # one entry for a single-host eval
```

On **each master host** (host1 and host2 in the HA default):

```bash
export CHANGO_MASTER_KEY=<from your secret manager>
export JAVA_HOME=/opt/openlogic-openjdk-17.0.7+7-linux-x64

sudo -u chango -E /opt/chango/bin/start-master.sh \
    -Dchango.zk.serverList=$ZK_LIST
```

Each master binds:

- `:8080` — admin HTTP (web UI + REST)
- `:19999` — internal NIO (master ↔ NM, encrypted with the cluster master key)

With two masters, the pair runs leader-election over ZooKeeper: one becomes leader, the other a warm follower that takes over on failure. Leadership is sticky — a restarted incumbent rejoins as leader without a needless toggle.

The first time a master starts, the KMS RocksDB under `/var/lib/chango/kms` is created and seeded from `CHANGO_MASTER_KEY`. From that point on, every later start of that master needs the **same** master key to decrypt it.

## 8. Start each node manager

On every node-manager host (all three in the HA default):

```bash
export CHANGO_MASTER_KEY=<from your secret manager>
export JAVA_HOME=/opt/openlogic-openjdk-17.0.7+7-linux-x64

sudo -u chango -E /opt/chango/bin/start-node-manager.sh \
    -Dchango.zk.serverList=$ZK_LIST
```

The NM binds `:19998` (internal NIO). It registers itself in ZooKeeper, the leader master picks it up, and the admin UI shows it as a target for new components.

## 9. Verify

From any host that can reach the master's `:8080`:

```bash
curl -s http://<master>:8080/admin/api/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"username":"admin","password":"admin"}'
```

A response containing `accessToken` means chango is up. The default login is `admin` / `admin`; the first login forces a password change.

Web UI: `http://<master>:8080/admin/`

## Logs and pid files

| What | Where |
|---|---|
| chango master log | `/var/log/chango/master.log` |
| chango master console | `/var/log/chango/master.out` |
| bundled ZK log | `/var/log/chango/zk.log` |
| node manager log | `/var/log/chango/nodemanager.log` |
| process pid files | `/opt/chango/bin/*.pid` |

Pid files are port-suffixed (`master-8080.pid`, `node-manager-19998.pid`) so multiple instances per host coexist.

## Adding hosts manually

Need to add a node manager later? Same recipe on the new host — host prep → install Java 11/17/25 → create `chango` user → extract chango tarball → `start-node-manager.sh` with the same `CHANGO_MASTER_KEY` and the existing cluster's ZK list. See [Add another node manager](../operations/cluster-operations.md#add-another-node-manager).

Adding a second master is the same recipe plus extending the bundled ZooKeeper quorum — see [Add another chango master](../operations/cluster-operations.md#add-another-chango-master).

## Next

- [Getting Started](getting-started.md) — first login + first managed component.
- [Cluster Operations](../operations/cluster-operations.md) — day-2 lifecycle.
