# Credits and sources

This package is aggregated from the public research and vendor documents below. **We thank their authors for publishing
them.** Each indicator in `iocs/iocs.csv` names the source it came from. Being listed here does not mean any of them
reviewed, tested or endorsed this package.

We copy facts (indicator values, version numbers and figures) with attribution, and link to everything else. Where a source
holds more than we reproduce, follow the link for the full, current version.

## The research that supplies the indicators

| Who | What we used | Link |
|---|---|---|
| **Google Threat Intelligence Group and Mandiant**, "ARMORY CASTLE Exploits vSphere" (published 2026-11-19; written by Jayson Frost, Bryce Greathouse, Dhanesh Kizhakkinan, John Palmisano, Steve Rasch, Daniel Sislo and John Wolfram) | The eleven addresses in the report's network indicator table and the six file names in its file indicator table; the paths, ports, account-name prefix, SSH banner, file and tool names and the described behaviour that our behaviour rows and hunt guide rely on; what the group did and when | https://cloud.google.com/blog/topics/threat-intelligence/armory-castle-exploits-vsphere |

## Further detail from the same report (linked, not reproduced)

- **The full indicator collection.** The report says its complete list is kept in a Google Threat Intelligence collection "for
  registered users": https://www.virustotal.com/gui/collection/1ea34ca05aa0bed8cce4f29cba20b8ecfd46c9401a3d6eae74769f2e4415b9d6/summary.
  We have not opened it. File hashes, domains and key fingerprints are not published in the report itself, so none are in
  this package. If you have access, take them from there.
- **Six detection rules** for the group's tools (SLIMSAVAGE, CAGEMATCH, SAWSNAKE, BONESAW, BEARBONES, LAZYBONES), in the
  report's "Detections" section. Detection rules are code under their publisher's terms, so we link and do not copy them.
- **The report's hardening table** and its description of how a hardened appliance would have blocked each step.
- **Names of the vendor's own detections** for this activity, listed in the report.

## The vendor documents behind the fixed versions and the hunt guide

| Who | What we used | Link |
|---|---|---|
| **Broadcom**, security advisory VMSA-2026-0009 (published 2026-11-19; copyright line on the page: "Copyright 2026 Broadcom. All rights reserved.") | The flaw's description, severity, the affected and fixed versions per product line, "no workaround", and the credit to Mandiant | https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/39495 |
| **Broadcom**, questions and answers for VMSA-2026-0009 | The emergency-change wording, the statement that it is a "new and distinct issue", the support position on older versions, the update behaviour (not eligible for Quick Patch, short vCenter outage) and the "Back-in-Time" upgrade restriction | https://github.com/vmware/vcf-security-and-compliance-guidelines/blob/main/security-advisories/vmsa-2026-0009/README.md |
| **Broadcom**, security advisory VMSA-2026-0011 (published 2026-11-24; copyright line on the page: "Copyright 2026 Broadcom. All rights reserved.") | The newer vCenter flaw and the critical ESX flaws, the fixed versions per product line (vCenter 9.1.2, 9.0.2.0300 and 8.0 U3m; ESX 9.1.2, 9.0.2.0200 and 8.0 U3m), "no workaround" as listed, the Telco knowledge base article | https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/39626 |
| **Broadcom**, questions and answers for VMSA-2026-0011 | That vCenter 8.0 U3m needs no second update, that patches are cumulative, that the ESX updates are not eligible for Live Patch, that the updates cause no "Back-in-Time" restriction and that Cloud Foundation 9.1.2 resolves the earlier one | https://github.com/vmware/vcf-security-and-compliance-guidelines/blob/main/security-advisories/vmsa-2026-0011/README.md |
| **CISA**, "CISA Adds Four Known Exploited Vulnerabilities to Catalog" (2026-11-20) | The fact and date that CVE-2026-123890 was added to the Known Exploited Vulnerabilities catalog | https://www.cisa.gov/news-events/alerts/2026/11/20/cisa-adds-four-known-exploited-vulnerabilities-catalog |
| **Broadcom**, vCenter Server Security Controls Reference (vSphere 8.0) | The 30-day default for vCenter's local task and event history, and the settings and read-only PowerCLI checks for log forwarding and SSH | https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere/8-0/vsphere-security/understanding-vsphere-hardening-and-compliance/understanding-vsphere-security-configuration-guide/vcenter-server-security-controls-reference.html |
| **Broadcom**, ESXi Security Controls Reference (vSphere 8.0) | Defaults for ESXi remote logging, audit records and shell access for the `vpxuser` account | https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere/8-0/vsphere-security/understanding-vsphere-hardening-and-compliance/understanding-vsphere-security-configuration-guide/esxi-security-controls-reference.html |
| **Broadcom**, knowledge base articles on the location of ESXi log files, vCenter log files, SSO logging and ESXi shell auditing | Where the logs the hunt guide names live | https://knowledge.broadcom.com/external/article/306962/location-of-esxi-log-files.html, https://knowledge.broadcom.com/external/article/312194/location-of-vcenter-server-log-files.html, https://knowledge.broadcom.com/external/article/310254/enabling-debug-logging-for-sso-and-authe.html and https://knowledge.broadcom.com/external/article/321910/auditing-esxi-shell-logins-and-commands.html |
| **Broadcom**, knowledge base article on refreshing the STS signing certificate | The procedure the hunt guide points to (it is written for expiry, not for theft: the guide says so) | https://knowledge.broadcom.com/external/article/318197/sts-signing-certificates-are-about-to-ex.html |
| **Mandiant**, "vSphere and BRICKSTORM Malware: A Defender's Guide" (author named on the page: Stuart Carrera) | Hardening and logging ideas for the vCenter appliance and ESXi hosts that the guide's last section paraphrases. It was written about a different malware family and we use none of its indicators | https://cloud.google.com/blog/topics/threat-intelligence/vsphere-brickstorm-defender-guide/ |
| **Mandiant**, VCSA Hardening and Logging Tool (Apache License 2.0, "Copyright 2026 Google LLC") | Linked only. We have not read or run the script and do not tell anyone to run it without their own review | https://github.com/mandiant/vcsa-hardening-tool |
| **Elastic**, detection rules for VMware vSphere (Elastic License 2.0) | Linked only. We took the fact that Elastic's published detections look for the text "SSH access has been enabled" on ESXi hosts and for SSH sessions by `vpxuser`. We copy no rule text | https://github.com/elastic/detection-rules/tree/main/rules/integrations/vsphere |
| **Splunk Threat Research Team**, "ESXi SSH Enabled" detection (Splunk Security Content, Apache License 2.0) | Linked only. The fact that it looks for the same message text. We copy no search text | https://research.splunk.com/application/b8003567-c5b6-445b-8966-ecdacc81c24d |

## What we left out on purpose

- **Hashes, domains, URLs and key fingerprints.** None are in the report's text, so none are in the dataset. The account
  name the group created is published only up to its prefix (`sps-`); the rest is redacted in the report and we did not
  guess it.
- **Detection rules and the hardening script.** Linked, not copied (see above).
