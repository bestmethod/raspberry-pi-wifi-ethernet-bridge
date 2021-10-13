# udprelay - install binary

```bash
apt -y install gcc make git
git clone https://github.com/bestmethod/udp-broadcast-relay-redux.git
cd udp-broadcast-relay-redux/
make
cp udp-broadcast-relay-redux /usr/sbin/udp-relay
chmod 755 /usr/sbin/udp-relay
```
