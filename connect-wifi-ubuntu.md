# Connect to WiFi - Ubuntu 20.04

```bash
cat <<EOF > /etc/netplan/50-cloud-init.yaml
network:
    wifis:
        wlan0:
            access-points:
                "WIFINAME":
                    password: "WIFIPASSWORD"
            dhcp4: true
            optional: true
    ethernets:
        eth0:
            dhcp4: false
            optional: true
    version: 2
EOF

rfkill unblock wifi
netplan generate
netplan apply
```
