# Configure parprouted and setup a service

```bash
cat <<'EOF' >/usr/lib/systemd/system/parprouted.service
[Unit]
Description=proxy arp routing service
Documentation=https://raspberrypi.stackexchange.com/q/88954/79866
Requires=sys-subsystem-net-devices-wlan0.device dhcpcd.service
After=sys-subsystem-net-devices-wlan0.device dhcpcd.service

[Service]
Type=forking
# Restart until wlan0 gained carrier
Restart=on-failure
RestartSec=5
TimeoutStartSec=30
# clone the dhcp-allocated IP to eth0 so dhcp-helper will relay for the correct subnet
ExecStart=-/usr/sbin/parprouted eth0 wlan0

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable parprouted
```
