# Connect to WiFi - Raspberry Pi OS

```bash
cat <<'EOF' > /etc/wpa_supplicant/wpa_supplicant.conf
country=UK
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
 ssid="YOURSSID"
 scan_ssid=1
 psk="YOURPASSWORD"
 key_mgmt=WPA-PSK
}
EOF
rfkill unblock wifi
reboot
```
