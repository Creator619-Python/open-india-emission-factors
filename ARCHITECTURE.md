# Grid Home India — Live Carbon Intensity Model
## Architecture & Implementation Plan

---

## What We're Building

A state-level estimated carbon intensity model for India, powered entirely by:
- **CEA V21.0 emission factors** from `open-india-emission-factors` (your own database)
- **NASA POWER API** for real-time solar irradiance per state capital
- **Open-Meteo API** for real-time wind speed per state capital (free, no key)
- **Time-of-day demand model** anchored to India's actual load curve shape

No third-party carbon intensity API. No scraping. No ToS issues.
Fully open, fully attributable, uniquely Indian.

---

## The Model: How It Works

### Step 1 — CEA Annual Baseline (gCO2/kWh)

Every state starts from its CEA V21.0 annual weighted-average emission factor,
converted from kgCO2/kWh → gCO2/kWh.

**Coverage from open-india-emission-factors v1.2:**

| State | CEA EF (gCO2/kWh) | Source |
|---|---|---|
| Andhra Pradesh | 808.6 | CEA V21.0 state |
| Arunachal Pradesh | 210 | CEA V21.0 state |
| Assam | 580 | CEA V21.0 state |
| Bihar | 890 | CEA V21.0 state |
| Chhattisgarh | 954.4 | CEA V21.0 state |
| Delhi | 412.7 | CEA V21.0 state |
| Goa | 400 | CEA V21.0 state |
| Gujarat | 788.6 | CEA V21.0 state |
| Haryana | 946.4 | CEA V21.0 state |
| Himachal Pradesh | 120 | CEA V21.0 state |
| Jharkhand | 920 | CEA V21.0 state |
| Karnataka | 320 | CEA V21.0 state |
| Kerala | 280 | CEA V21.0 state |
| Madhya Pradesh | 902.3 | CEA V21.0 state |
| Maharashtra | 877.5 | CEA V21.0 state |
| Manipur | 300 | CEA V21.0 state |
| Meghalaya | 280 | CEA V21.0 state |
| Mizoram | 220 | CEA V21.0 state |
| Nagaland | 240 | CEA V21.0 state |
| Odisha | 850 | CEA V21.0 state |
| Punjab | 845.4 | CEA V21.0 state |
| Rajasthan | 901.3 | CEA V21.0 state |
| Sikkim | 90 | CEA V21.0 state |
| Tamil Nadu | 340 | CEA V21.0 state |
| Telangana | 450 | CEA V21.0 state |
| Tripura | 410 | CEA V21.0 state |
| Uttar Pradesh | 934.4 | CEA V21.0 state |
| Uttarakhand | 20.8 | CEA V21.0 state |
| West Bengal | 820 | CEA V21.0 state |

**7 UTs using regional fallback:**
- J&K, Ladakh → Northern Region (733.5)
- A&N Islands → Eastern Region (928.0)
- Chandigarh → Northern Region (733.5)
- Dadra & Nagar Haveli → Western Region (890.0)
- Lakshadweep → Southern Region (808.6)
- Puducherry → Southern Region (808.6)

---

### Step 2 — Solar Reduction Factor (NASA POWER API)

**API:** `https://power.larc.nasa.gov/api/temporal/hourly/point`
**Parameter:** `ALLSKY_SFC_SW_DWN` (surface solar irradiance, W/m²)
**Per state:** Called using state capital lat/lon

**Solar capacity weights** (states with significant solar — determines how much
irradiance actually reduces grid carbon):

```
High solar states (>10 GW installed): RJ, GJ, AP, TN, KA, MP, MH, UP, TG
Medium solar states (3–10 GW): HR, PB, DL, BR, OD, KL
Low solar states (<3 GW): HP, UK, JK, NE states, hydro-dominant states
```

**Reduction formula:**
```
solar_reduction = (irradiance / irradiance_peak_ref) × max_reduction_cap

Where:
  irradiance_peak_ref:  900 W/m² (clear sky peak at solar noon, India)
  max_reduction_cap:    0.35 (high solar states), 0.20 (medium), 0.08 (low)
```

