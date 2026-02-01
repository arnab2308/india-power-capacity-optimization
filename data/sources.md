# Data Sources & References

Every input in the model is traceable to a published source. This file documents each one, what specific data point was used, and where to verify it.

---

## Demand Projections

| Parameter | Value Used | Source | Notes |
|---|---|---|---|
| Peak Demand 2024 | 250 GW | CEA, PIB Press Release (May 2024) | National record set during summer peak |
| Peak Demand 2020–2023 | 148→207 GW | CEA Monthly Demand Reports | Historical actuals |
| Peak Demand 2025–2030 | 270→355 GW | Ministry of Power, IEA *Electricity 2025* | MoP projects ~355 GW by 2030 |
| Energy Consumption 2024 | 1,694 BU | CEA Annual Energy Statistics | 1 BU = 1 billion units = 1 TWh |

**Key assumption:** Demand growth is non-linear. The 2022→2023 jump (174→207 GW, +19%) reflects post-COVID industrial recovery. The 2029→2030 growth (~4.7%) is the steady-state projection. The model uses year-by-year figures rather than a single CAGR to preserve this structure.

---

## Installed Capacity (Baseline 2020–2025)

| Technology | 2025 Value | Source |
|---|---|---|
| Solar PV | 111 GW | CEA; solar crossed 100 GW milestone in January 2025 (PIB) |
| Onshore Wind | 51.3 GW | CEA Installed Capacity Report |
| Coal | 221 GW | CEA; includes all thermal coal (super-critical and sub-critical) |
| Gas (CCGT) | 24.5 GW | CEA; gas capacity has been flat since ~2014 due to LNG import costs |
| Large Hydro | 48 GW | CEA; excludes small hydro (< 25 MW) |
| Nuclear | 8.8 GW | Department of Atomic Energy; Kakrapar-3 commissioned 2024 |
| BESS | 0.5 GW | CEA; battery storage still nascent but accelerating |

**Historical trend note:** Solar nearly tripled from 38 GW (2020) to 111 GW (2025) — the fastest capacity addition of any technology globally during this period. Coal added only 15 GW in the same window.

---

## Technology Cost Parameters

### Capital Costs

| Technology | ₹ Cr/MW | $/kW equivalent | Source |
|---|---|---|---|
| Solar PV | 3.2 | ~$350/kW | BNEF *New Energy Outlook 2024*; Berkeley IECC (2025) auction data |
| Onshore Wind | 5.2 | ~$550/kW | BNEF 2024; IEA *World Energy Outlook 2024* |
| Coal (Super-Critical) | 11.5 | ~$1,150/kW | CEA project cost estimates; IEA Coal report |
| Gas (CCGT) | 6.5 | ~$650/kW | IEA WEO 2024; consistent with recent NTPC CCGT projects |
| Large Hydro | 16.0 | ~$1,600/kW | CEA; highly variable by site (pumped storage is more expensive) |
| Nuclear | 50.0 | ~$5,000/kW | DAE cost estimates for indigenous PHWR; Kudankulam actuals |
| BESS (4-hr) | 3.8 | ~$400/kWh system | BNEF 2024; falling rapidly (~15-20%/year learning curve) |

**Currency note:** All ₹ figures use an approximate rate of ₹85/USD (2024 average). The model is denominated entirely in Indian Rupees.

### Capacity Factors

| Technology | CF Used | Source | Rationale |
|---|---|---|---|
| Solar PV | 22% | CEA Plant Load Factor data FY2024-25 | National average across all states. Rajasthan/Gujarat are higher (~25%), but the model uses the national figure conservatively. |
| Onshore Wind | 28% | CEA PLF FY2024-25 | Tamil Nadu/Gujarat sites perform better; national average used. |
| Coal (Super-Critical) | 68% | CEA PLF FY2024-25 | Super-critical plants run at higher PLF than the fleet average (~60%) which includes older sub-critical units. |
| Gas (CCGT) | 55% | CEA; adjusted for gas availability | India's gas plants historically under-utilise due to LNG supply constraints. 55% is the design-adjusted figure, not actual PLF (which is lower). |
| Large Hydro | 42% | CEA | Seasonal variation is significant; 42% is the annual average. |
| Nuclear | 80% | DAE annual reports | Indian nuclear plants have historically achieved high PLFs once commissioned. |
| BESS (4-hr) | 20% | IEA; industry benchmarks | 4-hour batteries discharge once per day during evening peak. 20% CF corresponds to ~4.8 hrs of equivalent full-load output per day. |

