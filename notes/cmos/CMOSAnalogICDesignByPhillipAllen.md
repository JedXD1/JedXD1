# CMOS Analog IC Design — Course Study Notes
**Instructor:** Phillip E. Allen (with Douglas R. Holberg) — *AnalogDesign.org*
**Source:** 12-Lecture Overview Course on Analog Integrated Circuit Design (CMOS focus)

> [!NOTE]
> This document expands the raw lecture transcripts into a technically rigorous reference. Equations use LaTeX; every symbol is defined; circuit topologies are rendered as ASCII schematics; and historical/named concepts are explained for context. Where the lecturer's spoken explanation was compressed or implicit, additional silicon-level intuition has been added (marked with 🔬).

---

## Table of Contents
1. [Lecture 1 — Introduction & Overview](#lecture-1--introduction--overview)
2. [Lecture 2 — Technology](#lecture-2--technology)
3. [Lecture 3 — Modeling](#lecture-3--modeling)
4. [Lecture 4 — The Analog IC Design Process](#lecture-4--the-analog-ic-design-process)
5. [Lecture 5 — Key Principles, Concepts & Techniques](#lecture-5--key-principles-concepts--techniques)
6. [Lecture 6 — Characteristics of a Successful Analog Designer](#lecture-6--characteristics-of-a-successful-analog-designer)
7. [Lecture 7 — Independent Sources](#lecture-7--independent-sources)
8. [Lecture 8 — Amplifiers](#lecture-8--amplifiers)
9. [Lecture 9 — Operational Amplifiers (Op Amps)](#lecture-9--operational-amplifiers-op-amps)
10. [Lecture 10 — Comparators](#lecture-10--comparators)
11. [Lecture 11 — D/A and A/D Converters](#lecture-11--da-and-ad-converters)
12. [Lecture 12 — The Future of Analog IC Design](#lecture-12--the-future-of-analog-ic-design)
13. [Global Symbol & Acronym Glossary](#global-symbol--acronym-glossary)
14. [Cheat Sheet — Key Equations & Rules of Thumb](#cheat-sheet--key-equations--rules-of-thumb)

---

## Lecture 1 — Introduction & Overview

### Core Concept: Design vs. Analysis
- **Analysis**: Given a system (topology + component values), find its properties. The solution is **unique**.
- **Design (Synthesis)**: Given a set of target properties/specifications, find a system that realizes them. The solution is **never unique** — many topologies (System 1, 2, 3…) may satisfy the same spec, and the designer must evaluate trade-offs among candidates.

> [!TIP]
> This asymmetry (unique analysis vs. non-unique design) is *the* reason analog design is judgment-heavy: a "correct" analysis has one answer, but a "good" design is a value judgment about which trade-off best fits the application.

### Analog vs. Digital Circuits — Comparison Table

| Property | Analog | Digital |
|---|---|---|
| Signal | Continuous in amplitude; continuous or discrete in time | Discontinuous (discrete) in both amplitude and time (e.g., binary = 2 levels) |
| Design level | Circuit level | System/higher level |
| Component values | Continuum (0 → large) | Fixed/quantized; parasitic passives are usually *unwanted* |
| Dynamic range | Limited above by supply rails, below by noise/linearity | Effectively unlimited (add bits: ~6 dB/bit) |
| Modeling accuracy needed | High | Only timing accuracy needed |
| Programmability | Fixed, hard to vary | Easily reprogrammed in software |
| Hierarchy / blocks | Irregular blocks → complex hierarchy | Regular blocks → scalable hierarchy |
| CAD tool maturity | Difficult to apply broadly | Highly mature, widely used |

🔬 **Why 6 dB per bit?** Each additional binary bit doubles the number of quantization levels, and $20\log_{10}(2) \approx 6.02\ \text{dB}$. This is the origin of the common "6 dB/bit" dynamic-range rule used throughout data-converter design (see Lecture 11).

### Discrete vs. Integrated Analog Design

| Characteristic | Discrete Analog Design | Integrated Analog Design |
|---|---|---|
| Component tolerance | Whatever designer pays for | Poor **absolute** tolerance, much better **relative** (matching) tolerance |
| Range of values | Unlimited (e.g., 100 µF caps) | Limited by die area (e.g., ~10 pF caps) |
| Power dissipation | Limited by heat-sinking | Limited by technology & device type |
| Voltage | Large or small, per technology used | Constrained by process breakdown voltages |
| Cost | Expensive per circuit | Inexpensive per circuit (amortized over volume) |
| Electrical/physical co-design | Can be separated | Strongly co-dependent (layout parasitics affect electrical behavior) |
| Fabrication | Designer builds it | Foundry (IC manufacturer) builds it |
| Testing | Easy — full node access, can swap parts | Must be planned in from the start; limited node access |

> [!NOTE]
> **Matching vs. absolute accuracy** is one of the most important IC design intuitions: two adjacent, identically-drawn transistors on the same die will track each other (same process gradients, same temperature) far better than either one tracks an "ideal" textbook value. This underlies current mirrors, differential pairs, and bandgap references throughout the course.

### Evolution of Analog Design
- **Pre-computer era:** hand calculation (paper, pencil, slide rule/calculator, design tables) + breadboard construction and lab measurement as the "simulation" step.
- **Post-computer era:** hand calculation for *understanding*, followed by **circuit simulation** with accurate models for *verification*; breadboarding is largely replaced by simulation (though prototype builds still occur).
- **Pre-IC-technology era:** circuit specs *define* the components (you pick discrete R, L, C to hit a spec).
- **Post-IC-technology era:** the *technology* (physical layout rules, process parameters) defines what components are achievable — the causality reverses.

### The "Three-Legged Stool" of Analog IC Design
```
                _______________
               |   ANALOG IC   |
               |    DESIGN     |
               |_______________|
                /      |       \
               /       |        \
        Technology  Modeling  Circuit
                              Understanding
         (leg 1)    (leg 2)     (leg 3)
```
- **Technology** — process capability, device physics, parasitics.
- **Modeling** — ability to predict circuit performance via models (hand + simulation).
- **Circuit understanding** — intuitive grasp of *how* and *why* a circuit behaves as it does.

### Is Analog Design an Art or a Science?
- Tongue-in-cheek definitions given in lecture:
  - *Art*: something you can't explain how it works, or something someone else can explain but you can't.
  - *Science*: something you can explain, but someone else can't (yet).
- Conclusion: analog design is **both**, but the course's philosophical position is that it should be pushed toward being **more science than art** — i.e., made teachable and systematic through principles, concepts, and techniques (Lecture 5).
- Reference: *Analog Circuit Design: Art, Science, and Personalities*, ed. Jim Williams — a historical anthology on analog design culture.

### Lecture 1 Summary
- Design = process of finding a circuit that best meets specifications (non-unique solution space).
- Analog signals: continuous in amplitude; digital: discrete in amplitude and time.
- Discrete tech: specs define components. IC tech: technology defines components.
- Three legs of analog IC design: technology, modeling, circuit understanding.
- Analog design blends art and science, with an emphasis toward science.

---

## Lecture 2 — Technology

### Evolution of Active Devices

| Era | Device | Conduction Mechanism | Notes |
|---|---|---|---|
| 1930s | Vacuum tube | Field emission (thermionic emission) | Cathode heated to "boil off" electrons; controlled by a grid; anode collects |
| 1950s | Bipolar Junction Transistor (BJT) | Diffusion current | Lower voltage, no heater — enabled transistor radios |
| 1970s | MOSFET | Drift current | Dominant modern device; two terminals modulate current between two others |

🔬 Diffusion current (BJT) arises from carrier **concentration gradients** (minority carriers diffusing across the base), while drift current (MOSFET) arises from an **electric field** accelerating carriers in the channel ($J = qn\mu E$). This physical distinction is why BJTs and MOSFETs have very different transconductance-vs-bias relationships (exponential vs. square-law/linear).

> [!NOTE]
> Every active device historically follows the same abstract pattern: **two terminals control the voltage/current relationship between two other terminals**, enabling amplification. The lecturer predicts future devices will preserve this structural pattern even if the physical mechanism changes.

### Basic IC Process Steps

| Step | Description | Analog Design Influence |
|---|---|---|
| **Oxidation** | Growth of SiO₂ (insulator), from ~1 nm to ~1 µm thick | Sets oxide capacitance quality, threshold voltage, breakdown voltage, reliability |
| **Diffusion** | High-temperature introduction of dopant atoms | Sets p-n junction C–V characteristics, breakdown voltage, bulk (ohmic) resistance |
| **Ion Implantation** | Dopants accelerated at high velocity into the substrate (low temp) | Same electrical effects as diffusion but at lower thermal budget; introduces photolithographic-invariance issues; requires an anneal step to repair lattice damage |
| **Deposition** | Blanket film deposition (metal, semiconductor, or insulator) | Determines quality of resulting layers |
| **Etching** | Selective removal of a layer via a mask | Affects component size and matching; imperfections: *selectivity* (etch-rate mismatch between mask/film/underlying layer) and *undercut* (lateral vs. vertical etch, sometimes referred to loosely as "anisotropy") |
| **Epitaxy** | Growth of single-crystal Si on the wafer | Allows substrate doping to differ from surface doping |

```
Photolithography (patterning polysilicon), simplified cross-section:

  UV Light  UV Light
    ↓  ↓      ↓  ↓
  ┌────┐    ┌────┐          <- Mask (opaque regions block UV)
  ══════════════════        <- Photoresist (exposed regions altered)
  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓        <- Polysilicon (to be patterned)
  ------------------        <- Substrate
```
- **Positive photoresist:** UV-exposed resist is removed; unexposed (shadowed) resist remains and protects the underlying poly during etch.
- After etch + resist strip → patterned polysilicon matching the mask geometry.
- **Analog impact:** photolithographic linewidth control varies across the wafer surface, directly causing **mismatch** between "identical" devices at different die locations — a first-order limiter of matching accuracy.

### Advanced Process Steps

| Step | Description | Analog Influence |
|---|---|---|
| **Planarization** | Minimizes wafer surface height variation | Enables more metal levels; affects integrity of thin-film components stacked on metal; can introduce dielectric-thickness variation affecting parasitic extraction/modeling |
| **CMP (Chemical-Mechanical Polishing)** | Wafer flipped, polished chemically + mechanically | Flatness depends on spacing between structures; may require dummy pattern fill to equalize density |
| **Salicidation** (self-aligned silicide) | Deposits a thin metal (Ti, W, Ta) on poly/diffusion, reacted to form silicide | Lowers bulk/poly resistance and contact resistance; must be masked off ("salicide block") where not wanted |
| **Shallow Trench Isolation (STI)** | Trench ~0.5 µm deep, filled with oxide/dielectric | Increases device density, reduces mechanical stress and encroachment |
| **Deep Trench Isolation** | Trench 6–13 µm deep | Same purpose as STI but for deeper isolation needs |

```
Salicide illustration (cross-section):

        Gate
         │
   ▓▓▓▓▓▓▓▓▓▓▓▓▓        <- "poly-side" (silicide on poly)
   ─────┴─────┴─────
   ▓▓▓         ▓▓▓      <- "salicide" (silicide on source/drain diffusion)
  ┌───┐       ┌───┐
  │ S │  Ch.  │ D │
  └───┴───────┴───┘
```

### Active vs. Passive Devices

- **Active device**: requires an external energy source to operate; output is a function of input; typically ≥3 terminals, 2 defined ports (a port = 2 terminals, sometimes sharing a common terminal between input/output).
  - Examples: MOSFET, BJT, vacuum tube, SCR (silicon-controlled rectifier).
  - Rare exception: a two-terminal *negative resistor* can be considered active.
- **Passive device**: no energy source required; current is a function of voltage (or vice versa), described by an I–V curve.
  - Examples: resistor, capacitor, inductor, diode.

#### Signal-Flow Rules for Transistor Terminals

| Terminal | MOSFET | BJT | Can be Input? | Can be Output? |
|---|---|---|---|---|
| Gate / Base | Gate | Base | ✅ Only input | ❌ Never output |
| Drain / Collector | Drain | Collector | ❌ Never input | ✅ Only output |
| Source / Emitter | Source | Emitter | ✅ | ✅ (either) |

> [!TIP]
> Memorizing this table lets you trace signal flow through any unfamiliar schematic instantly: gate/base = input only, drain/collector = output only, source/emitter = flexible. Also note the **polarity**: gate→drain (or base→collector) is *inverting*; all other paths (source→drain, emitter→collector, base→emitter, gate→source) are *non-inverting*.

### Reliability

> [!NOTE]
> **Reliability criterion (analog):** no parameter may shift more than **10% in 10 years** under worst-case bias, assuming a **1% duty cycle** (circuit "on" only 1% of the time). The equivalent digital criterion is roughly a **tenth of a year** — because digital circuits spend virtually all their time fully "on" or fully "off," avoiding the intermediate bias regions where most degradation mechanisms are worst.

#### Key Reliability Mechanisms
- **HCI** — Hot Carrier Injection
- **NBTI** — Negative Bias Temperature Instability
- **Gate Oxide Integrity** (breakdown)
- **Off-State Drain Stress / TDDB** — Time-Dependent Dielectric Breakdown
- **Electromigration** — conductor (metal interconnect) degradation under high current density

#### Avalanche Multiplication (Impact Ionization) — Mechanism
```
     Gate ─────────┐
                    │  (Channel current flowing L→R, saturation region)
   Source ══════════▓▓▓▓▓▓▓▓══════ Drain (large V_DS)
                    │        ╲
                    │  High-field depletion region (channel–drain)
                    │        ╲
                electron  →●  hits lattice atom → knocks off e⁻/hole pair
                             ╲              ↘ electron → toward +V (gate or drain)
                              ╲              ↘ hole → toward substrate (I_sub)
```
- A high electric field across the channel-to-drain depletion region accelerates carriers until they gain enough energy to **impact-ionize** lattice atoms, generating electron-hole pairs (avalanche multiplication).
- Roughly **4× more electrons than holes** are generated in this process (per the lecture).
- Electrons are drawn toward positive potentials (drain, or gate if energetic enough → gate current / oxide trapping); holes are drawn toward negative potentials (substrate) → **substrate current** $I_{sub}$.
- Electrons injected into the gate oxide create **oxide traps** or **interface states**, concentrated near the gate-drain overlap (the highest-field region) — this is the physical root of **Hot Carrier Injection (HCI)** degradation.

### Lecture 2 Summary
- Technology evolves and directly shapes achievable circuit performance.
- Basic process steps: oxidation, diffusion, implantation, deposition, etching, epitaxy.
- Photolithography defines feature geometry, and its imperfections directly cause mismatch.
- Advanced steps: planarization, CMP, salicidation, shallow/deep trench isolation.
- Active devices need energy and have input-controlled output; passive devices don't.
- Reliability requirements for analog (10%/10yr @ 1% duty) are far stricter than digital (10%/0.1yr), driven by mechanisms like HCI, NBTI, TDDB, and electromigration.

---

## Lecture 3 — Modeling

### Why Models?
- **Models**: the means (mathematical equations, circuit representations, graphs, tables) by which the electrical properties of a circuit/system are represented.
- **Purpose**: predict/verify performance; a key element of the design & simulation flow.
- **Golden rule**: know what *is* and *is not* modeled — especially for simulation models, whose accuracy is bounded to specific bias/geometry ranges.

### Two Types of Models

| Aspect | Thinking Model | Simulation Model |
|---|---|---|
| Complexity | Simple | Complex |
| Accuracy | Not critical — simplicity > accuracy | Must be highly accurate |
| Purpose | Understand *how* the circuit works | Verify & explore/optimize performance |
| Tooling | Hand analysis (paper/pencil/calculator) | Computer (SPICE-class simulator) |

```
Flow:
Technology & Usage → [Thinking Model] → Answers "what do I change to get effect X?"
                                              │
                                              ▼
                                   [Computer Simulation] → refine & optimize
                                              │
                        ┌─────────────────────┘
                        ▼
           Compare simulation vs. thinking-model prediction
                        │
                        ▼
        Disagreement? → extract parameters / re-check model validity
```

### MOSFET "Thinking" (Hand) Models

#### 1. Simple Square-Law Model (Active/Saturation Region)
$$
I_D = \frac{K'W}{2L}\left(V_{GS}-V_{T}\right)^2\left(1+\lambda V_{DS}\right)
$$

| Symbol | Meaning |
|---|---|
| $I_D$ | Drain current |
| $K' = \mu C_{ox}$ | Process transconductance parameter (mobility × oxide capacitance per area); sometimes just written $\mu C_{ox}$ |
| $W, L$ | Transistor width and length |
| $V_{GS}$ | Gate-to-source voltage |
| $V_T$ | Threshold voltage |
| $\lambda$ | Channel-length modulation parameter (models finite output resistance) |
| $V_{DS}$ | Drain-to-source voltage |

- Parameters shown in **red** in the original slides ($K'$, $V_T$, $\lambda$) are **model/process parameters** — properties of the technology, obtained either from the foundry or by *extraction* from simulation/measured data.

#### 2. Body-Effect (Threshold Voltage vs. Bulk-Source Voltage)
$$
V_T = V_{T0} + \gamma\left(\sqrt{2\phi_F + V_{SB}} - \sqrt{2\phi_F}\right)
$$

| Symbol | Meaning |
|---|---|
| $V_{T0}$ | Zero-bias (V_SB = 0) threshold voltage |
| $\gamma$ | Bulk-threshold (body-effect) parameter |
| $\phi_F$ | Fermi potential |
| $V_{SB}$ | Source-to-bulk voltage |

#### 3. Velocity-Saturation Model (short-channel devices)
- Adds a parameter $\theta$ (velocity-saturation parameter) alongside $K'$, $V_T$, $\lambda$ to capture the reduction in effective mobility at high lateral fields in short-channel devices.

#### 4. Subthreshold (Weak-Inversion) Model
- Applies when $I_D \lesssim 1\ \mu\text{A}$; conduction is dominated by **diffusion**, not drift — behaves more like a BJT (exponential in $V_{GS}$).

| Symbol | Meaning |
|---|---|
| $I_{sub-t}$ | Constant multiplying the exponential expression (technology-dependent) |
| $n$ | Subthreshold slope factor |
| $V_A$ | Early voltage (≈ reciprocal of $\lambda$) — borrowed terminology from BJT theory |

> [!NOTE]
> **Named concept — Early Voltage.** Named after **James M. Early**, who characterized the finite output resistance of BJTs due to base-width modulation (the "Early effect"). The MOSFET analog — channel-length modulation via $\lambda$ — plays the same conceptual role, and $V_A \approx 1/\lambda$ links the two device families' output-resistance behavior.

### Simulation Models — BSIM Family
- **BSIM3v3** (BSIM = Berkeley Short-channel IGFET Model) and **BSIM4** are industry-standard compact models.
- Simulators compute:
  - $I_D$ as a function of $V_{DS}, V_{GS}, V_{SB}$ (and bulk voltage).
  - All terminal **capacitances**: $C_{gs}, C_{gd}, C_{gb}, C_{sb}, C_{db}$.
  - **Temperature dependence** typically modeled from about $-50\,^{\circ}\text{C}$ to $150\,^{\circ}\text{C}$.
  - **Geometric scalability** across a verified $W$/$L$ range.
  - **Noise**: flicker ($1/f$) noise and thermal noise.
  - **Statistical variation**: Monte Carlo and DC-mismatch models.
  - **Process corners**: typically a **5-corner model** (see below) aligned to fab variation.
- BSIM3v3 is **faster** but less accurate than BSIM4 in some regimes (e.g., substrate-current "rebound" effects).
- PMOS uses the *same* model equations as NMOS, but with different (device-specific) parameter values.

### Process/Corner Modeling
```
              PMOS Threshold (V_Tp)
                    slow          fast
              ┌───────────┬───────────┐
       slow   │  SLOW-SLOW│ SLOW-FAST │
NMOS          │  (SS)     │  (SF)     │
Threshold     ├───────────┼───────────┤
       fast   │  FAST-SLOW│ FAST-FAST │
              │  (FS)     │  (FF)     │
              └───────────┴───────────┘
                (TYP = center of box, not shown)
```
- **3-corner box**: fast/slow along one axis for both device types together (fast-fast, slow-slow, typical).
- **5-corner model**: adds the two cross corners (fast-slow, slow-fast), capturing independent variation between NMOS and PMOS speed.
- Simulating across these corners lets the designer bound worst-case performance.

### Model Maturity vs. Technology Maturity

| Technology Stage | Model State |
|---|---|
| Prior to silicon | Based on previous-generation technology (least accurate) |
| Early silicon | Rapidly changing process → rapidly changing/unreliable models |
| Process-of-record silicon | Test structures + initial data available |
| Frozen silicon | Process locked (minor tweaks only); model accuracy improving |
| Manufacturing silicon | Statistical data available |
| Final silicon | Complete data + full statistical distributions (most mature) |

```
Inaccuracy/            Model
Maturity  │             Inaccuracy
    ▲     │            ╲
    │     │  Process     ╲___
    │     │  Maturity  __╱    ╲______
    │     │        ___╱               ╲________________
    │     │  ______╱
    └─────┴──────────────────────────────────────────────► time
      prior   early   proc-of-  frozen  manufacturing  final
      silicon silicon record            silicon        silicon
```
- **Process maturity** rises monotonically toward a plateau (frozen process).
- **Model inaccuracy** starts high and *decreases* over time as more data accumulates — the two curves move in complementary directions.

### Designer Expectations of Simulation Models

| Well-Modeled (trust it) | Poorly-Modeled (be cautious) |
|---|---|
| Threshold voltage $V_T$ | Small-signal output conductance $g_{ds}$ (i.e., $r_{ds}$) |
| Small-signal transconductance $g_m$ | The "knee" of the $I_D$–$V_{DS}$ output characteristic |
| Gate-source capacitance $C_{gs}$ | Gate-drain capacitance $C_{gd}$ |

> [!TIP]
> **Design implication:** place transistors whose behavior relies on *poorly modeled* parameters (like $r_{ds}$) in **non-critical** roles. Example: in a **cascode** configuration, the top transistor's exact $r_{ds}$ matters little because the *current* through it is set by the bottom (current-source) transistor — so a less-accurate top-device model does not compromise the design.

### Lecture 3 Summary
- Models predict/verify circuit performance; know what is/isn't captured.
- Thinking models: simple, not necessarily accurate, for understanding.
- Simulation models: complex, accurate, for verification/optimization (e.g., BSIM3v3/BSIM4).
- Models mature in lockstep with process maturity (prior-silicon → final silicon).
- Designers must know which parameters are trustworthy ($V_T$, $g_m$, $C_{gs}$) vs. untrustworthy ($r_{ds}$, output "knee," $C_{gd}$), and design around weak models' limitations.

---

## Lecture 4 — The Analog IC Design Process

### Generic IC Design Process Flow
```
Conception → Definition (Specs) → Electrical Design → Simulation ──┐
                                                                     │ (compare to spec)
                    ┌────────────────────────────────────────────────┘
                    ▼
      Physical Design (Layout) → Physical Verification (DRC/LVS)
                    │
                    ▼
        Parasitic Extraction → Re-simulate (with parasitics)
                    │
                    ▼
             Tapeout / Fabrication
                    │
                    ▼
        Testing & Verification → Manufacturing / Product
```

### Inputs & Outputs of Analog Design

| Stage | Inputs | Outputs |
|---|---|---|
| **Electrical Design** | Static/dynamic/physical performance specs; process/technology, power supply, temperature constraints | Schematic, DC currents, component values/ratings |
| **Physical Design (Layout)** | Electrical design outputs | Component geometry/sizes, connections, biasing/buses |
| **Test** | Design outputs + specs | Stimuli, probes, loading, verification procedures |

### Electrical Design — Detailed Steps
1. **Selection** of a candidate solution — from prior designs, literature, or patents; prefer *simple* topologies with potential to meet spec.
2. **Investigation** — hand-analyze the chosen circuit; understand strengths/weaknesses.
3. **Modification** — iteratively improve the topology/sizing to address weaknesses; re-evaluate via analysis.
4. **Verification** — simulate with accurate (technology) models; investigate any large disagreement between hand analysis and simulation; sweep temperature/process corners if statistical models unavailable.

> Electrical-design outputs: **W/L ratios** (MOSFET), **emitter areas** (BJT), **circuit topology**, **passive component values**, and **DC bias currents**.

### Physical Design (Layout) — Detailed Steps
1. **Inputs**: W/L values, schematic (from schematic-entry/simulation tool).
2. **Layout entry**: designer specifies location, shape, and mask level of each geometry.
3. **Design Rule Check (DRC)**: layout must obey foundry design rules (ensures manufacturability/robustness — *not* electrical correctness).
4. **Layout Versus Schematic (LVS)**: verifies the physical layout is electrically equivalent to the intended schematic.
5. **Parasitic Extraction**: extract capacitance-to-ground, inter-conductor capacitance, and bulk resistance.
6. **Re-simulation**: insert extracted parasitics back into the simulation database and re-verify against spec.

```
Example: Push-Pull Inverter
       VDD
        │
      ──┴── PMOS (gate=Vin, drain=Vout)
Vin ──┤
      ──┬── NMOS (gate=Vin, drain=Vout)
        │
       GND
```
- Schematic (above) → Layout (rectangles across multiple mask levels representing diffusion, poly, metal, contacts/vias) → 3D fabricated structure.
- **Parasitics revealed by layout**: bulk resistance, stray capacitance, self- and mutual inductance.

### Test Design
- **Objective**: compare fabricated performance against both specification and simulation.
- **Test types**:
  - **Functional** — verify nominal specs are met.
  - **Parametric** — verify characteristics to within tolerance.
    - **Static** — DC/AC characteristic verification.
    - **Dynamic** — transient characteristic verification.
- **Considerations**: wafer-level vs. package-level test; how to de-embed the measurement system's own influence from the measured result.

### Packaging
- **Functions of a package**:
  1. Protect the die.
  2. Power it (deliver supply voltages/currents).
  3. Cool it (manage thermal/electrical properties).
  4. Provide electrical & mechanical connection to the outside world.
- **DIP (Dual In-line Package) assembly steps**: die attach → wire bonding (pads → lead frame) → plastic molding (encapsulation) → cleanup → lead trim/finish → marking.
- **Package-level considerations**: added parasitic inductance and capacitance affect high-speed performance — should ideally be considered *early*, not as an afterthought.

> [!TIP]
> Packaging is listed last in the lecture sequence purely for presentation order — in real projects, package parasitics (bond-wire inductance, lead-frame capacitance) should be budgeted **early**, since they can dominate high-frequency or high-current performance.

### Lecture 4 Summary
- Process flow: electrical design → physical design (layout) → fabrication → test.
- Electrical design inputs = specs; outputs = topology, W/L, emitter areas, currents, passive values.
- Electrical design steps: selection → investigation → modification → verification.
- Physical design = translating schematic into geometric layers, followed by DRC, LVS, extraction, and re-simulation.
- Test design compares specs vs. actual performance; packaging affects performance and must be planned for.

---

## Lecture 5 — Key Principles, Concepts & Techniques

### What Is Innovation?
- **Innovation** = the ability to apply principles, concepts, and techniques to derive a *unique* solution to a given specification/problem.
- Characteristics: organized thinking (grasping cause and effect, avoiding "magical thinking"), experience, and intuition.

### Principles — "Fundamental laws that are precise and never change"
Examples cited:
- **Kirchhoff's Voltage Law (KVL)** and **Kirchhoff's Current Law (KCL)**
- **Thevenin/Norton equivalence**
- **Ohm's Law**
- **Superposition**
- **Linearity**, **time invariance**
- $Q = CV \Rightarrow i = C\dfrac{dv}{dt}$ (charge–voltage relation, differentiated to get capacitor current)
- $\Phi = Li \Rightarrow v = L\dfrac{di}{dt}$ (flux–current relation, differentiated to get inductor voltage)
- **Bode criterion** and **root-locus criterion** (stability)
- **Fourier analysis**, **Nyquist relationships**, **double-correlated sampling**, **Z-domain relationships**, **Laplace transforms**

#### Worked Principle Example 1 — Thévenin Equivalent + Superposition (Charge-Redistribution DAC)
- A charge-scaled capacitor-array DAC (used later in Lecture 11) can be analyzed by replacing each sub-network with its **Thévenin equivalent**, then applying **superposition**: short one source, compute the contribution at the output node; repeat for each source; sum the results.

#### Worked Principle Example 2 — Nyquist Frequency / Sampling
$$
f_{Nyquist} = 2 f_B
$$
where $f_B$ is the signal bandwidth. For faithful reconstruction of a sampled signal, the sampling frequency must satisfy $f_S \geq f_{Nyquist}$.

> [!NOTE]
> **Named concept — Nyquist Criterion**, after **Harry Nyquist**, whose work (later formalized in the Nyquist–Shannon sampling theorem) established the minimum sampling rate needed to avoid aliasing.

### Concepts — "Relationships / soft laws, generally true, worth remembering"
Examples cited: mOSFET modeling itself, poles/zeros and how to find them "by inspection," root locus, current matching, Cascode input/output resistance estimation, dynamic range, feedback (positive/negative) effects and identifying loops, feedforward, "**component accuracy is inversely proportional to size**," accuracy-enhancement techniques (averaging, interpolation, dynamic element matching), Bode plots/frequency response.

#### Worked Concept Example 1 — Current Matching in MOSFETs
For two matched transistors $M_1, M_2$ with gate, source, and drain terminals held at the same respective potentials (and matched physical layout):
$$
\frac{I_{D1}}{I_{D2}} = \frac{(W/L)_1}{(W/L)_2}
$$
- Requires both **electrical** condition (equal $V_{GS}$, equal $V_{DS}$) and **physical matching** (careful layout — common-centroid, same orientation, etc.) to hold in practice.

#### Worked Concept Example 2 — Feedback's Effect on Port Resistance

| Feedback Type | Resulting Resistance |
|---|---|
| Negative series feedback | $R(1+LG)$ — increased |
| Negative shunt feedback | $R/(1+LG)$ — decreased |
| Positive series feedback | $R(1-LG)$ — decreased (or negative if $LG>1$) |
| Positive shunt feedback | $R/(1-LG)$ — increased (as $LG\to1^-$) |

where $LG$ = loop gain. (Example: $LG=100 \Rightarrow$ negative-series resistance increases ×101.)

### Techniques — "Assumptions, tricks, tools, methods to simplify/understand"
Examples cited:
- Decomposing a differential voltage: $V_{AB} = V_A - V_B$.
- Assuming one pole is much closer to the origin than another (dominant-pole approximation) in a 2-real-root system.
- Estimating RC transient response via $v(t) = A + Be^{-t/RC}$, solving $A,B$ from initial/final conditions.
- MOSFET rule-of-thumb ratios: $g_{m}(\text{top gate}) \approx 10\times g_{mbs}(\text{bulk-source transconductance})$; and $g_m \approx 100\times g_{ds}$ (reciprocal of small-signal drain-source resistance/conductance) — rough sizing heuristics, not universal constants.
- **Source degeneration**: quickly writing the effective transconductance of a transistor with a source resistor.
- **Zero-temperature-coefficient (ZTC) MOSFET biasing** (detailed in Lecture 7).
- **Worst-case analysis** methods.

#### Worked Technique Example — Input-Offset Voltage of a Differential Pair (Worst-Case Analysis)
Starting point (square-law model, matched currents $I_D$, matched $W/L$):
$$
V_{OS} = V_{T1}-V_{T2} + (V_{GS1}-V_{T1}) - (V_{GS2}-V_{T2})
$$
Rewriting using the SAH (Sah) equation and factoring:
$$
V_{OS} = \Delta V_T + \sqrt{\frac{2 I_D}{K'(W/L)}}\left(\frac{1}{\sqrt{K_1}}-\frac{1}{\sqrt{K_2}}\right)
$$
Define **worst-case average/difference** pairs:
$$
K_{avg}=\frac{K_1+K_2}{2}, \quad \Delta K = K_1-K_2 \;\Rightarrow\; K_{1,2}=K_{avg}\pm\tfrac12\Delta K
$$
$$
V_{T,avg}=\frac{V_{T1}+V_{T2}}{2}, \quad \Delta V_T = V_{T1}-V_{T2}
$$
Substituting and applying the small-perturbation approximations
$$
\frac{1}{\sqrt{1\pm x}} \approx 1 \mp \frac{x}{2}, \qquad \sqrt{X}\approx \dots
$$
yields the final worst-case result:
$$
\boxed{V_{OS} = \Delta V_T - (V_{GS}-V_T)\cdot\frac{\Delta K}{2K}}
$$

| Symbol | Meaning |
|---|---|
| $V_{OS}$ | Input-referred offset voltage of the differential pair |
| $\Delta V_T$ | Threshold-voltage mismatch between $M_1, M_2$ |
| $(V_{GS}-V_T)$ | Overdrive voltage (common to both devices under the matched-current assumption) |
| $\Delta K/2K$ | Relative (fractional) mismatch in the transconductance parameter $K=K'(W/L)$ |

> [!TIP]
> This derivation is the canonical template for **all** MOSFET mismatch analyses: express the offset as (absolute $V_T$ mismatch) + (overdrive) × (relative $K$/gain-factor mismatch). It reappears throughout comparator, op-amp, and DAC mismatch/noise budgeting.

### Lecture 5 Summary
- Innovation = applying principles + concepts + techniques to synthesize a unique solution.
- Principles = precise, unchanging fundamental laws (KVL/KCL, Ohm's law, superposition, Laplace, etc.).
- Concepts = generally-true soft relationships/analytical tools (matching, feedback resistance scaling, poles/zeros).
- Techniques = assumptions/tricks/methods (decomposition, dominant-pole approximation, worst-case analysis).

---

## Lecture 6 — Characteristics of a Successful Analog Designer

### The Analog IC Design Environment
- Requires understanding **technology** (how much does a given process feature affect analog performance?), **models** (thinking vs. simulation), and the ability to manage **complexity via hierarchy**:
```
Components  →  Circuits  →  Systems
 (low level)              (high level, high complexity)
```
- Must also appreciate the influence of **packaging** and **testing** on the overall design.

### Required Skills
- Fundamentals: principles, concepts, techniques (Lecture 5).
- **Making assumptions**: simplify without removing the essential problem; use the simplification purposefully; **check the assumption's validity** afterward.
- **Troubleshooting/debugging** both simulation and measurement results.
- Knowing **when (and when not) to use the simulator** — tongue-in-cheek: *"simulator use × common sense = constant"* (heavy reliance on simulation tends to correlate with less physical intuition, and vice versa as designers mature).
- Avoiding **magical thinking** (reasoning not grounded in physical reality); learning from failure.

### Troubleshooting & Debugging
- **Definition**: examining incorrect circuit behavior and identifying its cause, leading to a solution — a form of **systematic problem-solving**.
- Analogy: like detective work — observe symptoms/clues → form a **hypothesis** → **test** the hypothesis.

### Designing for Failure (Post-Silicon Fixability)
- Since mistakes *will* happen, include **extra passive and active components** (various W/L values for NMOS/PMOS, various R/C values) in unused die area — "fill the vacant space."
- After finding a bug, use a **metal-mask-only respin** (cheaper than a full mask set) to reroute connections: disconnect bad components, connect spare good ones.
```
Spare-component array (conceptual):
  [NMOS W/L=1] [NMOS W/L=2] [NMOS W/L=4] ...
  [PMOS W/L=1] [PMOS W/L=2] [PMOS W/L=4] ...
  [R=1k] [R=2k] [R=5k] ...
  [C=1p] [C=2p] [C=5p] ...
   (metal routing re-patched via new metal mask only)
```

### Learning From Mistakes
- Failure = an **opportunity to learn**: (1) understand what failed and why, (2) build understanding to avoid repeating it, (3) build character.
- *"What happens to you is not nearly as important as how you respond to what happened."*
- **Under pressure**, avoid: randomly trying alternative "fixes" without understanding, avoiding the problem, or refusing to acknowledge it. If a problem gets fixed without understanding why, **analyze it later** when time permits.

### Frequency-Response Knowledge
- General transfer function:
$$
H(j\omega) = \frac{a_0 + a_1(j\omega) + a_2(j\omega)^2 + \dots + a_n(j\omega)^n}{b_0 + b_1(j\omega) + b_2(j\omega)^2 + \dots + b_m(j\omega)^m}, \quad m \ge n
$$
- Characterized via **magnitude** (Bode magnitude plot) and **phase** (argument) vs. $\omega$ — determines bandwidth, delay, and related behavior.
- Understanding how **poles and zeros** shape both curves is essential.

### Circuit-Analysis Knowledge
Systematic techniques every analog designer must be fluent in:
- Mesh & nodal analysis
- Superposition
- Source substitution/transformation
- Network reduction
- **Miller simplification**
- Thévenin's & Norton's equivalents
- Independent vs. dependent sources
- Differential-mode / common-mode decomposition

> [!NOTE]
> **Named concept — Miller Effect / Miller Simplification**, after **John Milton Miller**, who described how a feedback capacitor between the input and (inverted, amplified) output of a gain stage appears, from the input's perspective, as a much larger effective capacitance $C_{Miller} \approx C(1+|A_v|)$. This is central to compensation techniques used in two-stage op amps (Lecture 9).

#### Worked Circuit-Reduction Example
- A dependent current source controlled by a branch current can be "moved," combined with a parallel resistor, and reduced: if a controlled current $i$ flows into a resistor $R$ alongside another source current, the combined effect can often be expressed as $R(1+i)$ scaling — simplifying the network topology for input–output analysis (see lecture's simple worked reduction).

### Lecture 6 Summary
- Analog design demands managing technology, models, and hierarchy simultaneously.
- Core skills: fundamentals, disciplined assumption-making, troubleshooting/debugging, judicious simulator use, avoiding magical thinking.
- Design-for-failure: leave spare components on-die; fix via metal-only respins.
- Treat failure as a learning opportunity; avoid panic-driven "random fix" behavior under pressure.
- Deep fluency in frequency response (poles/zeros, Bode) and classical circuit-analysis techniques (mesh/nodal, superposition, Miller, Thévenin/Norton) is mandatory.

---

## Lecture 7 — Independent Sources

### Definition
- An **independent voltage source** maintains a constant output voltage $V_0$ regardless of the current through it.
- An **independent current source** maintains a constant output current $I_0$ regardless of the voltage across it.
- **Ideal goal — PVT independence**: the source should be independent of **P**rocess, supply **V**oltage, and **T**emperature.

```
Ideal current source I-V:        Ideal voltage source I-V:
   I                                  V
   │                                  │
I₀ ┤─────────────                  V₀ ┤─────────────
   │                                  │
   └─────────────► V                 └─────────────► I
```

### Power-Supply Independence

#### Approach 1 — Circuit (Feedback) Approach
```
        VDD
         │
        ┌┴┐              ┌────[ Op Amp ]────┐
        │ │◄── (neg. FB path) │            │ ◄── (pos. FB path)
        └┬┘                  │            │
         │                V1 ●            ● V2
   I1 ───┤                   (kept equal by op-amp feedback)
         │                I1 = I2 (also kept equal)
        GND
```
- Op amp with **both** negative and positive feedback forces $V_1 = V_2$ and $I_1 = I_2$.
- Plotting $I_2$ vs. $I_1$: one branch gives a straight line through the origin ($V_2 = I_2 R$); the other gives a nonlinear relation ($V_1$ as a function of $I_1$, e.g. from a diode-connected device).
- **Two intersection (operating) points** exist:
  - The **zero-current point** (undesired, degenerate) — requires a **start-up circuit** to avoid getting stuck there.
  - The **desired operating point** — where current becomes essentially independent of $V_{DD}$.

#### Approach 2 — Inherent (Diode Breakdown / Load-Line) Approach
```
   I
   │                    ╱  (steep reverse-breakdown branch of diode)
   │                  ╱
   │                ╱●────── operating point (V_ref)
   │       load line (slope = -1/R, from V_DD)
   │  ╲___________╱
   └──────────────────────► V
       V_ref          V_DD
```
- Uses a diode's steep reverse-breakdown I–V curve intersected with a resistive load line from $V_{DD}$.
- Because the breakdown branch is so **steep**, changes in $V_{DD}$ (shifting the load line up/down — shown as light-gray lines) cause only a **small** shift in $V_{ref}$ at the intersection point.

### Process Independence

#### MOSFET Threshold Voltage & Process Dependence
$$
V_{T0} = V_{FB} + 2\phi_F + \gamma\sqrt{2\phi_F} \quad (\text{illustrative form}; V_{SB}=0)
$$
- $\phi_F$ (Fermi potential) depends on substrate doping concentration.
- $C_{ox} \propto 1/t_{ox}$ (oxide thickness) — thinner oxide → larger $C_{ox}$.
- In practice, **threshold implants** further shift $V_{T0}$, adding process dependence that is harder to characterize simply.
- **Transconductance parameter**: $K' = \mu C_{ox}$, and since $C_{ox}\propto 1/t_{ox}$, $K'$ is process (oxide-thickness) dependent too.

#### Corner Modeling (Process Variation)
- See Lecture 3's corner-box diagram — 3-corner (fast/slow/typical) or 5-corner (adding FS, SF) models used to bound worst-case process spread in simulation.

### $V_{PTAT}$ — Proportional-to-Absolute-Temperature Reference (Process- & Supply-Independent)

- Based on **two diodes of unequal area** $A_1 \neq A_2$ carrying currents $I_1, I_2$:
$$
\Delta V_D = V_{D1}-V_{D2} = \frac{kT}{q}\ln\!\left(\frac{I_1/A_1}{I_2/A_2}\right)
$$
- If $I_1 = I_2$, this reduces to (with $A_2 > A_1$ chosen, ratio $n = A_2/A_1$):
$$
V_{PTAT} = \Delta V_D = \frac{kT}{q}\ln(n)
$$

| Symbol | Meaning |
|---|---|
| $k$ | Boltzmann's constant |
| $T$ | Absolute temperature (Kelvin) |
| $q$ | Elementary charge |
| $A_1, A_2$ | Diode (junction) areas |
| $n$ | Area ratio $A_2/A_1$ |
| $V_T^{th}=kT/q$ | **Thermal voltage** (~26 mV at 300 K) — *note: distinct from MOSFET threshold voltage $V_T$, same symbol overload* |

- Crucially, $\Delta V_D$ depends **only** on the area ratio $n$ (a geometric, not process, quantity) and temperature — hence independent of absolute process variation and of supply voltage.

```
Practical PTAT circuit:

         VDD
          │
        ┌─┴─┐     ┌─┴─┐
        │ M │     │ M │      (current mirror, or R1/R2 network)
        └─┬─┘     └─┬─┘
          │          │
          ●──────────●───── Op Amp forces these two nodes equal
          │          │       (without direct connection)
        ┌─┴─┐      ┌─┴─┐
        │ R1│      │ D2│ (area = n × D1)
        └─┬─┘      └─┬─┘
          │           │
        ┌─┴─┐         │
        │ D1│         │
        └─┬─┘         │
          │            │
         GND          GND
```
- The op amp forces equal voltages at its two inputs without a direct wire connection, forcing the PTAT voltage $\Delta V_D$ across $R_1$.
- The resulting PTAT current can be mirrored into a second resistor $R_2$ to rescale the reference voltage by $R_2/R_1$.

### Temperature Coefficient (TC)
$$
TC = \frac{V_{max}-V_{min}}{V_{nominal}\,(T_{max}-T_{min})}\times 10^6 \quad \left[\frac{\text{ppm}}{^\circ\text{C}}\right]
$$

| Symbol | Meaning |
|---|---|
| $V_{max}, V_{min}$ | Maximum/minimum reference voltage over the temperature sweep |
| $V_{nominal}$ | Nominal (design-center) reference voltage |
| $T_{max}, T_{min}$ | Temperature range endpoints |

> [!NOTE]
> A "good" stable voltage reference typically targets **TC ≤ 10 ppm/°C**.

### Temperature-Stable References — The Bandgap Principle
$$
V_{REF} = V_{PTAT}\cdot K + V_{CTAT}
$$

| Symbol | Meaning |
|---|---|
| $V_{PTAT}$ | Voltage **P**roportional **T**o **A**bsolute **T**emperature (positive slope) |
| $V_{CTAT}$ | Voltage **C**omplementary **T**o **A**bsolute **T**emperature (negative slope; e.g., a $V_{BE}$ or diode drop) |
| $K$ | Temperature-independent scaling constant chosen to make the slopes cancel |

```
  V
  │      \______________     <- V_CTAT (negative slope)
  │                       \
  │      __________________  <- ideal flat V_REF (slopes cancel exactly)
  │     /
  │    /                     <- K·V_PTAT (positive slope)
  └──────────────────────────► T
```

> [!NOTE]
> **Named concept — Bandgap Reference.** So named because the resulting reference voltage numerically comes out close to silicon's **bandgap voltage** (~1.12 eV → ~1.2 V), even though — as the lecturer notes — the circuit's *operating principle* has nothing physically to do with the bandgap energy itself; it's a historical/coincidental naming.

#### Bandgap Curvature Problem
- A **true** CTAT voltage is not perfectly linear in $T$; it has a slight curvature.
- Even with $K$ chosen to cancel the *linear* slopes at one temperature, residual curvature causes $V_{REF}$ to bow away from flat at other temperatures — the **bandgap curvature effect**.
- Without correction, typical achievable TC is roughly **10–50 ppm/°C**; correcting curvature (higher-order compensation) is needed to do better.
```
  V
  │      _____
  │     /     \_____        <- actual (curved) V_REF vs T
  │    /
  └──────────────────────────► T
        (dip/bow = curvature error)
```

### Zero-Temperature-Coefficient (ZTC) Point — For Current References
- Plotting $I_D$ vs. $V_{GS}$ for a MOSFET at multiple temperatures (e.g., $-50\,^\circ\text{C}$ to $150\,^\circ\text{C}$), all curves cross at a **single point** — the **ZTC point** (in the lecture's example, roughly $I_D \approx 3\ \mu\text{A}$, $V_{GS}\approx 1\ \text{V}$).
```
  I_D
   │      T=-50°C  T=150°C
   │        ╲        ╱
   │         ╲      ╱
   │          ╲    ╱
   │           ╲  ╱
   │            ●●  <-- ZTC crossing point (all T-curves intersect here)
   │           ╱  ╲
   └──────────────────────► V_GS
```
- Biasing a MOSFET exactly at this $(I_D, V_{GS})$ point makes drain current **inherently temperature-stable**, without relying on the bandgap-curvature-limited voltage cancellation trick.
- **Design procedure**: generate a PTAT current, pass it through a resistor $R_3$ to create a voltage; size $R_3$ so this voltage equals the ZTC gate-source voltage — then the resulting drain current through the biased MOSFET is temperature-independent.
- **Caveats**: (1) the ZTC point itself may drift slightly with temperature in some processes, limiting the usable temperature range; (2) if the underlying reference voltage suffers bandgap curvature, the derived current will **inherit** that curvature error.

### Lecture 7 Summary
- Independent sources ideally reject process, supply-voltage, and temperature dependence (PVT-independent).
- Power-supply independence: circuit (feedback) approach or inherent (steep-diode load-line) approach — both need care re: undesired zero-current operating points / start-up circuits.
- Process independence exploits $\Delta V_{BE}$ (or diode) ratios that depend only on geometric area ratio, not process parameters.
- $V_{PTAT} = (kT/q)\ln(n)$ is fundamentally process- and supply-independent.
- Bandgap references cancel $V_{PTAT}$ against $V_{CTAT}$; curvature is the residual, higher-order error source.
- Current references can achieve temperature stability via the MOSFET's ZTC bias point, but are still susceptible to inherited bandgap curvature and possible ZTC-point drift.

---

## Lecture 8 — Amplifiers

### Three Grounded-Terminal Amplifier Types

```
Common Source (CS)         Common Gate (CG)          Common Drain (CD) / Source Follower
      VDD                        VDD                        VDD
       │                          │                           │
     [Load]                    [Load]                      (Gate=Vin)
       │                          │                          ┌┴┐
Vin──►[Gate]                    Vin──►[Source]                │M│
       │M                         │M                          └┬┘
      [Drain]──►Vout           [Drain]──►Vout                  │ ──►Vout (at Source)
       │                          │(Gate=AC gnd)                │
      GND                       GND                          [Load]
                                                                 │
                                                                GND
```

| Config | Input | Output | Polarity |
|---|---|---|---|
| Common Source / Common Emitter | Gate/Base | Drain/Collector | **Inverting** |
| Common Gate / Common Base | Source/Emitter | Drain/Collector | Non-inverting |
| Common Drain / Common Collector (follower) | Gate/Base | Source/Emitter | Non-inverting |

> Only the **common source/emitter** configuration inverts; the other two are non-inverting. This matches the gate→drain (base→collector) inversion rule from Lecture 2.

### Load Options (CS or CG stage)
- Simple resistor
- Diode-connected MOSFET (gate tied to drain)
- Current-source load (biased for constant current) — highest resistance among simple options
- **Cascode load** (two stacked transistors) — highest output resistance of all, by "bootstrapping" the output impedance

### Push-Pull (CMOS) Inverting Amplifier
```
        VDD
         │
       ──┴── PMOS  (gate = Vin)
Vin ──┤
       ──┬── NMOS  (gate = Vin)
         │
        GND
                    Vout taken between drains
```
- Voltage-transfer curve: input low → output high; input high → output low.
- **Digital logic** operates at the extremes (near rails); **analog** amplification occurs in the steep transition region between them.
- The **slope** of the VTC in that region corresponds to the small-signal **voltage gain**.
- One of the best CMOS/BJT topologies for **sourcing and sinking output current** — the basis of Class-AB output stages.

### Differential Amplifier

```
Symbol:
        V1 ──►┐
               │ Diff Amp │──► Vout = Avd·Vid ± Avc·Vic
        V2 ──►┘
```

$$
V_{id} = V_1 - V_2 \qquad V_{ic} = \frac{V_1+V_2}{2}
$$
$$
V_o = A_{vd}V_{id} \pm A_{vc}V_{ic} \;=\; A_{vd}(V_1-V_2) \pm A_{vc}\left(\frac{V_1+V_2}{2}\right)
$$

| Symbol | Meaning |
|---|---|
| $V_{id}$ | Differential-mode input voltage |
| $V_{ic}$ | Common-mode input voltage |
| $A_{vd}$ | Differential-mode voltage gain (want **large**) |
| $A_{vc}$ | Common-mode voltage gain (want **small**; unpredictable sign due to mismatch/interference, hence "$\pm$") |

**Common-Mode Rejection Ratio (CMRR)**:
$$
CMRR = \left|\frac{A_{vd}}{A_{vc}}\right|
$$
- Want $CMRR$ as **large** as possible.

#### Practical Differential-Amplifier Topologies (NMOS input pair examples)
```
1) Resistor-loaded diff pair:

     VDD          VDD
      │            │
     [R]          [R]
      │            │
Vout1●┤            ├●Vout2
      │M1        M2 │
Vin1─►┤            ├─◄Vin2
      └─────┬──────┘
            │
          [I_tail]  (constant current source)
            │
           GND

2) Diode-connected (MOS-diode) load, current-source load,
   and current-mirror-load variants replace the resistors above
   with: diode-connected MOSFETs, fixed current sources, or an
   active current mirror (enabling single-ended output recovery,
   at the cost of half the differential gain if taken single-ended
   without the mirror; the mirror recovers full gain single-endedly).
```

#### The "Conflicting-Current" Problem (Current-Source-Load Diff Amp)
- If the tail-current-derived branch currents $I_1, I_2$ don't naturally match the load-defined currents $I_3, I_4$ (i.e., $I_1 \ne I_3$), a conflict arises: the transistor demanding the larger current is forced **out of saturation into the triode/active region** until its current reduces to match the smaller value.
- **Resolution**: requires **common-mode feedback (CMFB)** — covered in more depth in the full CMOS Analog Circuit Design course.

### Slew Rate
$$
SR = \left.\frac{dV}{dt}\right|_{max} = \frac{I}{C_L}
$$

| Symbol | Meaning |
|---|---|
| $SR$ | Slew rate (max rate of change of output voltage) |
| $I$ | Maximum current available to charge/discharge the load capacitor (e.g., tail current $I_{SS}$ or bias current $I_{D}$) |
| $C_L$ | Load capacitance |

🔬 **Physical origin**: slew-rate limiting occurs when one side of a differential pair is driven so hard that **all** the tail current is steered into a single branch — no more current is available to charge the output node faster, so the output ramps at the fixed rate $I/C_L$ regardless of how large the input step is.

### Common-Gate Amplifier — Input Resistance Deep Dive
Looking into the **source** of a common-gate MOSFET, with different drain-side loads:

| Drain-Side Condition | Input Resistance Looking Into Source |
|---|---|
| Drain shorted directly to $V_{DD}$ (no load) | $R_{in} \approx \dfrac{1}{g_m}$ |
| Current-source load | $R_{in} \approx \dfrac{2}{g_m}$ (roughly double — current sharing between transistors) |
| Cascode load | $R_{in} \approx \dfrac{g_{m}r_{ds}}{g_m} \cdot(\text{large factor})$ — becomes large (order $r_{ds}$-scaled) |

- General rule: **input resistance into a common-gate/follower node depends strongly on the impedance connected at its drain** — the same principle applies to BJT common-base stages.

### Current Amplifier (Current Mirror)
```
        VDD                          VDD
         │                            │
   Iin──►●──────┐                   ┌●──────► Iout = K·Iin
         │M1    │  (mirror node)    │M2
        [dc I1] │                   │[dc I2]
         │      │                   │
        GND    GND                 GND
```
- **Desired characteristics**: **low input resistance**, **high output resistance**.
- Simple MOSFET mirror: $R_{in}\approx 1/g_m$ (low — good), $R_{out}\approx r_{ds}$ (moderately high).
- $I_{out}/I_{in}$ ratio set by the $(W/L)$ ratio between mirror transistors (matching-dependent, per Lecture 5's current-matching concept).

### Lecture 8 Summary
- Three single-transistor configurations: common source (inverting), common gate (non-inverting), common drain/follower (non-inverting).
- Loads range from simple resistors to diode-connected, current-source, and cascode loads (increasing output resistance).
- Push-pull CMOS inverter is excellent for sourcing/sinking current; slope of VTC = small-signal gain.
- Differential amplifiers amplify $V_{id}$, reject $V_{ic}$; CMRR quantifies rejection quality.
- Current-source-load diff amps need common-mode feedback to resolve current conflicts.
- Slew rate $= I/C_L$ — a hard current-limited, non-linear large-signal effect.
- Common-gate input resistance scales with the load impedance at the drain.
- Current mirrors: want low $R_{in}$, high $R_{out}$; gain set by $W/L$ ratio.

---

## Lecture 9 — Operational Amplifiers (Op Amps)

### Definition & Requirements
- **Op amp**: a high-gain, DC-coupled differential amplifier designed to be used with **negative feedback** to precisely define a closed-loop transfer function.
- Requirements: sufficiently large differential voltage gain (accuracy-dependent), differential inputs, frequency response permitting **stable** operation under feedback, high input impedance, low output impedance, adequate speed/bandwidth.

### Closed-Loop Transfer Function
```
        ┌──────────────┐
Vin ──►(+)──► [ Op Amp, Av ] ──┬──► Vout
        (−)▲                  │
           └──[ Feedback, F ]──┘
```
$$
\frac{V_{out}}{V_{in}} = \frac{A_v}{1+A_vF}
$$
If $A_vF \gg 1$ (large **loop gain**):
$$
\frac{V_{out}}{V_{in}} \approx \frac{1}{F}
$$

| Symbol | Meaning |
|---|---|
| $A_v$ | Open-loop differential voltage gain of the op amp |
| $F$ | Feedback-network transfer function (feedback factor) |
| $A_vF$ | **Loop gain** |

> [!TIP]
> This is *the* foundational insight of op-amp-based design: if $F$ (built from passive, precisely-matched components) dominates the closed-loop behavior, the imprecise, PVT-sensitive open-loop gain $A_v$ becomes largely irrelevant — **precision is transferred from the active device to the passive feedback network.**

### Stability & Phase Margin
- **Stability criteria** (equivalent forms):
  1. Loop-gain **magnitude** must be **< 1** when the loop **phase shift = 180°**.
  2. Loop **phase shift** must be **≤ 180°** when the loop-gain **magnitude = 1**.
- **Phase margin (PM)**: $PM = 180^\circ - (\text{loop phase shift at unity loop-gain magnitude})$.
  - Example: phase shift of 135° at unity loop gain → $PM = 45°$.

### Settling Time
- **Definition**: time for a **unity-gain-configured** ($F=1$, worst-case configuration) op amp's step response to settle within a specified error tolerance of the ideal final value.
```
 Vout
  │        ___
  │       /   \___          <- 45° PM: more overshoot/ringing, longer settling
  │      /
  │     /   ┌─────────  <- 70° PM: faster, cleaner settle
  │    /   /
  │───/───/────────────────────► t
     step  (error-tolerance band shown as dashed lines around final value = 1)
```
- **Lower phase margin (e.g., 45°) → more ringing/overshoot → longer settling time.**
- **Higher phase margin (e.g., ~70°) → faster, cleaner settling.**
- An op amp can be *stable* yet still have unacceptably long settling time for a given application — settling time, not just stability, must be designed for.

### Op-Amp Architectures

#### 1. Single (Dominant) Pole — e.g., Folded Cascode
$$
A_v(j\omega) = \frac{A_{v0}}{1+j\omega/\omega_{p1}}
$$
```
|Av| (dB)
 │────────────
 │            \
 │             \  -20 dB/decade (6 dB/octave)
 │              \
 │               \___________ 0 dB (GB = gain-bandwidth)
 └──────────────────────────────► ω (log)
        ω_p1
```
#### 2. Two-Pole — Classical Two-Stage Op Amp
$$
A_v(j\omega) = \frac{A_{v0}}{(1+j\omega/\omega_{p1})(1+j\omega/\omega_{p2})}
$$
```
|Av| (dB)
 │──────────
 │          \
 │           \ -20 dB/decade
 │            \___
 │                \  -40 dB/decade (past 2nd pole)
 │                 \____ 0 dB
 └───────────────────────────────► ω (log)
       ω_p1      ω_p2
```

### Op-Amp Dynamics — Unity-Gain Step Response

#### One-Pole Op Amp
- As $t\to\infty$: $V_{out}\to \dfrac{A_0}{1+A_0}$
- **Gain error**:
$$
\varepsilon_{gain} = 1 - \frac{A_0}{1+A_0} = \frac{1}{1+A_0}
$$
- As $A_0\to\infty$ (ideal gain): $V_{out}(t) = \left(1-e^{-t\cdot GB}\right)V_{in}$ (step-response form using gain-bandwidth $GB$).
- **Half-LSB settling time**:
$$
T_S = \frac{(N+1)\ln 2}{GB}
$$

| Symbol | Meaning |
|---|---|
| $A_0$ | Open-loop DC gain |
| $GB$ | Gain-bandwidth product |
| $N$ | Number of bits of accuracy required (e.g., for a data-converter application) |
| $T_S$ | Settling time to within ½ LSB |

#### Two-Pole Op Amp
- Same gain-error expression, $\varepsilon_{gain}=1/(1+A_0)$, applies as $t\to\infty$.
- As $A_0\to\infty$, the step response is an **exponentially-decaying sinusoid settling to 1** (ringing behavior from the complex pole pair), and an analogous (more complex) settling-time formula relates $T_S$, $GB$, phase margin, and $N$.

> [!NOTE]
> **LSB** = Least Significant Bit. The "half-LSB settling" criterion equates to roughly a 3 dB-scale accuracy requirement per added bit of resolution and is the standard metric linking op-amp dynamics to data-converter accuracy (see Lecture 11).

### Compensation

#### Single-Pole Op Amps — Inherently Self-Compensated
- Any capacitor at the output creates the dominant pole; **larger $C_L$ → lower $GB$ but always ~90° phase margin** — excellent, "free" stability.
```
|Av| (dB), increasing C_L:
 │───╲
 │    ╲___ (small C_L: higher GB)
 │        ╲____
 │             ╲___ (larger C_L: lower GB, same ~90° PM)
 └────────────────────► ω (log)
```

#### Two-Pole Op Amps — Miller (Pole-Splitting) Compensation
```
       ┌──────[ Cc ]──────┐
Stage1 ──►●───────────────●──► Stage2 ──► Vout
       (high-Z node)   (output node)
```
- A **Miller capacitor** $C_c$ placed around the second (output) gain stage **splits** the two poles: pushes $p_1$ **lower** in frequency (more dominant) and pushes $p_2$ **higher** (out of the way), giving a clean **-20 dB/decade** roll-off all the way to 0 dB (defining $GB$).
- **Caveat**: Miller compensation introduces a **right-half-plane (RHP) zero**, which *adds* phase lag (unlike an LHP zero) and must be pushed well beyond $GB$ (a common target: RHP zero ≥ 10×GB) or cancelled via a nulling resistor — detailed treatment reserved for the full CMOS course.

### One-Pole Op-Amp Design Procedure (Given Specs)
Given: $A_{v0}$, $GB$, input common-mode range, $C_L$, slew rate, output swing, power dissipation.
1. Use **slew-rate** spec → define the tail/bias current.
2. Use the relationship between tail current and branch currents (to prevent any transistor's current from reaching zero) → define remaining DC currents.
3. Use **max output swing** → size the "top" transistors.
4. Use **min output swing** → size the "bottom" transistors.
5. Use **$GB$** → size the input differential-pair transistors.
6. Use **min input common-mode range** → size remaining bias transistors.
7. *Afterward*, **check** DC gain and power dissipation (not directly used in sizing) — iterate if insufficient/excessive.

### Two-Pole Op-Amp Design Procedure (Given Specs)
Given: $A_{v0}$, $GB$, phase margin, settling time, input CM range, $C_L$, slew rate, output swing, power, and RHP-zero constraint ($\geq 10\times GB$).
1. Relationship between $C_c$ (Miller cap) and $C_L$ for a target phase margin (e.g., **60° PM**) → sizes $C_c$.
2. Slew rate & $C_c$ → first-stage DC current.
3. **Max input CM range** → sizes input-pair transistors.
4. **$GB$** → sizes remaining input-stage transistors.
5. **Min input CM range** → sizes a bias transistor.
6. Relationship between stages → determine $g_{m6}$ (second-stage transconductance) → second-stage DC current → sizes remaining output-stage transistor.
7. *Afterward*, **check** DC gain (not directly used) and power dissipation — iterate if needed.

> [!TIP]
> Both design procedures follow the same meta-pattern: **use each spec exactly once to size a specific set of transistors**, then **verify** the specs that weren't directly used (gain, power) at the end — a systematic, checklist-driven design methodology rather than ad-hoc guessing.

### Lecture 9 Summary
- Op amp = high-gain DC amplifier meant for negative feedback; closed-loop gain → $1/F$ when loop gain $\gg 1$.
- Stability requires loop-gain magnitude < 1 at 180° phase shift (equivalently, phase margin > 0°); larger PM → shorter, cleaner settling.
- One-pole and two-pole are the two dominant architectures; one-pole op amps are inherently self-compensated (via $C_L$), two-pole op amps need Miller (pole-splitting) compensation, at the cost of introducing an RHP zero.
- Settling-time formulas link $GB$, accuracy bits $N$, and phase margin — directly relevant to data-converter design.
- Systematic spec-driven design procedures exist for both one-pole and two-pole op amps.

---

## Lecture 10 — Comparators

### Definition
- A **comparator** compares an analog input against another analog signal or reference and outputs a **binary** signal — effectively a **1-bit A/D converter**.
```
Analog In ──►┌──────────────┐
             │ 1-bit         │──► Encode ──► Digital Out (0/1)
Vref ───────►│ Quantizer     │
             └──────────────┘
```

### Comparator Definitions
- **Output swing**: $V_{OH}-V_{OL}$ (difference between max and min output).
- **Minimum input voltage**: input needed to swing the output fully between $V_{OH}$ and $V_{OL}$.
- **Propagation time delay**: time from a step input until the output crosses the halfway point $(V_{OH}+V_{OL})/2$.
- **Input offset voltage**: input difference required to bring the output to $(V_{OH}-V_{OL})/2$ (i.e., the midpoint referenced from a defined baseline).
- Two basic types: **one-pole** and **two-pole** open-loop comparators (built from uncompensated op amps).

### Open-Loop Comparator Dynamics
- An **uncompensated** op amp makes an excellent comparator (no feedback stability requirement).
- **Two response regimes**:
  1. **Linear region**: exponential-like rise (RC/pole-dominated); the larger the input overdrive, the **shorter** the propagation delay (crosses the threshold sooner along the same rising curve, then saturates at $V_{OH}$).
  2. **Slew-limited region**: if the input is large enough that the required rate of rise exceeds $I/C$, the output ramps **linearly** (slewing), and:
$$
t_{p} = \frac{(V_{OH}+V_{OL})/2}{SR}
$$

### Input-Offset Cancellation — Auto-Zeroing
```
Auto-Zero Phase:                    Comparison Phase:
      ┌───[ Op Amp ]───┐                  ┌───[ Op Amp ]───┐
Vin=0─┤(+)          (−)├──┐        Vin ───┤(+)          (−)├──┐
      └────────────────┘  │              └────────────────┘  │
             │             │                     │            │
           [Caz]───────────┘  (unity-gain          [Caz] (in series, polarity
             │                 feedback charges       reversed to cancel Vos)
            GND                Caz to Vos)
```
- **Auto-zero phase**: comparator configured in **unity-gain feedback**, charging an auto-zero capacitor $C_{AZ}$ to the input-offset voltage $V_{OS}$.
- **Comparison phase**: $C_{AZ}$ is switched in series (polarity reversed) with the input, **cancelling** $V_{OS}$.
- Practical limit: residual offset reducible to roughly **≤ 1 mV**, limited by switch non-idealities (**charge injection/clock feedthrough**).
- **Requirement**: the comparator must be **stable during auto-zero** (since it's in unity-gain feedback) — a **two-pole** comparator needs its compensation capacitor switched **in** during auto-zero and switched **out** during comparison (for maximum speed, since compensation slows the amplifier).

### Latch Comparators (Regenerative Comparators)
```
        VDD                              (PMOS latch, similarly structured)
         │
       [I1]      [I2]
         │         │
   Vo1●──┤M1     M2├──●Vo2
         │         │
         └────┬────┘   (cross-coupled: gate of M1 → drain of M2, and vice versa)
              │
             GND
```
- Uses **positive feedback** to regenerate a small input difference into a full-swing output.
- **Two operating periods**:
  1. **Track/reset (not enabled)**: current sources disconnected; inputs $V_{o1}, V_{o2}$ connect to the unknown differential signal, charging parasitic latch-node capacitances to the input values.
  2. **Regeneration (enabled/"Apply")**: current sources reconnected; positive feedback drives one side high, the other low, based on which side started with the (slightly) higher initial voltage.
- **Dynamics**: if the latch loop gain $g_mr_{ds} > 1$ (often written $g_m r_o$ or similar), the step response is a **growing (positive) exponential** — larger initial unknown input → **shorter** propagation delay.
```
V_latch
  │                    ____ (large input diff: fast rise)
  │                  _/
  │              ___/          (small input diff: starts slow,
  │           __/                then accelerates — "starts slow,
  │       ___/                    ends fast")
  └────────────────────────────► t
```

### What Limits Comparator Speed?
- **Linear-response regime**: speed limited by **pole locations** — poles farther from the origin (in the complex-frequency plane) → wider bandwidth → faster rise time.
- **Slew-rate regime**: delay $\propto C/I$ (capacitance ÷ available charge/discharge current) — pole/zero locations become **irrelevant**; it's purely a charge-delivery problem.

### High-Speed Comparator Architecture — Preamp + Latch Cascade
```
Vin ──►[Preamp 1]──►[Preamp 2]──►[Preamp 3]──►[Latch]──► Digital Out
        (fast rise,   (fast rise,   (fast rise,   (finishes the
         moderate      moderate      moderate       full-swing
         gain)         gain)         gain)          regeneration)
```
- **Open-loop (preamp) stages** are placed first because their step response rises **quickly at the start** (good initial speed) but slows later.
- The **latch** is placed last because its step response starts **slowly** but accelerates — completing the full-swing output efficiently.
- **Why cascade several lower-gain, wider-bandwidth preamps (commonly 3 stages) rather than one high-gain stage?** Lower-gain/wider-bandwidth stages achieve better *overall* speed than a single narrow-bandwidth high-gain stage, due to the gain-bandwidth trade-off at each stage.

### Lecture 10 Summary
- Comparator = 1-bit A/D converter; compares analog input vs. reference, outputs binary.
- Open-loop comparators are just uncompensated (one- or two-pole) op amps; response can be linear (pole-limited) or slew-limited.
- Auto-zeroing (clocked, unity-gain-feedback + capacitor storage) cancels input-offset voltage to ~1 mV, limited by switch charge injection.
- Latches use positive feedback for regeneration; their step response starts slow but accelerates (exponential growth if loop gain > 1).
- High-speed comparators cascade multiple open-loop preamp stages (fast initial rise) followed by a latch (fast full completion) — combining the strengths of both response shapes.

---

## Lecture 11 — D/A and A/D Converters

### Definitions
- **DAC (Digital-to-Analog Converter)**: converts digital (typically binary) data into an analog signal (current, voltage, or charge).
- **ADC (Analog-to-Digital Converter)**: converts a continuous physical quantity (typically voltage) into a digital number representing its amplitude; inherently involves **quantization** (deciding if a signal is above/below a threshold), which introduces **quantization error**.

```
DAC block diagram:                     ADC block diagram:
                                         Analog In
DSP System                                  │
   │ (parallel digital bits)                ▼
   ▼                                  [Sample & Hold]
[D/A Converter] ◄── Vref                    │
   │                                        ▼
Analog Out (raw) ──► [Filter] ──► [Amp]  [Quantizer] ◄── Vref
                                             │
                                             ▼
                                   [Digital Output] ──► DSP System
```

### D/A Conversion Process
```
         Vref
          │
          ▼
  ┌───────────────┐
  │ Scaling Network │──► Vout = D · Vref  (0 ≤ D ≤ 1, set by bits b0..b(n-1))
  └───────────────┘
          │
          ▼
     [optional Amplifier] ──► Analog Output
```
- $D$ is determined by the input bits, from the **MSB** ($b_0$) to the **LSB** ($b_{n-1}$).
- The primary active device in a DAC is typically the **op amp**.

#### DAC Transfer Function & Quantization Error
```
Vout
  │                         ●
  │                      ● ┌┘  <- ideal (infinite-resolution) line: dotted
  │                   ● ┌──┘
  │                ●┌───┘         <- actual staircase (finite resolution), solid
  │             ●┌──┘
  │          ●┌──┘
  └───────────────────────────────► Digital Input (000...0 → 111...1)
```
$$
\text{Quantization Error}(D) = V_{ideal}(D) - V_{actual}(D), \qquad \text{range: } \pm\tfrac12\,\text{LSB}
$$
- **1 LSB** = the height of one staircase step = $V_{ref}/2^N$ for an $N$-bit converter.
- **Design implication**: once component accuracy is within **±½ LSB**, further improving component accuracy is wasted effort — resolution is capped by the number of bits, not component precision beyond that point. To improve further, you must **increase $N$**.

### Types of DACs

| Category | Speed | Mechanism |
|---|---|---|
| **Serial** (e.g., charge-redistribution DAC) | Slow: $T_{conv} = N\times T_{clk}$ | Converts one bit at a time |
| **Parallel** | Fast: typically **one clock period** | Converts all bits simultaneously |
- Can be implemented via **current**, **voltage**, or **charge** scaling (or combinations).
- DACs can be **continuous** (unclocked) or **clocked**, depending on architecture.

#### Binary-Weighted Resistor DAC
```
                     Rf
              ┌──────[R]──────┐
              │                │
   Vref ──►[SW_MSB]──[R]───────┤
   Vref ──►[SW]──────[2R]──────┤
   Vref ──►[SW]──────[4R]──────┤──────► Vout
    ⋮                          │  (summing-amplifier / virtual-ground node)
   Vref ──►[SW_LSB]──[2^(N-1)R]┤
                                │
                              [Op Amp]
                                │
                               GND (virtual, at inverting input)
```
- Binary-weighted resistors ($R, 2R, 4R, \dots, 2^{N-1}R$) feed a summing-amplifier virtual-ground node.
- Each switch connects its resistor either to $V_{ref}$ (injecting current $\propto 1/R_{branch}$) or to ground (no current).
- Output voltage is the weighted sum of the bit currents through the feedback resistor $R_f$.

### A/D Conversion Process — Sample & Hold
```
Sampling (switch closed):        Holding (switch open):
Vin ──[SW closed]──●── Vout      Vin ──[SW open] ●── Vout (frozen at
                    │                              │   last sampled value)
                   [C]                             [C]
                    │                               │
                   GND                             GND
```
```
Vin, Vout
  │  Vin (continuously varying)
  │╲  ╱╲    ╱╲
  │ ╲╱  ╲  ╱  ╲
  │      ╲╱    
  │ ── (Vout tracks Vin while sampling, then holds flat during hold)
  └───────────────────────────────► t
```
- ADCs **cannot convert continuously** — they must **sample and hold** the input, then convert, then re-sample.
- Requires a **pre-filter** (anti-aliasing filter) ahead of the sample-and-hold, to prevent aliasing at the sampling frequency.
- The primary active device in an ADC is typically the **comparator**.

### Oversampling (Delta-Sigma-Style) Converters
```
Analog In ──►(+/−)──►[Loop Filter/Integrator]──►[Quantizer]──►[1-bit or multi-bit output]──►[DSP/Digital Filter]──► Digital Out
     ▲                                                │
     └────────────────[Feedback: subtract]◄───────────┘
```
- The **modulator** runs at a very high sample rate $f_S$ and pushes quantization noise to **high frequencies**, out of the signal band; a subsequent **digital low-pass filter** removes it.
- Only the quantizer/modulator is analog — the **rest of the signal chain is digital**, which is highly compatible with VLSI scaling.

#### Nyquist vs. Oversampled Bandwidth Comparison
```
Nyquist (conventional) ADC:                 Oversampled ADC:
|H(f)|                                       |H(f)|
 │███ (signal BW)                            │█ (signal BW, much smaller)
 │   ╲                                        │  ╲
 │    ╲___(hard-to-build anti-alias filter)    │   ╲___(easy anti-alias filter,
 └──────────────► f                            │       wide guard band)
      f_B   f_S/2 = Nyquist freq              └──────────────► f
      (f_B close to f_S/2 → steep filter          f_B  (guard band)  f_S/2
       required)
```
$$
f_S = M \cdot f_{Nyquist} = M \cdot 2f_B
$$

| Symbol | Meaning |
|---|---|
| $M$ | **Oversampling ratio** — power-of-2 values (2, 4, 8, 16, 32, 64, 128, …) for oversampled converters; $M=1$ for conventional Nyquist-rate converters |
| $f_B$ | Signal bandwidth |
| $f_S$ | Sampling frequency |

> [!TIP]
> All ADCs are technically "sampled" converters; the distinction is purely the value of $M$. Nyquist converters ($M=1$) need aggressive, hard-to-build anti-aliasing filters because the guard band between $f_B$ and $f_S/2$ is minimal. Oversampled converters trade sample-rate for a much easier analog anti-alias filter, pushing complexity into the (cheap, scalable) digital domain.

### Types of ADCs by Conversion Speed

| Type | Conversion Time | Notes |
|---|---|---|
| **Serial** | $\approx 2^N\times T_{clk}$ | Simple, but slow |
| **Successive Approximation (SAR)** | $N\times T_{clk}$ | One bit resolved per clock cycle |
| **Pipeline** | 1 clock period (throughput), but $N\times T_{clk}$ **latency** | Repeated identical stages; high throughput despite pipeline delay |
| **Algorithmic** | $N\times T_{clk}$ | Same architecture as pipeline but reuses **one** stage repeatedly (area-efficient, slower) |
| **High-Speed / Flash** | 1 clock period | Limited typically to ~6–8 bits; power & area scale poorly with bits |
| **Oversampled (ΔΣ)** | Flexible — can be configured for either high-resolution/slow or low-resolution/fast | Same fundamental modulator architecture serves both extremes |

#### Pipeline ADC Stage
```
Stage i:  Vin ──►[Delay]──►[Comparator]──►(bit_i out)
                     │
                     ▼
              [×2 Gain]──[Sum/Subtract Vref]──► Vin (to next stage)
```
- Each stage: 1 comparator (resolves 1 bit), a **×2 gain**, a summing junction, and reference addition/subtraction — **MSB resolved first**, then progressively less-significant bits down the pipeline.
- Data flows down the pipe each clock cycle; **once filled**, a new output is available every clock period, though the **latency** to the first valid output is $N\times$(number of stages)$\times T_{clk}$.

#### Flash ADC
```
Vin ──►(+)─┐  (+)─┐  (+)─┐   ...   (+)─┐
           │      │      │            │
Vref_ladder─►[Comp1] [Comp2] [Comp3] ... [CompM]
           │      │      │            │
           ▼      ▼      ▼            ▼
      Thermometer code ──► [Decoder] ──► Digital Output
```
- Uses **parallel comparators**, each referenced to one tap of a resistor-ladder-generated reference voltage.
- Produces a **thermometer code** (all comparators below $V_{in}$ trip "1," all above trip "0," or vice versa), decoded into a binary output word.
- Can be pipelined into a **single clock cycle**: half the cycle to generate the thermometer code, half to output the encoded word.

### Lecture 11 Summary
- DAC: digital → analog (current/voltage/charge); ADC: analog → digital, inherently quantized (± ½ LSB error).
- DACs categorized as slow-serial vs. fast-parallel; binary-weighted resistor DAC is a canonical example.
- ADCs require sample-and-hold + anti-aliasing pre-filter; the comparator is the core active device.
- Oversampled (ΔΣ) converters trade sample rate ($M\times f_{Nyquist}$) for relaxed anti-alias filtering and mostly-digital signal processing, shaping quantization noise out of the signal band.
- ADC architectures span serial (slowest) → SAR/algorithmic ($N\cdot T_{clk}$) → pipeline (1 clk throughput, $N\cdot T_{clk}$ latency) → flash (fastest, but bit-limited); oversampled converters can flexibly occupy any resolution/speed trade-off point using the same core architecture.

---

## Lecture 12 — The Future of Analog IC Design

### Where Is Analog IC Design Today?
- **Market maturity**: shakeout of inefficient companies; consolidation via mergers/acquisitions into large, established players, alongside a smaller population of startups.
- **Consumer electronics** (smartphones, personal communication/entertainment devices) strongly shape circuit/architecture demands.
- Analog has become largely a **commodity** — similar products offered by many vendors, differentiated mainly by cost.
- **Mixed-signal challenges** grow as more analog and digital content is integrated onto the same die.
- Technology bifurcation: some companies use **analog-optimized processes**; others push analog design into the **latest short-channel CMOS** nodes (motivating "digitally assisted" techniques below).
- Innovation focus has shifted somewhat from *inventing new functions* toward *cleverly using available technology to accomplish existing functions*.
- Reduced R&D and fewer analog-specific startups; new startups skew toward web-based products/applications.

### Technology's Influence on Analog Design
- **Digital circuits scale well** with technology shrinkage and have driven short-channel process development.
- **Analog benefits less**: speed increases, but **gain decreases**, **matching worsens**, **non-linearity increases**, and new parasitic issues (e.g., **gate leakage**) emerge.
- Central analog challenge: managing **trade-offs** among linearity, speed, precision, and power.
```
Area ↑  →  Non-linearity ↓, Precision ↑        (larger devices/caps → better matching & linearity)
Speed ↑ →  Power ↑, Precision ↓                (faster circuits cost more power, less accuracy)
```
- As analog/digital integration increases, **substrate coupling/interference** between blocks worsens.

### Digitally Assisted Analog Circuits
- **Core idea**: modern (deep-submicron) digital logic is so energy-efficient that spare "logic budget" can be spent **correcting** analog imperfections, making small-geometry nodes attractive for analog again.
- **Illustrative energy comparison** (lecture's own numbers):
  - 0.5 µm technology: ~1.3 pJ per logic operation.
  - 90 nm technology: ~4.5 fJ per logic operation.
  - A 90 nm ADC operation might cost ~21 nJ — dividing by 4.5 fJ/op implies **~4,700 equivalent logic operations** could be performed for the same energy budget.
  - At higher target dynamic ranges (50, 70, 90 dB), the number of "affordable" logic operations per conversion grows to ~38,000 → ~300,000 → ~2.4 million — i.e., **as required resolution increases, the case for spending logic to fix analog imperfections gets stronger, not weaker.**
- **Digitally assisted techniques** mentioned:
  - Oversampling-based mismatch correction
  - Digital linearization of amplifiers
  - Digital correction of dynamic errors
  - "Systematic/synergistic" error correction — exploiting known properties of the application-specific signal to correct analog performance

### The Role of Tools
- **Electrical design tools**: schematic capture, circuit simulation (block/circuit/mixed-signal/statistical or Monte Carlo levels), reliability simulators (including aging effects), leakage/node simulators (standby-mode high-current detection), IP verification/discovery tools.
- **Physical design tools**: layout editors, extraction tools, verification tools, ESD/latch-up robustness checkers.
- Increasing tool maturity/sophistication → fewer design mistakes, reduced design time.

### Influence of the Internet & Media
- Instant worldwide communication enables **geographically distributed design teams** to coordinate on complex chips.
- Design tools/resources increasingly available **remotely** (including from home) — no need to physically own design infrastructure.
- Blurring line between **individual/small design houses** and **large design companies**.
- **IP (Intellectual Property)** as a tradeable design resource, though **patent-based IP monetization is becoming harder**.

### A Web-Based ("Crowd-Design") Paradigm

```
Traditional paradigm:
Market Identified → IC Designed & Fabricated → IC Marketed
(money invested up front; no profitability guarantee; high overhead)

Proposed web-based paradigm:
Consumer posts IC spec (open website, profit-sharing terms)
        │
        ▼
Independent designers use web-based design & fab tools to meet spec
        │
        ▼
Designs fabricated → Consumer selects winning design
        │
        ▼
Consumer buys & markets IC → Profit shared (designer + web platform)
```
- Characteristics: **no upfront investment** until the product sells; demand is **consumer-driven** (someone who actually wants/will pay for the IC initiates it); opens the **design space to a wide pool** of designers; **no massive market required** for profitability; minimizes time/overhead — described as the **analog-design equivalent of crowdfunding ("crowd-designing")**.

### Challenges in Teaching Analog IC Design
- **Problem**: disconnect between academic curricula and industry needs; the skill/commitment level required deters students, leading to fewer analog-design students overall.
- **Proposed solution**: divide responsibilities —
  - **Academia** → theory, tools, foundational background.
  - **Industry** → application, technology specifics, implementation practice.
- Advocate for **online learning platforms** covering design → fabrication → test for those pursuing IC design in depth.

### What's Next?
- Follow-on course referenced: **"CMOS Analog Circuit Design"** — a **40-lecture** in-depth online course (≈1 hour/lecture, split into four 15–20 minute segments, each with a 10-question multiple-choice quiz), roughly equivalent to a **45-hour graduate university course** (without exams/homework).
- **Recommended prerequisites**: BS in Electrical Engineering (or equivalent); solid grounding in circuit theory, technology, circuit modeling, and basic design skills — though the course is stated to be accessible to those from other backgrounds seeking deeper appreciation of the field.

### Lecture 12 Summary
- Analog IC design today: mature, consolidated market; commoditized products; mixed-signal integration challenges.
- Technology scaling helps digital far more than analog (gain, matching, linearity, gate leakage all work against analog at small geometries).
- Digitally assisted analog circuits exploit cheap, abundant digital logic (in nm-scale nodes) to correct analog imperfections — increasingly attractive as required resolution (dynamic range) rises.
- Tool maturity, global/remote collaboration, and IP trading are reshaping the design ecosystem.
- A speculative "crowd-designed," consumer-driven, web-based IC design/fabrication/marketing paradigm is proposed as a possible future model.
- Teaching analog design should split theory (academia) from application/technology (industry), supplemented by full online design-fabricate-test learning platforms.

---

## Global Symbol & Acronym Glossary

### Acronyms

| Acronym | Full Name | Context |
|---|---|---|
| IC | Integrated Circuit | Overall subject of the course |
| CMOS | Complementary Metal-Oxide-Semiconductor | Dominant modern IC technology (NMOS + PMOS) |
| MOSFET | Metal-Oxide-Semiconductor Field-Effect Transistor | Primary active device discussed |
| BJT | Bipolar Junction Transistor | Alternate active device (diffusion-current based) |
| SCR | Silicon-Controlled Rectifier | Example 3-terminal active device |
| CAD | Computer-Aided Design | Tools for IC design/simulation/layout |
| CMP | Chemical-Mechanical Polishing | Advanced planarization process step |
| STI | Shallow Trench Isolation | Isolation structure (~0.5 µm depth) |
| HCI | Hot Carrier Injection | Reliability degradation mechanism |
| NBTI | Negative Bias Temperature Instability | Reliability degradation mechanism |
| TDDB | Time-Dependent Dielectric Breakdown | Gate-oxide reliability mechanism (off-state drain stress) |
| BSIM | Berkeley Short-channel IGFET Model | Industry-standard SPICE MOSFET compact model (v3, v4) |
| SPICE | Simulation Program with Integrated Circuit Emphasis | Generic term for circuit simulators |
| DRC | Design Rule Check | Physical-layout verification against foundry rules |
| LVS | Layout Versus Schematic | Physical-vs-electrical equivalence check |
| DIP | Dual In-line Package | Classic IC package type |
| KVL / KCL | Kirchhoff's Voltage Law / Kirchhoff's Current Law | Fundamental circuit principles |
| CMRR | Common-Mode Rejection Ratio | $\lvert A_{vd}/A_{vc}\rvert$ for differential amplifiers |
| CMFB | Common-Mode Feedback | Resolves current-source-load diff-amp operating-point conflicts |
| PTAT | Proportional To Absolute Temperature | Reference-voltage/current type with positive T-slope |
| CTAT | Complementary To Absolute Temperature | Reference-voltage type with negative T-slope (e.g., $V_{BE}$) |
| TC | Temperature Coefficient | ppm/°C metric for reference stability |
| PVT | Process, Voltage, Temperature | The three variation axes an "independent source" should reject |
| ZTC | Zero Temperature Coefficient | MOSFET bias point where $I_D$ vs. $T$ curves cross |
| GB / GBW | Gain-Bandwidth (Product) | Op-amp dynamic figure of merit |
| PM | Phase Margin | Stability margin metric |
| LSB / MSB | Least/Most Significant Bit | Data-converter bit-weighting terminology |
| DAC | Digital-to-Analog Converter | Lecture 11 topic |
| ADC | Analog-to-Digital Converter | Lecture 11 topic |
| SAR | Successive Approximation Register (converter) | ADC architecture, $N\times T_{clk}$ |
| ΔΣ | Delta-Sigma | Oversampling modulator/converter family |
| ESD | Electrostatic Discharge | Physical-design robustness concern |
| IP | Intellectual Property | Reusable/tradeable design blocks |

### Symbols

| Symbol | Meaning |
|---|---|
| $I_D$ | MOSFET drain current |
| $V_{GS}, V_{DS}, V_{SB}$ | Gate-source, drain-source, source-bulk voltages |
| $V_T, V_{T0}$ | Threshold voltage (general; zero-bias) |
| $K' = \mu C_{ox}$ | Process transconductance parameter |
| $W, L$ | Transistor width, length |
| $\lambda$ | Channel-length modulation parameter |
| $\gamma$ | Body-effect (bulk-threshold) parameter |
| $\phi_F$ | Fermi potential |
| $\theta$ | Velocity-saturation parameter |
| $V_A$ | Early voltage ($\approx 1/\lambda$) |
| $g_m$ | Small-signal transconductance |
| $g_{ds}$ (or $r_{ds}$) | Small-signal drain-source conductance (resistance) |
| $C_{gs}, C_{gd}, C_{gb}, C_{sb}, C_{db}$ | MOSFET terminal capacitances |
| $k$ | Boltzmann's constant |
| $q$ | Elementary charge |
| $T$ | Absolute temperature |
| $V_{PTAT}, V_{CTAT}$ | PTAT / CTAT reference voltages |
| $A_{vd}, A_{vc}$ | Differential-mode / common-mode voltage gain |
| $V_{id}, V_{ic}$ | Differential-mode / common-mode input voltage |
| $SR$ | Slew rate |
| $C_L$ | Load capacitance |
| $A_v, A_0$ | Open-loop voltage gain |
| $F$ | Feedback factor/network transfer function |
| $\omega_{p1}, \omega_{p2}$ | First/second pole frequencies |
| $C_c$ | Miller compensation capacitance |
| $GB$ | Gain-bandwidth product |
| $N$ | Number of accuracy bits |
| $T_S$ | Settling time |
| $M$ | Oversampling ratio |
| $f_B, f_S, f_{Nyquist}$ | Signal bandwidth, sampling frequency, Nyquist frequency |

---

## Cheat Sheet — Key Equations & Rules of Thumb

| # | Equation / Rule | Where Used |
|---|---|---|
| 1 | $I_D=\dfrac{K'W}{2L}(V_{GS}-V_T)^2(1+\lambda V_{DS})$ | Square-law MOSFET hand model (L3) |
| 2 | $V_T=V_{T0}+\gamma(\sqrt{2\phi_F+V_{SB}}-\sqrt{2\phi_F})$ | Body effect (L3, L7) |
| 3 | $I_{D1}/I_{D2}=(W/L)_1/(W/L)_2$ | Current matching (L5) |
| 4 | $V_{OS}=\Delta V_T-(V_{GS}-V_T)\dfrac{\Delta K}{2K}$ | Diff-pair worst-case offset (L5) |
| 5 | Neg. series FB: $R(1+LG)$; Neg. shunt FB: $R/(1+LG)$ | Feedback resistance scaling (L5) |
| 6 | $\Delta V_D=(kT/q)\ln(n)$ | $V_{PTAT}$ reference (L7) |
| 7 | $TC=\dfrac{V_{max}-V_{min}}{V_{nom}(T_{max}-T_{min})}\times10^6$ ppm/°C | Reference stability metric (L7) |
| 8 | $V_{REF}=K\cdot V_{PTAT}+V_{CTAT}$ | Bandgap reference (L7) |
| 9 | $SR = I/C_L$ | Slew rate (L8) |
| 10 | $CMRR = |A_{vd}/A_{vc}|$ | Diff amp (L8) |
| 11 | $V_{out}/V_{in}=A_v/(1+A_vF)\to 1/F$ for $A_vF\gg1$ | Op-amp closed-loop gain (L9) |
| 12 | $PM = 180^\circ - \angle(\text{loop gain})|_{|LG|=1}$ | Stability (L9) |
| 13 | $T_S = (N+1)\ln2 / GB$ | One-pole settling time (L9) |
| 14 | $t_p = \dfrac{(V_{OH}+V_{OL})/2}{SR}$ | Slew-limited comparator delay (L10) |
| 15 | Quantization error $=\pm\tfrac12$ LSB $=\pm V_{ref}/2^{N+1}$ | DAC/ADC (L11) |
| 16 | $f_S = M\cdot 2f_B$ | Oversampling ratio (L11) |
| 17 | 6 dB of dynamic range per added bit | Digital dynamic range / data converters (L1, L11) |

### Core Trade-offs Recap

| Trade-off | Direction |
|---|---|
| **Area vs. Matching/Precision** | ↑ Area → ↑ Matching accuracy, ↓ non-linearity |
| **Speed vs. Power vs. Precision** | ↑ Speed → ↑ Power required, generally ↓ achievable precision |
| **Gain vs. Bandwidth** | Single-stage/single-pole ↑ Gain often ↓ Bandwidth (classic $GB$ constancy) |
| **Compensation vs. Speed** | Adding compensation (stability) → ↓ speed (esp. two-pole op amps/comparators) |
| **Resolution (bits) vs. Conversion Time/Power** | ↑ Resolution (ADC/DAC) → generally ↑ conversion time and/or power and/or die area |
| **Technology scaling (nm↓)** | Helps digital greatly; hurts analog gain/matching/linearity; motivates "digitally assisted analog" |

> [!NOTE]
> The recurring meta-lesson across all 12 lectures: analog IC design is fundamentally about **navigating trade-offs** (area↔matching, speed↔power↔precision, gain↔bandwidth) using a disciplined toolkit of **principles** (unchanging laws), **concepts** (generally-true relationships), and **techniques** (practical approximations/assumptions) — validated by a two-tier modeling approach (simple hand models for intuition, complex simulation models for verification).