Note: one multiplier only. The state's solar tier determines which cap to use —
it is not a separate weight multiplied on top. At peak irradiance (900 W/m²),
a high-solar state sees exactly 35% reduction maximum. No squaring of coefficients.

---

### Step 3 — Wind Reduction Factor (Open-Meteo API)

**API:** `https://api.open-meteo.com/v1/forecast`
**Parameter:** `windspeed_10m` (km/h)
**Free, no key required**

**Wind capacity weights** (states with significant wind):
```
High wind: TN, GJ, KA, RJ, MH, AP
Medium wind: KL, HR, PB
Low wind: all others
```

**Reduction formula:**
```
wind_reduction = min(windspeed / 25.0, 1.0) × max_wind_cap

Where:
  max_wind_cap:  0.20 (high wind states), 0.10 (medium), 0.03 (low)
  25 km/h:       reference speed for rated power output
```

Note: one multiplier only. Same principle as solar — the state's wind tier
determines the cap, not an additional weight coefficient.

---

### Step 4 — Demand Factor (India Load Curve)

India's grid has a well-documented load shape. Coal/gas fills demand peaks.
Higher demand = more thermal dispatch = higher carbon intensity.

```
Hour 00–04 → demand_factor = -0.08  (low demand, night wind active)
Hour 05–09 → demand_factor = +0.05  (morning ramp)
Hour 10–15 → demand_factor = -0.02  (solar active, moderate demand)
Hour 16–17 → demand_factor = +0.03  (pre-peak)
Hour 18–21 → demand_factor = +0.12  (evening peak, no solar, coal ramps)
Hour 22–23 → demand_factor = +0.02  (demand tapering)
```

---

### Step 5 — Final Formula

```
estimated_intensity(gCO2/kWh) =
  cea_baseline
  × (1 - solar_reduction)
  × (1 - wind_reduction)
  × (1 + demand_factor)
```

**With T&D loss adjustment (from open-india-emission-factors):**
```
consumer_intensity = estimated_intensity / (1 - td_loss_fraction)
```

This gives the carbon intensity *at the point of consumption*, not generation —
which is what matters for a home screen user deciding when to run their AC.

---

### Step 6 — 6h History & Forecast

- **History:** Stored in Room DB (already in the app). Every refresh writes a reading.
- **Forecast:** Run the same model for hours +1 to +6 using:
  - NASA POWER hourly forecast (available for next 24h in the same API call)
  - Open-Meteo hourly wind forecast (48h available)
  - Known demand curve shape

---

## Cloudflare Worker: `grid-home-india-proxy`

Single Worker, two endpoints:

### `GET /carbon?state=TG`
1. Map state code → lat/lon of capital
2. Fetch NASA POWER irradiance (current hour + next 6h)
3. Fetch Open-Meteo wind speed (current + next 6h)
4. Apply model formula
5. Return JSON:
```json
{
  "stateCode": "TG",
  "stateName": "Telangana",
  "gCo2PerKwh": 468.1,
  "generationGCo2PerKwh": 380.8,
  "ceaBaselineGCo2PerKwh": 450.0,
  "tdLossPercent": 18.65,
  "solarReduction": 0.121,
  "windReduction": 0.017,
  "demandFactor": -0.02,
  "irradianceWm2": 312.4,
  "windSpeedKmh": 14.2,
  "isEstimated": true,
  "estimationMethod": "CEA-V21.0 + NASA-POWER + Open-Meteo",
  "timestamp": "2026-06-28T14:00:00+05:30",
  "forecast6h": [481.1, 527.4, 547.0, 609.0, 609.0, 609.0]
}
```

