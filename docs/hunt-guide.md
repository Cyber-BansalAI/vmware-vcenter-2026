# Hunt guide: did a group get into our vCenter before the fix?

As of **2026-11-19**. Read [DISCLAIMER.md](../DISCLAIMER.md) first. We have not tested these commands on your systems. Run
them yourself, on copies of the logs where you can, and adjust names and paths to your setup. Nothing in this package runs
anything for you, and every command shown only reads.

**Who this is for.** The person who runs VMware vCenter for a business: an IT lead, or the provider that does it for them.
The business-owner version, with five questions to put to that person, is in
[our blog post](https://blog.bansalai.com/news/vmware-vcenter-flaw-what-to-ask/?ref=github).

## What happened, in five lines

- On 2026-11-19 Broadcom published advisory VMSA-2026-0009. One flaw in it, CVE-2026-123890, lets someone with network
  access to vCenter get in without a password. Broadcom lists no workaround.
- The same day Google's threat researchers (Google Threat Intelligence Group and Mandiant) said a suspected China-linked
  group had used the flaw since **early September 2026**, before any fix existed. They describe one break-in.
- In that break-in the group was already inside the victim's network. It got an administrator session on vCenter, copied
  the directory database and the key that signs single sign-on tokens, made forged sign-in tokens, switched SSH on and off
  through the vCenter management interface, took the vCenter's master key and database password, then logged in to ESXi
  hosts as root and copied files out of virtual machines, including Active Directory files.
- The group removed its own tools after use, and vCenter keeps its own event history for 30 days by default. Neither fact
  is good for finding it now.
- The indicators the report publishes are few: eleven addresses, six file names and some behaviour. No file hashes, domains
  or key fingerprints are in its text. Google says its full list is in
  [a collection for registered users](https://www.virustotal.com/gui/collection/1ea34ca05aa0bed8cce4f29cba20b8ecfd46c9401a3d6eae74769f2e4415b9d6/summary).
  We have not opened it: if you have access, load the hashes from there.

## Two questions, kept apart

1. *Did anyone use the flaw, or hold administrator rights on our vCenter, without our knowing?* Answered from vCenter, its
   single sign-on records and the network.
2. *Did they go further: onto ESXi hosts, into our virtual machines, into our directory?* Answered from the hosts, the
   Linux appliances near vCenter and the data in the virtual machines.

A clean answer to the first does not answer the second.

## 0. Before you touch anything

Our reasoning, not Broadcom's: **copy the evidence first, then patch.** Patching restarts services and some of it reboots
hosts. Take these steps in this order:

1. Copy what logs you can reach (section 3) to somewhere the vCenter cannot write to.
2. Take offline snapshots of the vCenter appliances. Broadcom asks for this before it refreshes the token-signing
   certificate, and it also gives you something to compare against.
3. Install the fixed version (section 1). Patching stops new use of the flaw. It does not remove anything already done.
4. Hunt (sections 2 to 6) and, if you find something, see section 8.

Set your window: the report puts the use of the flaw in **early September 2026**. Search from 2026-08-01 if your records go
back that far. If they do not, write that down: you cannot rule out the earlier period.

## 1. Are we on a fixed version?

Broadcom's matrix for this flaw, read on 2026-11-19. No version is listed as safe for older builds:

| You run | Fixed version | Note |
|---|---|---|
| vCenter in Cloud Foundation or vSphere Foundation 9.1 | 9.1.1.0200 | |
| vCenter in Cloud Foundation or vSphere Foundation 9.0 | 9.0.2.0200 | |
| vCenter 8.0 | 8.0 U3m | |
| Cloud Foundation 5.x | vCenter 8.0 U3m through the asynchronous patching process (Broadcom KB 88287) | Cloud Foundation itself moves to 5.2.5 |
| vCenter 7.0 | No public fix | Only under an extended support contract, through Broadcom Support |
| Telco Cloud Platform or Infrastructure | See Broadcom KB 463116 | |

Things that matter:

- A vCenter patched in July or August for the earlier flaws (VMSA-2026-0006) **is still open** to this one. Broadcom calls
  this flaw "new and distinct".
- Google says its attack tool works against vCenter 7.x and 8.x. Broadcom lists 7.0, 8.0, 9.0 and 9.1. Treat every version
  as affected until you have the fixed build.
- The update is not eligible for Quick Patch, and vCenter is unavailable for a short while. Virtual machines keep running.
- 8.0 U3m and 9.0.2.0200 block a later upgrade to Cloud Foundation 9.1.1 until Broadcom's next release (Broadcom calls it a
  "Back-in-Time" restriction). If an upgrade is in progress, read the FAQ before you patch.

Read your build number in the vSphere Client (Summary tab) or with PowerCLI:

```powershell
Connect-VIServer -Server <your-vcenter>
$global:DefaultVIServer.Version
$global:DefaultVIServer.Build
```

## 2. The network around vCenter

The report says the group reached vCenter on **TCP 2012**, which Google describes as the control interface for single
sign-on, and scanned for 2012, 5480 (the appliance management interface), 9080, 443, 389 and a few others from a Linux
appliance inside the network.

Ask your firewall or flow-data owner for every connection to the vCenter addresses on these ports over the last 90 days:
2012, 389, 636, 5480, 22 and 9080. Then look for:

- **Hosts that have no reason to be there.** Which machines legitimately need TCP 2012 on version 8.0 and 9.0 is not
  something we could confirm: Broadcom's current ports page would not load for us. Start from this: your other vCenter
  servers in the same single sign-on domain, and your administrator workstations. Ask your VMware integrator or Broadcom for
  the real list.
- **Port scans.** One source sending connection attempts to several of those ports on many servers in a short time. The
  report shows five such scans from inside the victim's network.
- **Backup and monitoring servers that can reach 22 or 5480.** Mandiant's guidance warns that trusting a server "for all
  ports" hands it the management ports too.

Then compare against the dataset. Read the addresses from `iocs/iocs.csv` and search an exported firewall or proxy log (adjust
the file name):

```powershell
$ips = (Import-Csv .\iocs\iocs.csv | Where-Object { $_.type -eq 'ip' }).value
Select-String -Path .\firewall-export.log -SimpleMatch -Pattern $ips
```

How to read a match:

- The eleven addresses are what Google calls a **suspected obfuscation network**: other people's hacked firewalls and routers
  that the group used as a hop. The report gives no date, no port and no link to a victim. Such addresses change owners
  quickly. A match deserves a look at **when** and **what it did**. It is not proof, and a miss proves nothing.
- Do not mass-block them without looking: an address can belong to an innocent owner by now.

Outbound from vCenter and the Linux appliances near it:

- vCenter's normal outbound traffic is short: its ESXi hosts, DNS, time servers, your log collector, directory servers if it
  is joined, and Broadcom's update sites. Anything else to the internet is worth a question.
- The report's backdoor on a Linux appliance talks to **Slack** over HTTPS every 150 to 450 seconds. Slack is a legitimate
  service. A server that has no reason to use it, calling it on a steady beat, is the signal.
- An SSH server answering on **TCP 53 or TCP 33992** is unusual. The report also found an SSH banner of
  `SSH-2.0-pulsesecure_23345` on odd high ports of the hop machines. That banner imitates another vendor's naming: it is the
  group's choice, not a fact about that vendor.

## 3. Logs: where the evidence lives, and how much is left

**vCenter keeps its own task and event history for 30 days by default** (Broadcom's security controls guide, setting "Set Task
and Retention Interval"). Our reasoning: unless you raised it, vCenter's own record of early September is gone. Look to your
log collector or archives first.

Check what you have. Read-only, on the vCenter appliance and with PowerCLI:

```powershell
Connect-CISServer -Server <your-vcenter>
(Get-CisService -Name "com.vmware.appliance.logging.forwarding").get()
```

An empty result means the appliance forwards nothing: the only copy is on the appliance.

```sh
systemctl status auditd
auditctl -l
```

An empty rule list means nothing was watching file reads and commands on the appliance. Mandiant's guide says the appliance
does not forward its kernel audit log by default. We found no vendor page that tells you how to see, after the fact, that
the master key or database password files were read. Treat that as not detectable unless you had audit data forwarded.

On every ESX host, remote logging is off unless someone set it up:

```powershell
Get-VMHost | Get-AdvancedSetting -Name Syslog.global.logHost | Select-Object Entity, Name, Value
```

Time zones and clocks: make sure every record you compare uses the same time zone, and that the appliances' clocks were right.

## 4. vCenter: accounts, permissions, sign-ins and SSH

What the report says the group did on vCenter: created a **local administrator account whose name starts `sps-`** (the rest
is redacted), created another single sign-on administrator, gave it Global Administrator at the root folder, signed in as the
built-in `administrator` account of the single sign-on domain, made forged sign-in tokens, and turned SSH on and off through
the management API.

**Accounts.** In the vSphere Client, open Administration, then Single Sign On, then Users and Groups, and read the list of
users in your single sign-on domain. Any account nobody on the team created is a finding, **above all one starting `sps-`.**
(We took this navigation from Broadcom's usual layout; we did not fetch the page.)

**Single sign-on audit log.** Two Broadcom pages give two locations for it, so check both:

```sh
ls -l /var/log/audit/sso-events/audit_events.log /var/log/vmware/sso/audit_events.log
```

Broadcom says the log records account changes, privilege changes, security changes and sign-ins. Look for those events
outside your change windows. The same events show in the vSphere Client under Monitor. Broadcom's own list of log files also
names the vmdir logs under `/var/log/vmware/vmdird/` and `/var/log/vmware/sso/`. How much the exploit leaves in them at the
default log level is **not documented**: do not treat an empty log as a clean one.

**Permissions.** Look for a permission with the Administrator role at the root folder, or a global permission, that nobody
granted. In the vSphere Client, open Monitor, then Events, and look for "permission added" events. With PowerCLI (we did not
run these, check the output makes sense on your version):

```powershell
Get-VIPermission | Where-Object { $_.Role -eq 'Admin' } | Select-Object Principal, Entity, Role, Propagate, IsGroup
Get-VIEvent -MaxSamples 10000 | Where-Object { $_.GetType().Name -eq 'PermissionAddedEvent' } | Select-Object CreatedTime, UserName, FullFormattedMessage
```

**Sign-ins.** Look for sign-ins by the built-in `administrator` account of the single sign-on domain from anywhere but your jump host. The event type for a
sign-in carries the address and browser string. Our reasoning, not a vendor's: tokens forged with a stolen signing key never
touch your identity provider, so a sign-in with **no matching sign-in at your identity provider** is worth a look.

**SSH on the vCenter appliance.** The group turned it on through the management interface, used it, and turned it off again.
You can read the current state, which should be off:

```powershell
(Get-CisService -Name "com.vmware.appliance.access.ssh").get()
```

**We could not find where a past turn-on is recorded.** No Broadcom or Mandiant page we read names the log file or string for
that. Google's own detection for it is called "VMware vCenter VAMI API SSH Enablement", but its text is not public. If you
forward appliance logs to a collector, search them for requests to `/rest/appliance/access/ssh` and for SSH logins.
Logins over SSH to the appliance are recorded in its authentication log (`auth.log`; check the system journal as well):

```sh
ls -l /var/log/auth.log
journalctl -u sshd --since "2026-08-01" --no-pager
```

Look for logins from unexpected addresses, and for logins by a local account named like `sps-...`.

## 5. ESXi hosts

The report says the group signed in to the ESXi host client as `vpxuser`, then over SSH as `root`, added its own key to root's
authorized keys, hid a folder on a datastore, and copied files out of virtual disks. These checks run on each host (or with
PowerCLI).

**Is SSH on?** The service should normally be stopped:

```powershell
Get-VMHost | Get-VMHostService | Where-Object { $_.Key -in 'TSM-SSH','TSM' } | Select-Object VMHost, Key, Running, Policy
```

In the host logs, the message `SSH access has been enabled` records a turn-on. Both Splunk's and Elastic's published
detections look for exactly this text. On the host, the logs are in `/var/run/log` (Broadcom KB 306962):

```sh
ls -l /var/run/log
```

**Root logins and `vpxuser`.** In `auth.log`, look for `Accepted publickey for root` on a host where you only use passwords,
and in `hostd.log` for the text `SSH session was opened for` followed by `vpxuser`. vCenter manages hosts through its API, not SSH, so any SSH
session by `vpxuser` is odd (Elastic's detection says the same). On 8.0 hosts older than update 3h, SSH logs may be in
`syslog.log` instead (this comes from a search result, so check).

**Commands run.** `shell.log` records every command, tagged with a session number. Match the number to `auth.log` to find
who and from where. Look for `scp`, `tar xzf`, `vmfs_browser.py`, `vmfs-pydist.tar.gz`, `o.tar.gz` and anything under a
hidden folder in `/vmfs/volumes/`.

**Root's authorized keys.** The file is `/etc/ssh/keys-root/authorized_keys` (we saw the directory only in a search result;
the file name is Broadcom's). Compare its keys with the keys you know, and look at the file's date:

```sh
ls -l /etc/ssh/keys-root/authorized_keys
cat /etc/ssh/keys-root/authorized_keys
```

**Hidden folders and tools on datastores.** The report names a hidden folder `.a`, the files `vmfs-pydist.tar.gz`,
`vmfs_browser.py` and `o.tar.gz`. The group deleted the folder, so absence proves little. This is our idea, no vendor
publishes a method:

```sh
ls -la /vmfs/volumes/<datastore-name>/
```

**Remote logging and whether `vpxuser` can open a shell.** Both are settings to review, not signs of harm:

```sh
esxcli system syslog config get
esxcli system account list
```

An empty remote log host means a hacker who clears the local logs leaves you nothing. In Broadcom's guide, shell access for
`vpxuser` is on by default on 8.0 and later hosts, and Mandiant recommends turning it off. That is a change: ask your
administrator, do not do it during a hunt.

## 6. Linux appliances next to vCenter

The report says most of the group's tools ran on **other Linux appliances** in the victim's network, not on the vCenter.
Hunt the machines that can reach vCenter port 2012: appliances, backup servers, monitoring servers. Search each for the
paths in the dataset:

```sh
ls -l /tmp/123 /tmp/sss /tmp/dbr /tmp/123.key /usr/bin/vmtoolscs
find /bin/env -perm -4000
ss -ltn
```

How to read it:

- A file at one of those paths is a strong finding. The group can rename its tools, so a miss proves nothing.
- `/bin/env` should not have the set-user-ID bit. If `find` prints it, the group's privilege step has been done.
- `ss -ltn` lists listening ports: look for an SSH server on 53 or 33992.
- Look in web server folders for `ActiveSer.jsp` and `u.jsp`, the report's two webshells.
- A service that restarts every 50 minutes and starts `vmtoolscs` is the report's backdoor: list recently changed service
  files in `/etc/systemd/system`.

Google also published six detection rules for these tools. We link to them, we do not copy them:
[the report's Detections section](https://cloud.google.com/blog/topics/threat-intelligence/armory-castle-exploits-vsphere).

## 7. What the results mean

| You found | It suggests | Next step |
|---|---|---|
| The vCenter is on a version below the fixed one, and nothing else | Open to the flaw, no sign of use | Patch (section 0), then keep looking |
| An address from the dataset in firewall data | A hop machine, or a reassigned address | Look at date and what it did. Keep the logs |
| An account or permission nobody can explain, above all one starting `sps-` | Likely administrator access that was not yours | Treat as an incident (section 8) |
| SSH was turned on for vCenter or an ESXi host and nobody asked | Possible hands-on-keyboard access | Treat as an incident |
| A key in root's authorized keys that you do not own | Persistence on that host | Treat as an incident |
| A file or listener from section 6 | The group's tooling on that machine | Treat as an incident |
| Slack traffic from a server that has no use for it | Possible backdoor | Look at the server; treat as an incident if confirmed |
| Nothing | Not proof of safety: 30-day history, tools removed, no hashes published | Fix the basics in section 9 anyway |

## 8. If you think it is real

Preserve first, change later:

1. Keep the logs, the snapshots and the appliances as they are. Do not rebuild yet.
2. Call your IT provider or an incident-response firm. Do not use machines that may be affected to talk about it.
3. Report to CISA and to the FBI's Internet Crime Complaint Center through their websites.
4. Then, in this order. **This order is our reasoning, not Broadcom's**: Broadcom's pages we read give patch advice but no
   compromise-response steps, and none covers rotating the vCenter master key or database password.
   1. Patch vCenter to the fixed build, so the same flaw cannot be used again.
   2. Refresh the single sign-on **token-signing certificate.** The report says the group copied that key, and forged tokens
      stay valid until it changes. Broadcom's procedure (KB 318197) is written for expiry, not theft: it asks for offline
      snapshots first, and a forced refresh can need every vCenter appliance in the domain restarted.
   3. Reset the password of every local single sign-on account and remove any you did not create (an `sps-` account).
   4. On every ESXi host: remove keys you do not own from root's authorized keys, change root's password, and have vCenter
      rotate the `vpxuser` password.
   5. Decide whether to **rebuild vCenter** from a known-good image, patched, restoring configuration only from a backup taken
      before September. Our recommendation, because the group held the master key and the database password. No vendor page
      says to rebuild after this flaw.
   6. If a domain controller or any other virtual machine that holds secrets ran on a host where the group had root, treat its
      secrets as taken. The report shows the group copying Active Directory files from one. Rotate those credentials.

## 9. Reduce the risk

These come from Google's and Mandiant's guidance and Broadcom's security configuration guide. Read them for the full list.

- Patch vCenter within the emergency window. Broadcom calls this an emergency change.
- Allow the management ports (22, 443, 5480, and 2012 where needed) only from named administrator machines. The default
  appliance firewall is managed through the same interface an attacker can reach, and it does not log refused connections,
  so Mandiant recommends a firewall in front. Also control **outbound** traffic from the appliance.
- Forward vCenter and ESXi logs to a collector, including the appliance's audit log, over an encrypted channel. Mandiant
  suggests keeping them at least 400 days. Make sure clocks agree.
- Mandiant publishes a hardening tool for the vCenter appliance under the Apache 2.0 licence. We have not read the script.
  Take a snapshot, read it, test it and keep console access before you run it: it can lock you out.
- Keep a file-based backup of the vCenter that an intruder cannot delete, and test a restore.
- Treat the built-in `administrator` account of the single sign-on domain as a break-glass account: its password in a
  password safe, rarely used, alerted on.

## What we could not confirm

- Where a past turn-on of SSH through the management API is logged.
- Which hosts legitimately need TCP 2012, 389 and 636 on 8.0 and 9.0, and what TCP 9080 is (the group scanned it).
- Hashes, domains and key fingerprints. The report does not publish them in its text (see the collection link at the top).
- A second source for any indicator. Everything in `iocs/iocs.csv` comes from one publisher, so nothing is marked verified.
- Whether CISA lists the flaw as exploited. Check its catalog again in a day or two.

## Not sure what you found?

Bring the log lines to your IT person first. If you want a second pair of eyes,
[SOC-Lite by BansalAI](https://cyber.bansalai.com/?ref=github) offers a free 20-minute
[visibility check](https://cyber.bansalai.com/?ref=github#book).

## Sources for this guide

Google Threat Intelligence Group and Mandiant, "ARMORY CASTLE Exploits vSphere" (2026-11-19); Broadcom, advisory
VMSA-2026-0009 and its questions and answers (2026-11-19), the vSphere security configuration guide, and knowledge base
articles 306962, 312194, 310254, 318197 and 321910; Mandiant, "vSphere and BRICKSTORM Malware: A Defender's Guide" and the
VCSA hardening tool; Elastic and Splunk published detections for ESXi SSH being enabled. Full credit, links and licence notes
are in [CREDITS.md](../CREDITS.md). Windows event names and PowerCLI commands are standard tools, not from the sources, and we
have not run them on your systems.
