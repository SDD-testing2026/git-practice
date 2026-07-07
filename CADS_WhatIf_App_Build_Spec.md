# CADS Workforce What-If App — Build Spec

**Purpose:** hand-off so a fresh session can continue the app without re-deriving history. Pair this file with **`CADS_Planning_Dataset_v9.xlsx`** (data source) and **`cads_whatif.html`** (the live app).

---

## 1. What the app is

A **capacity-planning what-if tool** for CADS Engineering 2027 planning — **not** a compliance dashboard. The user sets **target ratios per team**, the app compares them to **actual composition** (in fractional FTE), and surfaces **capacity gaps** (understaffed) and **opportunities** (overstaffed), with **cost** implications. Core loop: *configure targets → per-team / per-RT / portfolio gaps + cost → compare scenarios.*

**App file:** `/Users/holly.ma/projects/git-practice/cads_whatif.html` — single self-contained HTML (~226KB), all data/CSS/JS inline, no external dependencies. Share the file directly; anyone can open it in a desktop browser.

---

## 2. Data source — `CADS_Planning_Dataset_v9.xlsx`

- **Team Relations** (535 rows) — the member-level fact table. App reads this.
- **Teams** (86 rows) — team dimension: Team Number, Name, **Description**, **Parent Team**, **Team Type** (Café/Decaf), Cafe level, **Team Location**, Portfolio, Delivery Stream, Release Train. Join to Team Relations on **Team Number**.
- Data Quality Summary, Assumptions Log, Gaps & Issues, SOWs, CG Updates, PPM — reference only.

### Team Relations key columns
| Column | Meaning |
|---|---|
| Portfolio → Delivery Stream → Release Train → Team Name | 4-level hierarchy |
| Team Number | join key to Teams dim |
| Parent Team / Team Type / Team Location | from team dimension (denormalized for convenience) |
| Cafe level | Scrum Team / Kanban / AIT Team / Release Train / Delivery Stream |
| **Is Delivery Team** | TRUE = Café team at Scrum/Kanban/AIT level (50 teams) — the ratio scope |
| Team Role | functional role on the team |
| **Functional Area** | Engineering / Product / Data Science / Product Analytics / Solution Delivery / Other (Scrum Master = **Solution Delivery**) |
| **RTE** | Release Train Engineer for the row's RT (Scrum Masters report here) |
| **Is Extended Team** | TRUE = RT/DS Extended leadership team (not a delivery team) |
| **Is People Manager** | TRUE = Mgr Software Engineering / Engineering Leader (leadership, not delivery capacity) |
| Team Member | person, or `TBD` = open slot |
| Title / **Team Member Type** | `"Full-Time Employee"` or `"Contingent Worker"` (not "FTE") |
| **Allocation %** | capacity share for this row; sums to 100% per person = the FTE weight |
| Vendor | contractor vendor |
| Cost Confidence | Verified / Imputed / Unpriced / Open Slot |
| Verified / Estimated Annual Cost | full annual cost @100% |

---

## 3. Hierarchy & team taxonomy

**Roll-up:** Member → **Team** → **Release Train** → **Delivery Stream** → Portfolio.

**72 total teams tracked** (in `RAW.allTeamsFilter`), split into 3 groups:
- **Delivery Team** (50): `Is Delivery Team = TRUE` — ratio scope, shown in Delivery Team Scorecards tab.
- **RT / DS Core** (13): RT and Delivery Stream core teams — leadership/operational, no ratio targets.
- **Extended / Leadership** (9): `Is Extended Team = TRUE` — functional-area leaders above team-manager line.

**Ratio scope = `Is Delivery Team` (50 Café Scrum/Kanban/AIT teams).** Note: 4 Decaf operational teams (Data Marketplace & Snowflake Operations, Mischief Managed, Mischief Managed 2, VitaC) excluded from ratio targets.

