# Lab Router Appliance

This repository builds a small, focused router VM for appliance test labs. It
is intentionally not a full network appliance. It provides the basic pieces
needed to stand up repeatable labs quickly:

- WAN DHCP on one interface.
- One or more lab LANs on internal hypervisor networks.
- NAT from lab LANs to WAN.
- DHCP and DNS forwarding through dnsmasq.
- Optional DNS reservations and zone delegations for lab services.

The current implementation uses a Debian 13 generic-cloud image, cloud-init,
nftables, and dnsmasq. Hyper-V is the first supported hypervisor target.

## Repository Map

| Path | Purpose |
| --- | --- |
| `scripts/stage-router-artifacts.sh` | Mac-side artifact builder. Downloads/converts Debian cloud image and renders a NoCloud seed ISO. |
| `templates/cloud-init/` | cloud-init templates for the router VM. |
| `hypervisors/hyperv/New-LabRouter.ps1` | Creates a Hyper-V router VM from the staged VHDX and seed ISO. |
| `configs/` | Example lab network configs and dnsmasq snippets. |
| `docs/configuration.md` | Configuration model and planned runtime reconfigure path. |

## Quick Start

From macOS:

```bash
scripts/stage-router-artifacts.sh --extra-dnsmasq configs/samba-addc.dnsmasq.conf
```

This writes the reusable base VHDX and seed ISO to `/Volumes/ISO` by default:

- `/Volumes/ISO/debian-13-router-base.vhdx`
- `/Volumes/ISO/router1-seed.iso`

Copy the Hyper-V script to the host share if needed, then build the VM:

```bash
cp hypervisors/hyperv/New-LabRouter.ps1 /Volumes/ISO/lab-scripts/
ssh nmadmin@server 'pwsh -File D:\ISO\lab-scripts\New-LabRouter.ps1'
```

Verify:

```bash
ssh -J nmadmin@server hm@10.10.10.1 'cat /var/log/router-ready.marker; sudo nft list table ip nat'
```

## Current Scope

This project currently supports the single-LAN lab flow extracted from the
Samba AD DC appliance lab. The next step is to move from command flags plus
dnsmasq snippets to a small YAML config that can describe multiple LANs and
VLAN interfaces while keeping the appliance simple.

See `docs/configuration.md` for the target config shape.
