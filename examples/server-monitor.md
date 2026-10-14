# Example: "a minimal, easy to use server monitoring utility"

A worked example of phases 1 to 4, to calibrate depth. Facts were checked on
2026-10-14; re-verify before relying on them in a real project.

## Phase 1: Restate and frame

> You want a small tool that tells you how your server(s) are doing (CPU,
> memory, disk, maybe services) without the setup cost of a full monitoring
> stack. v1 is done when you can install it in one step and see the current
> health of a server from somewhere convenient.

Ambiguous terms to pin down:

- **"monitoring"** - live view of now? history and graphs? alerts when
  something breaks? These are three different products.
- **"minimal"** - small binary? few features? no external services (no DB,
  no Grafana)? low resource use on the server?
- **"easy to use"** - easy to install? easy to read? zero config?
- **"server"** - one box? a handful of VPSes? Linux only?

Framing questions (only these before research):

1. How many servers: 1, 2 to 20, or more?
2. Do you want history and alerts, or just "what is happening now"?
3. Any hard constraints (Linux only, no Docker, must not open ports, ...)?

## Phase 2: Landscape

### Existing software

| Name           | Shape                                      | Good at                         | Why not (maybe)                   | License    |
|----------------|--------------------------------------------|---------------------------------|-----------------------------------|------------|
| htop / btop    | Local TUI, run over SSH                    | Instant live view, zero setup   | No history, no alerts, one host   | GPL/Apache |
| Glances        | Local TUI, optional web UI and REST API    | Broad metrics, one tool         | Python runtime, heavier           | LGPL-3.0   |
| Beszel         | Go agent on each host, hub with web UI     | Light, multi-host, history      | Very close to the idea already    | MIT        |
| Netdata        | Agent with built-in dashboard, per-second  | Huge coverage, auto-detection   | Heavy, many features, cloud tie   | GPL-3.0    |
| node_exporter  | Agent exposing /metrics, needs Prometheus  | The de facto metrics standard   | Needs Prometheus + Grafana stack  | Apache-2.0 |
| Telegraf       | Push agent with many input/output plugins  | Flexible collection             | Needs a backend to store/view     | MIT        |
| Monit          | Local daemon, checks + restarts services   | Self-healing, simple config     | Not about metrics or graphs       | AGPL-3.0   |
| Uptime Kuma    | External prober, web UI                    | Is it up from outside?          | Cannot see CPU/disk inside host   | MIT        |
| Zabbix         | Server + agents + DB, enterprise           | Everything                      | Opposite of minimal               | AGPL-3.0   |
| Healthchecks   | Dead-man switch: hosts ping it             | Detects silent failure (cron)   | Only liveness                     | BSD-3      |

### Standards and references

- **Linux `/proc` and `/sys`** (proc(5), sysfs docs): the actual data source
  for CPU, memory, load, disk, network on Linux. Exact field semantics matter
  (CPU is cumulative jiffies; you need two samples and a delta).
- **Prometheus text exposition format / OpenMetrics**: the standard way to
  expose metrics over HTTP. Speaking it means users can plug into existing
  Grafana / Prometheus setups for free.
- **Nagios plugin exit codes** (0 OK, 1 WARNING, 2 CRITICAL, 3 UNKNOWN):
  the long-standing convention for check scripts; many tools still consume it.
- **systemd** (`systemctl show`, D-Bus API): standard way to check service
  state on modern Linux.
- **SMART** (`smartctl` JSON output): disk health, often forgotten.

### Architecture patterns

- **Local only** (htop style): run the tool on the server, look at it. Zero
  network surface, no history unless it writes files.
- **Agent + pull** (node_exporter + Prometheus): agent exposes an HTTP
  endpoint; a central collector scrapes it. Needs an open port per host.
- **Agent + push** (Telegraf, Beszel agent to hub): agent sends to a central
  place. Works behind NAT; the central place needs to be reachable.
- **Agentless over SSH**: a central tool SSHes in and reads `/proc` or runs
  commands. Nothing to install on targets; needs SSH keys and is slower and
  more fragile at scale.

### Build vs use

Beszel already matches "minimal, easy, multi-host, Go agent, web UI". htop
over SSH already covers "what is happening now on one box". Reasons to still
build: learning, a different interface (for example CLI or TUI only, no web
UI), agentless SSH operation, or specific checks Beszel lacks. Ask the user
which of these applies.

## Phase 3: Questions informed by research (sample batch)

1. **Placement** - Agentless over SSH from your laptop (Recommended for 1 to
   5 hosts, nothing to install) / Small agent on each server / Local only.
2. **How you read it** - CLI that prints a summary / Live TUI / Web dashboard
   / Only alerts, no UI.
3. **History** - None, live only (Recommended for v1) / Short local history
   (24h) / Long term (needs storage decisions).
4. **Interop** - Expose Prometheus /metrics so it can plug into Grafana later?
   Yes, cheap to add / No.

## Phase 4: Unknowns a domain expert would raise

| Item                                | Why it matters                                          | Rec.   |
|-------------------------------------|---------------------------------------------------------|--------|
| Alerting and alert fatigue          | Monitoring without alerts is a dashboard nobody watches | later  |
| Who monitors the monitor            | If the agent or host dies, silence looks like "fine"    | v1     |
| Rates need two samples              | CPU and network from /proc are counters, not values     | v1     |
| Disk: space vs inodes vs SMART      | Inodes and dying disks fill or fail before space does   | v1/later |
| Containers and cgroups              | /proc inside a container lies about host resources      | decide |
| Network exposure and auth           | An open metrics port leaks host details                 | v1     |
| Least privilege                     | SMART and some systemd queries need root                | v1     |
| Clock skew and timestamps           | Multi-host timelines need consistent time               | later  |
| Retention and storage growth        | History grows forever unless downsampled or capped      | later  |
| Cross-platform                      | macOS/BSD have no /proc; Windows is a separate project  | never? |
| External reachability check         | A host can be healthy inside and unreachable outside    | later  |
