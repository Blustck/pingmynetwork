# Basic VLANs — Inter-VLAN Routing (Layer 3 Switch)

Configure VLANs and **SVIs** on a Cisco 3560 Layer 3 switch to route between networks without a separate router. Three Cisco 2960 access switches connect the PCs; the server connects directly to the Layer 3 switch.



**Lab built using [PingMyNetwork.com](https://pingmynetwork.com/).**


## IP Addressing

| Device | VLAN | IP Address | Default Gateway |
| --- | --- | --- | --- |
| PC1 | 10 | `10.1.10.10/24` | `10.1.10.254` |
| PC2 | 20 | `10.1.20.20/24` | `10.1.20.254` |
| PC3 | 30 | `10.1.30.30/24` | `10.1.30.254` |
| Server | 100 | `10.1.100.100/24` | `10.1.100.254` |

**Switch management (VLAN 1):** L3-SW `10.1.1.254`, SW1 `10.1.1.1`, SW2 `10.1.1.2`, SW3 `10.1.1.3`.

## L3-SW — Layer 3 Switch

```cisco
conf t
hostname L3-SW
vlan 10
vlan 20
vlan 30
vlan 100
exit
ip routing

interface vlan 1
 ip address 10.1.1.254 255.255.255.0
 no shutdown
exit
interface vlan 10
 ip address 10.1.10.254 255.255.255.0
 no shutdown
exit
interface vlan 20
 ip address 10.1.20.254 255.255.255.0
 no shutdown
exit
interface vlan 30
 ip address 10.1.30.254 255.255.255.0
 no shutdown
exit
interface vlan 100
 ip address 10.1.100.254 255.255.255.0
 no shutdown
exit

interface range gi1/0/1 - 3
 switchport mode trunk
exit
interface gi1/0/20
 switchport mode access
 switchport access vlan 100
end
write memory
```

## SW1 — VLAN 10

```cisco
conf t
hostname SW1
vlan 10
exit
interface fa0/1
 switchport mode access
 switchport access vlan 10
exit
interface gi0/1
 switchport mode trunk
exit
interface vlan 1
 ip address 10.1.1.1 255.255.255.0
 no shutdown
exit
ip default-gateway 10.1.1.254
end
write memory
```

## SW2 — VLAN 20

```cisco
conf t
hostname SW2
vlan 20
exit
interface fa0/1
 switchport mode access
 switchport access vlan 20
exit
interface gi0/1
 switchport mode trunk
exit
interface vlan 1
 ip address 10.1.1.2 255.255.255.0
 no shutdown
exit
ip default-gateway 10.1.1.254
end
write memory
```

## SW3 — VLAN 30

```cisco
conf t
hostname SW3
vlan 30
exit
interface fa0/1
 switchport mode access
 switchport access vlan 30
exit
interface gi0/1
 switchport mode trunk
exit
interface vlan 1
 ip address 10.1.1.3 255.255.255.0
 no shutdown
exit
ip default-gateway 10.1.1.254
end
write memory
```

## Verification

On **L3-SW**:

```cisco
show vlan brief
show interfaces trunk
show ip interface brief
show ip route
```

From **PC1**:

```text
ping 10.1.20.20
ping 10.1.30.30
ping 10.1.100.100
ping 10.1.1.254
```

**Expected:** All pings succeed when the VLANs, trunks, SVIs, host gateways, and `ip routing` are working.

**Key takeaway:** The Layer 3 switch routes between VLANs through SVIs; no external router is needed.
