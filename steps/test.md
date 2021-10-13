# Testing

## Reboot

```bash
reboot
```

## Check ip assignments and routes

```bash
ip addr sh
ip link ls
ip route ls
```

## Check services

```bash
systemctl status replicateip
journalctl -u replicateip --no-pager

systemctl status udprelay
journalctl -u udprelay --no-pager
```

## Further testing

* test pinging an IP on the eth0 side (e.g. on the internet: `ping 8.8.8.8`)
* test pinging an IP on the wlan0 side
* test DNS lookup: `dig google.com`
* testing pinging an IP on wlan0 side from a machine on the eth0 side
* optionally run tcpdump to see packet flows for multicasts and broadcasts
