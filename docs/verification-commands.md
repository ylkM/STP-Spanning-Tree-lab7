# Verification Commands

Run these on each switch (SW1, SW2, SW3, SW4) at each stage of the lab.

## Core STP status

```
show spanning-tree
show spanning-tree vlan 1
show spanning-tree vlan 2
show spanning-tree summary
```

## Bridge ID / priority check

```
show spanning-tree vlan 1 | include Priority
show spanning-tree vlan 2 | include Priority
```

## Per-interface detail (cost, priority, role, state)

```
show spanning-tree interface fa0/1 detail
show spanning-tree interface fa0/2 detail
show spanning-tree interface fa0/3 detail
```

## VLAN / trunk sanity checks

```
show vlan brief
show interfaces trunk
show interfaces switchport
```

## PortFast / BPDU Guard state (after Step 5)

```
show spanning-tree interface fa0/3 detail
show running-config interface fa0/3
```
A properly configured access port should show:
```
Port fa0/3 ... Portfast enabled
BPDU Guard is enabled
```
