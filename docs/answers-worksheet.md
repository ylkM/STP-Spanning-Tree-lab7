# Answers Worksheet

Exact root-bridge/port-role output in Packet Tracer depends on the **randomly
assigned MAC address** baked into each switch instance, so this worksheet gives you
the *method* to read your own `show spanning-tree` output, plus the *deterministic*
answers for Steps 2-5 (once priorities are manually set, MAC address no longer
decides the outcome).

---

## Question 1 — Current STP topology (before any config)

Run on every switch:
```
show spanning-tree vlan 1
show spanning-tree vlan 2
```

How to read it:
- **Root Bridge**: the switch whose `show spanning-tree` output says
  `This bridge is the root`. Before any config, all 4 switches have the default
  priority of **32768**, so the tiebreaker is the **lowest MAC address** — whichever
  switch has it becomes root for both VLAN 1 and VLAN 2 (same switch, since no
  per-VLAN priority has been set yet).
- **Port roles** (per switch, per VLAN), read from the `Interface / Role / Sts`
  columns:
  - `Root` (Sts: FWD) — the one port on a non-root switch with the lowest-cost path
    back to the root bridge.
  - `Desg` (Sts: FWD) — the port on a given network segment responsible for
    forwarding traffic onto that segment (the root switch's ports are always
    Designated).
  - `Altn` (Sts: BLK) — a redundant/backup port that is blocked to prevent a loop.

Record your actual output — root bridge + all 12 interface roles (3 links per
switch x 4 switches) — the specific answer will vary per Packet Tracer instance.

---

## Question 2 — After setting root primary/secondary

Config applied (see `configs/switches/SW1.txt` and `SW2.txt`):
```
SW1: spanning-tree vlan 1 root primary     (priority -> 24576, or lower again if needed)
SW1: spanning-tree vlan 2 root secondary   (priority -> 28672)
SW2: spanning-tree vlan 2 root primary     (priority -> 24576)
SW2: spanning-tree vlan 1 root secondary   (priority -> 28672)
```

Deterministic result:
- **VLAN 1 root bridge = SW1** (lowest priority for VLAN 1: 24576)
- **VLAN 2 root bridge = SW2** (lowest priority for VLAN 2: 24576)
- SW1 and SW2 no longer depend on MAC address to decide the root — priority now wins
  outright, since 24576 and 28672 are both lower than the default 32768 held by
  SW3 and SW4.

Now trace port roles per VLAN:
- **VLAN 1**: SW1 is root -> both of SW1's other links (to SW3, to SW4) are
  Designated on SW1's side. SW2, SW3, SW4 each pick a **Root port** = the interface
  on their shortest (lowest total cost) path back to SW1. Any switch with two paths
  to SW1 will block the higher-cost/higher-port-ID one.
- **VLAN 2**: same logic, but rooted at SW2 instead.

Because VLAN 1 and VLAN 2 now have *different* root bridges, it's entirely possible
(and expected) for the **same physical trunk port** to be Forwarding for VLAN 1 but
Blocking for VLAN 2 (or vice versa) — this is the normal behavior of PVST+
(Per-VLAN Spanning Tree).

Re-run `show spanning-tree vlan 1` / `vlan 2` on all four switches and record the
new role/state table.

---

## Question 3 — SW4 Fa0/2 VLAN1 cost raised to 100

Config (see `configs/switches/SW4.txt`):
```
interface FastEthernet0/2
 spanning-tree vlan 1 cost 100
```

**Does SW4 select a different VLAN1 root port? Yes, if an alternate path with a
lower total cost exists.**

Why: SW4 has two paths back to the VLAN1 root (SW1):
- Direct link: SW1<->SW4 via Fa0/2 — originally cost 19 (default FastEthernet cost),
  now manually raised to 100.
- Indirect link: SW4 -> SW2 (Fa0/1, cost 19) -> SW2's link back to SW1 (Fa0/3,
  cost 19) = total cost 38 to reach the VLAN1 root through SW2.

Before the change: direct path (19) < indirect path (38), so Fa0/2 was root port.
After the change: direct path (100) > indirect path (38), so **Fa0/1 (toward SW2)
becomes SW4's new VLAN1 root port**, and Fa0/2 moves to Blocking for VLAN1.

If your topology's indirect path total cost is still higher than 100, the port
would NOT change — always compare the two actual path costs from your own
`show spanning-tree vlan 1` output before and after.

---

## Question 4 — SW1 Fa0/1 VLAN1 port priority raised to 240

Config (see `configs/switches/SW1.txt`):
```
interface FastEthernet0/1
 spanning-tree vlan 1 port-priority 240
```

Port priority only affects **which port SW1 uses to send out BPDUs on that
segment (i.e., which port SW1 elects as Designated on a shared link)** — it does
NOT by itself change what SW3 selects as its own root port, because SW3's root
port decision is based on the **root path cost it receives in BPDUs**, and
secondarily its own port priority/ID as a tiebreaker — not SW1's port priority.

**Does SW3 select a different root port? Typically No.**

Why: SW3 still has only two paths back to root (SW1):
- Direct: SW3 <-> SW1 via Fa0/1, cost 19.
- Indirect: SW3 -> SW2 (Fa0/2, cost 19) -> SW2 -> SW1 (Fa0/3, cost 19) = total 38.

The direct path (19) is still cheaper than the indirect path (38) regardless of
SW1's local port-priority value, because SW1's port-priority only breaks ties
**between equal-cost paths on the same switch**, and doesn't propagate into SW3's
own root-path-cost calculation. SW3 keeps Fa0/1 as its VLAN1 root port.

(Port priority *would* matter if two paths to root had **identical total cost** —
then the receiving switch breaks the tie using the sender's bridge ID first, then
the sender's port priority/ID. That's not the case here.)

---

## Question 5 — PortFast + BPDU Guard on SW3/SW4 Fa0/3

Config (see `configs/switches/SW3.txt` and `SW4.txt`):
```
interface FastEthernet0/3
 spanning-tree portfast
 spanning-tree bpduguard enable
```

- **PortFast** skips the Listening/Learning STP states on an access port so the
  connected PC gets network access immediately instead of waiting ~30-50 seconds.
- **BPDU Guard** automatically **err-disables** the port if it ever receives a BPDU
  — protecting against someone accidentally plugging a switch/hub into what should
  be an end-device-only port and creating a loop.

Verify with:
```
show spanning-tree interface fa0/3 detail
show spanning-tree summary totals
```