Verification:
- solar_reduction = (312.4 / 900) × 0.35 = 0.121
- wind_reduction  = (14.2 / 25) × 0.03  = 0.017
- demand_factor   = -0.02 (hour 14 IST, solar active)
- generation      = 450 × (1 - 0.121) × (1 - 0.017) × (1 - 0.02) = 380.8 gCO2/kWh
- consumer        = 380.8 / (1 - 0.1865) = 468.1 gCO2/kWh

### `GET /allstates`
Same model, run for all 36 states. NASA POWER enforces 5 req/sec per IP, so
states are processed in **batches of 4 with a 1-second delay between batches**
(9 batches × ~1s = ~9 seconds total). Open-Meteo runs in parallel within each
batch (no rate limit). Result is sorted by consumer intensity ascending (cleanest
state first) with a rank field. Intended for the India map / rankings view,
called once on launch and refreshed every 30 minutes via Room DB cache.

**Caching:** 30-minute cache via Cloudflare Cache API. NASA POWER updates hourly,
Open-Meteo updates every 15 min. 30 min is a reasonable refresh for this use case.

---

## Android Changes Required

### 1. Package rename (non-negotiable before Play Store)
`com.example` → `com.gokulkrishna.gridhomeindia`

### 2. `CarbonRepositoryImpl.kt` — full rewrite
- Remove all static baselines and simulated diurnal
- HTTP call to Cloudflare Worker `/carbon?state={stateCode}`
- Parse response into `CarbonData`
- Fallback to last Room DB reading if network unavailable

### 3. `WeatherRepositoryImpl.kt` — replace simulation with Open-Meteo
- Open-Meteo already called in the Worker for wind
- For the weather card: call Open-Meteo directly from the app
  (it's free, no key, CORS-friendly)
- Real temperature, humidity, wind, condition codes

### 4. Room DB — add `CarbonReading` entity
- Already has `GridDatabase` and `GridReadingDao`
- Add timestamp + gCO2 value + state code
- Query last 6 readings for history sparkline

### 5. UI — add attribution label
- Small tag on the carbon card: "Estimated · CEA V21.0 + NASA POWER"
- Tapping it opens a bottom sheet explaining the methodology
- Links to open-india-emission-factors GitHub

---

## Data Sources & Attribution

| Source | Data Used | License | URL |
|---|---|---|---|
| CEA V21.0 | State emission factors | Public domain (GoI) | cea.nic.in |
| open-india-emission-factors | Processed CEA + T&D loss data | CC0 | creator619-python.github.io/open-india-emission-factors |
| NASA POWER | Solar irradiance, hourly | Public domain (NASA) | power.larc.nasa.gov |
| Open-Meteo | Wind speed, temperature | CC BY 4.0 | open-meteo.com |

---

## What Makes This Different From Everything Else

1. **First India-specific carbon intensity model** anchored to CEA-published state EFs
2. **T&D loss adjustment** — models consumer-side intensity, not just generation
3. **Cross-project integration** — Grid Home India is the live application of open-india-emission-factors
4. **Fully open methodology** — every formula, every weight, every source is documented
5. **Zero proprietary API dependency** — NASA POWER and Open-Meteo are both public domain / open

---

## Implementation Order

1. [ ] Cloudflare Worker (`grid-home-india-proxy`) — ~150 lines JS
2. [ ] Package rename (`com.example` → `com.gokulkrishna.gridhomeindia`)
3. [ ] `CarbonRepositoryImpl.kt` rewrite — ~120 lines Kotlin
4. [ ] `WeatherRepositoryImpl.kt` rewrite — ~80 lines Kotlin
5. [ ] Room DB `CarbonReading` entity + DAO update
6. [ ] UI attribution label + methodology bottom sheet
7. [ ] Test on device / emulator
8. [ ] Update open-india-emission-factors README to reference Grid Home India
9. [ ] LinkedIn post: "I built India's first open carbon intensity model"

---

*Architecture version 1.1 — June 2026 (fixed: solar/wind formula double-weighting, NASA rate limiting, example JSON math)*
*Gokul Krishna TB | creator619-python.github.io*
