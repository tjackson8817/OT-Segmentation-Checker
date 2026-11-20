# OT Segmentation Checker

**[Open the live tool →](https://tjackson8817.github.io/OT-Segmentation-Checker/)**

A browser tool that checks whether an OT network is segmented the way it should be. Give it three things:

- an Nmap scan
- a Wireshark conversations export (optional)
- a few lines describing your intended zones

It places every host on the Purdue model and builds the IEC 62443 zone-to-zone conduit matrix. It then flags traffic that crosses a boundary it shouldn't, such as the internet reaching control systems, IT talking straight to PLCs, remote access that skips the DMZ, or extra hosts reaching the safety system.

Everything runs in your browser. Scan and traffic data are never uploaded anywhere.

Part of the Sale Fish OT Tools suite, alongside [OT Risk Analyzer](https://github.com/tjackson8817/OT-Risk-Analyzer), [OT Asset Inventory](https://github.com/tjackson8817/OT-Asset-Inventory) and the OT CVE Live Tracker.

## What you need

| Input | Required? | How to get it |
|---|---|---|
| Nmap XML | Recommended | `nmap -sS -sV -O -oX scan.xml 10.30.0.0/16`. Scan OT networks gently and with approval: use `-T2` or `--scan-delay`, in a maintenance window. |
| Traffic conversations | Optional, strongly recommended | Wireshark → **Statistics → Conversations** → **TCP** tab → **Copy → As CSV**. Repeat for the **UDP** tab and paste below the TCP rows (or load both files together). |
| Zone rules | Yes, for conduit checks | One line per zone: `10.30.40.0/24 = L1 Line 2 Packaging`. Or click **Suggest zones from data** and edit the result. |

### Zone levels

| Level | Meaning |
|---|---|
| `L1` | Level 0–1: controllers, RTUs, I/O, drives, relays |
| `L2` | Level 2: HMIs, SCADA, engineering workstations |
| `L3` | Level 3: site operations, historians, OPC servers, MES |
| `DMZ` | Level 3.5: industrial DMZ |
| `ENT` | Level 4–5: enterprise / business IT |
| `SIS` | Safety instrumented systems |
| `PARTNER` | Vendors, utilities, remote sites reached over a WAN |

Public addresses are placed in **Internet** automatically. A single-IP rule such as `10.50.30.30 = SIS Triconex` overrides its subnet's rule, and you can move any host to a different zone from the Hosts table.

## What it checks

| Check | Severity |
|---|---|
| Internet ↔ Level 0–2 | Critical |
| Internet ↔ Level 3 without a DMZ | High |
| Enterprise IT ↔ Level 0–2 | High |
| DMZ ↔ Level 0–1 or SIS | High |
| More than one outside host reaching a safety system | High |
| Devices in the traffic that aren't in the Nmap scan | High (if they speak OT protocols) |
| Port-scan patterns | High |
| OT services running in an IT or DMZ zone | High |
| Enterprise ↔ Level 3 without a DMZ | Medium |
| Third-party connections ending inside a control zone | Medium |
| Cell-to-cell traffic between Level 0–1 zones | Medium |
| Remote access (RDP, VNC, SSH, Telnet, remote-support tools) into control zones from outside the DMZ | Medium |
| Cleartext management (Telnet, FTP, HTTP, SNMP) crossing into control zones | Medium |
| SMB / MS-RPC crossing into control zones | Medium |
| Windows / IT hosts inside a Level 0–1 zone | Medium |
| Level 3 polling Level 1 directly | Low |
| DMZ ↔ Level 2 | Low |
| Converged hosts exposing both OT and IT services | Low |
| Telnet / FTP open on control devices | Low |

Each finding cites the relevant IEC 62443-3-3 system requirement and MITRE ATT&CK for ICS techniques and mitigations (IDs checked against the October 2026 ATT&CK release).

## Outputs

- **Export CSV**: findings, conduit list and host table in one spreadsheet.
- **Save session (.json)**: the inputs, zone rules and host overrides, to reload later with **Import session**.
- **Print / PDF report**: a printable report from the browser's print dialog.
- The last session is also kept in the browser and offered back on your next visit.

## Try it

Click **Load sample plant** in the tool. To try other networks, the `samples/` folder has three plant networks (small water plant, medium food and beverage plant, large power and chemical complex). Each comes with an Nmap scan, a conversations export and a zone file. `sample_segmentation_report.*` shows the output for the medium plant.

## Hosting

The tool is a single file, `index.html`, with no build step and no dependencies. It is served by GitHub Pages (Settings → Pages → Deploy from branch `main`, folder `/ (root)`). To run it offline, download `index.html` and open it in any modern browser.

## Limitations

- It checks the traffic you give it. A path that didn't carry traffic during the capture won't appear in the matrix, so review firewall rules as well.
- Service direction is inferred from well-known ports. Uncommon proprietary ports are reported as "port N".
- Findings are a starting point for review, not a certified IEC 62443 assessment.

---
Sale Fish Marketing & Consulting · [salefishmarketing.com](https://www.salefishmarketing.com/contact.html)
