# Configuring the pi as a wifi to ethernet bridge

## Document version

1.5

## The legal blah-blah

I do take no responsibility for the below not working or doing damage, etc etc. You get the point.

## Synopsis

Since wlan0 cannot be simply bridged to eth0, we achieve this by reflecting the same wlan0 IP on eth0. We then forward the ARP requests between interfaces. The parprouted and replicateip services are the ones that make this happen. dhcp-helper is for dhcp routing. A custom forwarding binary and startup script allow for other udp broadcast and multicast forwarding.

## Project moved

[New Location](https://github.com/rglonek/raspberry-pi-wifi-ethernet-bridge)
