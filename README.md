# Road Diet Cost-Benefit Analysis Calculator

**Rutgers–New Brunswick, Edward J. Bloustein School of Planning and Public Policy
Alan M. Voorhees Transportation Center**

A self-contained, client-side HTML tool for evaluating the economic feasibility of 4-lane to 3-lane road diet conversions (two through lanes + two-way left-turn lane). All calculations run in the browser with no server, backend, or installation required.

---

## Overview

A road diet converts a four-lane undivided roadway into a three-lane cross-section comprising two through lanes and a center two-way left-turn lane (TWLTL). This reconfiguration typically reduces vehicle conflicts and crash frequency. The calculator quantifies the trade-off between:

- **Benefits** — monetized reduction in crash costs (using USDOT Value of Statistical Life parameters)
- **Costs** — capital restriping cost plus present-value travel delay from any speed reduction

Outputs include Net Present Value (NPV), Benefit-Cost Ratio (BCR), an annual year-by-year table, and a break-even crash-reduction threshold. A print/PDF summary is also available.

---

## Getting Started

No installation or build step is required.

1. Save or download `index.html`.
2. Open the file in any modern web browser (Chrome, Firefox, Safari, Edge).
3. All inputs, calculations, and outputs are handled locally in the browser.

---

## Inputs

### Project

| Field | Description |
|---|---|
| Project name | Optional label used in the printed PDF summary |

### Road characteristics

| Field | Default | Notes |
|---|---|---|
| Road length (miles) | 1.0 | Segment length subject to the road diet |
| Initial speed limit (mph) | 40 | Pre-conversion posted speed |
| Target speed limit (mph) | 35 | Post-conversion posted speed |
| AADT (veh/day) | 12,000 | Average Annual Daily Traffic |
| Commercial traffic (%) | 10 | Share of trucks/freight; delay is costed at the commercial VOT rate |
| Analysis period (years) | 20 | Evaluation horizon; 5–40 years accepted |
| Project opening year | 2024 | Year costs begin escalating from the 2024 base; must be ≥ 2024 |
| Discount rate (%) | 3.0 | Real discount rate for present-value calculations |

### Crash data

Crash counts should be **average annual values**; five-year averages are recommended to smooth year-to-year variation. Decimals are accepted.

The calculator supports two input scales. Both panels are **linked**: editing one automatically converts and updates the other using the FHWA AIS-to-KABCO probability matrix.

#### KABCO scale

| Severity code | Description | Default unit cost (2024 $) |
|---|---|---|
| K | Fatal | $13,700,000 |
| A | Incapacitating injury | $1,302,300 |
| B | Non-incapacitating injury | $256,300 |
| C | Possible injury | $122,400 |
| O | Property damage only | $5,500 |

#### AIS scale

| AIS level | Description | Default unit cost (2024 $) |
|---|---|---|
| AIS 6 | Unsurvivable | $13,700,000 |
| AIS 5 | Critical | $8,440,000 |
| AIS 4 | Severe | $5,820,000 |
| AIS 3 | Serious | $1,560,000 |
| AIS 2 | Moderate | $256,300 |
| AIS 1 | Minor | $122,400 |
| AIS 0 | Property damage only | $5,500 |

Unit costs for both scales can be expanded and edited inline; a **Reset to USDOT defaults** button restores original values.

The **"Calculate using"** radio button controls which scale's values are used for the cost calculation, independent of which panel is currently displayed.

### Cost parameters

| Field | Default | Notes |
|---|---|---|
| Capital cost — total project ($) | $50,000 | One-time Year 1 expense; not discounted |
| Personal VOT ($/person-hr) | $21.10 | USDOT 2023 personal travel time value; escalates 1.6%/yr |
| Commercial VOT ($/veh-hr) | $32.10 | USDOT 2023 commercial/truck travel time value; escalates 1.6%/yr |
| Vehicle occupancy (persons/veh) | 1.2 | Applied to personal vehicles only to convert veh-hrs to person-hrs |

---

## Methodology

### Annual crash cost (baseline)

The baseline annual crash cost is the dot product of crash counts and unit costs for the active scale:

**KABCO:**
```
annualCrashCost = (K × $13,700,000) + (A × $1,302,300) + (B × $256,300)
                + (C × $122,400)    + (O × $5,500)
```

**AIS:**
```
annualCrashCost = (AIS6 × $13,700,000) + (AIS5 × $8,440,000) + (AIS4 × $5,820,000)
                + (AIS3 × $1,560,000)  + (AIS2 × $256,300)   + (AIS1 × $122,400)
                + (AIS0 × $5,500)
```

### VSL escalation and discounting

Unit costs are expressed in 2024 dollars. For each year `t` of the analysis period, the crash cost is escalated and discounted to present value:

