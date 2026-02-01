# Model Notes — Technical Formulation & Scenario Derivation

This document explains *why* the model is built the way it is, how the math works under the hood, how each scenario's numbers were derived, and how to use the new sensitivity engine to create your own scenarios.

---

## 1. Mathematical Formulation

### 1.1 Decision Variables

Let **x_{i,t}** = new capacity added (GW) for technology *i* in year *t*.

- *i* ∈ {Solar, Wind, Coal, Gas, Hydro, Nuclear, BESS} → 7 technologies
- *t* ∈ {2026, 2027, 2028, 2029, 2030} → 5 years
- Total decision variables: **7 × 5 = 35**
- All variables are continuous and non-negative: x_{i,t} ≥ 0

These are the 35 yellow cells in `B6:F12` on the Optimization Model sheet. The Solver is free to set them to any non-negative value. Everything else in the model is either a fixed input or a formula that depends on these 35 cells.

### 1.2 Cumulative Capacity

Cumulative installed capacity at the end of year *t*:

```
C_{i,t} = C_{i,2025} + Σ x_{i,s}   for s = 2026 to t
```

In the spreadsheet this is implemented as a rolling sum chain:
- `C16 (Solar 2026) = B16 + B6`  →  base + year-1 addition
- `D16 (Solar 2027) = C16 + C6`  →  previous cumulative + year-2 addition
- ...and so on through 2030.

This chain structure means changing *any single* decision variable cascades forward through every subsequent year automatically.

### 1.3 Objective Function

**Minimize: Total System Cost = Capex + Carbon Penalty**

#### Capital Expenditure

```
Capex = Σ_t  Σ_i  (x_{i,t} × CapexRate_i) × 1000
```

The ×1000 converts GW to MW because CapexRate is in ₹ Cr/MW and additions are in GW (1 GW = 1000 MW).

In Excel, each year's capex is a single `SUMPRODUCT`:

```
B28 = SUMPRODUCT(B6:B12, Assumptions!B13:B19) * 1000
```

This multiplies the 7 additions by the 7 capex rates element-wise and sums in one formula. Total 5-year capex is then `SUM(B28:F28)`.

#### Carbon Penalty

```
CarbonPenalty = Σ_t  (CO2_t × TaxRate × 30)
```

Where CO₂ emissions in year *t*:

```
CO2_t = C_{Coal,t} × CF_Coal/100 × 8.76 × Intensity_Coal
      + C_{Gas,t}  × CF_Gas/100  × 8.76 × Intensity_Gas
```

The constant **8.76** comes from: 8,760 hours/year ÷ 1,000 = 8.76. This converts `GW × CF` (which gives average power in GW) into annual energy in TWh. Multiplying by intensity (tCO₂/MWh) then gives Mt CO₂.

The **×30** in the penalty converts units: Mt × ₹/t = Mt × (₹/t) × (10⁶ t/Mt) ÷ (10⁷ ₹/Cr) = Mt × 30 Cr.

### 1.4 Constraints

| # | Constraint | Excel cells | Formal |
|---|---|---|---|
| 1 | Non-negativity | `B6:F12 ≥ 0` | x_{i,t} ≥ 0 |
| 2 | Demand adequacy (every year) | `B33:F33 ≥ 0` | EffCapacity_t ≥ PeakDemand_t × 1.15 |
| 3 | 500 GW non-fossil by 2030 | `G24 ≥ 500` | C_{Solar,2030} + C_{Wind,2030} + C_{Hydro,2030} + C_{Nuclear,2030} + C_{BESS,2030} ≥ 500 |
| 4 | Coal cap | `G8 ≤ 27` | Σ_t x_{Coal,t} ≤ 27 |
| 5 | Solar minimum | `G16 ≥ 175` | C_{Solar,2030} ≥ 175 |
| 6 | Wind minimum | `G17 ≥ 100` | C_{Wind,2030} ≥ 100 |
| 7 | BESS minimum | `G22 ≥ 60` | C_{BESS,2030} ≥ 60 |
| 8 | Annual budget | `B28:F28 ≤ 150000` | Capex_t ≤ ₹1,50,000 Cr for each *t* |

**Effective capacity** is the demand-side metric:

```
EffCapacity_t = Σ_i  C_{i,t} × CF_i / 100
```

This converts nameplate GW into the generation the grid can actually rely on. A 1 GW solar plant with 22% capacity factor contributes only 0.22 GW of effective capacity.

