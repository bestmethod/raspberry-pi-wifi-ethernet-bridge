# Ubuntu 20.04 only: Reconfigure netplan to not acquire an IP - dhcpcd5 will handle this from now on

```bash
cat <<EOF > /etc/netplan/50-cloud-init.yaml
network:
    wifis:
        wlan0:
            access-points:
                "WIFINAME":
                    password: "WIFIPASSWORD"
            dhcp4: false
            optional: true
    ethernets:
        eth0:
            dhcp4: false
            optional: true
    version: 2
EOF

netplan generate
```
