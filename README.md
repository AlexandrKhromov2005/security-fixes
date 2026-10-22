# Security fixes

Memory-safety, denial-of-service, information-disclosure, and undefined-behaviour bugs I reported and fixed upstream.
Every row links to the public commit, pull request, or issue.

## Linux kernel

| Date | Subsystem | Fix | Class | Commit |
|------|-----------|-----|-------|--------|
| 2026-08-24 | `fs/smb/server` | Initialize `FileSystemControlFlags` in `FS_CONTROL_INFORMATION`, preventing an uninitialized-field information leak in compound SMB2 responses | Information disclosure (CWE-200) | [`c0cd3fc68241`](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/commit/?id=c0cd3fc6824122014da2b3b0cb7ddeaa2946ec8e) |
| 2026-08-27 | `drivers/net/slip` | Drop the `slip_devs[]` entry from `ndo_uninit`, so `sl_sync()` can no longer dereference a device that `netdev_run_todo()` already freed after releasing RTNL | Use-after-free (CWE-416) | [`2c4e7c42d77e`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=2c4e7c42d77e78ad595dbb9e4b5886b58b45d89d) |

Root-caused, reproduced and fixed by me from a Syzkaller report. Applied by Paolo Abeni to
`netdev/net`, now in mainline, `Cc: stable@vger.kernel.org`.

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
libraries they bundle. Found with clang libFuzzer under AddressSanitizer,
UndefinedBehaviorSanitizer, and LeakSanitizer. Each report ships a minimal reproducer;
no live exploits.

### Merged fixes

| Date | Project | Fix | Class | PR |
|------|---------|-----|-------|----|
| 2026-08-16 | [rapidyaml](https://github.com/biojppm/rapidyaml) | Reject cyclic references instead of overflowing the stack in `resolve()` | Uncontrolled recursion (CWE-674) | [#657](https://github.com/biojppm/rapidyaml/pull/657) |
| 2026-08-12 | [fkYAML](https://github.com/fktn-k/fkYAML) | Fix an out-of-bounds read from a truncated BOM in the char encoding detector | Out-of-bounds read (CWE-125) | [#545](https://github.com/fktn-k/fkYAML/pull/545) |
| 2026-08-11 | [fkYAML](https://github.com/fktn-k/fkYAML) | Guard the empty context stack in the key-separator and `add_new_key` paths | Out-of-bounds read (CWE-125) | [#544](https://github.com/fktn-k/fkYAML/pull/544) |
| 2026-08-10 | [fkYAML](https://github.com/fktn-k/fkYAML) | Fix an out-of-bounds read on an explicit key without a parent context | Out-of-bounds read (CWE-125) | [#543](https://github.com/fktn-k/fkYAML/pull/543) |
| 2026-08-07 | [Throne](https://github.com/throneproj/Throne) | Harden the bundled quirc QR decoder against a crafted-image hang and a NULL `memcpy` | Infinite loop / DoS (CWE-835) | [#1745](https://github.com/throneproj/Throne/pull/1745) |

### Reported, fixed upstream

| Date | Project | Bug | Class | Report → Fix |
|------|---------|-----|-------|--------------|
| 2026-08-21 | [Throne](https://github.com/throneproj/Throne) | Crafted Clash subscription YAML crashes the bundled fkYAML deserializer (three distinct over-reads) | Out-of-bounds read (CWE-125) | [#1746](https://github.com/throneproj/Throne/issues/1746) → [fkYAML v0.4.4](https://github.com/fktn-k/fkYAML/releases/tag/v0.4.4) |
| 2026-08-16 | [rapidyaml](https://github.com/biojppm/rapidyaml) | Unbounded alias expansion ("billion laughs") in `resolve()` | Recursive expansion (CWE-776) | [#656](https://github.com/biojppm/rapidyaml/issues/656) → [#658](https://github.com/biojppm/rapidyaml/pull/658) |
| 2026-08-15 | [fkYAML](https://github.com/fktn-k/fkYAML) | Stack overflow from a self-referential alias | Uncontrolled recursion (CWE-674) | [#536](https://github.com/fktn-k/fkYAML/issues/536) → [#549](https://github.com/fktn-k/fkYAML/pull/549) |

Additional findings are under coordinated disclosure and will be listed here once they are fixed and public.
