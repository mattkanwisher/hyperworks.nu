---
title: Open Source
description: Accepted contributions to large open-source projects, and top personal GitHub repositories.
---

Accepted pull requests and substantial work on public repos, ranked by project size. Personal projects are listed separately below.

GitHub: [mattkanwisher](https://github.com/mattkanwisher)

---

## Upstream contributions

Merged PRs and maintainer-level work outside my own user account. Larger projects first.

### [prometheus/prometheus](https://github.com/prometheus/prometheus)

The monitoring system and time-series database. ~65k stars.

- [Make Chunk and ChunkIterator public for reuse](https://github.com/prometheus/prometheus/pull/2011) — exposed storage internals so external systems (including DigitalOcean’s Vulcan path) could reuse Prometheus chunk/iterator types without forking the core.

### [mudler/LocalAI](https://github.com/mudler/LocalAI)

Open-source alternative to OpenAI APIs — run LLMs, vision, voice, and more locally. ~48k stars.

- [Keep whisper models in memory](https://github.com/mudler/LocalAI/pull/233)
- [Fix upload transcription API not reading POST body](https://github.com/mudler/LocalAI/pull/229)

Also related: site work on [go-skynet/localai-website](https://github.com/go-skynet/localai-website).

### [joonspk-research/generative_agents](https://github.com/joonspk-research/generative_agents)

Stanford’s **Generative Agents** paper implementation — interactive simulacra of human behavior. ~22k stars.

- [Make the project buildable on M1 Apple Silicon Macs](https://github.com/joonspk-research/generative_agents/pull/11)

### [hashicorp/nomad](https://github.com/hashicorp/nomad)

Workload orchestrator for containers and non-containerized apps. ~17k stars.

- [Jobs tests were sending invalid jobs](https://github.com/hashicorp/nomad/pull/3480)

### [lokalise/i18n-ally](https://github.com/lokalise/i18n-ally)

All-in-one i18n extension for VS Code. ~5k stars.

- [Add Svelte 2.0 / SvelteKit detection](https://github.com/lokalise/i18n-ally/pull/815)
- [Fix English spelling in choice prompt](https://github.com/lokalise/i18n-ally/pull/814)

### [thrasher-corp/gocryptotrader](https://github.com/thrasher-corp/gocryptotrader)

Multi-exchange cryptocurrency trading bot and framework in Go. ~3.5k stars.

- [Consistent sandbox access on Gemini and GDAX](https://github.com/thrasher-corp/gocryptotrader/pull/60)

### [ConsenSys / ganache-cli](https://github.com/ConsenSys-archive/ganache-cli-archive)

Fast Ethereum RPC client for local testing (now Truffle/Ganache lineage). ~3.3k stars on the archived repo.

- [Add ability to save accounts and private keys on startup](https://github.com/ConsenSys-archive/ganache-cli-archive/pull/406)
- [truffle-hdwallet-provider: add personal_eth_sign](https://github.com/ConsenSys-archive/truffle-hdwallet-provider/pull/67)

### [CryptozombiesHQ/cryptozombie-lessons](https://github.com/CryptozombiesHQ/cryptozombie-lessons)

Lesson content for [cryptozombies.io](https://cryptozombies.io) — interactive Solidity education. ~1k stars.

As co-founder of Loom / CryptoZombies: lesson content, lander, licenses, and ongoing course maintenance (e.g. [lesson 6](https://github.com/CryptozombiesHQ/cryptozombie-lessons/pull/207)).

### Loom Network (founder)

Built and maintained the DAppChain stack and tooling. Representative repos with accepted merges:

| Repo | Role |
|------|------|
| [loomnetwork/loomchain](https://github.com/loomnetwork/loomchain) | DAppChain engine — perf, DPoS, validators, storage (94 merged PRs) |
| [loomnetwork/go-loom](https://github.com/loomnetwork/go-loom) | Go SDK / plugins / smart contracts (27 merged PRs) |
| [loomnetwork/plasma-cash](https://github.com/loomnetwork/plasma-cash) | Plasma Cash contracts & client — ERC-721/20/ETH (13 merged PRs) |
| [loomnetwork/loom-js](https://github.com/loomnetwork/loom-js) | Browser & Node client for DAppChains |

### Other accepted work

| Project | Contribution |
|---------|----------------|
| [heroiclabs/nakama-dotnet](https://github.com/heroiclabs/nakama-dotnet) | [WebSocket compatibility for WASM and Blazor](https://github.com/heroiclabs/nakama-dotnet/pull/89) — .NET client for Nakama game server |
| [certusone/yubihsm-go](https://github.com/certusone/yubihsm-go) | [Additional HSM methods](https://github.com/certusone/yubihsm-go/pull/3) — Go client for YubiHSM2 |
| [certusone/aiakos](https://github.com/certusone/aiakos) | [Flexible Tendermint versioning](https://github.com/certusone/aiakos/pull/1) — PrivValidator on YubiHSM2 |
| [wantedly/apig](https://github.com/wantedly/apig) | [MySQL project generation support](https://github.com/wantedly/apig/pull/117) — Go REST API generator |

---

## Top 10 personal projects

Repositories under [github.com/mattkanwisher](https://github.com/mattkanwisher), ordered by stars (with a couple of active/recent projects mixed in where they matter more than star count).

### 1. [microservices-book-code](https://github.com/mattkanwisher/microservices-book-code)

Companion source for **Microservices in Go** ([microservicesingo.com](http://www.microservicesingo.com)). Reference implementations for service structure, messaging, and ops patterns from the book. · Go · 62★

### 2. [driftwood.js](https://github.com/mattkanwisher/driftwood.js)

Dirt-simple logging framework for JavaScript — minimal API for structured / leveled logs without a heavy framework. · JavaScript · 24★

### 3. [cryptofiend](https://github.com/mattkanwisher/cryptofiend)

Opinionated Go libraries for crypto trading bots and exchange integration experiments. · Go · 13★

### 4. [distributedtrace](https://github.com/mattkanwisher/distributedtrace)

Zipkin-compatible distributed tracing backend written in Go — spans, storage, and query path for request tracing. · Go · 6★

### 5. [iphone_dj](https://github.com/mattkanwisher/iphone_dj)

DJ from your iPhone — early mobile audio / performance experiment. · Objective-C · 5★

### 6. [html5_tetris_multiplayer](https://github.com/mattkanwisher/html5_tetris_multiplayer)

Netris-style multiplayer Tetris clone in HTML5 (real-time browser game). · JavaScript · 2★

### 7. [python_bkk](https://github.com/mattkanwisher/python_bkk)

Blockchain examples from a Python meetup in Bangkok — teaching material and demos. · JavaScript · 2★

### 8. [ranger](https://github.com/mattkanwisher/ranger)

Unix system monitor in Go — lightweight process / host introspection tooling. · Go · 1★

### 9. [gaime_mods](https://github.com/mattkanwisher/gaime_mods)

Reverse engineering the G'AIM'E Time Crisis light-gun console (Allwinner A523 / Android 13): firmware encryption, FEL/FES flash protocol, full backup, root shell. See also the [blog write-up](/blog/gaime-lightgun-reverse-engineering/). · C

### 10. [demo_stocks_digitalocean](https://github.com/mattkanwisher/demo_stocks_digitalocean)

Realtime market-data demo from Fintegrate Mumbai 2017 — streaming data in the cloud on DigitalOcean. · JavaScript · 1★

---

More under [github.com/mattkanwisher](https://github.com/mattkanwisher), including this site ([hyperworks.nu](https://github.com/mattkanwisher/hyperworks.nu)) and TensorFleet / Project Lam org work.
