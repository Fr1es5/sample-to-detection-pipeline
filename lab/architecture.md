# Lab Architecture

## Overview
Isolated lab for analyzing a public malware sample and validating the
resulting detections in a Wazuh deployment. No analysis VM has internet
access once the build phase is complete. All malware handling happens
inside disposable VMs that can be reverted to a clean snapshot.

## Diagram

```
        ┌───────────────────── Host (Windows + VMware) ─────────────────────┐
        │                                                                   │
        │      Host-only network: 192.168.10.0/24  (no internet access)     │
        │                                                                   │
        │  ┌──────────────────┐  ┌──────────────────┐  ┌─────────────────┐  │
        │  │  Analysis VM     │  │  REMnux VM       │  │  Wazuh server   │  │
        │  │  Windows 10      │  │  INetSim         │  │  (planned,      │  │
        │  │  FLARE-VM        │  │  fake DNS/HTTP/  │  │   week 6)       │  │
        │  │  Sysmon          │  │  SMTP/FTP        │  │                 │  │
        │  │  192.168.10.10   │  │  192.168.10.20   │  │  192.168.10.30  │  │
        │  └────────┬─────────┘  └────────▲─────────┘  └────────▲────────┘  │
        │           │  DNS + gateway      │                     │           │
        │           └─────────────────────┘                     │           │
        │           └──────────── logs (Wazuh agent) ───────────┘           │
        └───────────────────────────────────────────────────────────────────┘
```

## Machines

| VM | Role | OS | IP | Key tools | Status |
|----|------|----|----|-----------|--------|
| analysis-win | Malware detonation and analysis | Windows 10 (TODO: edition/build) | 192.168.10.10 | FLARE-VM, Sysmon (TODO: confirm installed), Wazuh agent (week 6) | Built |
| remnux | Fake internet and traffic capture | REMnux | 192.168.10.20 | INetSim, Wireshark, tcpdump | Built |
| wazuh-server | Detection platform | Ubuntu Server | 192.168.10.30 | Wazuh manager, indexer, dashboard | Planned (week 6) |

Hypervisor: VMware Workstation (17.6.3 build-24583834).

### Analysis VM specs
- Firmware: BIOS (legacy)
- vCPUs: 4 (1 processor x 4 cores)
- RAM: 6 GB
- Disk: 80 GB virtual disk, dynamically allocated, split into multiple files
- Removed hardware: USB controller, sound card
- Windows Defender disabled and kept off across reboots (lab VM only)
- Local account used. No Microsoft account is linked to the VM.

### REMnux VM specs
- Imported from the official REMnux OVA
- vCPUs: 2
- RAM: 2 GB

## Network design

- Both VMs are attached **only** to a VMware host-only network
  (`192.168.10.0/24`). No NAT and no bridged adapters.
- The local DHCP service on the host-only network is **disabled**.
  All addresses are static.
- The analysis VM uses REMnux (`192.168.10.20`) as both **default gateway
  and DNS server**, so all of the malware's lookups and connections land
  on INetSim and not on the real internet.
- REMnux has a static IP configured through netplan and **no default
  gateway**, so it has no route out.
- The VMs were connected to a NAT adapter temporarily during the build
  phase (to install FLARE-VM, Sysmon and updates). The NAT adapter was
  removed before any sample was run.

### Fake internet (INetSim)
INetSim on REMnux is bound to `192.168.10.20` and emulates:
DNS (53), HTTP (80), HTTPS (443), SMTP (25), SMTPS (465), POP3 (110),
POP3S (995), FTP (21), FTPS (990).

Request logs are written to `/var/log/inetsim/service.log`. The terminal
only shows startup messages, so logs are the source of truth for what the
malware requested.

`systemd-resolved` is stopped on REMnux so INetSim can use port 53.

## Isolation controls

- [x] Host-only networking on every VM, with no NAT or bridged adapters
- [x] Local DHCP disabled, static IPs only
- [x] Gateway and DNS of the analysis VM point at REMnux
- [x] Shared folders disabled 
- [x] Clipboard sharing and drag-and-drop disabled 
- [x] USB controller removed
- [x] Sound card removed
- [x] Windows Defender disabled on the analysis VM only
- [x] Samples stored outside the repo and outside OneDrive
- [x] Clean baseline snapshot taken before any detonation (see `snapshot-policy.md`)

## Isolation tests

Run from the analysis VM after the NAT adapter was removed.

| Test | Expected | Result |
|------|----------|--------|
| `ping 192.168.10.20` | succeeds |
| `ping 8.8.8.8` | fails | 
| `nslookup google.com` | returns 192.168.10.20 |
| Browse to a real website | INetSim default page |

Evidence: TODO (screenshot of the tests, and the matching lines from
`/var/log/inetsim/service.log`, saved in `testing/screenshots/`)

## Snapshots
See [`snapshot-policy.md`](snapshot-policy.md).

## Known limitations
- To be updated