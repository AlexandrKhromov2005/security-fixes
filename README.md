# Security fixes

Memory-safety, denial-of-service, information-disclosure, and undefined-behaviour bugs I reported or fixed upstream.
Every row links to the public commit, pull request, or issue.

Last checked: **2026-10-28**. Only merged fixes and reports with a confirmed upstream fix are listed.

## Linux kernel

| Date | Subsystem | Fix | Class | Commit |
|------|-----------|-----|-------|--------|
| 2026-08-24 | `fs/smb/server` | Zero `FS_OBJECT_ID_INFORMATION` before filling it, preventing disclosure of up to 31 bytes from a previous, unsent response in a compound SMB2 request | Information disclosure (CWE-200) | [`399aa12450a6`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=399aa12450a61a5c73dc77e73f069ece9687c95d) |
| 2026-08-24 | `fs/smb/server` | Initialize `FileSystemControlFlags` in `FS_CONTROL_INFORMATION`, preventing an uninitialized-field information leak in compound SMB2 responses | Information disclosure (CWE-200) | [`c0cd3fc68241`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=c0cd3fc6824122014da2b3b0cb7ddeaa2946ec8e) |
| 2026-08-24 | `fs/smb/server` | Fill `FileSysIdentifier` in `FS_POSIX_INFORMATION` from `statfs()` with little-endian conversion, preventing an eight-byte information leak in compound SMB2 responses | Information disclosure (CWE-200) | [`db2267b27c05`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=db2267b27c054a6c2151ff7fbb67927e784f31d6) |
| 2026-08-27 | `drivers/net/slip` | Drop the `slip_devs[]` entry from `ndo_uninit`, so `sl_sync()` can no longer dereference a device that `netdev_run_todo()` already freed after releasing RTNL | Use-after-free (CWE-416) | [`2c4e7c42d77e`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=2c4e7c42d77e78ad595dbb9e4b5886b58b45d89d) |

Root-caused, reproduced and fixed by me from a Syzkaller report. Applied by Paolo Abeni to
`netdev/net`, now in mainline, `Cc: stable@vger.kernel.org`.

