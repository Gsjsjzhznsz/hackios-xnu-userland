# HackiOS — userland

The user-space half of **HackiOS**: Apple's open-source XNU kernel on Android phone hardware (POCO X5 Pro 5G) and in
QEMU, running the unmodified iOS 16.5 user space. The kernel, loader, UEFI and the `common/` scripts are in
[hackios-xnu-kernel](https://github.com/oosexgang/hackios-xnu-kernel) — **start there**.

> **No Apple files are in this repository.** `ios165/fetch.sh` downloads the iOS 16.5 IPSW from Apple's CDN to your
> machine and extracts what is needed there. HackiOS is an independent research project by OOSEX GANG and is not
> affiliated with, authorised or endorsed by Apple Inc.

## Getting it

The two repositories are built as one tree. Clone the kernel repository and let it bring this one in:

```sh
git clone https://github.com/oosexgang/hackios-xnu-kernel.git hackios
cd hackios
common/setup.sh        # checks this repository out into the same directory (git dir .userland.git)
common/ugit status     # git for this repository inside that tree
```

## What is here

| Path | What |
|---|---|
| `userland/build.sh` | builds the APFS images: `root.apfs` (our init, libraries, launchd jobs, firmware) and `system.apfs` (the selected part of iOS 16.5 and its cryptexes); every option is documented at the top |
| `userland/src/` | `init.c` (process 1), injected libraries (`inject.c` backboardd, `inject_sb.c` SpringBoard, `inject_media.c` mediaserverd, `inject_tweaks.c`), the CoreAudio output device (`audio_plugin.c`), the OpenGL ES bridge to Mesa (`eagl.c`), Metal driver work, tests |
| `userland/libmini/` | a tiny libc for the static programs |
| `userland/rc.*`, `com.redwood.*.plist` | job lists and launchd jobs for test scenarios (`RC_JOBS=rc.jobs.tour`, `rc.jobs.ssh`, ...) |
| `userland/mkjobs.py`, `jobinfo.py` | the stock iOS launchd jobs: which are loaded, who provides a Mach service |
| `userland/var/` | our default preferences for `/private/var` |
| `ios165/` | `fetch.sh` / `fetch-from-ipsw.py` (take iOS out of the IPSW), `select-system.py` (what goes into the image) |
| `tools/apfs/` | APFS on the computer: `mkapfs.py` (image writer), reader, checks, write tests |
| `tools/mk-mesa.sh`, `tools/mesa/` | Mesa 25.0.7 with our patches: freedreno on Darwin, the AIR (Metal IR) frontend |
| `tools/air/`, `tools/metal/`, `tools/metallib.py` | Metal shader (AIR) checks and analysis; notes in `docs/air.md` |
| `tools/mk-jb.sh`, `jb-preinstall.py` | Procursus bootstrap + Sileo for the package manager prefix |
| `tools/font_trim.py` | trims two oversized iOS fonts so the image fits |
| `prebuilt/userland/` | our programs and libraries, built — used when there is no cross toolchain |

## Building the images

```sh
ios165/fetch.sh               # once
REST=1 userland/build.sh      # -> userland/root.apfs, userland/system.apfs
GL=1 REST=1 userland/build.sh # with OpenGL ES (Mesa) for the window server; on the phone with gpu=1
```

Without the cross toolchain (theos + iPhoneOS16.5.sdk, see the kernel repository) the build takes our programs from
`prebuilt/userland`; with it, it compiles `userland/src`.

A job list (`RC_JOBS=file`) is run by our `init`, not by a shell: iOS 16.5's root has no shell, lines are split on
blanks, there is no quoting and no variable assignment; only `>> FILE` and a trailing `&` are understood.

## Licenses

A license for our own code here is not chosen yet: until a LICENSE file appears, all rights are reserved by OOSEX GANG. Mesa (MIT) is not in the
repository: `tools/mk-mesa.sh` fetches release 25.0.7, `tools/mesa/*.patch` are our changes to it. apfsprogs
(GPL-2.0) is not in the repository either: `tools/apfs/check-image.sh` fetches and builds only `apfsck`.

OOSEX GANG — [t.me/oosexgangdevs](https://t.me/oosexgangdevs)
