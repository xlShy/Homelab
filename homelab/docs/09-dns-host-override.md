# DNS Host Override (fileserver.lab)

## Goal
Create a local DNS name, `fileserver.lab`, so machines on LAN-A can reach a host by name instead of by IP address, then practice troubleshooting a broken record.

## Concept
**DNS** translates names into IP addresses. A **host override** is a local record on the firewall's DNS service that answers for a name you choose, which is how many small networks provide internal names. If the record points to the wrong IP, the name resolves but leads to the wrong place, a common real-world problem.

## Steps
1. In OPNsense, added a host override (TODO: confirm menu path, e.g. Services > Dnsmasq DNS & DHCP > Hosts) with:

   | Field | Value |
   |---|---|
   | Host | fileserver |
   | Domain | lab |
   | IP address | TODO: correct IP |

2. From the Ubuntu client, tested the name:
   ```bash
   nslookup fileserver.lab
   ping fileserver.lab
   ```
   (TODO: paste the output.)

## Break and fix exercise
1. **Broke it on purpose:** changed the record to a wrong IP address (TODO: which IP).
2. **Symptom:** the name resolved to the wrong address, so (TODO: what happened, e.g. ping timed out).
3. **Diagnosis:** compared the `nslookup` result against the expected IP, which showed the record was wrong.
4. **Fix:** corrected the IP in the override, applied the change, and tested again. The name resolved correctly.

## Problems and fixes
- TODO: add any other problems (for example, the client caching the old answer). If none, write "None encountered."

## What I learned
- TODO: one or two sentences. For example: a name that resolves isn't proof it resolves correctly, so compare the result against the expected IP.
