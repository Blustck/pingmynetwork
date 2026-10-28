# LACP EtherChannel Lab (Cisco 2960)

Combine two physical switch links into one logical **Port-channel** using LACP (`active` mode). If one link fails, the other can continue forwarding traffic.

## Topology

- **SW1 Gi0/1–2 ↔ SW2 Gi0/1–2** (LACP EtherChannel)
- **PC1 → SW1 Fa0/1:** `192.168.10.10/24`
- **PC2 → SW2 Fa0/1:** `192.168.10.20/24`
- **VLAN:** 10

## SW1 Configuration

```cisco
conf t
hostname SW1
vlan 10
 name LAB
exit
interface fa0/1
 switchport mode access
 switchport access vlan 10
exit
interface range gi0/1 - 2
 switchport mode trunk
 channel-group 1 mode active
exit
interface port-channel 1
 switchport mode trunk
end
write memory
```

## SW2 Configuration

```cisco
conf t
hostname SW2
vlan 10
 name LAB
exit
interface fa0/1
 switchport mode access
 switchport access vlan 10
exit
interface range gi0/1 - 2
 switchport mode trunk
 channel-group 1 mode active
exit
interface port-channel 1
 switchport mode trunk
end
write memory
```

## Verify

Run on both switches:

```cisco
show etherchannel summary
show interfaces trunk
```

Expected: **`Po1(SU)`** and **`Gi0/1(P)` / `Gi0/2(P)`** (bundled ports).

From PC1:

```text
ping 192.168.10.20
```

## Optional: Link Failure Test

On SW1, disable one member and test the ping again:

```cisco
conf t
interface gi0/2
 shutdown
end
show etherchannel summary
```

Restore the member:

```cisco
conf t
interface gi0/2
 no shutdown
end
```

**Takeaway:** Two physical links, one logical EtherChannel, with link redundancy.
