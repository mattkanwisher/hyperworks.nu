---
title: "Chaos Monkey on my Laptop"
date: 2015-11-24
year: 2015
event: GothamGo
city: New York
youtube: "-H_Dyx2sct8"
slides: /slides/chaos-monkey-on-my-laptop.pdf
tags: [go, testing, devops, chaos]
description: "Simulating harsh server conditions and netsplits in local integration tests with VMs and Docker."
---

**GothamGo 2015 · New York**

Chaos engineering for apps that cannot tolerate silent netsplits — from Thomson Reuters **Eikon Messenger** experience.

### Story
- Netflix Chaos Monkey randomly kills production processes; we needed the same *before* production
- Spin up multi-VM / Docker topologies on a **developer laptop**
- Script NIC failures, host churn, latency between replicas (MySQL/Redis)
- Push the same suite into **Jenkins** and cloud VMs even when prod is bare metal
- Cut release-cycle risk by making ops confidence automatic

### Audience takeaways
Bulletproof middleware and microservices with integration tests that actually model datacenter failure.

Related appearance around **Velocity NYC**.
