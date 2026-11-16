# Sample-to-Detection Pipeline

End-to-end malware analysis and detection engineering project: analyze a
public malware sample, turn the findings into YARA and Sigma detections,
and validate them in a Wazuh lab.

> ⚠️ This repo contains **no malware samples**. Only hashes, analysis notes,
> and detection rules. All analysis was done in an isolated lab.

## Status
🚧 In progress: Week 1 of 8 complete (lab built)

## Goal
Show the full workflow of a malware analyst and detection engineer:
analysis → IOCs and ATT&CK mapping → detections → testing → report.

## Sample
- Family: _TBD (week 2)_
- SHA256: _TBD_
- Source: _MalwareBazaar link, TBD_

## Lab
Three VMs on an isolated host-only network (`192.168.10.0/24`):

| VM | Role |
|----|------|
| Windows 10 + FLARE-VM + Sysmon | Malware analysis and detonation |
| REMnux (INetSim) | Fake internet and traffic capture |
| Wazuh server (week 6) | Detection platform and alerting |

Details, diagram and isolation tests: [`lab/architecture.md`](lab/architecture.md).
Snapshot rules: [`lab/snapshot-policy.md`](lab/snapshot-policy.md).

## Roadmap
- [x] Week 1: Lab setup and isolation tests
- [ ]