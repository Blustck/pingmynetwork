# VTP Lab
<img width="1377" height="595" alt="image" src="https://github.com/user-attachments/assets/d2a02ebe-1c24-47c1-9c53-c12d5a4715c2" />


Configure **VTP version 2** across three Cisco 3560 switches to share VLANs automatically.

- **S1 — Server:** Creates and advertises VLANs.
- **S2 — Transparent:** Maintains local VLANs and forwards VTP updates.
- **S3 — Client:** Learns VLANs from the server.
- **Domain:** `ccna`
- **Topology:** S1 ↔ S2 ↔ S3

**Lab created using [PingMyNetwork.com](https://pingmynetwork.com/).**

## S1 — VTP Server

```cisco
conf t
hostname S1
vtp domain ccna
vtp version 2
vtp mode server

interface gi1/0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit

vlan 10
 name IT
end
write memory
```

## S2 — VTP Transparent

```cisco
conf t
hostname S2
vtp domain ccna
vtp version 2
vtp mode transparent

interface range gi1/0/1 - 2
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit

vlan 11
vlan 22
vlan 33
vlan 44
vlan 55
vlan 66
 name TEST
end
write memory
```

## S3 — VTP Client

```cisco
conf t
hostname S3
vtp domain ccna
vtp version 2
vtp mode client

interface gi1/0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
end
write memory
```

## Verification

Run on all switches:

```cisco
show vtp status
show vlan brief
show interfaces trunk
```

**Expected Results:**

- **S1:** VLAN 10 exists.
- **S2:** VLANs 11, 22, 33, 44, 55, and 66 exist locally.
- **S3:** Automatically learns VLAN 10 from S1.

**Key takeaway:** VTP synchronizes VLAN information between servers and clients, while transparent switches maintain their own VLAN database.
