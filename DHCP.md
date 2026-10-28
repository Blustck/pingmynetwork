# DHCP Server, Relay & Client Lab

Configure **R2 as a DHCP server** for two remote LANs, **R1 and R3 as DHCP relay agents**, and **R2's WAN interface as a DHCP client**.

<img width="1567" height="922" alt="image" src="https://github.com/user-attachments/assets/8fc091d4-8d9b-43b7-9917-e7dd2b44c240" />



**Lab built using [PingMyNetwork.com](https://pingmynetwork.com/).**

## Lab Overview

| Device | Role | Network / Address |
|---|---|---|
| R1 | DHCP relay for PC1 | `192.168.10.0/24` · GW `192.168.10.1` |
| R2 | Central DHCP server | Pools for both LANs |
| R3 | DHCP relay for PC2 | `192.168.30.0/24` · GW `192.168.30.1` |
| R2 `Gi0/1` | DHCP client | Address from ISP |

> **Prerequisite:** Router interfaces and routes between R1, R2, and R3 must already be configured.

## R2 — DHCP Server & Client

```cisco
conf t
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10

ip dhcp pool R1-LAN
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.20.254
exit

ip dhcp pool R3-LAN
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 192.168.20.254
exit

interface GigabitEthernet0/1
 ip address dhcp
 no shutdown
exit
end
write memory
```

## R1 — DHCP Relay

```cisco
conf t
interface GigabitEthernet0/0
 ip helper-address 10.1.1.2
exit
end
write memory
```

## R3 — DHCP Relay

```cisco
conf t
interface GigabitEthernet0/0
 ip helper-address 10.2.2.2
exit
end
write memory
```

## Verification

**On R2:**

```cisco
show ip dhcp pool
show ip dhcp binding
show ip interface brief
```

**On R1 and R3:**

```cisco
show ip interface GigabitEthernet0/0
```

Set **PC1 and PC2 to DHCP**, then verify they receive addresses in `192.168.10.0/24` and `192.168.30.0/24`, respectively, with the correct default gateways.

**Key takeaway:** `ip dhcp pool` assigns addresses; `ip helper-address` relays DHCP requests across routers; `ip address dhcp` lets a router interface obtain its own address.
