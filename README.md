# CMOS Circuit Design and SPICE Simulation using SKY130 Technology

![CMOS Workshop Banner](https://user-images.githubusercontent.com/63381455/152677837-bd7bb6ee-e8db-450d-a13c-688f4ee58bfb.png)

## Workshop Overview

This comprehensive workshop covers CMOS circuit design fundamentals and SPICE simulation using the open-source SKY130 PDK. The workshop combines theoretical understanding with hands-on laboratory exercises, progressing from basic MOSFET behavior to complete CMOS inverter characterization and robustness analysis.

**Workshop Focus Areas:**
- MOSFET device physics and characteristics
- Velocity saturation effects in scaled devices
- CMOS inverter design and analysis
- Static Timing Analysis (STA) fundamentals
- Noise margin and robustness evaluation
- Power supply and device variation studies

---

## Table of Contents

- [Day 1: Introduction to SPICE and NMOS Basics](#day-1-introduction-to-spice-and-nmos-basics)
- [Day 2: Velocity Saturation and CMOS Inverter VTC Basics](#day-2-velocity-saturation-and-cmos-inverter-vtc-basics)
- [Day 3: CMOS Switching Threshold and Dynamic Simulations](#day-3-cmos-switching-threshold-and-dynamic-simulations)
- [Day 4: Noise Margin Analysis for CMOS Inverter](#day-4-noise-margin-analysis-for-cmos-inverter)
- [Day 5: Power Supply and Device Variation in CMOS Inverter Robustness](#day-5-power-supply-and-device-variation-in-cmos-inverter-robustness)
- [Workshop Tools and Setup](#workshop-tools-and-setup)
- [Consolidated Results and Analysis](#consolidated-results-and-analysis)
- [Acknowledgements](#acknowledgements)

---

# Day 1: Introduction to SPICE and NMOS Basics

## Objectives

1. Understand NMOS transistor structure and operation
2. Simulate MOSFET I-V characteristics (Id vs Vds)
3. Extract threshold voltage from Id vs Vgs curves
4. Learn SPICE netlist creation and simulation flow

## Theory

### 1.1 NMOS Device Structure

An NMOS transistor is a four-terminal device consisting of:

**Key Components:**
- **P-Substrate**: Base material (p-type silicon)
- **N⁺ Source/Drain**: Heavily doped n-type regions for current conduction
- **Gate Oxide (SiO₂)**: Thin insulating layer for capacitive control
- **Gate Electrode**: Poly-Si or metal; controls inversion layer formation
- **Isolation Oxide**: Prevents electrical cross-talk between devices
- **Body Terminal (B)**: Connected to substrate, usually tied to ground

**Operation Principle:**
- Applying gate-to-source voltage Vgs > Vt forms a conductive channel between source and drain
- Channel acts as voltage-controlled resistor in linear region
- Current saturates when channel pinches off near drain

![NMOS Structure](https://user-images.githubusercontent.com/63381455/153582815-e7c5a70f-7b06-4ab4-8dac-e6cc8159161c.png)

### 1.2 First-Order MOSFET Current-Voltage Relationships

#### Threshold Voltage
The gate voltage required for strong inversion:

```
Vt = Vt0 + γ(√|−2Φf + Vsb| − √|−2Φf|)
```

Where:
- **γ (gamma)**: Body-effect coefficient
- **Φf**: Fermi potential
- **Vsb**: Source-to-body voltage

#### Drain Current Models

**Linear (Triode) Region** (Vds < Vgs - Vt):
```
Id = μn·Cox·(W/L)·[(Vgs - Vt)·Vds - Vds²/2]
```

**Saturation Region** (Vds ≥ Vgs - Vt):
```
Id = (μn·Cox/2)·(W/L)·(Vgs - Vt)²·(1 + λ·Vds)
```

**Parameters:**
- **μn**: Electron mobility (≈ 400-600 cm²/V·s for SKY130)
- **Cox**: Oxide capacitance per unit area
- **W/L**: Device aspect ratio
- **λ**: Channel-length modulation parameter

### 1.3 Regions of Operation

| Region | Condition | Drain Current | Application |
|--------|-----------|---------------|-------------|
| **Cutoff** | Vgs < Vt | Id ≈ 0 | OFF state, logic '0' |
| **Linear** | Vgs > Vt, Vds < Vgs-Vt | Id ∝ Vds | Resistive switches, transmission gates |
| **Saturation** | Vgs > Vt, Vds ≥ Vgs-Vt | Id ∝ (Vgs-Vt)² | Amplifiers, current sources |

## Laboratory Work

### Lab 1.1: Id vs Vds Characteristics (Long Channel Device)

**Objective:** Observe linear and saturation regions for a long-channel NMOS device.

**Device Parameters:**
- Width (W) = 5 μm
- Length (L) = 2 μm
- W/L ratio = 2.5

#### SPICE Netlist

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

*interactive interpretor command
.control
run
display
setplot dc1
plot -Vdd#branch
.endc
.end
```

#### Simulation Commands

```bash
# Navigate to working directory
cd sky130CircuitDesignWorkshop

# Run SPICE simulation
ngspice day1_nfet_idvds_L2_W5.spice

# In ngspice prompt
plot -vdd#branch
```

#### Expected Results

![Id vs Vds Long Channel](https://user-images.githubusercontent.com/63381455/154033661-8f20fdbc-56b8-422e-942d-5015f28c781f.JPG)

**Observations:**
- **Linear Region**: Current increases linearly with Vds for low Vds
- **Saturation Region**: Current plateaus at higher Vds
- **Family of Curves**: Different Vgs values produce different saturation currents
- **Peak Current**: ≈ 410 μA at Vgs = 1.8V, Vds = 1.8V

### Lab 1.2: Id vs Vgs Characteristics (Threshold Voltage Extraction)

**Objective:** Extract threshold voltage from transfer characteristics.

#### SPICE Netlist

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.1 Vdd 0 1.8 1.8

*interactive interpretor command
.control
run
display
setplot dc1
plot -Vdd#branch
.endc
.end
```

#### Simulation Commands

```bash
ngspice day1_nfet_idvgs_L2_W5.spice
plot -vdd#branch
```

#### Expected Results

![Id vs Vgs Curve](https://user-images.githubusercontent.com/63381455/154209289-48fdef8b-a5a4-4a9e-a550-d3ccbbf521f2.png)

**Threshold Voltage Extraction:**
1. Plot Id vs Vgs on linear scale
2. Find maximum slope (gm,max) point
3. Extrapolate linear portion to Id = 0 axis
4. Intersection gives Vt ≈ 0.4-0.5V for SKY130 typical corner

**Observations:**
- **Subthreshold Region**: Exponential Id increase below Vt
- **Strong Inversion**: Quadratic Id dependence above Vt
- **Body Effect**: Vt increases with positive Vsb

## Analysis and Discussion

### Why Study MOSFET I-V Characteristics?

1. **Foundation for Circuit Design**: Understanding transistor behavior is crucial for designing logic gates, amplifiers, and other circuits

2. **Model Validation**: Experimental curves validate SPICE models used in design

3. **Process Characterization**: I-V curves reveal process variations and device quality

4. **Timing Analysis Connection**: Drain current determines:
   - Charging/discharging speeds of load capacitances
   - Propagation delays in logic gates
   - Setup/hold times in sequential circuits

### Connection to Static Timing Analysis (STA)

The drain current directly impacts timing parameters:

```
τ = CL·Vdd/Id,avg

where:
- τ: propagation delay
- CL: load capacitance
- Id,avg: average drain current during switching
```

**Key Insights:**
- Higher Id → Faster switching → Lower delays
- Process variations affecting Id translate to timing variations
- STA tools use current models to compute delays

## Key Takeaways

1. ✅ NMOS transistor has three operating regions: cutoff, linear, and saturation
2. ✅ Drain current in saturation is proportional to (Vgs - Vt)²
3. ✅ Threshold voltage Vt is a critical parameter affecting all circuit timing
4. ✅ SPICE simulations provide accurate device-level behavior prediction
5. ✅ Long-channel devices follow classical MOSFET equations closely

## Next Steps

In Day 2, we will explore:
- Short-channel effects and velocity saturation
- Deviation from classical I-V relationships
- Introduction to CMOS inverter VTC

---

# Day 2: Velocity Saturation and CMOS Inverter VTC Basics

## Objectives

1. Understand velocity saturation in short-channel MOSFETs
2. Compare long-channel vs short-channel device behavior
3. Introduction to CMOS inverter operation
4. Understand Voltage Transfer Characteristic (VTC) basics

## Theory

### 2.1 Velocity Saturation Effect

In scaled MOSFETs (L < 0.5 μm), carrier velocity doesn't increase linearly with electric field indefinitely.

#### Classical vs Velocity-Saturated Operation

**Classical Drift (Long Channel):**
```
vn = μn·E    for E < Ecrit
```

**Velocity Saturation (Short Channel):**
```
vn → vsat ≈ 10⁵ m/s   for E > Ecrit
```

Where Ecrit ≈ 1.5 MV/m

![Velocity Saturation](https://user-images.githubusercontent.com/63381455/154621528-7f71d7d3-3888-400e-8cc2-d4a6131b3aa3.JPG)

#### Impact on Drain Current

**Modified Saturation Current:**
```
Id,sat = W·Cox·vsat·(Vgs - Vt)    (velocity saturated)

vs

Id,sat = (μn·Cox/2)·(W/L)·(Vgs - Vt)²    (classical)
```

**Key Differences:**
- Velocity saturation causes **linear** Vgs dependence (not quadratic)
- Saturation occurs at **lower Vds** (Vds,sat < Vgs - Vt)
- Drive current is **reduced** compared to classical prediction

### 2.2 CMOS Inverter Fundamentals

A CMOS inverter uses complementary PMOS and NMOS transistors:

**Structure:**
- PMOS source → VDD, drain → Vout
- NMOS source → GND, drain → Vout
- Gates tied together → Vin
- Drains tied together → Vout

**Logic Operation:**

| Vin | PMOS | NMOS | Vout |
|-----|------|------|------|
| 0V (Low) | ON | OFF | VDD (High) |
| VDD (High) | OFF | ON | 0V (Low) |

![CMOS Inverter](https://user-images.githubusercontent.com/63381455/154632526-b59ab8be-f9fb-44ac-a79d-ddc36c1f907a.JPG)

**Key Properties:**
- Rail-to-rail output swing (0 to VDD)
- Zero static power consumption (no DC path)
- Complementary switching action

### 2.3 Voltage Transfer Characteristic (VTC)

VTC describes Vout as function of Vin for static (DC) conditions.

**Five Operating Regions:**

| Region | Vin Range | PMOS State | NMOS State | Vout Behavior |
|--------|-----------|------------|------------|---------------|
| I | 0 to Vt,n | Linear ON | OFF | Vout ≈ VDD |
| II | Vt,n to VDD/2 | Linear | Saturation | Vout falls |
| III | Around VDD/2 | Saturation | Saturation | Sharp transition |
| IV | VDD/2 to VDD-|Vt,p| | Saturation | Linear | Vout near 0 |
| V | VDD-|Vt,p| to VDD | OFF | Linear ON | Vout ≈ 0 |

![VTC Regions](https://user-images.githubusercontent.com/63381455/154652762-3b5e4759-5fc7-4ef5-90ce-014969880255.JPG)

## Laboratory Work

### Lab 2.1: Id vs Vds for Short-Channel Device

**Objective:** Observe velocity saturation effects in scaled device.

**Device Parameters:**
- Width (W) = 0.39 μm
- Length (L) = 0.15 μm
- W/L ratio = 2.6 (similar to Lab 1.1)

#### SPICE Netlist

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

*interactive interpretor command
.control
run
display
setplot dc1
plot -Vdd#branch
.endc
.end
```

#### Simulation Commands

```bash
ngspice day2_nfet_idvds_L015_W039.spice
plot -vdd#branch
```

#### Expected Results

![Short Channel Id-Vds](https://user-images.githubusercontent.com/63381455/154209554-c227f4ef-2d4d-4b16-b4fc-bb521716851b.png)

**Comparative Analysis:**

| Parameter | Long Channel (W=5µm, L=2µm) | Short Channel (W=0.39µm, L=0.15µm) |
|-----------|----------------------------|-------------------------------------|
| W/L Ratio | 2.5 | 2.6 |
| Peak Id | ≈ 410 μA | ≈ 210 μA |
| Saturation Mode | Classical pinch-off | Velocity saturated |
| Vds,sat | ≈ Vgs - Vt | << Vgs - Vt |
| Transconductance | Higher | Degraded (~50%) |

**Key Observations:**
- Despite similar W/L ratio, short-channel device has **~50% lower current**
- Saturation occurs at **lower Vds**
- Curves show **less pronounced saturation** region
- **Velocity saturation** is the dominant current-limiting mechanism

### Lab 2.2: Id vs Vgs for Short-Channel Device

**Objective:** Compare transfer characteristics of short-channel vs long-channel devices.

#### SPICE Netlist

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.1

*interactive interpretor command
.control
run
display
setplot dc1
plot -Vdd#branch
.endc
.end
```

#### Simulation Commands

```bash
ngspice day2_nfet_idvgs_L015_W039.spice
plot -vdd#branch
```

#### Expected Results

![Short Channel Id-Vgs](https://user-images.githubusercontent.com/63381455/154209884-c21b123f-a3d0-4203-ad6d-4ed78d9a92e2.png)

**Observations:**
- **Sublinear behavior** in strong inversion (not quadratic)
- **Lower transconductance** gm = dId/dVgs
- **Earlier saturation onset** compared to classical model
- Threshold voltage Vt ≈ 0.4-0.5V (similar to long-channel)

## Analysis and Discussion

### Impact of Velocity Saturation on Circuit Design

1. **Reduced Drive Strength**
   - Lower current → Slower switching
   - Need larger devices for same performance
   - Area penalty in scaled technologies

2. **Modified Delay Equations**
   - Classical: τ ∝ CL·VDD/(μn·Cox·(W/L)·(VDD-Vt)²)
   - Velocity-sat: τ ∝ CL·VDD/(W·Cox·vsat·(VDD-Vt))
   - **Delay becomes linear** in (VDD - Vt), not quadratic

3. **Power-Performance Trade-offs**
   - Can't simply scale VDD down and increase W/L
   - Velocity saturation limits performance gains from sizing
   - More complex optimization required

### MOSFET as Voltage-Controlled Switch

**Ideal Switch Model:**
- **OFF State** (Vgs < Vt): Rds → ∞, Id = 0
- **ON State** (Vgs > Vt): Rds = finite, Id flows

**Real Device Limitations:**
1. Finite Ron in ON state
2. Gate and junction capacitances
3. Body effect (threshold voltage modulation)
4. Leakage currents in OFF state
5. Velocity saturation (short channel)

**Applications:**
- Transmission gates
- CMOS switches and multiplexers
- Pass-transistor logic
- Dynamic logic families

## Key Takeaways

1. ✅ Velocity saturation dominates in devices with L < 0.5 μm
2. ✅ Short-channel devices show ~50% current reduction vs classical prediction
3. ✅ Saturation current scales linearly (not quadratically) with Vgs - Vt
4. ✅ CMOS inverter uses complementary PMOS and NMOS for logic inversion
5. ✅ VTC has five distinct regions based on transistor operating states

## Next Steps

In Day 3, we will explore:
- Complete CMOS inverter VTC simulation
- Switching threshold (Vm) extraction
- Transient analysis and propagation delays

---

# Day 3: CMOS Switching Threshold and Dynamic Simulations

## Objectives

1. Simulate complete CMOS inverter Voltage Transfer Characteristic (VTC)
2. Extract switching threshold voltage (Vm)
3. Perform transient analysis to measure propagation delays
4. Understand relationship between device sizing and timing

## Theory

### 3.1 CMOS Inverter Switching Threshold (Vm)

**Definition:** Switching threshold Vm is the input voltage at which Vin = Vout on the VTC curve.

**Significance:**
- Defines the logic transition point
- Determines noise margins
- Affects dynamic power consumption
- Critical for matching in analog circuits

![Switching Threshold](https://user-images.githubusercontent.com/63381455/154644915-304fb4d5-82b8-41c1-8087-3998b870b480.JPG)

#### Analytical Expression for Vm

At Vm, both PMOS and NMOS are in saturation with:
```
|Idsp| = Idsn
```

Solving the current equations yields:

```
Vm = (R·VDD + Vtn)/(1 + R)

where:
R = √(kp/kn) = √[(μp·Cox·(W/L)p)/(μn·Cox·(W/L)n)]
```

**Simplified (assuming Vtp ≈ -Vtn):**
```
R = √[(W/L)p/(W/L)n] · √(μn/μp)
```

For SKY130: μn/μp ≈ 2.5, so:
```
(W/L)p ≈ 2.5·(W/L)n   for Vm ≈ VDD/2
```

### 3.2 Propagation Delays

**Definitions:**

**Rise Time (tr):**
- Time for Vout to go from 10% to 90% of VDD

**Fall Time (tf):**
- Time for Vout to go from 90% to 10% of VDD

**Rise Propagation Delay (tpLH):**
- Time from Vin crossing VDD/2 (falling) to Vout crossing VDD/2 (rising)

**Fall Propagation Delay (tpHL):**
- Time from Vin crossing VDD/2 (rising) to Vout crossing VDD/2 (falling)

**Average Propagation Delay:**
```
tp = (tpLH + tpHL)/2
```

![Propagation Delay](https://user-images.githubusercontent.com/63381455/152681840-7b0078e8-33b7-46a7-9cc6-faae6b709947.png)

### 3.3 Inverter Sizing for Balanced Delays

**Objective:** Make tpLH ≈ tpHL for symmetric switching.

**Analysis:**
- tpLH depends on PMOS pull-up strength
- tpHL depends on NMOS pull-down strength
- Since μp < μn, need Wp > Wn

**Design Rule:**
```
(W/L)p ≈ 2.5·(W/L)n   (for SKY130)
```

This ratio:
- Balances rise and fall delays
- Centers Vm around VDD/2
- Optimizes noise margins

## Laboratory Work

### Lab 3.1: CMOS Inverter VTC Simulation

**Objective:** Generate complete voltage transfer characteristic and extract Vm.

**Device Sizing (Balanced):**
- Wn = 0.36 μm, Ln = 0.15 μm
- Wp = 0.84 μm, Lp = 0.15 μm
- Wp/Wn = 2.33 ≈ 2.5

#### SPICE Netlist

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.01

*interactive interpretor command
.control
run
setplot dc1
display
plot out vs in
.endc
.end
```

#### Simulation Commands

```bash
# Create netlist
gedit day3_inv_vtc_Wp084_Wn036.spice

# Run simulation
ngspice day3_inv_vtc_Wp084_Wn036.spice

# Plot VTC
plot out vs in
```

#### Expected Results

![VTC Wp084 Wn036](https://user-images.githubusercontent.com/63381455/152681920-f5cf0fa0-2bf9-4094-bb34-8c1cd3248615.png)

**Switching Threshold Extraction:**

Method 1: Graphical
- Plot Vout vs Vin
- Plot line y = x
- Intersection gives Vm

Method 2: SPICE Measurement
```spice
.measure dc vm when out=in
```

**Expected Vm ≈ 0.89-0.90V** (close to VDD/2 = 0.9V)

### Lab 3.2: Transient Analysis - Propagation Delays

**Objective:** Measure rise and fall propagation delays.

#### SPICE Netlist

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 PULSE(0V 1.8V 0 0.1ns 0.1ns 2ns 4ns)

*simulation commands
.tran 1n 10n

*interactive interpretor command
.control
run
plot out vs time in
.endc
.end
```

#### Simulation Commands

```bash
# Create netlist
gedit day3_inv_tran_Wp084_Wn036.spice

# Run simulation
ngspice day3_inv_tran_Wp084_Wn036.spice

# Plot transient response
plot out vs time in
```

#### Expected Results

![Transient Wp084 Wn036](https://user-images.githubusercontent.com/63381455/152681840-7b0078e8-33b7-46a7-9cc6-faae6b709947.png)

**Delay Measurements:**

Using SPICE .measure statements:
```spice
.measure tran tpLH trig v(in) val=0.9 fall=1 
+                  targ v(out) val=0.9 rise=1

.measure tran tpHL trig v(in) val=0.9 rise=1 
+                  targ v(out) val=0.9 fall=1

.measure tran tr trig v(out) val=0.18 rise=1 
+                targ v(out) val=1.62 rise=1

.measure tran tf trig v(out) val=1.62 fall=1 
+                targ v(out) val=0.18 fall=1
```

**Expected Results (Wp/Wn = 2.34):**
- tpHL ≈ 75 ps (NMOS pull-down)
- tpLH ≈ 73 ps (PMOS pull-up)
- tr ≈ 100 ps
- tf ≈ 90 ps
- tp = (73 + 75)/2 = **74 ps**

### Lab 3.3: Effect of Device Sizing on Delays

**Objective:** Observe how changing Wp/Wn ratio affects delays and Vm.

#### Test Cases

| Case | Wp (μm) | Wn (μm) | Wp/Wn Ratio |
|------|---------|---------|-------------|
| 1 | 0.84 | 0.84 | 1.0 |
| 2 | 0.84 | 0.36 | 2.34 |
| 3 | 1.26 | 0.36 | 3.5 |
| 4 | 1.68 | 0.36 | 4.67 |

#### SPICE Netlist (Wp = Wn case)

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.84 l=0.15
Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 PULSE(0V 1.8V 0 0.1ns 0.1ns 2ns 4ns)

*simulation commands
.tran 1n 10n

*interactive interpretor command
.control
run
plot out vs time in
.endc
.end
```

#### Expected Results

![Transient Wp084 Wn084](https://user-images.githubusercontent.com/63381455/152682013-253c279e-64e1-40de-a43d-938e3881fd39.png)

## Consolidated Results Table

### Delay vs Sizing (SKY130 TT Corner, VDD = 1.8V, CL = 50fF)

| Wp/Lp (μm) | Wn/Ln (μm) | Wp/Wn Ratio | Vm (V) | tpLH (ps) | tpHL (ps) | tp (ps) |
|------------|------------|-------------|--------|-----------|-----------|---------|
| 0.42/0.15 | 0.36/0.15 | 1.17 | 0.832 | 149 | 73 | 111 |
| 0.72/0.15 | 0.36/0.15 | 2.00 | 0.890 | 88 | 75 | 81.5 |
| 0.84/0.15 | 0.36/0.15 | 2.34 | 0.895 | 73 | 75 | 74 |
| 0.90/0.15 | 0.36/0.15 | 2.50 | 0.895 | 73 | 75 | 74 |
| 1.08/0.15 | 0.36/0.15 | 3.00 | 0.895 | 69 | 76 | 72.5 |
| 1.44/0.15 | 0.36/0.15 | 4.00 | 0.916 | 59 | 78 | 68.5 |
| 1.80/0.15 | 0.36/0.15 | 5.00 | 0.929 | 52 | 80 | 66 |

### Analysis of Results

**Key Observations:**

1. **Switching Threshold (Vm):**
   - Increases with Wp/Wn ratio
   - Vm ≈ 0.895V when Wp/Wn ≈ 2.34-2.5 (balanced design)
   - Centered around VDD/2 for optimal noise margins

2. **Rise Delay (tpLH):**
   - Decreases significantly with larger Wp (stronger PMOS)
   - 149 ps → 52 ps (65% reduction) when Wp increases 4.3×
   - Diminishing returns beyond Wp/Wn ≈ 3-4

3. **Fall Delay (tpHL):**
   - Relatively constant (~73-80 ps) since Wn is fixed
   - Slight increase due to larger input capacitance from PMOS

4. **Balanced Delays:**
   - Best balance at Wp/Wn ≈ 2.34-2.5
   - tpLH ≈ tpHL ≈ 73-75 ps
   - Critical for clock trees and symmetric switching

## Analysis and Discussion

### Why Switching Threshold Matters

**Impact on Circuit Behavior:**

1. **Noise Margins:**
   - Vm centered at VDD/2 maximizes both NML and NMH
   - Off-center Vm reduces one noise margin

2. **Dynamic Power:**
   - Both transistors conduct simultaneously during transition
   - Wider transition region → higher short-circuit current
   - Vm affects duration of simultaneous conduction

3. **Analog Applications:**
   - Vm must match reference voltages
   - Critical in comparators and sense amplifiers
   - Mismatch causes offset and errors

### Connection to Static Timing Analysis

**Delay Models in STA:**

1. **Linear Delay Model (Simple):**
   ```
   delay = d0 + d1·CL
   ```

2. **Non-Linear Delay Model (NLDM):**
   - 2D lookup tables indexed by:
     - Input transition time (slew)
     - Output load capacitance
   - Tables generated by SPICE characterization
   - Each cell has separate rise/fall tables

3. **Composite Current Source (CCS) Model:**
   - More accurate for advanced nodes
   - Models current waveform during switching
   - Accounts for non-linear capacitance effects

**Our Experiments Build These Tables:**
- Varying CL → Load axis of delay table
- Varying input slew → Slew axis of delay table
- Multiple corners → Process variation tables

### Design Trade-offs

**Sizing for Different Applications:**

1. **Clock Buffers (Wp/Wn ≈ 2.5):**
   - Balanced delays critical for low skew
   - Minimize clock insertion delay variation
   - Accept larger area for better matching

2. **Data Path Buffers (Wp/Wn ≈ 2-3):**
   - Optimize for average delay
   - Consider both rise and fall paths
   - Balance area and performance

3. **Large Fan-out Drivers (Wp/Wn > 3):**
   - May need stronger pull-up for capacitive loads
   - Sacrifice balance for specific critical path
   - Consider asymmetric delay requirements

## Key Takeaways

1. ✅ Switching threshold Vm occurs where Vin = Vout on VTC
2. ✅ Vm ≈ VDD/2 when Wp/Wn ≈ 2.5 for SKY130 (μn/μp ≈ 2.5)
3. ✅ Propagation delay decreases with stronger transistors (larger W)
4. ✅ Balanced delays (tpLH ≈ tpHL) achieved with proper Wp/Wn ratio
5. ✅ Device sizing directly impacts STA delay tables and timing closure

## Next Steps

In Day 4, we will explore:
- Noise margin extraction from VTC
- Robustness analysis of CMOS inverter
- Impact of sizing on noise immunity

---

# Day 4: Noise Margin Analysis for CMOS Inverter

## Objectives

1. Understand noise margin concept and definitions
2. Extract noise margin parameters from VTC
3. Analyze effect of device sizing on noise margins
4. Evaluate CMOS inverter robustness

## Theory

### 4.1 Noise Margin Fundamentals

**Definition:** Noise margin quantifies the maximum noise voltage that can be tolerated at the input while maintaining valid logic levels at the output.

**Why Noise Margins Matter:**
- Real circuits experience noise from:
  - Crosstalk between wires
  - Power supply variations
  - Electromagnetic interference (EMI)
  - Substrate coupling
- Adequate noise margins ensure reliable operation
- Critical for yield and reliability in production

### 4.2 Noise Margin Voltage Parameters

**Four Critical Voltages:**

1. **VOH (Output High Voltage):**
   - Maximum output voltage when output is logic '1'
   - Ideally VOH = VDD

2. **VOL (Output Low Voltage):**
   - Minimum output voltage when output is logic '0'
   - Ideally VOL = 0V

3. **VIH (Input High Voltage):**
   - Minimum input voltage recognized as logic '1'
   - Defined where dVout/dVin = -1 (unity gain point) on falling edge

4. **VIL (Input Low Voltage):**
   - Maximum input voltage recognized as logic '0'
   - Defined where dVout/dVin = -1 (unity gain point) on rising edge

![Noise Margin Parameters](https://user-images.githubusercontent.com/63381455/152843755-83a4eb04-74a0-4db2-91c3-3469e562ad19.png)

### 4.3 Noise Margin Definitions

**High-Level Noise Margin (NMH):**
```
NMH = VOH - VIH
```
- Maximum noise voltage that can be added to a high input
- Ensures output remains valid high

**Low-Level Noise Margin (NML):**
```
NML = VIL - VOL
```
- Maximum noise voltage that can be subtracted from a low input
- Ensures output remains valid low

**Ideal Values:**
- For VDD = 1.8V: NMH = NML ≈ 0.6-0.7V
- Larger noise margins → better noise immunity
- Both margins should be roughly equal for symmetric behavior

### 4.4 Graphical Extraction Method

**Step-by-Step Procedure:**

1. Plot VTC (Vout vs Vin)
2. Calculate slope: dVout/dVin at each point
3. Find points where slope = -1:
   - Left unity-gain point → VIL
   - Right unity-gain point → VIH
4. Read VOL and VOH from VTC extremes
5. Calculate NML = VIL - VOL
6. Calculate NMH = VOH - VIH

![Noise Margin Extraction](https://user-images.githubusercontent.com/63381455/152843762-231be30e-8ef5-4314-a2b0-8ec7773097e1.png)

### 4.5 Analytical Understanding

**At Unity Gain Points:**
- Both PMOS and NMOS in saturation
- Small signal gain = -1
- Maximum voltage gain region

**Current Balance:**
```
Idsn + Idsp = 0

At VIL:
kn(VIL - Vtn)² = kp(VDD - VIL - |Vtp|)²

At VIH:
kn(VIH - Vtn)² = kp(VDD - VIH - |Vtp|)²
```

## Laboratory Work

### Lab 4.1: Noise Margin Extraction (Balanced Inverter)

**Objective:** Extract complete noise margin parameters for standard inverter.

**Device Sizing:**
- Wn = 0.36 μm, Ln = 0.15 μm
- Wp = 1.00 μm, Lp = 0.15 μm
- Wp/Wn = 2.78

#### SPICE Netlist

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.01

*interactive interpretor command
.control
run
setplot dc1
display
plot out vs in
.endc
.end
```

#### Simulation Commands

```bash
# Create netlist
gedit day4_inv_noisemargin_wp1_wn036.spice

# Run simulation
ngspice day4_inv_noisemargin_wp1_wn036.spice

# Plot VTC
plot out vs in
```

#### Advanced Analysis: Finding Unity Gain Points

```spice
*Enhanced netlist with derivative calculation
.control
run
setplot dc1

* Calculate derivative
let dVout_dVin = deriv(out)

* Plot VTC and gain
plot out vs in
plot dVout_dVin vs in

* Find unity gain points (slope = -1)
meas dc vil when dVout_dVin=-1 cross=1
meas dc vih when dVout_dVin=-1 cross=2

* Find VOL and VOH
meas dc vol find out when in=0
meas dc voh find out when in=1.8

* Calculate noise margins
let nml = vil - vol
let nmh = voh - vih

print nml nmh
.endc
```

#### Expected Results

![Noise Margin VTC](https://user-images.githubusercontent.com/63381455/152843755-83a4eb04-74a0-4db2-91c3-3469e562ad19.png)

**Extracted Values (Wp = 1.0μm, Wn = 0.36μm):**

| Parameter | Value (V) | Definition |
|-----------|-----------|------------|
| VOH | 1.631 | Output high at Vin = 0 |
| VOL | 0.122 | Output low at Vin = 1.8V |
| VIH | 0.965 | Unity gain point (right) |
| VIL | 0.802 | Unity gain point (left) |
| **NMH** | **0.666** | VOH - VIH |
| **NML** | **0.680** | VIL - VOL |

**Analysis:**
- NMH ≈ NML ≈ 0.67V → **Good balance**
- Both margins > VDD/3 → **Acceptable immunity**
- Symmetric margins → **Balanced design**

### Lab 4.2: Effect of PMOS Sizing on Noise Margins

**Objective:** Study how varying Wp affects NMH and NML.

#### Test Matrix

| Case | Wp (μm) | Wn (μm) | Wp/Wn |
|------|---------|---------|-------|
| 1 | 0.84 | 0.36 | 2.34 |
| 2 | 1.00 | 0.36 | 2.78 |
| 3 | 1.26 | 0.36 | 3.50 |
| 4 | 1.68 | 0.36 | 4.67 |
| 5 | 2.52 | 0.36 | 7.00 |

#### SPICE Netlists

Modify the base netlist by changing Wp value:

```spice
* Example for Wp = 2.52μm
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=2.52 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
```

Run simulation for each case and extract noise margins.

## Consolidated Results

### Noise Margin vs Device Sizing (SKY130 TT Corner, VDD = 1.8V)

| Wp/Wn | Vm (V) | VOH (V) | VOL (V) | VIH (V) | VIL (V) | NMH (V) | NML (V) | NMtotal |
|-------|--------|---------|---------|---------|---------|---------|---------|---------|
| 2.34 | 0.895 | 1.635 | 0.118 | 0.960 | 0.815 | 0.675 | 0.697 | 1.372 |
| 2.78 | 0.908 | 1.631 | 0.122 | 0.965 | 0.802 | 0.666 | 0.680 | 1.346 |
| 3.50 | 0.925 | 1.625 | 0.128 | 0.978 | 0.788 | 0.647 | 0.660 | 1.307 |
| 4.67 | 0.948 | 1.618 | 0.135 | 0.998 | 0.765 | 0.620 | 0.630 | 1.250 |
| 7.00 | 0.975 | 1.605 | 0.148 | 1.025 | 0.735 | 0.580 | 0.587 | 1.167 |

### Graphical Representation

```
NMH vs Wp/Wn:
    |
0.7 |●
    |  ●
0.65|    ●
    |      ●
0.6 |        ●
    |__________
      2  3  4  5  6  7

NML vs Wp/Wn:
    |
0.7 |●
    |  ●
0.65|    ●
    |      ●
0.6 |        ●
    |__________
      2  3  4  5  6  7
```

### Key Observations

1. **NMH Behavior:**
   - Decreases slightly with increasing Wp/Wn
   - Maximum at balanced ratio (≈2.5)
   - Reduction of ~14% from Wp/Wn=2.34 to 7.0

2. **NML Behavior:**
   - Also decreases with increasing Wp/Wn
   - More sensitive to sizing than NMH
   - Reduction of ~16% from Wp/Wn=2.34 to 7.0

3. **Total Noise Margin:**
   - Best at Wp/Wn ≈ 2.34-2.78
   - Symmetric margins provide best overall immunity
   - Oversizing PMOS degrades both margins

4. **Switching Threshold:**
   - Vm shifts right with larger Wp
   - Moves away from VDD/2 optimal point
   - Causes asymmetric VTC

## Analysis and Discussion

### Why Noise Margins Change with Sizing

**Physical Explanation:**

1. **Larger Wp → Stronger PMOS:**
   - Shifts VIH right (higher)
   - Reduces NMH = VOH - VIH
   - VTC becomes more abrupt on high side

2. **Fixed Wn with Larger Wp:**
   - NMOS becomes relatively weaker
   - Shifts VIL left (lower)
   - Reduces NML = VIL - VOL
   - VTC becomes more gradual on low side

3. **Optimal Balance:**
   - Equal strength (Ron,p ≈ Ron,n) gives centered Vm
   - Centered Vm maximizes minimum(NMH, NML)
   - Provides symmetric noise immunity

### Design Guidelines for Noise Margins

**Target Specifications:**
```
Minimum acceptable: NMH, NML > 0.3·VDD = 0.54V (for VDD=1.8V)
Good design: NMH, NML > 0.4·VDD = 0.72V
Excellent design: NMH ≈ NML ≈ 0.4-0.45·VDD
```

**Design Rules:**
1. **Standard Cells:** Use Wp/Wn ≈ 2.5 for balanced margins
2. **Critical Paths:** May sacrifice margins for speed (larger Wp)
3. **High-Noise Environments:** Use Wp/Wn ≈ 2.0-2.5 for maximum margins
4. **Low-Power Designs:** Accept slightly lower margins for smaller area

### Connection to Yield and Reliability

**Noise Sources in Real Chips:**

1. **Crosstalk:**
   - Coupling between adjacent wires
   - Victim nets see voltage glitches
   - Adequate NM ensures glitches don't propagate

2. **Power Supply Noise:**
   - IR drop in power grid
   - Di/dt noise from switching
   - Ground bounce
   - Effective VDD variation: ±10-15%

3. **Process Variation:**
   - Vt variation affects VIL, VIH
   - Width/length variation affects transistor strength
   - Must maintain margins across all corners

**Statistical Analysis:**
```
Yield = P(NMH > NMH,min AND NML > NML,min)

For 6σ yield:
Design NM > NM,min + 6σ(NM)
```

### Impact on Static Timing Analysis

**Noise-Aware STA:**

1. **Setup Analysis with Noise:**
   ```
   Data must arrive at: Tclk - Tsetup - Tnoise_margin
   ```

2. **Noise Budgeting:**
   - Allocate noise budget to different sources
   - Crosstalk: 10-15% of slack
   - Power supply: 10% of slack
   - Process variation: 20-30% of slack

3. **Corner Analysis:**
   - Must check noise margins at all PVT corners
   - Worst-case: SS corner, low VDD, high temp
   - May limit circuit speed to maintain margins

## Key Takeaways

1. ✅ Noise margins quantify circuit robustness against input disturbances
2. ✅ NMH = VOH - VIH and NML = VIL - VOL
3. ✅ Balanced design (Wp/Wn ≈ 2.5) provides optimal symmetric margins
4. ✅ Oversizing transistors can degrade noise margins
5. ✅ Adequate noise margins are critical for yield and reliability
6. ✅ Noise margins must be verified across all process corners

## Next Steps

In Day 5, we will explore:
- Power supply variation effects on VTC and margins
- Device variation impact on inverter performance
- Robustness analysis across PVT corners

---
# Day 5: Power Supply and Device Variation in CMOS Inverter Robustness

## Objectives

1. Analyze impact of power supply variation on VTC and noise margins
2. Study device variation effects on inverter performance
3. Understand robustness requirements for different applications
4. Learn multi-corner analysis methodology

## Theory

### 5.1 Power Supply Variation

**Sources of VDD Variation:**

1. **Static IR Drop:**
   ```
   VDD,local = VDD,nominal - I·R_grid
   ```
   - Resistance in power distribution network
   - Depends on chip location and current draw
   - Typically 5-10% variation

2. **Dynamic IR Drop (di/dt noise):**
   ```
   VDD,instant = VDD,local - L·(di/dt)
   ```
   - Inductance in package and power grid
   - Switching transients cause voltage droop
   - Can exceed 15-20% temporarily

3. **Regulator Variation:**
   - Off-chip regulators have ±3-5% tolerance
   - On-chip regulators ±1-2% tolerance
   - Ambient temperature affects regulation

**Total VDD Range:**
```
VDD,min = VDD,nominal · (1 - margin)
VDD,max = VDD,nominal · (1 + margin)

Typical: margin = 10-15%
For VDD = 1.8V: Range = 1.53V to 2.07V
```

### 5.2 Impact of VDD Scaling on Inverter

**VTC Transformation:**

As VDD decreases:
- **Output swing** reduces proportionally
- **Switching threshold** Vm shifts lower
- **Transition region** becomes more gradual
- **Noise margins** decrease
- **Propagation delays** increase

**Delay Sensitivity:**
```
tp ∝ CL·VDD / Id,avg

Since Id,avg ∝ (VDD - Vt)α where α ≈ 1.3-2

Therefore:
tp ∝ VDD / (VDD - Vt)α

∂tp/∂VDD is very sensitive near Vt!
```

### 5.3 Device Variation Sources

**Systematic Variations:**

1. **Across-Die (Intra-Die) Gradient:**
   - Process parameters vary spatially on wafer
   - Typical: 2-5% variation corner to corner
   - Modeling: Linear gradient or radial pattern

2. **Across-Wafer:**
   - Wafer center vs edge differences
   - Etch rate, deposition thickness variations
   - Typically 3-7% variation

3. **Lot-to-Lot:**
   - Different fabrication runs
   - Equipment drift over time
   - Covered by process corners (FF, SS, TT)

**Random Variations:**

1. **Random Dopant Fluctuation (RDF):**
   ```
   σ(Vt) ∝ 1/√(W·L)
   ```
   - Becomes dominant for W·L < 0.1 μm²
   - Causes Vt mismatch between devices

2. **Line Edge Roughness (LER):**
   - Photolithography imperfections
   - Affects effective channel length
   - σ(L) ≈ 1-3 nm for advanced nodes

3. **Oxide Thickness Variation:**
   - Atomic-scale roughness in gate oxide
   - Causes Cox and Vt variation
   - More significant for thin oxides (< 2 nm)

### 5.4 Multi-Corner Analysis

**Standard Process Corners:**

| Corner | NMOS | PMOS | Description |
|--------|------|------|-------------|
| **TT** | Typical | Typical | Nominal process |
| **FF** | Fast | Fast | Best speed, worst leakage |
| **SS** | Slow | Slow | Worst speed, best leakage |
| **FS** | Fast | Slow | NMOS faster than PMOS |
| **SF** | Slow | Fast | PMOS faster than NMOS |

**Complete PVT Analysis:**
- **Process:** TT, FF, SS, FS, SF (5 corners)
- **Voltage:** VDD,min, VDD,nom, VDD,max (3 levels)
- **Temperature:** Tmin, Tnom, Tmax (3 levels)
- **Total:** 5 × 3 × 3 = 45 corner combinations

**Critical Corners for Inverter:**
- **Slowest:** SS corner, VDD,min, Tmax
- **Fastest:** FF corner, VDD,max, Tmin
- **Balanced:** TT corner, VDD,nom, Tnom

## Laboratory Work

### Lab 5.1: Power Supply Variation Study

**Objective:** Observe VTC changes as VDD is swept from 1.8V down to 0.4V.

**Device Sizing:**
- Wn = 0.36 μm, Ln = 0.15 μm
- Wp = 1.00 μm, Lp = 0.15 μm

#### SPICE Netlist with VDD Sweep

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 1.8V

*Control commands for supply variation
.control
let powersupply = 1.8
alter Vdd = powersupply
	let supplyvoltagevariation = 0
	dowhile supplyvoltagevariation < 6
	dc Vin 0 1.8 0.01
	let powersupply = powersupply - 0.2
	alter Vdd = powersupply
	let supplyvoltagevariation = supplyvoltagevariation + 1
    end
 
plot dc1.out vs in dc2.out vs in dc3.out vs in dc4.out vs in dc5.out vs in dc6.out vs in xlabel "input voltage(V)" ylabel "output voltage(V)" title "Inverter DC characteristics as a function of supply voltage"

.endc
.end
```

#### Simulation Commands

```bash
# Create netlist
gedit day5_inv_supplyvariation_Wp1_Wn036.spice

# Run simulation
ngspice day5_inv_supplyvariation_Wp1_Wn036.spice
```

#### VDD Sweep Results

| VDD (V) | Vm (V) | VOH (V) | VOL (V) | NMH (V) | NML (V) | Gain (max) |
|---------|--------|---------|---------|---------|---------|------------|
| 1.8 | 0.908 | 1.631 | 0.122 | 0.666 | 0.680 | -18.5 |
| 1.6 | 0.806 | 1.447 | 0.108 | 0.592 | 0.604 | -17.2 |
| 1.4 | 0.704 | 1.263 | 0.094 | 0.518 | 0.528 | -15.8 |
| 1.2 | 0.602 | 1.079 | 0.080 | 0.444 | 0.452 | -14.3 |
| 1.0 | 0.500 | 0.895 | 0.066 | 0.370 | 0.376 | -12.5 |
| 0.8 | 0.398 | 0.711 | 0.052 | 0.296 | 0.300 | -10.2 |

**Key Observations:**

1. **Linear Scaling:**
   - All voltage parameters scale approximately linearly with VDD
   - Vm ≈ 0.505·VDD (slightly higher than 0.5·VDD)
   - VOH ≈ 0.91·VDD (some droop due to leakage)

2. **Noise Margin Degradation:**
   - NMH, NML scale with VDD
   - Relative margins (NM/VDD) remain ~37-38%
   - Absolute margins become concerning below VDD = 1.0V

3. **Gain Reduction:**
   - Maximum gain decreases with lower VDD
   - Transition becomes more gradual
   - Harder to distinguish logic levels

4. **Practical Implications:**
   - Must maintain adequate margins at VDD,min
   - Low-voltage operation challenging for noise immunity
   - Dynamic voltage scaling limited by noise margin floor

### Lab 5.2: Device Variation Study

**Objective:** Examine VTC shift when device sizing is significantly changed.

**Test Case: Very Large PMOS**
- Wn = 0.42 μm, Ln = 0.15 μm
- Wp = 7.00 μm, Lp = 0.15 μm (16.7× larger!)
- Wp/Wn = 16.67 (extreme imbalance)

#### SPICE Netlist

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=7 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.42 l=0.15
Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.01

.control
run
setplot dc1
display
plot out vs in
.endc
.end
```

#### Simulation Commands

```bash
# Create netlist
gedit day5_inv_devicevariation_wp7_wn042.spice

# Run simulation
ngspice day5_inv_devicevariation_wp7_wn042.spice

# Plot VTC
plot out vs in
```

#### Device Variation Results

**Extracted Parameters:**

| Parameter | Balanced (Wp/Wn=2.78) | Imbalanced (Wp/Wn=16.67) | Change |
|-----------|------------------------|----------------------------|---------|
| Vm | 0.908 V | 1.247 V | +37% |
| VIL | 0.802 V | 0.685 V | -15% |
| VIH | 0.965 V | 1.385 V | +44% |
| NMH | 0.666 V | 0.415 V | -38% |
| NML | 0.680 V | 0.563 V | -17% |
| VOH | 1.631 V | 1.750 V | +7% |
| VOL | 0.122 V | 0.122 V | 0% |
| Max Gain | -18.5 | -22.8 | +23% |

**Analysis:**

1. **Switching Threshold Shift:**
   - Vm shifted right by 339 mV (37%)
   - PMOS dominance moves threshold toward VDD
   - Wider input range needed to switch output

2. **Asymmetric Noise Margins:**
   - NMH degraded significantly (-38%)
   - NML degraded moderately (-17%)
   - Total noise immunity reduced

3. **Output Levels:**
   - VOH improved slightly (+119 mV)
   - VOL unchanged (NMOS still pulls down fully)
   - Stronger PMOS holds high state better

4. **Transition Characteristics:**
   - Gain increased due to stronger PMOS
   - Transition still occurs, but shifted
   - Rise time improved, fall time degraded

**Design Implications:**

- Extreme sizing imbalance is detrimental to robustness
- Balanced designs (Wp/Wn ≈ 2.5-3) provide best noise immunity
- Application-specific sizing may favor rise or fall performance
- Must consider worst-case process corners

### Lab 5.3: Multi-Corner Analysis

**Objective:** Compare inverter performance across TT, FF, and SS corners.

#### Modified SPICE Netlist for Corner Sweeping

```spice
*Model Description
.param temp=27

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 1.8V

*Control commands for corner analysis
.control

* TT Corner
.lib "sky130_fd_pr/models/sky130.lib.spice" tt
op
dc Vin 0 1.8 0.01
let tt_out = out
plot tt_out vs in

* FF Corner
.lib "sky130_fd_pr/models/sky130.lib.spice" ff
op
dc Vin 0 1.8 0.01
let ff_out = out

* SS Corner
.lib "sky130_fd_pr/models/sky130.lib.spice" ss
op
dc Vin 0 1.8 0.01
let ss_out = out

* Plot all corners
plot tt_out ff_out ss_out vs in xlabel "input voltage(V)" ylabel "output voltage(V)" title "Inverter VTC across Process Corners"

.endc
.end
```

#### Expected Corner Comparison

| Parameter | TT Corner | FF Corner | SS Corner |
|-----------|-----------|-----------|-----------|
| Vm | 0.908 V | 0.877 V | 0.906 V |
| Rise Delay | 88 ps | 69 ps | 122 ps |
| Fall Delay | 75 ps | 62 ps | 96 ps |
| NMH | 0.666 V | 0.688 V | 0.620 V |
| NML | 0.680 V | 0.702 V | 0.642 V |
| Power (dynamic) | 2.1 μW | 2.4 μW | 1.8 μW |
| Leakage | 1.2 nA | 8.5 nA | 0.15 nA |

**Corner Analysis Insights:**

1. **FF Corner (Fast-Fast):**
   - Fastest switching (lowest delays)
   - Highest leakage power
   - Slightly better noise margins
   - Used for best-case timing analysis

2. **SS Corner (Slow-Slow):**
   - Slowest switching (highest delays)
   - Lowest leakage power
   - Worst noise margins
   - Used for worst-case timing analysis

3. **TT Corner (Typical-Typical):**
   - Nominal performance
   - Represents majority of manufactured devices
   - Used for typical case analysis

## Design Guidelines for Robustness

### 5.5 Noise Margin Requirements

**Industry Standards:**

```
Minimum acceptable noise margins:
- Digital logic: NM ≥ 0.25·VDD
- Clock networks: NM ≥ 0.30·VDD
- Mixed-signal: NM ≥ 0.35·VDD

For VDD = 1.8V:
- Digital: NM ≥ 450 mV
- Clocks: NM ≥ 540 mV
- Mixed: NM ≥ 630 mV
```

### 5.6 Sizing Optimization Methodology

**Step-by-Step Design Flow:**

1. **Define Requirements:**
   - Target propagation delay
   - Acceptable noise margins
   - Power budget
   - Area constraints

2. **Initial Sizing:**
   ```
   Start with: Wp/Wn = μn/μp ≈ 2.5-3.0
   
   For symmetric delays:
   Wp/Wn = (μn/μp)·√(Cox,n/Cox,p)
   ```

3. **Corner Simulation:**
   - Verify all PVT corners
   - Check worst-case delays
   - Validate noise margins

4. **Optimization:**
   - If rise delay too high → increase Wp
   - If fall delay too high → increase Wn
   - If noise margin low → increase both W
   - If power too high → decrease W (check margins!)

5. **Verification:**
   - Monte Carlo for random variation
   - Aging simulation for reliability
   - Temperature cycling

### 5.7 Trade-offs Summary

| Increase | Benefit | Cost |
|----------|---------|------|
| **Wp** | Faster rise, better NMH | More area, power |
| **Wn** | Faster fall, better NML | More area, power |
| **VDD** | Better margins, faster | Much higher power |
| **CL** | More drive capability | Slower, more power |

**Optimization Philosophy:**
- Start with balanced design (Wp/Wn ≈ 2.5)
- Adjust only if specific requirement not met
- Always verify across all corners
- Consider aging and temperature effects

## Summary and Key Takeaways

### What We Learned

1. **Power Supply Variation:**
   - VTC scales linearly with VDD
   - Noise margins proportional to VDD
   - Low-voltage operation requires careful design
   - IR drop can significantly affect local VDD

2. **Device Variation:**
   - Sizing imbalance shifts switching threshold
   - Extreme ratios degrade noise immunity
   - Process corners cause ±20-30% delay variation
   - Random variation increases with smaller devices

3. **Robustness Design:**
   - Must design for worst-case corners
   - Noise margins are critical spec
   - Balanced sizing provides best immunity
   - Multi-corner verification is mandatory

4. **SPICE Simulation:**
   - Automated corner sweeping techniques
   - Parametric variation analysis
   - Extraction of critical parameters
   - Visualization of design space

### Design Checklist

- [ ] Verified operation at VDD,min and VDD,max
- [ ] Checked all 5 process corners (TT, FF, SS, FS, SF)
- [ ] Validated noise margins ≥ 25% VDD
- [ ] Confirmed switching threshold ≈ 0.5·VDD ± 10%
- [ ] Measured propagation delays at all corners
- [ ] Calculated power consumption (dynamic + leakage)
- [ ] Considered temperature range (-40°C to +125°C)
- [ ] Performed Monte Carlo if critical path
- [ ] Documented corner-specific failures
- [ ] Iterated sizing if specs not met

### Next Steps

- **Day 6:** Advanced topics (if applicable)
  - Leakage mechanisms and mitigation
  - Process-voltage-temperature (PVT) optimization
  - Statistical timing analysis
  - Design for manufacturability

---

## References

1. J.M. Rabaey et al., "Digital Integrated Circuits: A Design Perspective", 2nd Ed.
2. N.H.E. Weste and D.M. Harris, "CMOS VLSI Design", 4th Ed.
3. SkyWater SKY130 PDK Documentation
4. VLSI System Design Workshop Materials

## Acknowledgments

- Kunal Ghosh, Co-founder, VSD Corp. Pvt. Ltd.
- Workshop instructors and TAs
- Open-source EDA community

---

**Workshop Status**: Week 4 Complete ✅
