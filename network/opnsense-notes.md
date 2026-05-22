# OPNsense network notes

Reference doc for the OPNsense VM configuration.
Not a full config export — key structural decisions and recovery procedures.

---

## Interface assignments

| Interface | VMware NIC | Role | Address |
|-----------|-----------|------|---------|
| WAN | vmx0 (VMXNET3) | Upstream to AT&T BGW320-505 | DHCP |
| LAN | vmx1 (VMXNET3) | Internal network | 10.0.0.1/24 |
| lo0 | — | Loopback | 127.0.0.1 / ::1 |

> Both interfaces use VMware VMXNET3 paravirtual adapters for optimal throughput inside ESXi.

### WAN settings

| Setting | Value | Note |
|---------|-------|------|
| IP | DHCP (from BGW320-505) | |
| MTU | 1492 | AT&T fiber requires PPPoE-style MTU even over DHCP |
| MSS | 1452 | Clamped to match MTU |
| Block private networks | Yes | Drops RFC1918 on WAN ingress |
| Block bogon networks | Yes | Drops unallocated/reserved IP ranges |
| Honour DHCP MTU | Yes | Accepts MTU from upstream DHCP server |
| IPv6 | DHCPv6 | DNS requests suppressed |

---

## vSwitch layout (ESXi)

| vSwitch | Uplink | Connected VMs |
|---------|--------|--------------|
| vSwitch-WAN | Physical NIC to BGW320-505 | OPNsense WAN (vmx0) |
| vSwitch-LAN | Internal only | OPNsense LAN (vmx1), TrueNAS, Docker VM |

---

## DHCP

- Plugin: ISC DHCPv4 (Dnsmasq alone insufficient for reliable static lease assignment)
- Static leases assigned to all hosts — see README for IP table

---

## DNS

- Resolver: Dnsmasq

---

## SSL / remote access

- Cert provider: Let's Encrypt via `os-acme-client` plugin
- DNS challenge provider: DuckDNS (`maximo-opnsense.duckdns.org`)

---

## Security features active

- WAN private network blocking (RFC1918)
- WAN bogon blocking
- IDS/IPS enabled
- QAT hardware crypto offload (Intel C3758R QAT engine)

---

## VLANs

Not yet configured — planned for future network segmentation work.

---

## Firewall rules

Not yet documented — ruleset to be exported and sanitized after VLAN configuration is complete.

---

## AT&T BGW320-505 — IP Passthrough

**Settings:** DHCPS-fixed mode using OPNsense WAN MAC address.

**Known issue:** BGW320-505 resets IP Passthrough after reboots (indicated by solid red light on the gateway).

**Recovery procedure:**
1. Set a static IP on the connected client in the `192.168.1.x` range
2. Access BGW320-505 admin UI at `192.168.1.254`
3. Navigate to Firewall → IP Passthrough
4. Re-enter DHCPS-fixed with the OPNsense WAN MAC address
5. Save and apply
6. In OPNsense: Interfaces → WAN → release and renew DHCP lease

---

## WAS-110 ONU stick (planned)

Goal: replace the BGW320-505 entirely with a WAS-110 ONU stick for direct fiber
termination, eliminating the IP Passthrough dependency and the associated reset issue.
