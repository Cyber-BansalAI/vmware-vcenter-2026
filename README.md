# VMware vCenter flaw CVE-2026-123890: indicators and hunt guide

> **Read first.** This package is aggregated from public research and vendor documents by SOC-Lite by BansalAI. It is not
> tested by us, it comes with no warranty, and it is not professional advice. It is as of **2026-11-19**, the day the
> flaw was published, and it will go stale. See [DISCLAIMER.md](DISCLAIMER.md). We are not affiliated with any company
> credited in [CREDITS.md](CREDITS.md), and none of them reviewed or endorsed this package.

On 2026-11-19 Broadcom published advisory VMSA-2026-0009. One flaw in it, **CVE-2026-123890**, lets someone with network
access to VMware vCenter get in without a password. Broadcom lists no workaround. The same day, Google's threat researchers
(Google Threat Intelligence Group and Mandiant) said a suspected China-linked group had used the flaw since early September
2026, before any fix existed. They describe one break-in: the group got an administrator session on vCenter, copied the
directory database and the key that signs single sign-on tokens, then moved on to ESXi hosts and copied files out of
virtual machines.

This repository copies **the indicators the report publishes** (with credit) and adds a guide to checking your own systems.

**Not technical? Start with the plain-language version:**
[VMware vCenter Update: What to Ask Whoever Runs Your Servers](https://blog.bansalai.com/news/vmware-vcenter-flaw-what-to-ask/?ref=github)
on the SOC-Lite blog. It gives five questions to put to your IT person or provider.

## What is in here

| File | What |
|---|---|
| [`iocs/iocs.csv`](iocs/iocs.csv) | One row per indicator, with the source, the date, how fragile it is, and a review date |
| [`docs/hunt-guide.md`](docs/hunt-guide.md) | How to check your version, the network, vCenter, ESXi hosts and the Linux machines near vCenter, what the results do and do not mean, and what to do if something looks real |
| [`CREDITS.md`](CREDITS.md) | The research and documents this is built from, and what we took from each |
| [`DISCLAIMER.md`](DISCLAIMER.md) | The full disclaimer |

## Where the indicators come from

**Rows:** 32 · **verified (two or more sources):** 0 · **single source:** 32

These counts match [`iocs/iocs.csv`](iocs/iocs.csv); we check them every time we change the file (a test checks them in our
private working copy). Every value was read from the publisher's own copy of the report. One source means no row is verified
yet. What we took, and where it came from, is in [CREDITS.md](CREDITS.md).

### Copied into `iocs.csv` (indicator values, with credit)

| Source | Rows | Rows by type | What we took | Link |
|---|---|---|---|---|
| **Google Threat Intelligence Group and Mandiant** | 32 | 11 ip, 5 filename, 6 file-path, 3 string, 1 account, 6 behaviour | The report's two indicator tables (eleven addresses and six file names), and the paths, ports, account-name prefix, SSH banner, tool file names and described behaviour that the report's text gives | https://cloud.google.com/blog/topics/threat-intelligence/armory-castle-exploits-vsphere |
| **Total** | 32 | | | |

### What is not here, and why

- **No file hashes, domains or key fingerprints.** The report does not publish them in its text. Google says its full list
  is in a collection "for registered users", which we have not opened:
  [the Google Threat Intelligence collection](https://www.virustotal.com/gui/collection/1ea34ca05aa0bed8cce4f29cba20b8ecfd46c9401a3d6eae74769f2e4415b9d6/summary).
  If you have access, load the hashes from there, because that is Google's own copy. We do not guess or reconstruct values.
- **No detection rules.** Google published six rules for the group's tools. They are code under Google's terms, so we link
  to them: see the report's "Detections" section (link above).
- **The rest of the account name.** The group created a local administrator whose name starts `sps-`. The report redacts the
  rest, and so do we.

## Read the results carefully

- **A match is a reason to look, not a verdict.** Most of the file and path rows describe where the report found a tool on
  a separate Linux machine, not on vCenter. The group can rename every file.
- **The eleven addresses are a suspected network of other people's hacked firewalls and routers.** The report gives no date,
  no port and no link to a victim. Such addresses change hands quickly. **Do not mass-block them**; look each one up against
  your own logs first.
- **The report describes one break-in.** We found no public count of victims.
- **Your own records may not reach back far enough.** vCenter keeps its own task and event history for 30 days by default,
  and the report says the group removed its tools after use. A clean result is not an all-clear.
- **This is one source.** Every row comes from one report, so no row is marked verified. We mark a row verified only when
  two or more independent publishers report it.
- **There is more in the report than we copy.** Read it for the full description of how the group worked (link in
  [CREDITS.md](CREDITS.md)).

## Using `iocs.csv`

| Column | Meaning |
|---|---|
| `id` | Stable row ID (`VC-nnnn`) |
| `wave` | Always `vcenter` in this release |
| `type` | `ip`, `filename`, `file-path`, `string`, `account` or `behaviour` |
| `value` | The indicator. Addresses are written as normal text, not defanged. Never open them in a browser |
| `match` | `exact`, `substring`, `prefix` or `wildcard` |
| `evidence` | Where it would show: `network`, `file`, `process`, `access-log` or `event-log` |
| `fragile` | `yes` if the attacker can change it trivially |
| `verified` | `yes` only when two or more independent primary sources report it. `no` means a single source |
| `source_url`, `source_date` | Where we took it from and the date of that source |
| `fetched`, `review_by` | When we read it, and when it should be re-checked (30 days for every row here) |
| `notes` | Context and caveats. Sentences starting "Our note" are ours, not the source's |

Search your own systems with the steps in the hunt guide. This package does not ship anything that runs on your systems or
collects data from them.

## Licence

The text we wrote is licensed under CC BY 4.0 ([LICENSE](LICENSE)). The indicator values and facts belong to the research
and documents that reported them: they are credited in [CREDITS.md](CREDITS.md), and we do not relicense them. Details and
how to credit us are in [NOTICE.md](NOTICE.md).

## Where to go for the full, current picture

Broadcom's advisory is the authority on which versions are affected and fixed. Google's report is the authority on what the
group did. Both may be updated: follow the links in [CREDITS.md](CREDITS.md).

## Help with a suspected compromise

If something looks real, preserve the logs first and call your IT provider. Section 8 of the hunt guide lists what to do and
in what order. [SOC-Lite by BansalAI](https://cyber.bansalai.com/?ref=github) helps small and mid-sized Central Florida
businesses review alerts and work with their IT person or MSP.
[Book a free 20-minute visibility check](https://cyber.bansalai.com/?ref=github#book). More plain-language security
write-ups are on the [SOC-Lite blog](https://blog.bansalai.com/?ref=github).
