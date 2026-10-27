# 06. Troubleshooting log

Real problems met while building the lab. Each entry: symptom, diagnosis, root cause, fix.

## 1. Overlapping subnets on LAN and MGMT

- **Symptom:** the laptop received an address from the wrong DHCP server and could not reach Proxmox.
- **Diagnosis:** `tcpdump -e -n -i nic0` and `bridge fdb show` on Proxmox showed untagged frames where tagged ones were expected.
- **Root cause:** the untagged LAN and tagged MGMT both used 10.10.10.1/24.
- **Fix:** moved the temporary LAN to a different subnet, enabled MGMT with its own gateway, and put the laptop port in access VLAN 10. The LAN address and DHCP were removed at the end.
- **Lesson:** never reuse a subnet on two interfaces, and check tags with a capture before blaming the config.

## 2. Gateway not answering after DHCP worked

- **Cause:** the MGMT interface had no pass rule. DHCP is auto-permitted, so a lease was granted, but ICMP was dropped.
- **Lesson:** a lease proves only that DHCP works.

## 3. "Bad archive mirror" in the Debian installer (about 90 minutes lost)

- **Symptom:** the installer got an address but could not reach the mirror. Packet capture showed DNS queries leaving but no replies.
- **Wrong path:** reconfiguring the DNS resolver (listen interfaces, localhost).
- **Root cause:** the INFRA block rule had destination `*` instead of the private-network alias, so it blocked everything, including the gateway.
- **Fix:** corrected the destination.
- **Lesson:** if an address is obtained but even the gateway does not answer, look at firewall rules first, not DNS. Diagnose in order: link, address, gateway ping, external IP ping, DNS name.

## 4. Wrong destination in zone exception rules

- **Cause:** *This Firewall* was used for DNS and ICMP. It matches every pfSense address, including other zones' gateways and the WAN.
- **Fix:** use the zone's own interface address.

## 5. Block rule did not block everything

- **Cause:** protocol was TCP instead of *any*, so ICMP and UDP passed.
- **Fix:** protocol *any*.

## 6. Minimal Debian quirks

- netinst left `sources.list` with only a CD-ROM entry. Fixed by adding the `deb.debian.org` repositories.
- `sudo` and `curl` are not installed by default. Used `su -` and `getent hosts`.
- Docker comes from Docker's own APT repository; `docker-compose-v2` is not in Debian.

## 7. SSH broken on a cloned VM

- **Cause:** host keys were deleted for the clone but not regenerated, and `/run/sshd` was missing.
- **Fix:** `ssh-keygen -A`, `mkdir -p /run/sshd`, correct ownership and mode.
- **Checklist for clones:** new hostname, `/etc/hosts`, `/etc/machine-id`, SSH host keys.

## 8. pfSense target UP in Prometheus but absent in Grafana

- **Cause:** the `uname` collector fails on pfSense, so `node_uname_info` is missing.
- **Fix:** textfile metric (see [05 Monitoring](05-monitoring.md)).

## 9. Unclean power-off of pfSense

- **Effect:** very long boot with filesystem checks.
- **Mitigation:** UPS recommended; AutoConfigBackup protects the configuration.

## General method

1. Is the link up and tagged correctly? (`ip link`, `bridge vlan show`)
2. Is there an address and a gateway?
3. Does the gateway answer ping?
4. Does an external IP answer ping?
5. Does a DNS name resolve?
6. Capture on both sides and compare with the firewall log.
