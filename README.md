# Open India Emission Factors

India's open data commons for GHG emission factors — built and validated 
by the practitioners who use it.

**Live database:** https://creator619-python.github.io/open-india-emission-factors  
**License:** CC0 1.0 Public Domain  
**Version:** 1.1 | **Last updated:** 27 June 2026

---

## Why this exists

Indian GHG practitioners have been forced to rely on DEFRA UK and IPCC 
defaults because India-specific emission factors aren't openly accessible 
in one place. This database fixes that.

Every factor includes its source document, methodology, reference year, 
and practitioner notes — so you know exactly what you're using and why.

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
| Industrial processes | Cement, steel (BF-BOF + EAF-DRI), aluminium, glass |
| Agriculture | Enteric fermentation, manure management, rice cultivation, synthetic fertilizers |
| DG sets | 4 capacity ranges — CPCB IV+ |

**95 emission factors. All with source document, methodology, reference 
year, and practitioner notes.**

---

## How to use

**Web interface:** Browse, filter, and search at the live URL above.

**JSON API:** Load the raw data directly:
https://creator619-python.github.io/open-india-emission-factors/emission_factors_v1_1.json
**Local use:** Download `emission_factors_v1_1.json` and use in any 
tool, script, or model. No attribution required.

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
  "last_updated": "2026-06-27",
  "notes": "practitioner guidance"
}
```

---

## Community

This is a community initiative — open to contributions from anyone with 
credible India-specific data.

Validation is community-driven, led by experienced practitioners. If you 
have domain expertise in a specific sector and want to be part of the 
validation process, reach out.

The goal is for this to be India's shared data commons — built and 
validated by the people who use it.

**Contribute:** Open an issue or submit a pull request.  
**Validate:** Comment on existing factors with corrections or updated sources.  
**Suggest:** Flag missing sectors, geographies, or activity types.

Priority gaps for v1.2:
- Fugitive emissions from oil and gas
- Construction materials
- Aviation (domestic routes)
- Cold chain and refrigerated transport
- State-specific agriculture factors

---

## Sources

- Central Electricity Authority (CEA) — CO2 Baseline Database V21.0
- Power Finance Corporation (PFC) — DISCOM Performance Report FY 2022-23
- Petroleum Planning & Analysis Cell (PPAC) — Ready Reckoner FY 2025-26
- Bureau of Energy Efficiency (BEE) — CCTS Technical Guidelines FY 2024-25
- Smart Freight Centre India — India Default GHG Values V1.0 (2025)
- India GHG Program (WRI India, TERI, CII) — Rail Transport EFs
- MoEFCC — Third National Communication to UNFCCC (2023)
- CPCB — DG Set Emission Standards (CPCB IV+)
- IPCC 2006 Guidelines — Volumes 2, 3, 4, and 5
- IPCC AR6 WGI — GWP100 values (2021)

---

## Built by

**Gokul Krishna T.B**  
Senior ESG Executive | GHG Practitioner | Open Data Builder  
[LinkedIn](https://www.linkedin.com/in/gokul-k-148624117/) · 
[open-epd-india](https://creator619-python.github.io/open-epd-india)

*Also building open infrastructure for Indian sustainability practice.*

---

*CC0 1.0 Universal — No rights reserved. Use freely for any purpose.*
