This image is designed to be used as part of the [www.pool.ntp.org](https://ntppool.org) project.
Default is for the Australian zone

```bash

# Build
git pull https://github.com/rouing/docker-ntpsec/
cd docker-ntpsec
docker build t ardoin/ntpsec .

# Run
docker run -it --rm --name ntpsec -p123:123/udp -p123:123/tcp --cap-add SYS_TIME --cap-add SYS_NICE --cap-add NET_RAW --cap-add NET_BIND_SERVICE ardoin/ntpsec

# Run detached
docker run -d --rm --name ntpsec -p123:123/udp -p123:123/tcp --cap-add SYS_TIME --cap-add SYS_NICE --cap-add NET_RAW --cap-add NET_BIND_SERVICE ardoin/ntpsec

# Run detached, use host network and limit logfile size
docker run -d --rm --name ntpsec --net=host --cap-add SYS_TIME --cap-add SYS_NICE --log-opt max-size=1m --log-opt max-file=3 ardoin/ntpsec

# BYO (bring your own) config file
docker run -it --rm --name ntpsec -p123:123/udp -p123:123/tcp -v /etc/ntp.conf:/etc/ntp.conf --cap-add SYS_TIME --cap-add SYS_NICE --cap-add NET_RAW --cap-add NET_BIND_SERVICE ardoin/ntpsec

# Or your own arguments
docker run -it --rm --name ntpsec -p123:123/udp -p123:123/tcp --cap-add SYS_TIME --cap-add SYS_NICE --cap-add NET_RAW --cap-add NET_BIND_SERVICE ardoin/ntpsec --help
```
`--cap-add NET_RAW --cap-add NET_BIND_SERVICE` are needed due to a bug with user namespaces, docker, and the linux kernel for now. Fix is on the way!

## Prevent conntrack from filling up

```sh
# disable conntrack on NTP port 123 (UDP for NTP, TCP for NTS)
$ iptables -t raw -A PREROUTING -p udp -m udp --dport 123 -j NOTRACK
$ iptables -t raw -A OUTPUT -p udp -m udp --sport 123 -j NOTRACK
$ iptables -t raw -A PREROUTING -p tcp -m tcp --dport 123 -j NOTRACK
$ iptables -t raw -A OUTPUT -p tcp -m tcp --sport 123 -j NOTRACK
```
