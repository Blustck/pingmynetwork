# NAT & PAT (NAT Overload) Lab

Configure **PAT (NAT Overload)** on Cisco R1 to allow multiple private hosts to access an external server using **one shared IP address**. 

<img width="1112" height="931" alt="image" src="https://github.com/user-attachments/assets/9586023f-385c-45d7-b704-89e438da89ec" />


This lab was built using [PingMyNetwork.com](https://pingmynetwork.com/).

## Topology & IP Addressing

`PC1 / PC2 / PC3 → SW1 → R1 → R2 → SRV1`

| Device / Interface | IP Address |
|---|---|
| PC1 | `192.168.1.11/24` |
| PC2 | `192.168.1.12/24` |
| PC3 | `192.168.1.13/24` |
| R1 LAN (`G0/0`) | `192.168.1.1/24` |
| R1 WAN (`G0/3/0`) | `1.2.3.1/24` |
| R2 WAN | `1.2.3.2/24` |
| SRV1 | `1.1.1.100/24` |

**PC default gateway:** `192.168.1.1`  
**Routing and the external-link ACL** are assumed to be configured already, as in the original lab.

## R1 — PAT Configuration

```cisco
conf t
interface GigabitEthernet0/0
 ip nat inside
exit
interface GigabitEthernet0/3/0
 ip nat outside
exit
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/3/0 overload
end
write memory
```

## Verify

On R1:

```cisco
show ip nat translations
show ip nat statistics
show ip route
```

From each PC:

```text
ping 1.1.1.100
```

**Expected:** All PCs reach SRV1. NAT translations show their private addresses sharing R1's outside address (`1.2.3.1`). For ICMP, translations use ICMP identifiers rather than TCP/UDP port numbers.

**Key takeaway:** PAT uses `overload` to let multiple internal devices share a single external IPv4 address.
