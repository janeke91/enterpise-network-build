# 05. Monitoring

## Architecture

Prometheus (pull model) runs in Docker on the `monitoring` VM (10.10.40.10, VLAN 40) together with Grafana. Each node runs `node_exporter` on port 9100. The firewall allows exactly one flow for this: 10.10.40.10 to `NODE_EXPORTERS` on TCP/9100.

| Job | Target | Label |
|---|---|---|
| proxmox | 10.10.10.3:9100 | |
| vms | 10.10.20.10:9100 | test-infra |
| vms | 10.10.30.10:9100 | dmz |
| vms | 10.10.40.10:9100 | monitoring |
| pfsense | 10.10.40.1:9100 | pfsense |

Configuration: [configs/monitoring/prometheus.yml](../configs/monitoring/prometheus.yml) and [compose.yaml](../configs/monitoring/compose.yaml).

The monitoring VM scrapes itself: `node_exporter` is installed there by `ansible/node_exporter.yml`. A security audit found this gap (the monitoring server was not monitored); see [07 Automation](07-automation.md).

## Grafana

- Data source: Prometheus at `http://prometheus:9090` (Compose service name).
- Dashboard: Node Exporter Full (ID 1860). Its Job and Host selectors are built from `node_uname_info`.

## pfSense exporter

The pfSense `node_exporter` package is bound to the MONITORING interface only. Extra flags: `--log.level=warn --no-collector.zfs`.

Problem: the `uname` collector fails with "cannot allocate memory" on pfSense, so `node_uname_info` is missing and the dashboard cannot list the host. Workaround: a textfile metric.

```sh
printf 'node_uname_info{machine="%s",nodename="%s",release="%s",sysname="%s"} 1\n' \
  "$(uname -m)" "$(uname -n)" "$(uname -r)" "$(uname -s)" \
  > /var/tmp/node_exporter/uname.prom
```

The textfile directory must match the exporter's `--collector.textfile.directory`. If `/var/tmp` is RAM-backed the file may vanish on reboot, so re-check after restarting pfSense.

## Retention and persistence

Prometheus keeps 15 days of data in the `prom-data` volume. Grafana state lives in `grafana-data`. Both containers use `restart: unless-stopped`.

## Backups

- pfSense: AutoConfigBackup enabled. The encryption password is stored outside this repo.
- Proxmox: scheduled VM backups plus a snapshot of each VM before changes.