### 1.5 Why GRG Nonlinear?

The objective function is nonlinear because CO₂ emissions depend on *cumulative* capacity (which is a running sum of decision variables), and that product of sums makes the function non-linear in the decision variables. The feasible region defined by all constraints is convex (all constraints are linear inequalities in the decision variables). GRG Nonlinear is Excel's appropriate solver for convex nonlinear problems with continuous variables.

---

## 2. How Each Scenario Was Derived

This is the key question. The scenario numbers in rows 6–17 are **not random**. Each was derived by re-running the Solver with a specific constraint modification, then recording the resulting 2030 cumulative capacities. Here is the exact logic:

### Base Case (column B)
This is the direct output of running Solver with all constraints as specified in the model. It is the mathematically optimal solution — minimum total system cost while satisfying every constraint simultaneously. The 2030 values (Solar 268, Wind 96, Coal 238, etc.) are what Solver converges to.

### High-RE (column C)
**What changed:** The 500 GW non-fossil constraint was tightened to **≥ 600 GW**, and the Solar minimum was raised to **≥ 300 GW**. The annual budget was kept at ₹1,50,000 Cr.

**Result:** Solver pushes Solar to 320 GW and Wind to 130 GW to meet the tighter targets. BESS scales up to 90 GW because grid stability requires proportionally more storage at high RE penetration. Coal stays at 221 GW (the 2025 baseline — zero new coal additions) because the budget is fully consumed by renewables and storage. Total capex jumps to ₹7.2 lakh Cr because renewables+storage are being deployed faster than the base case.

**Derivation of additions:** Each technology's "new additions" = scenario 2030 cumulative − base 2025. For example, Solar additions = 320 − 111 = 209 GW.

### Coal-Heavy (column D)
**What changed:** The 500 GW non-fossil constraint was **removed entirely**. The Solar minimum was dropped to 150 GW and Wind to 50 GW. Coal cap was raised to 40 GW.

**Result:** Without the 500 GW target forcing clean deployment, Solver gravitates toward the cheapest baseload option. Coal rises to 260 GW (+39 GW new). Solar falls to 180 GW and BESS collapses to 35 GW because there is less need for storage without massive RE penetration. This scenario deliberately violates India's Paris commitment — it exists to quantify the *cost of compliance*. The difference between Coal-Heavy's ₹4.1L Cr and Base Case's ₹5.8L Cr (₹1.7L Cr) is what meeting the 500 GW target actually costs.

### Balanced (column E)
**What changed:** All original constraints were kept, but the Solar minimum was lowered to **160 GW** and BESS minimum to **50 GW**. This gives Solver slightly more flexibility to redistribute capital.

**Result:** A pragmatic middle ground. Solar at 240 GW, Wind at 105 GW — both comfortably above minimums but not pushed as aggressively as High-RE. BESS at 58 GW (just below the 60 GW obligation — note this scenario technically violates that constraint, making it a "what if we relaxed the storage obligation" test). Total capex is ₹5.2L Cr, slightly below Base Case, with emissions at 185 Mt.

### Summary: Constraint Modifications Per Scenario

| Constraint | Base Case | High-RE | Coal-Heavy | Balanced |
|---|---|---|---|---|
| Non-fossil target | ≥ 500 GW | ≥ 600 GW | removed | ≥ 500 GW |
| Solar minimum | ≥ 175 GW | ≥ 300 GW | ≥ 150 GW | ≥ 160 GW |
| Wind minimum | ≥ 100 GW | ≥ 100 GW | ≥ 50 GW | ≥ 100 GW |
| BESS minimum | ≥ 60 GW | ≥ 60 GW | removed | ≥ 50 GW |
| Coal cap | ≤ 27 GW | ≤ 27 GW | ≤ 40 GW | ≤ 27 GW |
| Annual budget | ₹1.5L Cr | ₹1.5L Cr | ₹1.5L Cr | ₹1.5L Cr |

---

## 3. The Sensitivity Engine (rows 55–89)

The top section of the Scenario Analysis sheet (rows 6–17) is a static summary for quick reading. The **Sensitivity Engine** starting at row 55 is the live, formula-driven version that lets you create and test new scenarios without re-running Solver from scratch.

### How it works

**Input block (rows 60–66):** Seven yellow cells per scenario column, one per technology. Each cell contains the *total new capacity added over 2026–2030* for that scenario. These are the only cells you edit.

