# Router Configuration Model

The router should stay focused: route lab networks, provide DHCP/DNS, NAT to a
WAN, and make common lab reconfiguration fast. It should not grow into a full
general-purpose network appliance.

## Current Model

Today `scripts/stage-router-artifacts.sh` accepts command-line options:

- router hostname and domain
- LAN IP and prefix
- DHCP pool start/end
- SSH public key
- output staging directory
- optional dnsmasq snippet

The optional dnsmasq snippet is how callers add reservations and DNS
delegations for a specific lab.

## Target Model

The next iteration should render from a small YAML file:

```yaml
router:
  hostname: router1
  domain: lab.test
  wan:
    mode: dhcp
    switch: "PCI 1G Port 1"
  lans:
    - name: lab
      switch: Lab-NAT
      interface: eth1
      address: 10.10.10.1/24
      dhcp:
        range: 10.10.10.100-10.10.10.200
        reservations:
          - name: WS2025-DC1
            mac: 00:15:5D:0A:0A:0A
            ip: 10.10.10.10
          - name: samba-dc1
            mac: 00:15:5D:0A:0A:14
            ip: 10.10.10.20
      dns:
        delegations:
          - zone: lab.test
            servers: [10.10.10.10, 10.10.10.20]
```

VLANs should be explicit and boring:

```yaml
vlans:
  - parent: eth1
    id: 20
    name: clients
    address: 10.10.20.1/24
    dhcp:
      range: 10.10.20.100-10.10.20.200
```

Hypervisor-specific VLAN trunk/access settings belong in the lab orchestration
tool. Inside the router, a VLAN is just a Linux interface plus dnsmasq and
nftables rules.

## Reconfiguration

Two workflows are useful:

1. Build-time rendering: create a new seed ISO and build/rebuild the router VM.
2. Runtime apply: render config locally, copy it to an existing router, validate
   dnsmasq/nftables, restart services, and print effective state.

Planned command shape:

```bash
scripts/apply-config.sh configs/samba-addc.yaml router1
```

The runtime path should be safe and obvious. It should validate before
restarting services and leave the previous known-good config available for
manual rollback.
