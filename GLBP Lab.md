# GLBP Load Balancing Lab — Four Cisco Routers

Configure **Gateway Load Balancing Protocol (GLBP)** with four Cisco routers to provide **gateway redundancy and load balancing** through one virtual IP (`10.1.1.1`). GLBP uses round-robin ARP replies to distribute hosts across available forwarders.


<img width="1152" height="917" alt="image" src="https://github.com/user-attachments/assets/c5a21b6e-6e8c-42f7-8ee6-28fa6559140c" />

**Lab built using [PingMyNetwork.com](https://pingmynetwork.com/).**

## Topology & IP Addressing

- **LAN:** `10.1.1.0/24` · **VLAN:** `10` · **GLBP Group:** `10`
- **Virtual gateway:** `10.1.1.1`
- **Connections:** Four routers → SW1 → four PCs; each router also connects to ISP.

| Device | LAN IP | WAN IP (/30) | GLBP Priority |
|---|---|---|---|
| R1 | `10.1.1.2` | `172.16.0.1` | 140 |
| R2 | `10.1.1.3` | `172.16.0.5` | 130 |
| R3 | `10.1.1.4` | `172.16.0.9` | 120 |
| R4 | `10.1.1.5` | `172.16.0.13` | 110 |

**PCs:** `10.1.1.101–104/24`, all with gateway `10.1.1.1`.

## R1 — GLBP Priority 140

```cisco
conf t
hostname R1
interface gi0/0
 ip address 10.1.1.2 255.255.255.0
 glbp 10 ip 10.1.1.1
 glbp 10 priority 140
 glbp 10 preempt
 glbp 10 load-balancing round-robin
 glbp 10 authentication md5 key-string $123#
 no shutdown
exit
interface gi0/1
 ip address 172.16.0.1 255.255.255.252
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 172.16.0.2
end
write memory
```

## R2 — GLBP Priority 130

```cisco
conf t
hostname R2
interface gi0/0
 ip address 10.1.1.3 255.255.255.0
 glbp 10 ip 10.1.1.1
 glbp 10 priority 130
 glbp 10 preempt
 glbp 10 load-balancing round-robin
 glbp 10 authentication md5 key-string $123#
 no shutdown
exit
interface gi0/1
 ip address 172.16.0.5 255.255.255.252
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 172.16.0.6
end
write memory
```

## R3 — GLBP Priority 120

```cisco
conf t
hostname R3
interface gi0/0
 ip address 10.1.1.4 255.255.255.0
 glbp 10 ip 10.1.1.1
 glbp 10 priority 120
 glbp 10 preempt
 glbp 10 load-balancing round-robin
 glbp 10 authentication md5 key-string $123#
 no shutdown
exit
interface gi0/1
 ip address 172.16.0.9 255.255.255.252
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 172.16.0.10
end
write memory
```

## R4 — GLBP Priority 110

```cisco
conf t
hostname R4
interface gi0/0
 ip address 10.1.1.5 255.255.255.0
 glbp 10 ip 10.1.1.1
 glbp 10 priority 110
 glbp 10 preempt
 glbp 10 load-balancing round-robin
 glbp 10 authentication md5 key-string $123#
 no shutdown
exit
interface gi0/1
 ip address 172.16.0.13 255.255.255.252
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 172.16.0.14
end
write memory
```

## ISP Router

```cisco
conf t
hostname ISP
interface gi0/0
 ip address 172.16.0.2 255.255.255.252
 no shutdown
exit
interface gi0/1
 ip address 172.16.0.6 255.255.255.252
 no shutdown
exit
interface gi0/2
 ip address 172.16.0.10 255.255.255.252
 no shutdown
exit
interface gi0/3
 ip address 172.16.0.14 255.255.255.252
 no shutdown
exit
interface loopback0
 ip address 203.0.113.10 255.255.255.255
exit
ip route 10.1.1.0 255.255.255.0 172.16.0.1
ip route 10.1.1.0 255.255.255.0 172.16.0.5
ip route 10.1.1.0 255.255.255.0 172.16.0.9
ip route 10.1.1.0 255.255.255.0 172.16.0.13
end
write memory
```

## SW1 — VLAN 10

```cisco
conf t
hostname SW1
vlan 10
 name GLBP-LAN
exit
interface range fa0/1 - 6
 switchport mode access
 switchport access vlan 10
exit
interface range gi0/1 - 2
 switchport mode access
 switchport access vlan 10
end
write memory
```

## Verification & Testing

**On routers:**

```cisco
show glbp brief
show glbp
show ip route
```

**On PCs:**

```text
arp -a
ping 10.1.1.1
ping 203.0.113.10
tracert 203.0.113.10
```

**Expected:** R1 is AVG, R2 is standby AVG, and all four routers can act as Active Virtual Forwarders (AVFs). PCs may resolve `10.1.1.1` to different GLBP virtual MAC addresses, showing load distribution.

**Key takeaway:** One virtual default gateway, multiple forwarding routers, and gateway redundancy.