The three ksmbd fixes were applied by Namjae Jeon and included in the
[ksmbd fixes for 7.3-rc2](https://lists.openwall.net/linux-kernel/2026/09/01/1409),
now in mainline, `Cc: stable@vger.kernel.org`. Their dates above are patch submission dates.

## DPDK

| Date | Subsystem | Fix | Class | Commit |
|------|-----------|-----|-------|--------|
| 2026-09-18 | `lib/net` | Cast the IPv6 next-header field to `uint32_t` before shifting by 24 in `rte_ipv6_phdr_cksum()`, avoiding signed-shift overflow for protocol values ≥ 128 | Undefined behaviour (signed left shift overflow) | [`bfbec118e9df`](https://git.dpdk.org/dpdk/commit/?id=bfbec118e9df73b35b375b668424c33ff36e62f2) |

Found with AFL++ and UBSan while fuzzing `fuzz_cksum`; reproduced and fixed by me.
[Patch accepted](https://patches.dpdk.org/project/dpdk/patch/20260918074729.79205-1-haa@amicon.ru/)
and applied by Thomas Monjalon, `Cc: stable@dpdk.org`. Intel functional testing reported PASS.
The date above is the patch submission date.

## Userspace parsers

Bugs I found by fuzzing open-source parsers — mostly proxy clients and the YAML, image, and QR
libraries they bundle. I used clang libFuzzer with AddressSanitizer, UndefinedBehaviorSanitizer,
and LeakSanitizer for C/C++, and `go test -fuzz` for Go.
Minimal reproducers are included in the reports or available on request; no live exploits.

Dates below are PR merge dates or upstream fix dates.

### Merged fixes

| Date | Project | Fix | Class | PR |
|------|---------|-----|-------|----|
| 2026-10-28 | [quirc](https://github.com/dlbeer/quirc) | Bound the alignment-pattern search to the image dimensions to prevent a crafted-image CPU hang; guard a zero-length `memcpy` with a NULL source | Excessive iteration / DoS (CWE-834, [CVE-2026-110596](https://www.cve.org/CVERecord?id=CVE-2026-110596)); undefined behaviour | [#159](https://github.com/dlbeer/quirc/pull/159) |
| 2026-09-20 | [sing](https://github.com/SagerNet/sing) | Clamp the JSON comment scanner's string index after an incomplete escape, preventing an out-of-range slice panic on malformed input | Uncaught panic / DoS (CWE-248) | [#108](https://github.com/SagerNet/sing/pull/108) |
| 2026-08-30 | [Throne](https://github.com/throneproj/Throne) | Update bundled fkYAML to [v0.4.4](https://github.com/fktn-k/fkYAML/releases/tag/v0.4.4), fixing crafted Clash subscription YAML crashes reported in [#1746](https://github.com/throneproj/Throne/issues/1746) | Out-of-bounds read (CWE-125) | [#1801](https://github.com/throneproj/Throne/pull/1801) |
| 2026-08-16 | [rapidyaml](https://github.com/biojppm/rapidyaml) | Reject cyclic references instead of overflowing the stack in `resolve()` | Uncontrolled recursion (CWE-674) | [#657](https://github.com/biojppm/rapidyaml/pull/657) |
| 2026-08-12 | [fkYAML](https://github.com/fktn-k/fkYAML) | Fix an out-of-bounds read from a truncated BOM in the char encoding detector | Out-of-bounds read (CWE-125) | [#545](https://github.com/fktn-k/fkYAML/pull/545) |
| 2026-08-11 | [fkYAML](https://github.com/fktn-k/fkYAML) | Guard the empty context stack in the key-separator and `add_new_key` paths | Out-of-bounds read (CWE-125) | [#544](https://github.com/fktn-k/fkYAML/pull/544) |
| 2026-08-10 | [fkYAML](https://github.com/fktn-k/fkYAML) | Fix an out-of-bounds read on an explicit key without a parent context | Out-of-bounds read (CWE-125) | [#543](https://github.com/fktn-k/fkYAML/pull/543) |
| 2026-08-07 | [Throne](https://github.com/throneproj/Throne) | Harden the bundled quirc QR decoder against a crafted-image hang and a NULL `memcpy` | Excessive iteration / DoS (CWE-834); undefined behaviour | [#1745](https://github.com/throneproj/Throne/pull/1745) |

The quirc and Throne QR-decoder rows cover the same two fixes in the upstream library and its vendored copy.

[CVE-2026-110596](https://www.cve.org/CVERecord?id=CVE-2026-110596) was published on 2026-10-14 for the
`find_alignment_pattern()` denial of service. The [official CVE record](https://github.com/CVEProject/cvelistV5/blob/main/cves/2026/110xxx/CVE-2026-110596.json)
credits me (`AlexandrKhromov2005`) as a co-discoverer (`finder`), alongside `JohannYellow` and
HaiND from the Post and Telecommunication Institute of Technology. The NULL-source `memcpy` guard
is an additional undefined-behaviour fix in the same PR.

### Reported, fixed upstream

| Date | Project | Bug | Class | Report → Fix |
|------|---------|-----|-------|--------------|
| 2026-08-16 | [rapidyaml](https://github.com/biojppm/rapidyaml) | Unbounded alias expansion ("billion laughs") in `resolve()` | Recursive expansion (CWE-776) | [#656](https://github.com/biojppm/rapidyaml/issues/656) → [#658](https://github.com/biojppm/rapidyaml/pull/658) |
| 2026-08-15 | [fkYAML](https://github.com/fktn-k/fkYAML) | Stack overflow from a self-referential alias | Uncontrolled recursion (CWE-674) | [#536](https://github.com/fktn-k/fkYAML/issues/536) → [#549](https://github.com/fktn-k/fkYAML/pull/549) |
| 2026-08-10 | [sing-box](https://github.com/SagerNet/sing-box) | Crafted binary `.srs` rule-set lengths trigger excessive allocation or a slice-length panic before the declared elements are read | Uncontrolled allocation / DoS (CWE-789) | [#4399](https://github.com/SagerNet/sing-box/pull/4399) → [`f260e771ea3f`](https://github.com/SagerNet/sing-box/commit/f260e771ea3f9ba3902e83cae1b63a66dc83c960) |
| 2026-08-10 | [sing](https://github.com/SagerNet/sing) | Crafted serialized domain-matcher lengths trigger excessive allocation or a slice-length panic, also reachable through AdGuard items in sing-box `.srs` rule-sets | Uncontrolled allocation / DoS (CWE-789) | [#107](https://github.com/SagerNet/sing/pull/107) → [`7c349dacf402`](https://github.com/SagerNet/sing/commit/7c349dacf402256d3a7029746073b05d2ead584a) |

For the two allocation reports, maintainers implemented the upstream fixes; my PRs were closed as superseded.

Additional findings are under coordinated disclosure and will be listed here once they are fixed and public.