```
PV_crash(t) = annualCrashCost
            × (1 + 0.0107)^(yearsToOpen + t)   [VSL escalation: 1.07%/yr]
            ÷ (1 + discountRate)^t              [discount to present value]
```

`yearsToOpen = openingYear − 2024`

Summing across all years yields **pvCrashFull** — the present value of eliminating crashes based on the percent selected in the slider.

### Travel delay cost

Delay is calculated from the change in travel speed (v₁ → v₂):

```
delayVehHrs/yr = AADT × 365 × length × max(0, 1/v₂ − 1/v₁)
```

Personal and commercial vehicle streams are separated and costed independently, then escalated (VOT at 1.6%/yr) and discounted each year before summing to the total delay PV.

### AIS ↔ KABCO conversion

When crash counts are entered on one panel, the linked panel is updated using the FHWA AIS-to-KABCO probability matrix. Each AIS level maps to a probability distribution across the five KABCO categories. The forward transform (AIS → KABCO) applies this matrix directly; the reverse (KABCO → AIS) uses a proportional back-allocation based on column weights.

### Summary metrics

| Metric | Formula |
|---|---|
| Annual crash cost (baseline) | Σ(count × unit cost) |
| Crash reduction benefit PV | pvCrashFull × reduction% |
| Total cost PV | capital cost + PV(delay) |
| NPV | PV(benefit) − total cost PV |
| BCR | PV(benefit) ÷ total cost PV |
| Break-even threshold | total cost PV ÷ pvCrashFull × 100% |

Capital cost is a one-time Year 1 expense and is not discounted.

---

## Outputs

### Summary tiles

Real-time metric cards display:

- Capital cost (Year 1)
- Personal and commercial delay cost (Year 1 and full PV)
- Total cost (PV)
- Annual crash cost (baseline)
- Crash reduction benefit PV at 100% reduction

A break-even callout shows the minimum crash reduction percentage required for BCR ≥ 1.

### Scenario slider

A slider (0–100%) lets users explore different crash reduction assumptions. FHWA research finds road diets typically reduce crashes **19–47%**, with a central estimate of **~29%**. The slider updates NPV, BCR, and the annual table in real time.

### Year-by-year table

Columns: Year · Crash Reduction Benefit (PV) · Delay Cost (PV) · Capital Cost · Present Value · Cumulative NPV.

Year 1 includes the capital cost. All values are in present-value dollars.

### Print / PDF summary

The **Print / Save PDF Summary** button opens a print-ready page in a new tab. Using the browser's "Save as PDF" option produces a formatted report containing all inputs, cost/benefit summary, break-even note, scenario results, and the year-by-year table.

---

## Parameters and data sources

| Parameter | Value | Source |
|---|---|---|
| KABCO unit costs | See table above | USDOT 2024 $ |
| AIS unit costs | See table above | USDOT 2024 $ |
| VSL escalation rate | 1.07%/yr | USDOT guidance |
| VOT escalation rate | 1.60%/yr | USDOT guidance |
| Personal VOT | $17.90–$21.10/hr | USDOT 2023 |
| Commercial VOT | ~$32.10/veh-hr | USDOT 2023 |
| AIS→KABCO matrix | FHWA probability distribution | AIS_to_KABCO_matrix.xlsx (FHWA) |
| Typical crash reduction | 19–47%; central ~29% | FHWA road diet research |
| BCA framework | — | [USDOT Benefit-Cost Analysis Guidance](https://www.transportation.gov/mission/office-secretary/office-policy/transportation-policy/benefit-cost-analysis-guidance) |

---

## Technical notes

- The tool is a **single HTML file** with no external dependencies and no network requests. It runs entirely offline once downloaded.
- All calculations are triggered automatically on any input change (120 ms debounce).
- The pop-up blocker must allow pop-ups from the file's origin for the Print/PDF feature to work.
- Light and dark mode are supported automatically via CSS `prefers-color-scheme`.
- The tool is mobile-responsive and touch-accessible.

---

## Limitations

- The delay model assumes a uniform speed change across the full segment length. Intersection delay is not modeled.
- Crash count inputs are treated as stationary averages; traffic growth over the analysis period is not modeled.
- The AIS ↔ KABCO conversion uses a fixed national probability matrix and may not reflect local crash patterns.
- The calculator does not model pedestrian or cyclist safety benefits, property value impacts, or induced demand effects.

---

## License and attribution

Developed by the **Alan M. Voorhees Transportation Center**, Edward J. Bloustein School of Planning and Public Policy, Rutgers–New Brunswick.

When citing or adapting this tool, please reference the Voorhees Transportation Center and the USDOT guidance documents listed above.
