# Router-on-a-Stick (Inter-VLAN Routing)

Configure **Router-on-a-Stick (RoaS)** to route between VLANs 10, 20, and 30 using a single router interface and an 802.1Q trunk.

![Uploading image.png…]()

**Lab built using [PingMyNetwork.com](https://pingmynetwork.com/).**

## Topology & IP Addressing

- **R1 Gi0/0 ↔ SW1 Gi0/1** — trunk link
- **PC1 → SW1 Fa0/1** — VLAN 10 (Sales), `192.168.10.10/24`, gateway `192.168.10.1`
- **PC2 → SW1 Fa0/2** — VLAN 20 (HR), `192.168.20.10/24`, gateway `192.168.20.1`
- **PC3 → SW1 Fa0/3** — VLAN 30 (IT), `192.168.30.10/24`, gateway `192.168.30.1`

## SW1 Configuration

```cisco
conf t
hostname SW1

vlan 10
 name SALES
vlan 20
 name HR
vlan 30
 name IT

interface fa0/1
 switchport mode access
 switchport access vlan 10
interface fa0/2
 switchport mode access
 switchport access vlan 20
interface fa0/3
 switchport mode access
 switchport access vlan 30

interface gi0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
end
write memory
```

## R1 Configuration

```cisco
conf t
hostname R1

interface gi0/0
 no shutdown

interface gi0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
interface gi0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
interface gi0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
end
write memory
```

## Verification

**SW1:**

```cisco
show vlan brief
show interfaces trunk
```

**R1:**

```cisco
show ip interface brief
show ip route
```

**From PC1:**

```text
ping 192.168.20.10
ping 192.168.30.10
```

**Key takeaway:** One physical router link, multiple VLAN gateways using 802.1Q subinterfaces.
