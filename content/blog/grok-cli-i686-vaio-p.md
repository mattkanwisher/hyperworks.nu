---
title: "Grok CLI on a 32-bit Vaio P — shipping i686 when nobody else does"
date: 2026-07-30T08:00:00+07:00
description: "Official Grok Build has no i686 binary. I cross-compiled the open-source CLI for a Sony Vaio P (Atom Z550, antiX) — aws-lc worked, the real pain was arch-gated Rust and flaky CDNs."
tags: [rust, linux, i686, vaio, grok, reverse-engineering, systems]
image: /images/og/blog-vaio-p.jpg
image_alt: "Ultra-portable silver netbook open on a dark desk — Vaio P era hardware"
draft: false
---

Official [Grok Build](https://github.com/xai-org/grok-build) does not ship an **i686** binary. My **Sony Vaio P** (Intel **Atom Z550**, pure 32-bit) running **antiX / Debian trixie** only speaks i386 userspace and has about **2 GB of RAM**.

So I built one.

The result lives in **[mattkanwisher/grok-i686](https://github.com/mattkanwisher/grok-i686)**: a prebuilt `dist/grok` (ELF 32-bit Intel 80386), the patches that made it compile, a reproduce script, and a longer narrative of every dead end.

![Vaio P–class ultra-portable on a dark desk](/images/blog/headers/vaio-p.jpg)
*The kind of machine this is for: tiny late-2000s ultra-portable, not a workstation.*

## Why bother?

Rust’s `i686-unknown-linux-gnu` target is **Tier 1**. That only means the *compiler* is happy. A large modern workspace still has:

- C/CMake crypto (AWS-LC via `aws-lc-rs`)
- Arch-gated `cfg` that assumes x86_64
- Seccomp filters written for 64-bit syscall ABIs
- Tooling that only downloads x86_64 / aarch64 assets
- A ~170 MB release binary that has to live with glibc on the *target*, not the build host

Target facts for the published artifact:

| Item | Value |
|------|--------|
| Package | `xai-grok-pager` (installed as `grok`) |
| Target | `i686-unknown-linux-gnu` |
| Upstream rev | `5da6962e…` (pinned in the build script) |
| Max GLIBC needed | **2.39** (trixie ships 2.41 → OK) |
| Repo | [github.com/mattkanwisher/grok-i686](https://github.com/mattkanwisher/grok-i686) |

On the Atom itself: expect a heavy fullscreen TUI. Codebase-indexing features may simply be too memory-hungry. That is fine — the point was **run the CLI on hardware that cannot run 64-bit userspace at all**.

## The hypothesis that failed (in a good way)

I expected **`aws-lc-sys`** to be the show-stopper. AWS-LC does not advertise 32-bit x86 Linux. Community lore said: evict every path onto `ring` or give up.

With a real **`-m32`** C toolchain, cmake, clang/libclang, and nasm, **`aws-lc-sys 0.39.1` built and linked for i686** on this lockfile. No feature surgery for the published binary. The ring escape hatch is still good documentation; it was not the path we needed.

## Do not build on the Atom

Cross-compile from **x86_64 multilib**. The ideal story — `docker run --platform linux/386 i386/debian:trixie` — burned time:

- Packages are i386, but `uname -m` still reports **x86_64**
- rustup follows uname unless forced
- Large Fastly downloads of i686 rustup bits stalled mid-stream

What worked:

```bash
export CARGO_TARGET_I686_UNKNOWN_LINUX_GNU_LINKER=gcc
export CC_i686_unknown_linux_gnu="gcc -m32"
export CXX_i686_unknown_linux_gnu="g++ -m32"
export CFLAGS_i686_unknown_linux_gnu="-m32"

cargo build -p xai-grok-pager-bin --release --target i686-unknown-linux-gnu
```

Or from the port repo:

```bash
git clone https://github.com/mattkanwisher/grok-i686.git
cd grok-i686
./scripts/build-i686.sh   # → dist/grok
```

## Network pain was half the project

On the build host, downloads from `static.rust-lang.org` (Fastly) repeatedly hung after ~35 MB — especially HTTP/2 / IPv6. Sparse crates.io through the same CDN family timed out under cargo’s parallel HTTP client.

| Symptom | Mitigation |
|---------|------------|
| rustup / rustc tarballs hang | Force IPv4 (`curl -4`); chunked downloads; mirrors when needed |
| cargo sparse index timeouts | `[registries.crates-io] protocol = "git"` |
| Flaky parallel HTTP | `git-fetch-with-cli`, retries, lower multiplexing |

If you only remember one ops lesson: **assume CDN flake early**, not after three hours of “cargo is broken.”

## Compile errors that actually were i686-specific

### 1. `fastant` — wrong arch module

Gated as `cfg(any(target_arch = "x86", "x86_64"))` but always imported `core::arch::x86_64::__cpuid`. On i686 that module does not exist.

**Fix:** local crate patch — `core::arch::x86` on 32-bit, `x86_64` on 64-bit (`patches/fastant`).

### 2. `nono` — missing syscall numbers

`SYS_OPENAT` / `SYS_OPENAT2` only defined for x86_64 and aarch64.

**Fix:** for `target_arch = "x86"`: openat **295**, openat2 **437** (`patches/nono`).

### 3. Sandbox — `SYS_accept` missing on i686

32-bit x86 socket ops historically go through **`socketcall`**. libc does not expose `SYS_accept` the way x86_64 does, so the child network seccomp filter would not compile.

**Fix:** full filter on x86_64/aarch64; **no-op network filter** on other Linux arches (including i686). Trade-off: weaker network sandboxing on 32-bit. Also set `AUDIT_ARCH_I386` for namespace lockdown.

### 4. `pprof::blocklist` is arch-gated

Only exists on some arches. Unconditional call breaks i686.

**Fix:** `cfg` the `.blocklist(...)` call in CPU profiling.

### 5. `rlim_t` width

Address-space caps assumed `u64`. `rlim_t` is **u32** on i686.

**Fix:** cast through `libc::rlim_t`.

### 6. Bundled ripgrep

Build scripts only auto-fetch `rg` for known x86_64 / aarch64 assets.

**Fix:** ship BurntSushi’s official i686 `rg` and set `GROK_TOOLS_BUNDLE_RG_PATH` / `GROK_SHELL_BUNDLE_RG_PATH`.

## Verification

```bash
file dist/grok
# ELF 32-bit LSB ... Intel 80386

objdump -T dist/grok | grep -o 'GLIBC_[0-9.]*' | sort -Vu | tail -5
# highest ≤ target glibc (we needed 2.39)

dist/grok --version
```

## Quick start on the Vaio

```bash
# needs git-lfs — dist/grok is ~170 MB
git clone https://github.com/mattkanwisher/grok-i686.git
cd grok-i686 && git lfs pull

scp dist/grok user@vaio:~/.local/bin/grok
ssh user@vaio 'chmod 755 ~/.local/bin/grok && grok --version'
```

Set `XAI_API_KEY` or complete the browser auth flow on first run.

## What I would do differently

1. Start with **multilib cross-compile**, not a 386 container, unless rustup’s i686 downloads are already proven healthy on that network.
2. Plan for **mirrors, IPv4, and git crates index** before the first full release link.
3. Run `cargo check --target i686-unknown-linux-gnu` before a multi-hour release build.
4. Budget for **“works on x86_64” cfg bugs**, not only crypto CMake.

## Links

- **Port repo:** [mattkanwisher/grok-i686](https://github.com/mattkanwisher/grok-i686)
- **Upstream:** [xai-org/grok-build](https://github.com/xai-org/grok-build) (Apache-2.0)
- **Long-form build diary (HTML in the repo):** [problems-and-solutions.html](https://github.com/mattkanwisher/grok-i686/blob/main/docs/problems-and-solutions.html)

This is a **personal / experimental** port for hardware I own — not an official xAI release. On i686 the sandbox network path is intentionally weaker; treat it accordingly.
