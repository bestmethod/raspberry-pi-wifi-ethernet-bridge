# Configuring the pi as a wifi to ethernet bridge

## Document version

1.5

## The legal blah-blah

I do take no responsibility for the below not working or doing damage, etc etc. You get the point.

## Synopsis

Since wlan0 cannot be simply bridged to eth0, we achieve this by reflecting the same wlan0 IP on eth0. We then forward the ARP requests between interfaces. The parprouted and replicateip services are the ones that make this happen. dhcp-helper is for dhcp routing. A custom forwarding binary and startup script allow for other udp broadcast and multicast forwarding.

## Default credentials

System | Username | Password
--- | --- | ---
Raspberry Pi OS | pi | raspberry
Ubuntu 20.04 | ubuntu | ubuntu

Once logged in, when following the below manual, use `sudo -i` to switch to root first.

### Safety note: do not forget to change the password for the default user to something secure!

## Steps

- Connect to WiFi
  - [Ubuntu 20.04](steps/connect-wifi-ubuntu.md)
  - Raspberry Pi OS - pick one from below:
    - [From Pi OS](steps/connect-wifi-rpi.md)
    - [Headless Setup](steps/headless.md)
- [Check WiFi connection status](steps/check-wifi-stat.md)
- [Install dependencies and openssh](steps/install-deps.md)
- [Get current IP, routes and link status](steps/get-ip.md)
- Connect to the Pi via `ssh`
- [Disable unattended-updates](steps/disable-unattended.md)
- [Enable services](steps/enable-srv.md)
- [Configure IP forwarding](steps/conf-ip-fwd.md)
- [Configure dhcpcd5](steps/dhcpcd5.md)
- [Configure dhcp-helper](steps/dhcp-helper.md)
- [Configure parprouted service](steps/conf-parprouted.md)
- Configure IP replicator
  - [Create bash script](steps/ip-repl-bash.md)
  - [Add systemd service](steps/ip-repl-systemd.md)
- [Ubuntu 20.04 only: Reconfigure netplan to not acquire an IP - dhcpcd5 will handle this from now on](steps/ubuntu-reconf-netplan.md)
- Restart your pi: `$ reboot`
- [resolv.conf - DNS](steps/resolvconf.md)
- Add support for broadcast/multicast
  - [Install binary](steps/udprelay-inst-binary.md)
  - [Install script (optinonally adjust to your needs)](steps/udprelay-inst-script.md)
  - [Install systemd unit](steps/udprelay-inst-systemd.md)
- [Optional: install wavemon to watch and monitor wifi](steps/wavemon.md)
- [Test: final reboot and check services](steps/test.md)
- [Optional: overlay root filesystem to set SD card as read-only](steps/overlayroot.md)
