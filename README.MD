# NTPsec

[NTPsec](https://ntpsec.org/) is a NTPd Network Time Server with TLS 1.3 security added to it. This allows for secure syncing/peering between 2 time servers. 

## Quick Reference
* __Main git repo__: https://gitlab.com/rouing/docker-ntpsec/
* __Mirrored git repo__: https://github.com/rouing/docker-ntpsec/
* __NTPsec issues__: https://gitlab.com/NTPsec/ntpsec/-/issues
* __NTPsec Docker issues__: https://gitlab.com/rouing/docker-ntpsec/-/issues
* __NTPsec Documentation__: https://docs.ntpsec.org/latest/NTS-QuickStart.html
* __IETF NTS Draft__: https://tools.ietf.org/html/draft-ietf-ntp-using-nts-for-ntp-28

## Public NTS capable time servers

* [time.cloudflare.com:1234](https://www.cloudflare.com/time/)
* [ntpsec.anastrophe.com](https://ntpsec.anastrophe.com/) [ntp.org listing](http://support.ntp.org/bin/view/Servers/PublicTimeServer001384)
* [ntp.keff.org](http://ntp.keff.org/)
* [pi4.rellim.com](https://pi4.rellim.com/)

## Supported tags and respective `Dockerfile` links

* [`latest`, `alpine`, `alpine-latest`, `alpine3`, `1.1.9-alpine`, `1.1.9-alpine3`](https://gitlab.com/rouing/docker-ntpsec/-/blob/master/ntpsec_alpine) 
* [`alpine-edge`, `1.1.9-alpine-edge`](https://gitlab.com/rouing/docker-ntpsec/-/blob/master/ntpsec_alpine-edge)
* [`centos`, `centos-latest`, `centos8`, `centos8.2`, `1.1.9-centos8`, `1.1.9-centos8.2`](https://gitlab.com/rouing/docker-ntpsec/-/blob/master/ntpsec_centos)
* [`debian`, `debian-latest`, `debian10`, `debian10.4`, `1.1.9-debian10`, `1.1.9-debian10.4`](https://gitlab.com/rouing/docker-ntpsec/-/blob/master/ntpsec_debian)

## CLI start examples 
```bash
# Run
docker run -it --rm --name ntpsec -p123:123/udp -p123:123/tcp -v /var/lib/ntp:/var/lib/ntp  --cap-add SYS_TIME --cap-add SYS_NICE --cap-add NET_RAW --cap-add NET_BIND_SERVICE ardoin/ntpsec

# Run detached
docker run -d --rm --name ntpsec -p123:123/udp -p123:123/tcp -v /var/lib/ntp:/var/lib/ntp --cap-add SYS_TIME --cap-add SYS_NICE --cap-add NET_RAW --cap-add NET_BIND_SERVICE ardoin/ntpsec

# Run detached, use host network and limit logfile size
docker run -d --rm --name ntpsec --net=host -v /var/lib/ntp:/var/lib/ntp  --cap-add SYS_TIME --cap-add SYS_NICE --log-opt max-size=1m --log-opt max-file=3 ardoin/ntpsec

# BYO (bring your own) config file
docker run -it --rm --name ntpsec -p123:123/udp -p123:123/tcp -v /etc/ntp.conf:/etc/ntp.conf -v /var/lib/ntp:/var/lib/ntp --cap-add SYS_TIME --cap-add SYS_NICE --cap-add NET_RAW --cap-add NET_BIND_SERVICE ardoin/ntpsec

# Or your own arguments
docker run -it --rm --name ntpsec -p123:123/udp -p123:123/tcp -v /var/lib/ntp:/var/lib/ntp  --cap-add SYS_TIME --cap-add SYS_NICE --cap-add NET_RAW --cap-add NET_BIND_SERVICE ardoin/ntpsec --help
```
`--cap-add NET_RAW --cap-add NET_BIND_SERVICE` are needed due to a bug with user namespaces, docker, and the linux kernel for now. Fix is on the way!
`-v /var/lib/ntp:/var/lib/ntp` mounts the folder that holds the drift file specific to that machine. Highly recommend you dont touch this.

## Sample ntp.conf with NTS Examples

```bash
driftfile /var/lib/ntp/ntp.drift # This should be bind mounted to your host as it is specific to the host its running on.

# Analyitcs
# statsdir /var/log/ntpstats/

# Pure Logs 
#statistics loopstats peerstats clockstats 
#filegen loopstats file loopstats type day enable
#filegen peerstats file peerstats type day enable
#filegen clockstats file clockstats type day enable

#logfile /var/log/ntpd.log 
#logconfig =syncall +clockall +peerall +sysall

# The interface directive is great when your docker is in host net mode. (It works)
#interface ignore all # Do no engage(ignore) all interfaces
#interface listen eth1 # Listen to that one interface with the public IP. 
# The last one overwrites the first one

#server clock.nyc.he.net iburst #CDMA, Stratum 1, iburst is a better disconnect netflow. 
#server time.cloudflare.com:1234 iburst nts #NTS Protocol Enabled, TLS1.3 Only by Default, Port 123 over TCP(!)
pool 0.pool.ntp.org iburst prefer # Prefer
pool 1.pool.ntp.org iburst
pool 2.pool.ntp.org iburst

# Enable NTS
nts enable (Mandatory for NTS)
# SSL Cert that is your Key (Mandatory for NTS Server(?))
nts key /etc/letsencrypt/live/can.be.yours/privkey.pem
# Full Chain Cert (Mandatory for NTS Server(?))
nts cert /etc/letsencrypt/live/can.be.yours/fullchain.pem
# Where auth cookies are stored (Mandatory for NTS)
nts cookie /var/lib/ntp/nts-keys

# Default safe security settings
restrict default kod limited nomodify nopeer noquery
restrict 127.0.0.1

# Remove all restrictions for things like some LAN machines. 
# Remote ntpq control
#unrestrict another.host.local
```

Reduce Conntrack Flooding
```sh
# disable conntrack on NTP port 123 (UDP for NTP, TCP for NTS)
$ iptables -t raw -A PREROUTING -p udp -m udp --dport 123 -j NOTRACK
$ iptables -t raw -A OUTPUT -p udp -m udp --sport 123 -j NOTRACK
$ iptables -t raw -A PREROUTING -p tcp -m tcp --dport 123 -j NOTRACK
$ iptables -t raw -A OUTPUT -p tcp -m tcp --sport 123 -j NOTRACK
```
