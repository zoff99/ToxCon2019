# Alpine Linux as an Embedded Systems Distribution Strategy

by strfry & nero

---

## Summary

1) Introduction to Alpine Linux
2) Typical embedded system challenges and how to solve them
3) Live Demo on Raspberry Pi

---

## Why Alpine Linux

What is Alpine Linux?

- Popular in Docker/container environments
- Binary distro
- Diskless mode
- OpenRC

---

## Why Alpine Linux

Low Binary Size

- musl libc
- busybox vs. coreutils
- Low memory footprint: Suited for boot-to-RAM

---

## musl libc vs. GNU libc

"It's called Linux-minus-GNU"

---

## musl libc vs. GNU libc

"It's called Linux-minus-GNU"

- MIT/BSD-licensed
- Binary incompatibilty
- Less functionality: No NSS, no Localization
- glibc breakages on update (2.27 ucontext)

---

## Packaging toolchain: abuild & friends

- easy to set up (compared to Debian...)
- APK format: Tarballs (can be easily generated by hand)
- apk vs dpkg: fast, pure C & shell, fewer runtime checks 
- Setting up a custom repository is easy

---

## Example APKBUILD

```bash
pkgname=toxic
pkgver=0.8.3
pkgrel=0
pkgdesc=" An ncurses-based Tox client"
url="https://github.com/JFreegman/toxic"
arch="all"
subpackages="$pkgname-doc"
makedepends="toxcore-dev libconfig-dev ncurses-dev openal-soft-dev linux-headers
    libvpx-dev opus-dev libnotify-dev libqrencode-dev curl-dev libx11-dev"
source="$pkgname-$pkgver.tar.gz::https://github.com/JFreegman/$pkgname/archive/v$pkgver.tar.gz"

build() { make ; }
package() { make PREFIX="$pkgdir"/usr install ; }
```


---


## APK Overlays

- /etc -> hostname.apkovl.tar.gz
- managed through `lbu` command
- fully defines system configuration state

---

## Diskless Mode

- System is installed on `tmpfs` root on every Boot
- APK Overlays determine system configuration

---

## apkcache

- loaded package can be stored at a configured location
- file layout is identical to "real" APK repository


---


## Deployment Options: 

1) At installation: `setup-disk -o my.apkovl.tar.gz`
2) On cmdline: `apkovl=/path/to/apkovl.tar.gz`
    - also works with http:// URLs
4) Automatic detection on boot medium


---

## End of Part 1

- Questions?


---

## Part 2: Embedded Distribution challenges

- Flash Storage
- Distributing Updates
- Cross-compile for non-PC architecture
- Assembling for Exotic boot architectures


---

## Flash Storage

- Limited Write Cycles
- Sudden power loss happens often and unexpected
    <!--- Raspberry Pi doesnt even have poweroff button -->
- In case of FS corruption, physical access might be difficult

<!-- mention overlayfs -->

---

## Distributing Upgrades

- Provide custom packages through an APK repository
- If network is available, everything is updated on boot
- apkcache helps with load times / bandwidth, and partially offline scenarios

---

## Cross-compile for non-PC architecture


---

## Cross-compile for non-PC architecture

- Actually not really supported

---

## Cross-compile for non-PC architecture

- Just use containers with qemu-static:

    \# enable binfmt-misc
    cp $(which qemu-arm-static) /armhf-chroot/usr/bin
    chroot /armhf-chroot

---

## Bootloader

- raspberrypi-bootloader: Firmware blobs for VideoCore etc.
- Device Tree Blobs & Overlays

<!-- /boot medium is cluttered with firmware files, otherwise not a real problem: managed by raspberry-bootloader packages


 -->

---

## Part 3: Demo

- Example Use Case: Raspberry Pi with Pulseaudio & Network Streaming

---

## Practical Experience with Alpine RPi Image

1) Get Raspi Release at https://alpinelinux.org/downloads/
2) Unzip on SD Card
3) Add custom.apkovl.tar.gz

That's it!

----


## Show minimal apkovl

tree /etc

---


# Links:

https://wiki.alpinelinux.org

Overlay generator & qemu test script: https://github.com/nero/apkovl

Raspberry Pi example apkovl:
https://github.com/strfry/pidac.strfry.org

Alpine Linux Online Package Search:
https://pkgs.alpinelinux.org


# APKOVL pain points

- packages also fill /etc (lower priority)
- LBU doesn't ignore package default configs
- git integration