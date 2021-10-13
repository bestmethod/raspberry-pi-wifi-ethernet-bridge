# udprelay - install systemd unit

```bash
cat <<'EOF' > /usr/lib/systemd/system/udprelay.service
[Unit]
Description=UDP Broadcast and Multicast Relay
Requires=sys-subsystem-net-devices-wlan0.device dhcpcd.service parprouted.service replicateip.service
After=sys-subsystem-net-devices-wlan0.device dhcpcd.service parprouted.service replicateip.service

[Service]
Type=forking
Restart=on-failure
RestartSec=5
TimeoutStartSec=30
ExecStart=/usr/sbin/udp-relay-start.sh

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable udprelay
systemctl start udprelay
systemctl status udprelay
jouranctl -u udprelay --no-pager
```
