# DNS Host Override (fileserver.lab)

## Goal
Create a local DNS name, `fileserver.lab`, so machines on LAN-A can reach a host by name instead of by IP address, then practice troubleshooting a broken record.

## Concept
**DNS** translates names into IP addresses. A **host override** is a local record on the firewall's DNS service that answers for a name you choose, which is how many small networks provide internal names. If the record points to the wrong IP, the name resolves but leads to the wrong place, a common real-world problem.

## Steps
1. In OPNsense, added a host override (TODO: confirm menu path, e.g. Services > Dnsmasq DNS & DHCP > Hosts) with:

   | Field | Value |
   |---|---|
   | Host | ubuntu-client |
   | Domain |  |
   | IP address | 10.10.10.50 |

2. From the Ubuntu client, tested the name:
   ```bash
   nslookup fileserver.lab
   ping fileserver.lab
   ```
   Server:		127.0.0.53
   Address:	127.0.0.53#53

   Non-authoritative answer:
   Name:	fileserver.lab
   Address: 10.10.10.50
   
   64 bytes from fileserver.lab (10.10.10.50): icmp_seq=1 ttl=64 time=0.024 ms
   64 bytes from fileserver.lab (10.10.10.50): icmp_seq=2 ttl=64 time=0.075 ms
   64 bytes from fileserver.lab (10.10.10.50): icmp_seq=3 ttl=64 time=0.035 ms
   64 bytes from fileserver.lab (10.10.10.50): icmp_seq=4 ttl=64 time=0.330 ms
   64 bytes from fileserver.lab (10.10.10.50): icmp_seq=5 ttl=64 time=0.036 ms
   64 bytes from fileserver.lab (10.10.10.50): icmp_seq=6 ttl=64 time=0.036 ms
   64 bytes from fileserver.lab (10.10.10.50): icmp_seq=7 ttl=64 time=0.038 ms
   64 bytes from fileserver.lab (10.10.10.50): icmp_seq=8 ttl=64 time=0.037 ms
   64 bytes from fileserver.lab (10.10.10.50): icmp_seq=9 ttl=64 time=0.040 ms
   64 bytes from fileserver.lab (10.10.10.50): icmp_seq=10 ttl=64 time=0.047 ms


