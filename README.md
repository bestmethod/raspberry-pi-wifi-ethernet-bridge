# Configuring the pi as a wifi to ethernet bridge

## Version

1.3

## The legal blah-blah

I do take no responsibility for the below not working or doing damage, etc etc. You get the point.

## Synopsis

Since wlan0 cannot be simply bridged to eth0, we achieve this by reflecting the same wlan0 IP on eth0. We then forward the ARP requests between interfaces. The parprouted and replicateip services are the ones that make this happen. dhcp-helper is for dhcp routing and avahi is for mDNS forwarding. A custom forwarding binary and startup script allow for other udp broadcast and multicast forwarding.

The following components are covered:

* netplan (ubuntu) OR wpa_supplicant (raspberry pi OS) - to connect to wifi itself
* dhcpcd5 - dhcp client for wlan0
* dhcp-helper - dhcp forwarding so dhcp clients attached to eth0 also work
* parprouted - routing of ARP requests, to allow for the "bridge" to work
* custom service script for replicating IP across 2 interfaces
* openssh-server - cause it's easier than console
* disabling the pesky unattended-upgrades (optional)
* udp-relay for broadcast/multicast udp relay (mdns, dlna/ssdp, netbios, etc)

## Steps

### Connect to wifi

#### Ubuntu 20.04

```
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
netplan generate
netplan apply
```

#### Raspberry Pi OS

```
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

### Check wifi connection status, fix above if required

```
iw wlan0 info; ip addr show wlan0; ip route; echo; journalctl -b --no-pager -q | grep -i wlan0 | tail -n 10 | sort -r
```

### Install dependencies and openssh

```
apt update && apt -y upgrade && apt -y install parprouted dhcp-helper dhcpcd5 openssh-server
systemctl enable ssh
systemctl start ssh
```

Note: if you see `Could not get lock /var/lib/dpkg/lock-frontend. It is held by process ... (unattended-upgr)`, you need to wait for first run of unattended upgrades to finish before repeating the above step.

### Get current ip, routes and link status

```
ip addr sh
ip route ls
ip link ls
```

### You can now ssh in to the pi to continue the installation, easier that way :)

### Disable unattented upgrades (if installed)

```
dpkg-reconfigure unattended-upgrades
# select "No" in the question screen
```

### Enable services

```
systemctl enable dhcp-helper
systemctl enable dhcpcd
systemctl disable avahi-daemon # in case it's installed, this may fail if avahi daemon is not installed
```

### Configure ip forwarding

```
sed -i'' s/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/ /etc/sysctl.conf
grep ip_forward /etc/sysctl.conf
```

### Configure dhcpcd5

```
grep '^option ip-forwarding 1$' /etc/dhcpcd.conf || printf "option ip-forwarding 1\n" >> /etc/dhcpcd.conf
grep '^denyinterfaces eth0$' /etc/dhcpcd.conf || printf "denyinterfaces eth0\n" >> /etc/dhcpcd.conf
egrep 'denyinterfaces|option ip-forward' /etc/dhcpcd.conf
```

### Configure dhcp-helper

```
cat > /etc/default/dhcp-helper <<EOF
DHCPHELPER_OPTS="-b wlan0"
EOF
```

### Configure parprouted and setup a service

```
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

### Add a monitoring bash script for replicating the IP if dhcp decides to change it

```
cat <<'EOF' > /opt/replicate-ip.sh
#!/bin/bash

DEBUG=0

function bootstrap {
    /sbin/ip link set dev eth0 up
    /sbin/ip link set wlan0 promisc on
    /sbin/ip link set eth0 promisc on
    /usr/sbin/iw wlan0 set power_save off
}

function replicate {
    WLANIP=$(/sbin/ip -4 -br addr show wlan0 | /bin/grep -Po "\\d+\\.\\d+\\.\\d+\\.\\d+/\\d+")
    ETHIP=$(/sbin/ip -4 -br addr show eth0 | /bin/grep -Po "\\d+\\.\\d+\\.\\d+\\.\\d+/\\d+")
    [ ${DEBUG} -eq 1 ] && echo "$(date) eth0=${ETHIP} wlan0=${WLANIP}"
    if [ "${WLANIP}" == "" ]
    then
        return
    fi
    if [ "${WLANIP}" == "${ETHIP}" ]
    then
        return
    fi
    echo "$(date) fixing_eth0_ip, found: eth0=${ETHIP} wlan0=${WLANIP}"
    if [ "${ETHIP}" != "" ]
    then
        /sbin/ip addr del ${ETHIP} dev eth0
    fi
    /sbin/ip addr add ${WLANIP} brd + dev eth0
}

if [ "$1" == "--debug" ]
then
    DEBUG=1
fi

[ ${DEBUG} -eq 1 ] && echo "$(date) bootstrapping"
bootstrap
[ ${DEBUG} -eq 1 ] && echo "$(date) bootstrap finished"
while true
do
    [ ${DEBUG} -eq 1 ] && echo "$(date) checking interface IPs"
    replicate
    [ ${DEBUG} -eq 1 ] && echo "$(date) sleep 5 seconds"
    sleep 5
done
EOF

chmod +x /opt/replicate-ip.sh

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

### Ubuntu 20.04 only: Reconfigure netplan to not acquire an IP - dhcpcd5 will handle this from now on

```
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

