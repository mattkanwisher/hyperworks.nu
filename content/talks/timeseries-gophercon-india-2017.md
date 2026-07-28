---
title: "Distributed Timeseries Database in Go"
date: 2017-02-24
year: 2017
event: GopherCon India
city: India
slides: /slides/distributed-timeseries-database.pdf
tags: [go, prometheus, timeseries, cassandra, kafka]
description: "Building Vulcan — a Prometheus-compatible distributed timeseries database in Go."
---

**GopherCon India 2017**

Deep dive into **Vulcan**, DigitalOcean’s Prometheus-compatible distributed timeseries database written in Go.

### Outline (from the deck)
- Why Go for NoSQL/NewSQL (Prometheus, Cockroach, Influx, etcd…)
- Timeseries properties: write-heavy, compressible, lossy-friendly
- From hash-sharded Prometheus to a pipelined microservice architecture
- Kafka ingestion, Cassandra/chunk storage, schema evolution (V1 series → V2 chunks + index)
- In-memory query engine and downsampling
- Hard numbers we designed for: multi-Gbit/s, 100k writes/s, tens of TB

### Links
- [Speaker Deck](https://speakerdeck.com/mattkanwisher/building-distributed-timeseries-database-in-go)
- Open source context: [digitalocean/vulcan](https://github.com/digitalocean/vulcan) (historical)
