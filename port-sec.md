# Cisco Port Security Lab

Configure **Port Security** on a Cisco switch to allow one MAC address per access port. Sticky learning records the authorized device's MAC address, while `shutdown` mode disables the port if another MAC address is detected.

<img width="1215" height="712" alt="image" src="https://github.com/user-attachments/assets/4b8fecb0-7531-4985-b35c-f1dcccdb8b2d" />


**Lab built using [PingMyNetwork.com](https://pingmynetwork.com/).**

## Topology

- **DSW1 (Cisco 3560)** — `Fa0/1` connects to PC1; `Fa0/2` connects to PC2.
- **PC1:** `192.168.1.10/24` (authorized)
- **PC2:** `192.168.1.20/24`
- **PC3:** `192.168.1.10/24` (test device with a different MAC address; replaces PC1)

## DSW1 — Port Security Configuration

```cisco
conf t
interface fa0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
end
write memory
```

## Verify

After PC1 sends traffic:

```cisco
show port-security interface fa0/1
show port-security address
```

Expected: **Secure-up**, **1 sticky MAC address**, and **0 violations**.

## Test Unauthorized Device

1. From PC1, verify connectivity to PC2: `ping 192.168.1.20`.
2. Disconnect PC1 and connect **PC3** to `Fa0/1`.
3. Generate traffic from PC3 and check the switch:

```cisco
show port-security interface fa0/1
show interfaces status err-disabled
```

Expected: **Port-security violation**; `Fa0/1` enters **err-disabled** state.

## Restore Port

Reconnect PC1, remove PC3, then on DSW1:

```cisco
conf t
interface fa0/1
 shutdown
 no shutdown
end
```

**Key takeaway:** Sticky MAC learning with `violation shutdown` limits an access port to the learned device's MAC address.