### Restart your pi

```
reboot
```

### resolv.conf - DNS

```
cat <<EOF > /etc/systemd/resolved.conf
[Resolve]
DNS=8.8.8.8
FallbackDNS=8.8.4.4
#Domains=
#LLMNR=no
#MulticastDNS=no
#DNSSEC=no
#DNSOverTLS=no
#Cache=no-negative
#DNSStubListener=yes
#ReadEtcHosts=yes
EOF
systemctl restart systemd-resolved
```

### Optional: install wavemon to watch and monitor wifi

```
apt install wavemon
wavemon
```

### Add support for boradcast/multicast (adjust the start script to your needs)

#### install binary

```
apt -y install gcc make git
git clone https://github.com/bestmethod/udp-broadcast-relay-redux.git
cd udp-broadcast-relay-redux/
make
cp udp-broadcast-relay-redux /usr/sbin/udp-relay
chmod 755 /usr/sbin/udp-relay
```

#### install script

```
cat <<'EOF' > /usr/sbin/udp-relay-start.sh
#!/bin/bash

# mDNS
/usr/sbin/udp-relay -f --id 1 --port 5353 --dev eth0 --dev wlan0 --multicast 224.0.0.251 -s 1.1.1.1
if [ $? -ne 0 ]
then
    /usr/sbin/udp-relay -f --id 1 --port 5353 --dev eth0 --dev wlan0 -s 1.1.1.1
    [ $? -eq 0 ] && echo "started mdns without adding multicast group" || echo "failed to start mdns"
else
    echo "started mdns"
fi

# ssdp / dlna
/usr/sbin/udp-relay -f --id 1 --dev eth0 --dev wlan0 --port 1900 --multicast 239.255.255.250
if [ $? -ne 0 ]
then
    /usr/sbin/udp-relay -f --id 1 --dev eth0 --dev wlan0 --port 1900
    [ $? -eq 0 ] && echo "started ssdp without adding multicast group" || echo "failed to start ssdp"
else
    echo "started ssdp"
fi

# lifx bulb discovery
/usr/sbin/udp-relay -f --id 1 --dev eth0 --dev wlan0 --port 56700
[ $? -eq 0 ] && echo "started lifx" || echo "failed to start lifx"

# broadlink IR emmiter
/usr/sbin/udp-relay -f --id 1 --dev eth0 --dev wlan0 --port 80
[ $? -eq 0 ] && echo "started broadlink IR" || echo "failed to start broadlink IR"

# unifi discovery
/usr/sbin/udp-relay -f --id 1 --dev eth0 --dev wlan0 --port 10001
[ $? -eq 0 ] && echo "started unifi" || echo "failed to start unifi"

# sonos
/usr/sbin/udp-relay -f --id 1 --dev eth0 --dev wlan0 --port 6969
[ $? -eq 0 ] && echo "started sonos 1" || echo "failed to start sonos 1"
/usr/sbin/udp-relay -f --id 1 --dev eth0 --dev wlan0 --port 1901
[ $? -eq 0 ] && echo "started sonos 2" || echo "failed to start sonos 2"

# netbios
/usr/sbin/udp-relay -f --id 1 --dev eth0 --dev wlan0 --port 136
[ $? -eq 0 ] && echo "started netbios 1" || echo "failed to start netbios 1"
/usr/sbin/udp-relay -f --id 1 --dev eth0 --dev wlan0 --port 137
[ $? -eq 0 ] && echo "started netbios 2" || echo "failed to start netbios 2"
/usr/sbin/udp-relay -f --id 1 --dev eth0 --dev wlan0 --port 138
[ $? -eq 0 ] && echo "started netbios 3" || echo "failed to start netbios 3"
/usr/sbin/udp-relay -f --id 1 --dev eth0 --dev wlan0 --port 139
[ $? -eq 0 ] && echo "started netbios 4" || echo "failed to start netbios 4"

# done
echo "init complete"
EOF
chmod 755 /usr/sbin/udp-relay-start.sh
```

#### install systemd unit

```
at <<'EOF' > /usr/lib/systemd/system/udprelay.service
[Unit]
Description=UDP Broadcast and Multicast Relay

[Service]
Type=forking
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
