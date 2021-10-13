# Add a monitoring bash script systemd service unit file

```bash
cat <<'EOF' >/usr/lib/systemd/system/replicateip.service
[Unit]
Description=ip monitoring and replication service
Requires=sys-subsystem-net-devices-wlan0.device dhcpcd.service parprouted.service
After=sys-subsystem-net-devices-wlan0.device dhcpcd.service parprouted.service

[Service]
Restart=on-failure
RestartSec=5
TimeoutStartSec=30
ExecStart=/opt/replicate-ip.sh

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable replicateip
```
