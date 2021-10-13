# Add a monitoring bash script for replicating the IP if dhcp decides to change it

```bash
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
```
