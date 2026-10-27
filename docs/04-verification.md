# 04. Verification

Every rule is tested from the zone it applies to. Results are from the built lab. Add screenshots to `../screenshots/` and link them in the Evidence column.

| # | From | Test | Expected | Result | Evidence |
|---|---|---|---|---|---|
| 1 | INFRA VM | `ping 10.10.20.1` | Reply | Pass | |
| 2 | INFRA VM | `getent hosts debian.org` | Resolves via pfSense | Pass | |
| 3 | INFRA VM | `ping 1.1.1.1` | Reply | Pass | |
| 4 | INFRA VM | `ping 10.10.30.10` (DMZ) | No reply, firewall log "Block internal networks" | Pass | |
| 5 | INFRA VM | `ping <ISP router address>` | No reply | Pass | |
| 6 | INFRA VM | `ping 10.10.10.3` (Proxmox) | No reply | Pass | |
| 7 | DMZ VM | gateway, DNS, internet | Work | Pass | |
| 8 | DMZ VM | ping INFRA, ISP router, Proxmox | No reply, logged | Pass | |
| 9 | MONITORING VM | `nc -zv 10.10.20.10 9100` | Open | Pass | |
| 10 | MONITORING VM | `nc -zv 10.10.20.10 22` | Blocked | Pass | |
| 11 | Laptop (MGMT) | SSH to each VM | Works | Pass | |
| 12 | Laptop (MGMT) | pfSense and Proxmox web UI | Open | Pass | |
| 13 | Laptop (MGMT) | `ping 10.10.10.1` | Reply | Pass | |
| 14 | Laptop (MGMT) | `ping <ISP router address>` | No reply, logged | Pass | |
| 15 | Laptop (MGMT) | `ping 10.10.20.10` while SSH works | No reply, SSH ok | Pass | |
| 16 | Laptop (MGMT) | `ping 1.1.1.1` | No reply (no ICMP rule to internet) | Pass | |
| 17 | All | Reboot pfSense, then Proxmox | MGMT, VMs and Grafana return without manual action | Pass | |


On pfSense: Status > System Logs > Firewall shows blocked traffic with the rule description. Diagnostics > Packet Capture sees packets before the firewall decides, so a packet visible in capture and absent in logs points to a rule problem, not a link problem.
