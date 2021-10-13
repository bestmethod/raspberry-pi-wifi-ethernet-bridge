# resolv.conf - DNS

```bash
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
