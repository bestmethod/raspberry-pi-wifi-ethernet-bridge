# Enable services

```bash
systemctl enable dhcp-helper
systemctl enable dhcpcd
systemctl disable avahi-daemon # in case it's installed, this may fail if avahi daemon is not installed
```
