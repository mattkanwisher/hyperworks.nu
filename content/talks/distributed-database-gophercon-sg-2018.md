---
title: "Build your own distributed database"
date: 2018-05-05
year: 2018
event: GopherCon Singapore
city: Singapore
youtube: k0-WyZCKF5I
youtube_backup: 8aV0XCwEcZU
tags: [go, databases, distributed-systems]
description: "Deep dive into building distributed databases in Go."
slides: /slides/distributed-database-gophercon-sg-2018.pdf
---

**GopherCon Singapore 2018**

A from-scratch mental model for **distributed databases in Go**.

### Deck outline
- Layers: file → KV → SQLite → relational
- When MySQL/Postgres are not enough (custom indexes, domain query languages)
- Use cases: blockchains, trading, logs, metrics
- Go examples: Influx, Prometheus, Vulcan, Ethereum, etcd, Consul
- Architecture: consensus (Raft/Paxos/BFT), storage (LevelDB/Rocks/Bolt), query layer
- Starting points: HashiCorp Raft, etcd Raft, toy SQL parsers

For people who want to *build* storage systems, not only use them.