**Where each role sits — all four ratio roles sit ON the delivery teams:**
- **Engineers, Product Owners, Scrum Masters, Engineering Managers** all sit on delivery-team rows → **all four ratios measured per delivery team** in fractional FTE (Allocation %).
- **Engineering Managers** (`Mgr Software Engineering`, `Is People Manager`=TRUE) are the team's people-manager, shared ~3 teams each (≈0.32 mgr-FTE/team across 48 of 50 teams).
- **Scrum Masters** belong to **Solution Delivery** and report to the **RTE** of their RT.
- **Extended teams** (`Is Extended Team`=TRUE, 9 teams) are a **separate leadership layer** — tracked for cost but **not part of the four ratios**.

---

## 4. Locked conventions the app MUST honor

1. **Ratio scope = `Is Delivery Team` = TRUE** (50 Café delivery teams).
2. **Fractional FTE counting** — a person's contribution = Allocation %, not a whole head.
3. **Cost = Annual Cost × Allocation %.** Portfolio = simple sum. Never sum gross @100%.
4. **Contractor annual = already in cost column (annual, not hourly × 1920).** FTE = burdened annual rate.
5. **Scrum + AIT both count** (22 people on both during transition).
6. **TBD = open slot**, 100% to one team, no cost — planned headcount. TBD keys use `name__teamNumber` to avoid collision.
7. **Confirmed-$0** rows (offboarding/prepaid/temp) = headcount, $0 budget.
8. **Cost confidence:** Verified (506), Imputed (5), Unpriced (4), Open Slot (20). Imputed/Unpriced carried as flagged (~$554K).
9. **Team Member Type** value is `"Full-Time Employee"` (not `"FTE"`) — check exact string when filtering.

---

## 5. Role → ratio buckets