### CO₂ Intensities

| Technology | tCO₂/MWh | Source |
|---|---|---|
| Coal (Super-Critical) | 0.85 | IEA *CO₂ Emissions from Fuel Combustion*; super-critical efficiency ~42-45% |
| Gas (CCGT) | 0.45 | IEA; CCGT efficiency ~58-62% |
| All others | 0 | Zero direct combustion emissions (lifecycle emissions not modelled) |

---

## Policy Targets & Constraints

| Parameter | Value | Policy Reference |
|---|---|---|
| 500 GW Non-Fossil by 2030 | 500 GW | India's updated Paris NDC (2022); PM Modi's COP26 pledge |
| Max New Coal | 27 GW | CEA: ~27 GW of coal capacity under construction across India as of 2024 |
| Solar ≥ 175 GW | 175 GW | MNRE target; India had 111 GW in Jan 2025, targeting 175+ |
| Wind ≥ 100 GW | 100 GW | IWTMA (Indian Wind Turbine Manufacturers Association) roadmap |
| BESS ≥ 60 GW | 60 GW | Ministry of Power Energy Storage Obligation (2024) |
| Carbon Tax | ₹300/tCO₂ | Indicative; within India's proposed carbon pricing range. GST on coal currently acts as an implicit carbon tax at a lower effective rate. |
| Reserve Margin | 15% | CEA adequacy standard; ensures grid resilience against plant outages |
| Annual Budget | ₹1,50,000 Cr | Approximate annual power sector capex (~$17.5B). India's total power sector investment was ~₹1.7–2.0 lakh Cr in FY2024. |

---

## Key Reports Referenced

1. **IEA, *Electricity 2025*** — Global electricity market outlook with India-specific demand forecasts.
2. **CEEW, *How Can India Meet Its Rising Power Demand? Pathways to 2030* (2025)** — Comprehensive India-specific capacity planning analysis from the Council on Energy, Environment and Water.
3. **Central Electricity Authority (CEA)** — Monthly installed capacity reports, PLF data, and project pipeline tracking. Primary source for all historical capacity and demand figures.
4. **Press Information Bureau (PIB)** — Official government press releases on power sector milestones (solar 100 GW crossing, demand records).
5. **Bloomberg New Energy Finance (BNEF), *New Energy Outlook 2024*** — Technology cost benchmarks and learning curve projections for solar, wind, and battery storage.
6. **IEA, *World Energy Outlook 2024*** — Global technology cost assumptions and emissions factors.
7. **Berkeley IECC, *Plummeting Solar+Storage Auction Prices in India* (2025)** — Documents India's solar auction price trajectory, supporting the ₹3.2 Cr/MW solar capex assumption.
8. **IBEF, *Power Sector in India*** — Industry overview and investment trend data.

---

## What Is NOT Modelled (and Why)

| Factor | Why it is excluded |
|---|---|
| Regional variation in capacity factors | Would require state-level disaggregation; adds complexity without changing the national-level strategic conclusions |
| Transmission constraints | Inter-state grid congestion is real but requires a separate network flow model |
| Land availability | Solar and wind are land-constrained in practice; a binding constraint in a more detailed model |
| Seasonal/hourly dispatch | This is a capacity planning model, not a dispatch model. Hourly simulation would require 8,760 time steps per year |
| Plant retirement | The model assumes no retirements. In reality, some older coal plants will be decommissioned. Adding retirement would *reduce* the coal baseline and make the 500 GW target easier to meet. |
| Learning curves | Solar and BESS costs are falling 15-20%/year. Using static costs makes this model *conservative* — actual future costs will likely be lower. |
| Demand-side management | EVs, efficiency programs, and demand response could reduce peak demand. Not modelled here. |
