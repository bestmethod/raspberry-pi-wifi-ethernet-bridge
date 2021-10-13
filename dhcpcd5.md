# Configure dhcpcd5

```bash
grep '^option ip-forwarding 1$' /etc/dhcpcd.conf || printf "option ip-forwarding 1\n" >> /etc/dhcpcd.conf
grep '^denyinterfaces eth0$' /etc/dhcpcd.conf || printf "denyinterfaces eth0\n" >> /etc/dhcpcd.conf
egrep 'denyinterfaces|option ip-forward' /etc/dhcpcd.conf
```
