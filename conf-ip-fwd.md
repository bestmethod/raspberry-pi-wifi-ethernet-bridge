# Configure ip forwarding

```bash
sed -i'' s/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/ /etc/sysctl.conf
grep ip_forward /etc/sysctl.conf
```
