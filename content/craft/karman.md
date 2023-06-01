---
title: "Karman Rockets"
date: 2023-05-30
mathjax: true
draft: false
tags: [craft, karman]
---

{{<quote text="Low-tech, two-stage sounding rockets." class="center">}}

{{<image src="/craftfiles/Karman 1.png"
    alt="rocket" maxw="7em" class="right-text">}}
## Karman 1-2
{{<quote text="Made of steel. Heavy, but it will fly." class="dark-3 left">}}

{{<download src="/craftfiles/Karman 1.craft" name="Karman 1">}}
{{<download src="/craftfiles/Karman 2.craft" name="Karman 2">}} (should be identical to 1)

- Two stage rockets with a single kick stage.
- Comfortably reaches the Karman line.
- No scientific instruments.
- Kick stage engine: Tiny Tim Booster
- Stage 1 engine: WAC-Corporal

---

{{<image src="/craftfiles/Karman 3.png"
    alt="rocket" maxw="7em" class="right-text">}}

## Karman 3-4

{{<quote text="Let the autopilot handle the ignition timing." class="dark-3 left">}}

{{<download src="/craftfiles/Karman 3.craft" name="Karman 3">}}
{{<download src="/craftfiles/Karman 4.craft" name="Karman 4">}} (mostly identical to 3, with some trims)

- Two stage rockets with two kick stages.
- Designed to reach 140 km altitude and return with experiments.
- Kick stage engines: Tiny Tim Booster
- Stage 1 engine: WAC-Corporal
- Stage 2 engine: WAC-Corporal
- Karman 4 relies on 3 seconds of overburn to reach 140 km.

## Karman 5

{{<quote text="Now with more efficient and powerful XASR-1 engines." class="dark-3 left">}}

{{<download src="/craftfiles/Karman 5.craft" name="Karman 5">}}

- Two stage rockets with one kick stage.
- Kick stage engines: Tiny Tim Booster
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
