# Snapshot Policy

## Purpose
Malware modifies the system it runs on. Snapshots let every analysis run
start from an identical, known-clean state, so results are comparable and
a compromised VM is never reused.

## Snapshot chain

### Analysis VM (`Analysis VM`)

| Snapshot | State | Taken |
|----------|-------|-------|
| `01-clean-windows` | Fresh Windows install, VMware Tools installed, no analysis tools | 11/09/2026 |
| `02-flarevm-installed` | FLARE-VM installed and tools verified, Defender off | 11/13/2026 |
| `03-baseline-isolated` | Sysmon installed, NAT removed, static IP set, isolation tests passed. **This is the state to revert to before every detonation.** | 16/11/2026 |

### REMnux VM (`REMnus VM`)

| Snapshot | State | Taken |
|----------|-------|-------|
| `03-baseline-isolated` | Static IP set, INetSim configured, isolation tests passed | 16/11/2026 |

### Wazuh server (added in week 6)
A baseline snapshot will be taken after the manager, dashboard and agent
connection are working, and before detection rules are loaded.

## Rules

1. **Revert before every run.** Before detonating the sample, always
   revert the analysis VM to `03-baseline-isolated`. Never analyze on a VM
   that has already run malware.
2. **Never run malware without a clean snapshot.** If no snapshot exists
   for the current state, create one first.
3. **Snapshot powered-off VMs when possible.** It gives the smallest and
   cleanest save point. Shut Windows down fully instead of suspending it.
4. **Take a snapshot after every milestone**, such as a finished install or
   a working config, so a failed step costs minutes and not hours.
5. **Never take a snapshot of an infected VM and treat it as a baseline.**
   Post-detonation snapshots, if kept for evidence, use a clearly
   different name (see below).
6. **Revert the network too.** If the network configuration is changed,
   re-run the isolation tests before creating a new baseline.

## Evidence snapshots (optional)

If a post-detonation state is worth keeping, for example to come back to a
memory capture later, name it with a prefix that cannot be confused with a
baseline:

```
EVIDENCE-<date>-<short-description>
```

Evidence snapshots are never used as a starting point for a new run.

## Workflow for each detonation

1. Revert `analysis-win` to `03-baseline-isolated`.
2. Make sure REMnux is running and INetSim is started.
3. Start the monitoring tools (Procmon, Wireshark, Regshot as needed).
4. Run the sample.
5. Record observations in `analysis/`.
6. Revert the VM to `03-baseline-isolated`.

## Housekeeping

- Snapshots grow as the VM changes. Delete intermediate snapshots that are
  no longer needed.
- Snapshots are not backups. If the virtual disk files are lost or
  corrupted, the snapshots are lost with them.
- VM files are stored on a local drive, never in OneDrive or any synced folder.