| Bucket | Team Roles | Current FTE | Measured at |
|---|---|---|---|
| **Engineer** | Software/Data/Platform/Delivery-Stream Engineer, Dev Ops, Technical Lead, Software Test Engineer | ~265 | per delivery team |
| **Product Owner** | Product Owner | ~28.7 | per delivery team |
| **Engineering Manager** | Mgr Software Engineering, Engineering Leader | ~17.9 | per delivery team (coverage) |
| **Scrum Master** | Scrum Master | ~24 | per delivery team |
| **Other (tracked, not ratio'd)** | QA, Analysts, Delivery Mgr, Product Mgr, Intern, etc. | ~24 | — |

---

## 6. Target ratios

**Defaults (per delivery team):** 4.0 Engineers, 0.5 Product Owner, 0.25 Engineering Manager, 0.25 Scrum Master. **User-configurable in left panel.**

---

## 7. Current-state baseline

50 delivery teams, 11 Release Trains (~5–6 teams/RT). Org-wide vs default targets: Engineers ~5.3/team (target 4), PO ~0.57, Eng Mgr ~0.36, SM ~0.48.

**Locations** (for app filtering/cards): Atlanta (32), Ho Chi Minh City (9), Remote (7), Irvine (5), Dallas (5), Austin (4), Burlington (3), Gurugram (4).

**Blended rates (verified actuals — allocated cost ÷ allocated FTE):**
- Engineer: $140,951 | Product Owner: $178,016 | Eng Manager: $274,370 | Scrum Master: $125,394

**Offshore blended rates (contractors only):**
- Ho Chi Minh City (HCMC): $62,303 | Gurugram (IND): $74,936

**Portfolio totals:**
- Delivery Teams Spend: ~$47.9M | Total Spend (all 72 teams): ~$71.5M
- FTE salary total (for merit calc): $33,779,731
- Engineers/Mgr FTE ratio: 14.1x

---

## 8. App features (implemented)

### Tabs
1. **Delivery Team Scorecards** — 50 team cards, each showing actual FTE vs target by role with progress bars, gap/surplus/mixed badge, spend, and gap cost.
2. **By Release Train** — RT and Delivery Stream rollup tables. Gap columns use **gross gaps only** (shortfalls only; surplus excluded from totals, not netted). Total Gap FTE shows 2 decimal places.
3. **Budget Gap** — interactive person/team removal explorer. Budget input fields (cloud, token, merit, other). Merit auto-calculates as FTE salaries × 3.5%, recalculates on any removal. Removal is full-person (affects all teams). Suggestion buttons for offshore removal.
4. **Scenario Compare** — save/compare named what-if scenarios (in-memory).
5. **Read Me** — data assumptions, navigation guide, KPI definitions.

### KPI strip (top of page, 6 columns)
| KPI | Definition |
|---|---|
| **Teams** | Count of delivery teams shown (50 default) |
| **PO Gap** | Total Product Owner FTE gap across all delivery teams (shortfalls only, gross) |
| **Cost to Close PO Gap** | PO Gap × blended PO rate ($178,016) |
| **Gap to PLRP** | Cost to Close PO Gap + all Budget Gap inputs (cloud + token + merit + other) |
| **Delivery Teams Spend** | Sum of allocated costs for 50 delivery teams only |
| **Total Spend** | Sum of allocated costs across all 72 teams (full org labor) |

### Left panel
- **Target Ratios** — user-editable inputs (Engineers, PO, Eng Mgr, SM)
- **Engineers / Mgr FTE** — computed ratio display (14.1x)
- **Actual Avg Ratios** — read-only, live-updating; reflects removed persons from Budget Gap tab
- **Blended Rates** — all-role rates + offshore contractor rates
- **Scenarios** — save/restore named scenarios

### Filters (toolbar)
- Release Train, Delivery Stream, Location, **Team** (all 72 teams in 3 optgroups), Status (All/Gap/Surplus/Mixed)

---

## 9. Key implementation details

### In-memory state
```javascript
removedPersonIds  // Set of personId strings removed in Budget Gap tab
bgCloud, bgToken, bgOther  // Budget gap input values
scenarios  // Array of saved scenario objects
filters    // {rt, ds, loc, team, status}
personMap  // Map of personId → person data
teamPersons // Map of teamNumber → array of persons
```

### Key functions
```javascript
computeActualAvgRatios()  // subtracts removed persons from team FTE totals
computeMerit()            // sum FTE salaries of non-removed × 0.035
computeLaborSavings()     // sum annualCost of removedPersonIds
recalc()                  // recomputes KPIs, cards, RT table, actual ratios
renderBudget()            // renders Budget Gap tab, calls recalc() at end
renderRT(t)               // RT/DS rollup tables, gross gaps only
applyFilters(team, gaps)  // checks rt, ds, loc, team, status filters
openSaveScenario() / confirmSaveScenario()  // inline input (not prompt())
```

### Gross gap logic (RT rollup)
```javascript
// Eng gap: shortfalls only, no netting with surplus
if (gaps.eng < 0) m.engGap += gaps.eng;
// Same for po, mgr, sm
// Total Gap FTE = sum of all role shortfalls (no surplus netting)
```

### Embedded data (`RAW` object in script tag)
- `teams` (50 delivery teams), `persons` (334: 164 FTE + 159 contractors + 11 TBD)
- `rates`, `offshoreRates`, `engPerMgrFTE`, `fteSalaryTotal`, `rts`, `dss`, `locations`
- `allTeamsFilter` (72 teams): `{number, name, group, rt, isDelivery}`

### Re-embedding data after JSON changes
```python
# Use string split approach (not regex) to avoid \u unicode escape issues
start_idx = html.index('const RAW = {')
end_idx = html.index('};', start_idx) + 2
html = html[:start_idx] + 'const RAW = ' + json_str + ';' + html[end_idx:]
```

---

## 10. Carried-forward open data items (non-blocking)
- **5 Imputed** (~$554K): Lalu Thota, Dmytro Ponomarchuk, Crystle Stamper, Pravin Hanagandi, Utkarsh Goyal.
- **4 Unpriced**: Jeffrey Delong (vendor "Hearst"), Daniel Emmanuel ×2, Dimitri Manoukis (no vendor).
- Reconciliation (info): 4 of 6 orphan SOW contractors (~$304K) off-roster; 15 External Labor resources with rates not on a team; 39 unmatched SOW records.

---

## 11. Continue in a new session

Open with: *"Continue building the CADS what-if capacity app. The app is at `cads_whatif.html`. Review the build spec for current state, then [describe next task]."*

**Known issues / potential next steps:**
- Non-delivery teams selected in the Team filter show 0 scorecards (expected behavior — no ratio targets for RT/DS core or extended teams).
- Scenario compare tab could be enhanced with side-by-side diff view.
- Per-team-type ratio targets (AIT teams vs Scrum teams) not yet implemented — currently global only.
