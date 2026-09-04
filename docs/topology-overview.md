# Topology Overview

## Physical / Logical Links

| Local Switch | Local Port | Remote Switch | Remote Port | Link Type |
|---------------|------------|-----------------|--------------|-----------|
| SW1           | Fa0/3      | SW2             | Fa0/3        | Trunk (VLAN 1,2) |
| SW1           | Fa0/1      | SW3             | Fa0/1        | Trunk (VLAN 1,2) |
| SW1           | Fa0/2      | SW4             | Fa0/2        | Trunk (VLAN 1,2) |
| SW2           | Fa0/1      | SW4             | Fa0/1        | Trunk (VLAN 1,2) |
| SW2           | Fa0/2      | SW3             | Fa0/2        | Trunk (VLAN 1,2) |
| SW3           | Fa0/3      | PC (VLAN1)      | Fa0          | Access (VLAN 1) |
| SW4           | Fa0/3      | PC (VLAN2)      | Fa0          | Access (VLAN 2) |

Note: SW3 and SW4 are **not** directly connected to each other — they only reach one
another through SW1 and/or SW2. This asymmetry is what makes the lab interesting: it's
a "square with two diagonals" (5 inter-switch links), not a full mesh.

## VLANs

| VLAN | Name  | Subnet             | Access Switch/Port |
|------|-------|----------------------|-----------------------|
| 1    | default | 172.16.0.0/25      | SW3 Fa0/3 |
| 2    | VLAN2 | 172.16.0.128/25     | SW4 Fa0/3 |

## Why Loops Exist

Because SW1, SW2, SW3, and SW4 form a graph with multiple paths between any two
switches (e.g., SW1 -> SW3 directly, or SW1 -> SW2 -> SW3), Layer 2 loops would form
without STP. STP blocks redundant paths to prevent broadcast storms/MAC table
instability while keeping a backup path available if a link fails.
