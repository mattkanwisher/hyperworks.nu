---
title: "Taking apart the G'AIM'E light gun — both halves of it"
date: 2026-07-28
description: "A ¥13,200 Time Crisis console ships a hardcoded AES key, no secure boot, and a licensed Namco arcade emulator. The gun is a second Linux computer with an NPU and a root shell on a header."
tags: [reverse-engineering, hardware, allwinner, android, lightgun, namco]
image: /images/og/blog-gaime.jpg
draft: false
---

A ¥13,200 Time Crisis console turned out to ship a **hardcoded AES key**, **no secure boot of any kind**, and a **licensed Namco arcade emulator**. The gun turned out to be a **second Linux computer** with a deliberately unmarked SoC, an NPU, and a root shell on an unpopulated header.

I bought the Japanese retail **G'AIM'E TIME CRISIS** kit in Akihabara. This is what was inside it — expanded from the thread on [Twitter/X](https://twitter.com/kanwisher).

![Gun chassis with camera and recoil motor](/images/blog/gaime/gun-chassis-camera-and-motor.jpg)
*Gun opened: main board, muzzle camera on a ribbon, recoil motor bottom-right.*

## What it is

G'AIM'E is a small HDMI mini-console from **Tassei Denki**, licensed by Bandai Namco for the Time Crisis 30th anniversary. You plug in a camera-based light gun, plug in a TV, shoot things. No settings screen, no file manager, no network stack, no way out of the launcher.

Bundles (JP retail): Basic ~¥13,200 (console + 1 gun), Premium (+ pedal), Ultimate (+ second gun). Games: Time Crisis, Point Blank, Steel Gunner / Steel Gunner 2 on the higher SKUs.

Two things made this project unusually productive:

1. The vendor publishes a **~4 GB full firmware image** as an **unauthenticated download** — not a delta patch.
2. There is **no secure boot** anywhere in the chain, so once you understand the image you can also put things back.

Everything below used that public download, a USB cable, a cheap serial adapter, and hardware I own. Full notes and tooling live in [gaime_mods](https://github.com/mattkanwisher/gaime_mods).

---

## Part 1 — The console

### A 4 GB download and a key left in the updater

The update package is a Windows-only vendor tool plus three `.dat` files. The `.dat` files start with ASCII magic `GAIMEENC` and have payload entropy ~7.9998 — genuinely encrypted.

The updater `GAIMEUpdater.exe` is a .NET 8 single-file bundle. Extracting `GAIMEUpdater.dll` and decompiling `FirmwareDecryptor` gives away the scheme, including a **hardcoded AES-256-GCM key**:

```
2C4B5B4D84F9A5E0E4D4C2F78B6A1D9E0A0B1C2D3E4F5061728394A5B6C7D8E9
```

Layout:

```
"GAIMEENC" (8) · version=1 (1) · max chunk (4, LE = 4 MiB) · base nonce (12)
then repeating { length (4, LE) · ciphertext · GCM tag (16) }

Per-chunk nonce = base nonce with LE int32 at bytes 8..12 XORed with chunk index.
```

Underneath is a plain **Allwinner IMAGEWTY** image (PhoenixSuit/LiveSuit container). Re-encrypting an unmodified image reproduces the vendor `.dat` **byte for byte** (same SHA-256). At that point the container is solved, not “probably understood.”

### What was actually in there

| | |
|---|---|
| SoC | Allwinner A527 (`sun55iw3`), board `a527-pro2`, octa-core Cortex-A55 |
| DRAM | 512 MiB LPDDR4 |
| Storage | ~58 GiB eMMC |
| OS | Android 13, `user` build, **test-keys** |
| Kernel | Linux 5.15.x, built as `root@dashine-namco` |
| Boot chain | BOOT0 → BL31 (ATF v2.5) → SCP/arisc → U-Boot 2018.07 |
| Board | `LBQ-1585-A-V1.1` (2025-07-23) |
| Camera on console | **None** — zero CSI/MIPI in `sys_config.fex` |

ODM is **Dashine** (`com.dashine.*` packages, leftover Windows build paths in the flasher config).

**SKU twist:** Basic / Premium / Ultimate are almost the same firmware. Basic just omits three game APKs. Premium vs Ultimate differs by effectively one launcher APK (5 bytes of compression noise). First-launch “authentication” is runtime accessory detection, not separate images.

### The emulator is a licensed Namco core, not MAME

Each game is Unity wrapping a native core. Time Crisis uses `libSys22Plugin.so`, stamped:

```
(C)2021 BANDAI NAMCO Research Inc.
```

So this is an in-house **Namco System 22** emulator — not FBNeo, not MAME. No GPL leverage, no source offer. The exported API is clean and shipped with debug hooks still present:

```
Sys22Plugin_SetRom            Sys22Plugin_DebugReadMemory
Sys22Plugin_SetInputPointing  Sys22Plugin_DebugWriteMemory
Sys22Plugin_SetEEPROM         Sys22Plugin_GetForceFeedback
Sys22Plugin_UpdateFrame       …
```

ROM data is fed via `Sys22Plugin_SetRom` from C#, living inside a Unity `data.unity3d` asset rather than loose MAME sets.

### Getting in: FEL was hiding in plain sight

Allwinner boot ROM mode **FEL** speaks a simple USB protocol for memory R/W/X. Entry sequence from the vendor’s own docs: hold the pinhole **Programming** button, insert USB-C into the **rear power jack** (power is also data), release after ~5s.

```
$ sunxi-fel version
AWUSBFEX soc=00001890(A523) …
```

That independently confirms the `sun55iw3` family from a live device.

**Hard lesson:** reading an unmapped address over FEL hangs the endpoint until a physical replug. Stay in known SRAM until DRAM is up.

### Writing a flasher carefully

FEL alone has no eMMC driver. To dump partitions you need Allwinner’s second-stage **FES** path: load `fes1` (DRAM init) → U-Boot (efex) → FES partition commands.

Design choice that kept the unit alive:

- **Dumper is read-only by construction** — write opcodes are not defined in the read client at all.
- **Writer** is a separate tool that refuses sectors whose contents match neither the recorded original nor the intended result, and read-backs everything it writes.

**The bug that almost ruined the backup:** the first full dump looked perfect (right sizes, no errors) and was entirely wrong. GPT LBAs sit **40960 sectors** above where FES addresses data. Every partition was read from 20 MiB past the truth. Fix: calibrate against known filesystem magics (`ANDROID!`, `AVB0`, …) and **refuse to dump** if calibration fails.

Result: full 27-partition backup of **factory firmware older than the public download** — something one OTA would have destroyed.

### No secure boot. Anywhere.

| Check | State |
|---|---|
| Secure boot fuse | clear |
| AVB | vbmeta flags zero; “avb” never appears in boot log |
| dm-verity | not enforcing |
| Integrity | 32-bit word sum on some images |

### UART and the one-byte ADB trap

Three unpopulated headers on the board, silkscreened: **J2** = UART (GND/TX/RX), **J3** = JTAG, **J4** = grounds. SW1 is literally labeled `FEL`.

Serial at 115200 gave a full boot log and a root shell. Then `logcat` explained why five carefully verified patches to enable ADB did nothing:

```
adbd: Could not set SELinux context
```

Our own `ro.secure=0` patch was the problem. On a `user` build, adbd tries a transition to `u:r:su:s0` that policy denies — so adbd fatals forever. Forcing `service.adb.root=0` fixed it. Four correct patches had been masked by one wrong assumption.

With adb working, the kiosk is just a door: **106 packages**, **24 launchable activities**, including stock Settings. Small APKs installed to `/data` (hello splash + file explorer) cannot affect boot and uninstall cleanly.

---

## Part 2 — The gun (the more interesting half)

### Five USB interfaces, one of them is a gift

```
VID 0x2E2C  PID 0x0631  — Tassei Denki / GAIME v1
```

| # | Class | Role |
|---|---|---|
| 0 | HID | Boot keyboard |
| 1 | HID | **Digitizer — absolute pointer** |
| 2 | HID | Vendor config (64-byte, Modbus CRC-16) |
| 3–4 | UVC | “Dashine UVC” camera gadget |

Interface 1 is a **Touch Screen digitizer**: tip switch (trigger), “in range”, X/Y as **absolute** 0–10000. That is exactly what light guns need — plug into any PC and you get an absolute mouse with no vendor software.

**All CV runs in the gun.** The console has no camera. For pointing, the console is optional: this is effectively a Sinden-class gun at a fraction of Sinden’s price.

### Measuring the jitter everyone complains about

Reviews mention pointer jump on trigger. Ten seconds of capture with the gun on a desk (not aimed at a display):

- Report rate wildly variable (~28–278 Hz)
- X/Y span the full 99–9900 clamp range
- Mean jump ~90 counts; worst jump ~half the screen in one report
- **`in_range` was set on every single garbage report** — useless as a validity gate

So filtering has to be geometric (median-of-5 + max-jump), not flag-based. On macOS the digitizer hijacks the system cursor; `gun_bridge.swift` seizes the device (`kIOHIDOptionsTypeSeizeDevice`) and only re-emits while RetroArch (or another app) is frontmost.

### The UVC stream is a test pattern

The gun enumerates UVC up to 1080p. Captured frames are **not imagery** — a synthetic ramp:

- every row perfectly uniform in X (σ = 0)
- scrolls at exactly +1.6 rows/frame
- bit-identical after roll offsets (zero sensor noise)

Looks like a bring-up leftover that shipped inactive. Console code never consumes video.

### Opening the gun

![Gun mainboard front](/images/blog/gaime/gun-front-board.jpg)
*Main board `LBQ-1585-C-V1.1` — note blank SoC at U8 and serial header J3.*

![SoC and UART header](/images/blog/gaime/gun-front-closeup-soc-and-j3-uart.jpg)
*Unmarked ~96-pin QFN at U8; J3 silkscreened GND / TX / RX / V3.3.*

![Board back](/images/blog/gaime/gun-mainboard-back.jpg)
*Nearly bare reverse — no external DRAM; memory is in-package.*

![Camera FPC](/images/blog/gaime/gun-front-closeup-flash-and-camera-fpc.jpg)
*SPI NAND near the camera end; camera module on its own FPC in the barrel.*

PID `0x0631` is **1585 decimal** — same project number as the board silk. Boot medium is a **GigaDevice SPI NAND** (not NOR — clips and `flashrom` won’t help). DRAM is stacked in-package.

### Three wires and a root shell

Receive-only on J3 first (can’t brick). After fixing two self-inflicted issues (unpowered gun → EMI garbage; a leftover `uart_watch.sh` still stealing half the bytes), one newline was enough:

```
# id
uid=0(root) gid=0(root) groups=0(root),10(wheel)
```

Root, no password. Boot log even shows `Starting telnetd: OK` — just no network interface for it to listen on.

### The blank chip has a name: LomboTech N7V5

Kernel strings and props give it away:

```
ro.product.chip=n7v5
ro.product.name=n7v5_alcor_lightgun
ro.build.user=dashine
ro.build.host=gaime-gun-001
```

Same ODM as the console. Cortex-A7 class Linux box, ~128 MiB RAM visible to Linux, NPU for the aim model. CV path runs **224×224 @ 60 fps** into an NN — not a 1080p pretty stream. Camera source is `/dev/video3` only after `libvideo` brings the ISP pipeline up.

There is **no USB DFU path** from the host; the console never ships gun firmware blobs. Inside the rootfs, `fw_upgrade` / `fw_ab_upgrade` exist but nothing over USB invokes them from the console side.

---

## What’s open

- Permanent one-byte ADB property fix on the console (button snapped during teardown — FEL still works)
- Namco System 22 debug hooks and the Unity ROM asset
- Real camera frames out of the gun (need clean shutdown of vendor `gun` process + libvideo bridge; abrupt kill reboots the box)
- In-circuit SPI NAND dump via the SoC once UART is the daily driver

## Tooling (safe by construction)

Decrypt/encrypt GAIMEENC, IMAGEWTY unpack, sparse/lpunpack, **read-only FES dumper**, gated FES writer, UART shell, gun HID probe, macOS gun bridge — all in the repo.

## Bottom line

| Half | Surprise |
|---|---|
| Console | Android 13 Allwinner kiosk, public AES key, no secure boot, **licensed System 22 core** |
| Gun | Separate **LomboTech N7V5** Linux + NPU, absolute HID, root on J3, UVC is a fake pattern |

If you only want a cheap absolute light gun for RetroArch/MAME, the gun already works on a PC — just filter the jitter and seize the digitizer on macOS.

Not affiliated with Tassei Denki, Bandai Namco, or Dashine. Device-unique IDs redacted; game ROMs not republished.

**Code & full lab notes:** [github.com/mattkanwisher/gaime_mods](https://github.com/mattkanwisher/gaime_mods)

Originally sketched as short posts on X: [console open](https://twitter.com/kanwisher/status/2082111462328414400) · [gun teardown](https://twitter.com/kanwisher/status/2082110983011729786).