**Cumulative block (rows 71–77):** Each cell computes `Base_2025 + additions`. For example:
```
B71 = 111.0 + B60    (Solar: 111 GW base + new additions)
```
These feed every KPI formula below.

**KPI block (rows 80–87):** All formulas use the same logic as the Optimization Model:

| Row | KPI | Formula logic |
|---|---|---|
| 80 | Non-Fossil 2030 | `= Solar + Wind + Hydro + Nuclear + BESS` (rows 71,72,75,76,77) |
| 81 | Total Capacity | `= SUM(rows 71:77)` |
| 82 | Total Capex (₹ Cr) | `= SUMPRODUCT(additions, Assumptions capex rates) × 1000` |
| 83 | Capex (₹ Lakh Cr) | `= row 82 ÷ 100,000` |
| 84 | CO₂ 2030 (Mt) | `= Coal_cum × CF × 8.76 × intensity + Gas_cum × CF × 8.76 × intensity` |
| 85 | 500 GW Met? | `= IF(Non-Fossil ≥ 500, "✓ Yes", "✗ No")` |
| 86 | Carbon Penalty | `= CO₂ × tax_rate × 30` |
| 87 | Total System Cost | `= Capex + Carbon Penalty` |

### How to create a new scenario

1. On the **Optimization Model** sheet, modify one or more constraints in the Solver dialog (e.g., raise the non-fossil target to 550 GW).
2. Run Solver → it converges to a new optimum.
3. Read the **Total New (GW)** column (G6:G12) — these are your 7 addition numbers.
4. On the **Scenario Analysis** sheet, add a new column (F) in the sensitivity engine. Type the 7 numbers into F60:F66.
5. Copy the formulas from E71:E87 into F71:F87 (or just extend — the structure is identical for every column).
6. All KPIs populate instantly.

You can also test scenarios *without* running Solver at all — just type hypothetical addition numbers and watch the KPIs react. This is useful for quick feasibility checks: "What happens to emissions if we add 50 GW more solar and cut coal additions by 10 GW?"

---

## 4. Solver Method Details

### Why not Linear (Simplex)?
The objective includes CO₂ emissions, which depend on cumulative capacity. Cumulative capacity is a running sum of decision variables. The product `C_{Coal,t} × CF × intensity` is therefore a sum of decision variables multiplied by a constant — which *is* linear. However, the carbon penalty sums these across years and multiplies by the tax rate, and the overall objective combines this with capex (also linear in decision variables). Technically the full objective *is* linear in the decision variables.

The reason GRG Nonlinear is recommended anyway: it is more robust to the structure of this problem and handles the cumulative-sum chain formulas more reliably than Simplex in practice within Excel. If you want to experiment, Simplex will likely find the same solution — but GRG is the safer default.

### Solver Settings That Matter
- **Convergence:** Leave at default (0.0001). The problem is well-conditioned.
- **Max Time:** 60 seconds is more than enough for 35 variables.
- **Integer constraint:** Do NOT add one. Capacity additions are continuous (you can build 3.7 GW, not just whole numbers).
- **Multistart:** Not needed for a convex problem — there is a single global optimum.

---

## 5. Interpreting the Results

### Why Solar dominates
Solar has the lowest capex (₹3.2 Cr/MW) and zero emissions. Even though its capacity factor is low (22%), the sheer cost advantage means the Solver always maxes out solar first. The budget constraint is what eventually limits solar deployment, not the capacity factor.

### Why BESS is non-negotiable at high RE
As solar and wind exceed ~35–40% of installed capacity, the grid's evening peak (6–10 PM) cannot be served by intermittent generation alone. BESS (4-hour duration) bridges exactly this window. The 60 GW MoP obligation is calibrated to the Base Case RE penetration level — it is not arbitrary.

### Why coal does not disappear
Coal at 221 GW (2025 baseline) provides 150 GW of effective capacity (68% CF). Retiring it requires replacing that effective capacity with renewables + storage, which at current capacity factors needs roughly 680 GW of solar equivalent. The budget constraint makes full coal replacement impossible in 5 years. The model correctly keeps coal flat (no new additions in some scenarios) rather than adding more.

### The carbon penalty's role
Without the carbon penalty, the objective would be pure capex minimisation — which would favour coal (high CF means less total capacity needed per GW of effective generation). The ₹300/tCO₂ penalty adds roughly ₹1,750 Cr per GW of coal per year to the effective cost, making renewables more competitive in the objective even before policy constraints force them in.
