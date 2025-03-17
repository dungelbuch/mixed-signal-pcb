# PCB Design and Routing

Before movong on to PCB design, layout and routing, it is important to remember the following:
1. Check your symbols and footprints, circuits, datasheets, component values/tolerances/ratings/types.
2. Annotate schematics in a logical manner (page by page, left-right, top-bottom).
3. Label all your nets and components.
4. Perform an electrical rules check (ERC) in KiCAD.
5. Check if the components are physically available.
6. Choose foorprints for eaach symbol in KiCAD.

**Pro Tip**: Print out schematic on paper, check them there. And if any errors are found, correct it and wait for a day before repeating the
process.

## PCB Stackup

![PCB Stackup](/notes/img/pcb_stackup.png)

Most (embedded) mixed-signal boards are **four layers** minimum. We have the choice between two stackups:
* A) L1: Sig, L2: GND, L3: VCC, L4: Sig
* B) L1: Sig, L2: GND, L3: GND, L4: Sig

Stackup A makes out power routing a bit easier, since it give us mor more space on signal layers. Anytime we need VCC or GND, we can just
drop a via to the appropriate layer. 

Stackup B is more preferred for the following reasons:
1. **Low impedance return paths for any (single ended) signal** on both signal layers.
2. Spacing between L2 and L3 is huge in 1.6mm four layer boards -> Near zwro capacitive effect when having VCC + GND plane pair.
3. We have multiple VCCs -> split planes, return currents harder to trace and control if using VCC planes as reference.
Routing power via traces is no problem for the "low frequency" PCBs.

**However, when changing signal layers with a via, we need to place a GND via close to the signal via to maintain good coupling and provide a
good return path.**

### Signal Path

Consider the following as **"facts"**:
* Signals flow in closed loops.
* **Signal energy traverls in dielectric metarial**, between copper traces and planes. -> Copper is simply a waveguide.
* For signals from kHz range, **return current will be directly underneath the forward current trace** -> I.e. in the plane below!

Therefore we need to consider both **FORWARD** and **RETURN** paths of signals. **We will use the internal planes as our return paths for traces on
signal layers**.

## Traces

We have broadly two types of traces: Signal or power traces. For the design to work as intended, we need to determine the width or limit the
following parameters:
1. Thickness (current carrying capability)
2. Width (current carrying capability, controlled impedance)
3. Maximum length (signal integrity, noise, EMI)
4. Spacing (track-to-track, track-to-pad, etc..., cross-talk, noise)
5. Matching (differential pair, matched-length bus)

Here are some guidelines from the course to follow:
* Width: 0.5mm traces for power and 0.3mm traces for signals are a good starting point.
* To determine the current carrying capability of a trace, use an online tool or the calculator in KiCAD!
* Spacing: at least 3x dielectric thickness between traces.
* **Always** minimize the length and maximize spacing!

### Controlled Impedance Traces

Signals propagate within the dielectric space of the PCB, between the copper trace and return plane. The copper simply acts as a waveguide. The
propagation speed is dependent on the dielectric constant of the material. The need to control the impedance of a trace arises when the
**rise/fall time** of signals are comparable to propagation times on PCB traces. **Traces now acts as a transmission line**. Therefore as a rule
of thumb: **Always route high-speed digital buses and RF signals as controlled impedance traces**. Controlling inpedance of traces to match
impedance of driver and receiver minimizes reflections and thus improves signal integrity and enables maximum possible power transfer between
devices.

![Controlled Impedance](/notes/img/controlled_impedance.png)

The trace impedance is primarily dependent on trace capacitance and inductance (loosely):
* Inductance (L) has to do with loop area, dependent on trace width (W).
* Capacitance (C) has to do with surface area, dependent on the distance between trace and return plane (H).

The impedance is calculated as follows:
$$
Z = sqrt(L/C)
$$

Increasing L decreases Z, while increasing C increases Z. L and C, and thus Z are defined by geometry (W and H) and the dielectric material.

### Single-Ended vs Differential Pairs

For single-ended and differential pairs, the impedance is calculated are different. Given a specific PCB stackup, how do we calculate the
required trace width to obtain a specific impedance? We can use the following steps:
1. Get bus specification: USB (90 Ohm differential), Ethernet (100 Ohm differential), RF (~50 Ohm single-ended).
2. Get PCB stackup, dimensions and materials (see PCB vendor).
3. Use an online calculator or KiCAD calculator to determine the trace width.

## Holes: VIAS and Through-Holes 

Holes in PCB can either be plated or non-plated. Plated holes are used for vias and through-holes. Non-plated holes are used for mounting
components. Standard sized brill bits are 0.1mm, 0.15mm, etc. The minimum hole size is determined by:
* Cost and manufacturer capability.
* Aspect ratio (drill width vs PCB height).
* Routing/component constraints

VIAs are not only determine by the hole size, but also the pad size. The pad size is determined by the current carrying capability of the via and
there are minimal specifications for it, given by the manufacturer. There are also different types of vias: standard, blind, buried, micro-vias.
These are recommendations for vias:
* **Small**: 0.45mm pad and 0.25mm drill hole.
* **Medium**: 0.7mm pad and 0.3mm drill hole.
* **Large**: 0.8mm pad and 0.4mm drill hole.

Use medium size if possible, since no additional cost is incurred. Use large for higher current carrying capability and small if space
constraints are present.