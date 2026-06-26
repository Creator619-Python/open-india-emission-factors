# Open India Emission Factors

An open database of India-specific GHG emission factors for carbon 
accounting, BRSR reporting, and climate disclosure.

**Live database:** https://creator619-python.github.io/open-india-emission-factors  
**License:** CC0 1.0 Public Domain  
**Version:** 1.0 | **Last updated:** June 2026

---

## Why this exists

Indian GHG practitioners have been forced to rely on DEFRA UK and IPCC 
defaults because India-specific emission factors aren't openly 
accessible in one place. This database fixes that.

---

## What's included

| Category | Coverage |
|---|---|
| Grid electricity | National, 5 regional grids, 20+ states — CEA V21.0 FY 2024-25 |
| AT&C losses | State-wise — PFC FY 2022-23 |
| Road fuels | Diesel, petrol, CNG, LPG |
| Indian Railways | Passenger and freight |
| Stationary combustion | Coal (domestic + imported + lignite), furnace oil, natural gas, biomass |
| Refrigerants | 8 types — IPCC AR6 GWP100 |
| Waste | MSW landfill, wastewater treatment |

**74 emission factors. All with source document, methodology, 
reference year, and practitioner notes.**

---

## How to use

**Web interface:** Browse, filter, and search at the live URL above.

**JSON API:** Load the raw data directly: https://creator619-python.github.io/open-india-emission-factors/emission_factors_v1_0.json
**Local use:** Download `emission_factors_v1_0.json` and use in 
any tool, script, or model. No attribution required.

---

## Data structure

Each emission factor record contains:

```json
{
  "id": "unique-identifier",
  "factor_name": "descriptive name",
  "geography": "India / State / Region",
  "geography_type": "national / state / regional_grid / global",
  "value": 0.7117,
  "unit": "kgCO2/kWh",
  "scope": "Scope 1 / 2 / 3",
  "sector": "Electricity",
  "source_document": "full source name",
  "source_url": "https://...",
  "publication_year": 2025,
  "reference_year": "FY 2024-25",
  "methodology": "how the value was derived",
  "last_updated": "2026-06-26",
  "notes": "practitioner guidance"
}
```

---

## Sources

- Central Electricity Authority (CEA) — CO2 Baseline Database V21.0
- Power Finance Corporation (PFC) — DISCOM Performance Report
- Petroleum Planning & Analysis Cell (PPAC) — Ready Reckoner FY 2025-26
- Bureau of Energy Efficiency (BEE) — CCTS Technical Guidelines
- Smart Freight Centre India — India Default GHG Values V1.0
- India GHG Program (WRI India, TERI, CII) — Rail Transport EFs
- IPCC 2006 Guidelines — Volumes 2 and 5
- IPCC AR6 WGI — GWP100 values

---

## Contributing

Found an error? Missing a factor? Open an issue or submit a pull request.

Priority gaps: agriculture, industrial processes, construction materials,
fugitive emissions from oil and gas.

---

## Built by

Gokul Krishna T B  
[LinkedIn](https://linkedin.com/in/yourprofile) · [open-epd-india](https://creator619-python.github.io/open-epd-india)

*Also building open infrastructure for Indian sustainability practice.*
