---
title: "Distributed Scheduler Hell"
date: 2017-02-17
year: 2017
event: microXchg
city: Berlin
youtube: mlNI7fYzckg
slides: /slides/distributed-tracing-hell-berlin.pdf
tags: [kubernetes, mesos, nomad, docker, devops]
description: "Choosing between container schedulers — Mesos, Kubernetes, Docker Swarm, Nomad."
---

**microXchg 2017 · Berlin**

A practical tour of **distributed container schedulers** drawn from running DigitalOcean’s monitoring stack at scale.

### What the talk covers
- What a distributed scheduler actually provides vs the Linux kernel (deploy, quotas, networking, scaling)
- **Vulcan** — DigitalOcean’s Prometheus-compatible timeseries system — as a real workload: ~3 Gbit/s network, 100k writes/s, sub-100ms reads, tens of TB storage
- Deploying that workload on **Mesos + Marathon**, custom Kafka schedulers, Cassandra pinning
- Failure modes (network partitions) and a counter-example with **Nomad**
- Moving to **Kubernetes**, internal deploy tooling (Docc / kube-diff style workflows), Flannel load balancing, metrics and logging

### Takeaways
How to choose an abstraction for middle-size apps growing into large multi-service fleets — and when *not* to reach for a full scheduler.

### Links
- [YouTube](https://www.youtube.com/watch?v=mlNI7fYzckg)
- [InfoQ presentation](https://www.infoq.com/presentations/container-schedulers/) (video, slides, MP3)
