<p align="center"><img src="https://raw.githubusercontent.com/go-gnulinux/brand/main/social/go-gnulinux.png" alt="go-gnulinux" width="640"></p>

<h1 align="center">go-gnulinux</h1>
<p align="center">Linux system interfaces in pure Go — reached through <code>/sys</code>, <code>ioctl</code> and ordinary file descriptors, with no cgo.</p>
<p align="center">
  <img src="https://img.shields.io/badge/Go-1.26-00ADD8?style=flat-square&logo=go&logoColor=white">
  <img src="https://img.shields.io/badge/license-BSD--3--Clause-0A6E96?style=flat-square">
  <img src="https://img.shields.io/badge/cgo-none-0079A8?style=flat-square">
  <a href="https://github.com/go-authn"><img src="https://img.shields.io/badge/protocol-go--authn-0079A8?style=flat-square"></a>
</p>

---

## What this is

The Linux side of a fleet that already has
[`go-macos`](https://github.com/go-macos) and
[`go-mswin`](https://github.com/go-mswin). Same rule throughout:
`CGO_ENABLED=0`, no wrapper around a C library, and nothing claimed that has
not been checked.

## Repos

| | Repo | |
|---|---|---|
| <img src="https://raw.githubusercontent.com/go-gnulinux/brand/main/avatar/go-gnulinux-fido.png" width="36"> | [`fido`](https://github.com/go-gnulinux/fido) | A FIDO security key over `hidraw`: sysfs enumeration, the `uevent` file parsed by hand, usage page `0xF1D0` through `HIDIOCGRDESC`. |
| <img src="https://raw.githubusercontent.com/go-gnulinux/brand/main/avatar/go-gnulinux-factors.png" width="36"> | [`factors`](https://github.com/go-gnulinux/factors) | The key as an [`mfa.Factor`](https://github.com/go-authn/mfa). What only Linux knows: where a key is, and what its absence means. |

## libudev is a C library, and it is avoidable

The reference implementation enumerates HID devices through `libudev`, which
would cost cgo. It does not have to: udev is used only to *list* the hidraw
nodes, and `/sys/class/hidraw` already lists them. Everything else libfido2
wants, it parses by hand out of the sysfs `uevent` file — so that is what
happens here.

## Three things this cost, written down so the next one is cheaper

**The obstacle is a permission, not an API.** `/dev/hidraw*` belongs to root on
a stock system. Reporting that as *no key found* would send a person to check a
cable when the answer is a udev rule, so the two are separate errors and the
one that matters names the rule.

**The ioctl number is not the same on every architecture.**
`asm-generic/ioctl.h` wraps `_IOC_SIZEBITS` in an `#ifndef`, and powerpc and
mips override it — 13 bits instead of 14, which moves the direction field down
one and changes every request number. A wrong request earns `ENOTTY`, which
would have been reported as *not a security key*: a real key found, filtered
out, never mentioned. **Cross-compiling all eleven architectures does not catch
it** — the arithmetic compiles perfectly and produces the wrong number.

**A hidraw write carries one byte more than the report; a read does not.**
Linux wants a leading report-id going out and returns the report without it. On
macOS that byte is a separate argument to IOKit, so a transport ported across
without noticing sends every report shifted by one — and a key answers that
with silence rather than an error.

## What has not been done

The hidraw half has **never run against a key**: the machine this was written
on is a Mac. The report-descriptor parser is checked against seventeen real
descriptors with IOKit's own verdict as the judge, and none of them is a
security key — which is the negative control. Both repos say so in their own
READMEs rather than leaving it to be discovered.

## Links

- 🎨 Brand assets — <https://github.com/go-gnulinux/brand>
- 🌐 <https://go-gnulinux.github.io>

---

<p align="center"><sub>No cgo. No libudev. Eleven architectures.</sub></p>
