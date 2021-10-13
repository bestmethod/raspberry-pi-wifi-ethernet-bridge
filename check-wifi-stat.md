# Check WiFi connection status

## One-liner

```bash
iw wlan0 info; ip addr show wlan0; ip route; echo; journalctl -b --no-pager -q | grep -i wlan0 | tail -n 10 | sort -r
```

## Or split into separate commands

```bash
iw wlan0 info
ip addr show wlan0
ip route
journalctl -b --no-pager -q | grep -i wlan0 | tail -n 10 | sort -r
```
