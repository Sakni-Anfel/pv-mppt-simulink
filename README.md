# PV Array MPPT Simulation — Simulink/MATLAB

Simulation of a photovoltaic array with Maximum Power Point Tracking (MPPT) using the Incremental Conductance algorithm, implemented in MATLAB/Simulink with Simscape Electrical (formerly SimPowerSystems).

---

## What it does

The model simulates a complete PV power conversion chain:

1. **PV Array** — models a 47×10 cell array (47 strings in parallel, 10 modules in series) with realistic I-V and P-V characteristics
2. **MPPT controller** — Incremental Conductance algorithm implemented as a MATLAB Function block; continuously tracks the maximum power point by adjusting a voltage reference
3. **PI controller** — regulates the converter duty cycle to follow the MPPT voltage reference (Kp = 0.001, Ki = 0.01)
4. **DC-DC converter** — boost-type topology with IGBT/Diode switch, bypass diode, and LC filter (Series RLC branches)
5. **PWM generation** — Repeating Sequence block generates the carrier waveform; duty cycle is updated by the PI output

---

## System parameters

| Parameter | Value |
|---|---|
| Array configuration | 47 strings × 10 modules |
| Cells per module | 60 |
| Open-circuit voltage (Voc) | 36.3 V |
| Short-circuit current (Isc) | 7.84 A |
| Maximum power per module (Pm) | 213.15 W |
| Voltage at max power (Vm) | 29 V |
| Current at max power (Im) | 7.35 A |
| Irradiance sweep | 100 / 500 / 1000 W/m² |
| Temperature sweep | 25°C / 45°C |
| IGBT on-resistance | 1 mΩ |
| Diode forward voltage | 0.8 V |

---

## MPPT algorithm (Incremental Conductance)

The MATLAB Function block implements the standard Incremental Conductance logic:

```matlab
P = V * I;
dv = V - Vold;
dp = P - Pold;

if dp < 0
    if dv < 0 → increase Vref
    else       → decrease Vref
else
    if dv < 0 → decrease Vref
    else       → increase Vref
```

`Vref` is clamped between `Vrefmini` and `Vrefmax` to prevent instability.  
Persistent variables (`Vold`, `Pold`, `Vrefold`) preserve state between time steps.

---

## Model structure

```
PV Array (47s×10p)
    ├── V, I measurements
    │       └── MPPT MATLAB Function → Vref
    │                   └── PI Controller → duty cycle
    │                               └── PWM (Repeating Sequence + comparator)
    │                                           └── IGBT/Diode switch
    └── Boost converter (LC filter + bypass Diode) → Load
```

Scopes log: PV voltage (V_PV), PV current (I_PV), output voltage, and duty cycle.

---

## Requirements

- MATLAB R2020b or later  
- Simulink  
- Simscape Electrical (formerly SimPowerSystems)

---

## How to run

1. Open `pv_array_solar.slx` in MATLAB/Simulink
2. Run the simulation (Ctrl+T or the Run button)
3. Observe results on the four Scope blocks (V_PV, I_PV, output voltage, duty cycle)
4. To test different irradiance/temperature conditions, modify the `Constant` and `Constant1` block values (default: 1000 W/m², 25°C)

---

## Results

Simulation run at 1000 W/m², 25°C, 1 second stop time.

**Model diagram**  
![Model](results/model.png)

**V_PV and I_PV convergence (Scope3)**  
![V and I](results/scope3_vi.png)  
V_PV (orange) converges to ~350 V and I_PV (blue) stabilizes around 290–300 A within ~0.05 s. The yellow trace shows the MPPT voltage reference tracking.

**Output voltage and current (Scope)**  
![Output](results/scope_output.png)  
Output voltage (yellow) reaches ~448 V with fast transient response. Output current (blue) stabilizes around 225 A. The boost converter steps up from the array MPP voltage successfully.

**Power (Scope2)**  
![Power](results/scope2_power.png)  
Array power converges to ~9.15–9.9 ×10⁴ W (~91–99 kW) during the MPPT search phase before settling.

**PWM duty cycle (Scope1)**  
![PWM](results/scope1_pwm.png)  
Steady-state duty cycle visible at ~0.6–1.0 s range, confirming the PI controller has locked onto the correct operating point.

---

## Context

Built as part of an embedded systems and power electronics coursework at ISIMG (Institut Supérieur de l'Informatique et de Mathématiques de Monastir, Gabès).
