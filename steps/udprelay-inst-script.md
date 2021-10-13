# udprelay - install script

```bash
cat <<'EOF' > /usr/sbin/udp-relay-start.sh
#!/bin/bash

# fail if IP is not provided
if [ "$(/sbin/ip -4 -br addr show wlan0 | /bin/grep -Po "\\d+\\.\\d+\\.\\d+\\.\\d+/\\d+")" == "" ]
then
        echo "ERROR: wlan0 does not have an IP"
        exit 1
fi
if [ "$(/sbin/ip -4 -br addr show eth0 | /bin/grep -Po "\\d+\\.\\d+\\.\\d+\\.\\d+/\\d+")" == "" ]
then
        echo "ERROR: eth0 does not have an IP"
        exit 1
fi

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
