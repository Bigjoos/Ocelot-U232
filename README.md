# Ocelot-U232

> **High-performance announce daemon for U-232 V5.5 “Chaos Edition”**  
> Forked from [WhatCD/Ocelot](https://github.com/WhatCD/Ocelot) — rebuilt from the ground up.

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![C++17](https://img.shields.io/badge/C%2B%2B-17-orange.svg)](https://en.cppreference.com/w/cpp/17)
[![Platform: Linux](https://img.shields.io/badge/Platform-Linux-lightgrey.svg)](https://kernel.org)
[![Status: Production](https://img.shields.io/badge/Status-Production-brightgreen.svg)]()

-----

## What is this?

Ocelot-U232 is a **heavily modified fork** of the classic WhatCD Ocelot C++ BitTorrent tracker daemon, surgically rebuilt to power the U-232 V5.5 tracker ecosystem. The original Ocelot was legendary — this version is something else entirely.

This is not a cosmetic fork. Every subsystem has been touched. The result is a production-hardened announce daemon with real-time swarm intelligence, modular accounting, anti-cheat enforcement, and geographic peer awareness — all running natively against U-232’s own database schema.

-----

## Native Schema Integration

Ocelot-U232 ditches Gazelle’s table structure entirely and operates directly against **U-232’s native schema**:

- `peers` — with `(torrent, userid)` unique key to handle peer_id rotation cleanly
- `torrents` — seeder/leecher/snatch counts updated in-daemon
- `users` — ratio, upload, download, hit-and-run tracking
- `snatched` — full snatch lifecycle management

No middleware. No shim layer. Direct, fast, correct.

-----

## Core Features

### ⚡ Dynamic Announce Intervals

Announce intervals are not static. The daemon calculates optimal intervals per-torrent based on swarm health:

- **Critical swarms** (very few seeds) → **30-second intervals** to maximise peer churn and connection opportunity
- Healthy swarms scale gracefully to standard intervals
- Reduces load on the tracker while keeping sick torrents alive

### 🧠 SwarmPromoter — Autonomous Swarm Health Management

A `SwarmPromoter` class runs inside the `docleanup()` cycle and automatically promotes struggling torrents:

- Detects swarms falling below configurable seed/leech thresholds
- Auto-applies **Freeleech** or **Silver** status to incentivise activity
- Reverses promotions automatically once swarm health recovers
- Zero manual intervention required — the tracker heals itself

### 🛡️ Anti-Cheat Detection Engine

A multi-layer cheat detection system runs on every announce and scrape:

|Detection Type        |Method                                                  |
|----------------------|--------------------------------------------------------|
|**Impossible Speed**  |Upload/download delta vs. time delta sanity check       |
|**Ghost Peers**       |Peers announcing without completing handshake validation|
|**Ratio Manipulation**|Cross-session upload inflation detection                |
|**Hit-and-Run**       |Snatch-without-seed tracking with threshold enforcement |

A `flagged_pairs` set prevents duplicate spam — each peer/torrent pair is flagged once per cycle, not on every announce. Offenders are logged for PHP-side action.

### 🌍 GeoIP Peer Selection

Peers are selected with geographic awareness:

- Prioritises local-region peers for better latency
- Falls back gracefully when regional peers are unavailable
- Reduces cross-continental tracker load on small swarms

### 📊 Modular `accounting.h` — Priority-Based Modifier Resolution

The accounting subsystem is fully modular with a clean priority stack:

- Multiple modifier sources (freeleech tokens, site-wide FL events, torrent-level overrides, staff grants) resolve via **priority order** — no conflicts, no double-dipping
- Upload/download multipliers are applied correctly at the announce level, not retrospectively
- Pluggable architecture — new modifier types can be added without touching core accounting logic

-----

## What Was Changed vs. Upstream

|Area              |Upstream Ocelot      |Ocelot-U232                        |
|------------------|---------------------|-----------------------------------|
|Database schema   |Gazelle/Ocelot tables|U-232 native schema                |
|Announce intervals|Static               |Dynamic per-swarm                  |
|Swarm management  |None                 |SwarmPromoter auto-promotion       |
|Cheat detection   |Basic                |Multi-layer with dedup             |
|Peer selection    |FIFO                 |GeoIP-weighted                     |
|Accounting        |Monolithic           |Modular priority stack             |
|`peers` unique key|`(torrent, peer_id)` |`(torrent, userid)` — rotation safe|

-----

## Requirements

- **C++17** compiler (GCC 9+ / Clang 10+)
- **Boost** (asio, system, filesystem)
- **MySQL Connector/C** or MariaDB equivalent
- **libmaxminddb** for GeoIP (optional but recommended)
- Linux (production-tested on Ubuntu 22.04 LTS)

-----

## Building

```bash
git clone https://github.com/Bigjoos/Ocelot-U232.git
cd Ocelot-U232
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
```

Copy `ocelot.conf.example` to `ocelot.conf` and configure your U-232 database credentials before starting.

-----

## Configuration

Key config values specific to Ocelot-U232 (beyond standard Ocelot settings):

```ini
# Dynamic interval thresholds
min_announce_interval = 30
critical_seed_threshold = 2

# SwarmPromoter
swarm_promoter_enabled = true
swarm_fl_threshold_seeds = 1
swarm_silver_threshold_seeds = 3
swarm_recovery_seeds = 5

# GeoIP
geoip_database = /usr/share/GeoIP/GeoLite2-City.mmdb
geoip_peer_selection = true

# Anti-cheat
cheat_detection_enabled = true
impossible_speed_mbps = 500
```

-----

## Integration with U-232 V5.5

Ocelot-U232 is purpose-built for the [U-232 V5.5 “Chaos Edition”](https://github.com/Bigjoos/U-232-V5.5) PHP tracker codebase. It expects:

- PDO-compatible MySQL UTF8MB4 schema
- U-232’s `peers`, `torrents`, `users`, and `snatched` table structure
- The tracker’s whitelist management via the standard Ocelot reload mechanism

It will **not** work drop-in with Gazelle or other tracker codebases without schema adaptation.

-----

## Production Status

Ocelot-U232 is running in production at **[Murder By Sound / U-232](https://u-232.com)** — a private tracker serving real users with real torrents. The anti-cheat system, SwarmPromoter, and dynamic intervals have all been validated against live traffic.

-----

## Lineage

```
WhatCD/Ocelot (original)
    └── Various community forks
            └── Ocelot-U232 (this repo)
                    — Schema: U-232 native
                    — Engine: rebuilt for Chaos Edition
```

-----

## Contributing

This is the announce daemon for a specific tracker ecosystem. PRs that align with U-232’s architecture are welcome. Generic Gazelle-compatibility PRs will not be merged — use upstream Ocelot for that.

-----

## License

GPL v3 — same as upstream Ocelot. See <LICENSE>.

-----

*Part of the U-232 V5.5 “Chaos Edition” open-source tracker project.*  
*Built with stubbornness, caffeine, and an unreasonable amount of C++ template debugging.*
