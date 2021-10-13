# Install dependencies and openssh

## install

```bash
apt update && apt -y upgrade && apt -y install parprouted dhcp-helper dhcpcd5 openssh-server
```

Note: if you see `Could not get lock /var/lib/dpkg/lock-frontend. It is held by process ... (unattended-upgr)`, you need to wait for first run of unattended upgrades to finish before repeating the above step.

## enable ssh

```bash
systemctl enable ssh
systemctl start ssh
```
