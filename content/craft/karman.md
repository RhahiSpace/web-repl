---
title: "Karman Rockets"
date: 2023-05-30
mathjax: true
draft: false
tags: [craft, karman]
---

{{<quote text="Low-tech, two-stage sounding rockets." class="center">}}

{{<image src="/craftfiles/Karman 2.png"
    alt="rocket" maxw="8em" class="right"
    title="Karman 2">}}
## Karman 1


{{<download src="/craftfiles/Karman 1.craft" name="Download">}}

- Two stage rockets with two kick stages.
- Designed to reach 140 km altitude and retrieve core.
- Comfortably reaches the Karman line and more.
- Kick stage engine: Tiny Tim Booster
- Stage 1 engine: WAC-Corporal

## Karman 2

{{<download src="/craftfiles/Karman 2.craft" name="Download">}}

- Two stage rockets with a single kick stage.
- Kick stage engine: Tiny Tim Booster
- Stage 1 engine: XASR-1
- Stage 2 engine: XASR-1

---

## Engine specifications

### Tiny Tim Booster
    - Solid
    - 1 ignition, cannot be shut down
    - Isp = specific impulse: 202s - 222s
    - mₚ = propellant mass: 66.22 kg
    - T = thrust : 146.6 kN
    - ṁ = mass flow: 67.34 kg/s
    - Residual: 3%
    - Fuel
        - NGNC (ρ = 1.6 kg/l)

### WAC-Corporal
    - Pressure-fed
    - 1 ignition, requires ullage
    - Isp = specific impulse: 195s - 226s
    - τ = rated burn time: 47s
    - T = thrust: 6.7 kN - 7.7 kN (not throttleable)
    - ṁ = mass flow: 3.49 kg/s
    - Residual: 4.85%
    - Effective spool-up time: 0.06s
    - Fuel
        - Anilline22 (ρ = 1.042 kg/l, ratio = 0.40854)
        - IRFNA-III (ρ = 1.658 kg/l, ratio = 0.59146)
        - Nitrogen (ρ = 0.001251 kg/l, ratio = 30.9)

### XASR-1
    - Pressure-fed
    - 1 ignition, requires ullage
    - Isp = specific impulse: 200s - 235.44s
    - τ = rated burn time: 40s
    - T = thrust: 11.7 kN - 13.8 kN (not throttleable)
    - ṁ = mass flow: 5.9608183 kg/s
    - Residual: 4.23%
    - Effective spool-up time: 0.08s
    - Fuel
        - Anilline37 (ρ = 1.0585 kg/l, ratio = 0.3796)
        - IRFNA-III (ρ = 1.658 kg/l, ratio = 0.6204)
        - Nitrogen (ρ = 0.001251 kg/l, ratio = 34.2)
