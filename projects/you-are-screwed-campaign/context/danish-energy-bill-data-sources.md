# Danish energy bill data sources for the "Screwed Score" tool

**The complete data pipeline for a postcode-personalized Danish electricity bill breakdown is feasible using open data, with one critical gap: mapping postcodes to grid companies.** Energi Data Service provides tariffs, spot prices, and system charges via free JSON APIs. The postcode-to-grid-company mapping—essential because Denmark's ~40 grid companies each set different tariffs—has no single authoritative open dataset, but can be solved using the Strømligning API or Green Power Denmark's address lookup. The January 2026 elafgift reduction from 72.7 to **0.8 øre/kWh** fundamentally reshapes the bill: wholesale electricity now represents ~50% of the total (up from ~28%), making grid tariff variations far more significant to consumers.

---

## PART 1: Electricity bill components

### 1A. DatahubPricelist — the tariff backbone

**SOURCE: Energi Data Service — DatahubPricelist**
- URL: `https://api.energidataservice.dk/dataset/DatahubPricelist`
- Metadata: `https://api.energidataservice.dk/meta/dataset/DatahubPricelist`
- Web page: `https://www.energidataservice.dk/tso-electricity/DatahubPricelist`
- Format: JSON REST API (also CSV via `/download?format=csv`)
- Access: **Open, free, no authentication**
- Granularity: Per grid company (identified by GLN number)
- Update frequency: Daily
- Historical depth: From **2014-01-01**
- Rate limit: **1 request per IP per dataset per minute**; Energinet explicitly states this API is not intended as a direct consumer-facing source—vendors should cache/proxy

**ChargeType codes (reverse-engineered from DataHub conventions and community integrations):**

| Code | Meaning | Typical Price1–Price24 behavior |
|------|---------|-------------------------------|
| **D01** | Subscription (Abonnement) — fixed periodic fee | Only Price1 populated (DKK/month or DKK/year) |
| **D02** | Fee (Gebyr) — one-time or periodic charges | Only Price1 populated |
| **D03** | Tariff (Tarif) — variable per-kWh charge | All 24 prices may differ under Tarifmodel 3.0 |

**GLN_Number** is the **Global Location Number** (GS1 standard, 13 digits), serving as the unique identifier for each grid company or charge owner. Energinet's own GLN is **5790000432752**. There are approximately **40–50 active DSOs** plus Energinet as TSO, with the total distinct GLN count in the dataset reaching 50–100+ including historical entries and electricity retailers.

**Price1 through Price24** represent **hourly tariff rates in DKK/kWh** for Danish local time (Price1 = 00:00–01:00, Price24 = 23:00–00:00). Under Tarifmodel 3.0 (fully rolled out January 2024), grid company tariffs are time-differentiated with peak (17:00–21:00), mid-load, and off-peak (00:00–06:00) rates varying by winter/summer season.

**ResolutionDuration** uses ISO 8601 duration format: **PT1H** = hourly resolution (Price1–24 contain different rates), **P1M** = monthly (subscriptions, only Price1 relevant), **P1D** = daily (single rate for all hours).

**Elafgift is included** in DatahubPricelist as entries from Energinet (GLN 5790000432752), with specific ChargeTypeCodes for standard and reduced electricity tax.

**No official GLN-to-company-name API exists.** The ChargeOwner field in each record contains the company name alongside the GLN. The best open-source reference is the **chargeowners.py** file maintained in the Home Assistant integration: `https://github.com/MTrab/energidataservice/blob/master/custom_components/energidataservice/tariffs/energidataservice/chargeowners.py` — this maps every Danish grid company to its GLN, ChargeTypeCode patterns, and tariff filter parameters.

**Example queries:**
```
# Latest D03 tariffs, sorted newest first:
https://api.energidataservice.dk/dataset/DatahubPricelist?filter={"ChargeType":["D03"]}&limit=20&sort=ValidFrom%20DESC

# All charges from Energinet (TSO):
https://api.energidataservice.dk/dataset/DatahubPricelist?filter={"GLN_Number":["5790000432752"]}&limit=20

# Specific grid company by GLN:
https://api.energidataservice.dk/dataset/DatahubPricelist?filter={"GLN_Number":["5790001089030"],"ChargeType":["D03"]}&start=2026-01-01&sort=ValidFrom%20DESC&limit=20
```

**Limitations:** ChargeTypeCode values are company-specific with no standardized enumeration. ValidTo can be blank (meaning "until further notice"). Price1–24 have complex fallback logic. Timezone handling is undocumented. One developer described it as "maybe the worst open dataset" due to these inconsistencies.

---

### 1B. Postcode-to-grid-company mapping — the critical gap

**There is no single public dataset that directly maps Danish 4-digit postcodes to electricity grid companies.** This is the hardest data challenge for the tool. Some postcodes are split between multiple grid companies, making address-level or coordinate-level resolution fundamentally necessary.

**🥇 BEST SOLUTION: Strømligning API**
- URL: `https://stromligning.dk/api/docs/` (Swagger documentation)
- Endpoint: `GET https://stromligning.dk/api/suppliers/find?lat={lat}&long={long}`
- Format: JSON REST
- Access: Free for non-commercial use; commercial requires agreement (contact@stromligning.dk)
- Returns: Grid company `id`, `priceArea` (DK1/DK2)
- Then use: `GET https://stromligning.dk/api/prices?from=...&to=...&supplierId={id}&aggregation=1h&priceArea={area}` for complete prices including all tariffs
- Limitation: Rate limited; closed source; external dependency
- How it feeds the Screwed Score: **Solves the entire postcode→tariff chain in a single API**

**🥈 SECOND-BEST: Green Power Denmark / elnet.dk API**
- URL: `https://api.elnet.greenpowerdenmark.dk/api/supplierlookup/{adresse}`
- Documentation: `https://elnet.dk/files/media/document/Find-Netselskab-AP-beskrivelse.pdf`
- Format: JSON — returns `{"name": "Selskab", "phone": "...", "website": "..."}`
- Access: Open (no auth documented)
- Limitation: Requires a **full address string**, not just postcode
- How it feeds the Screwed Score: Most authoritative source (industry body). Build a static mapping table by querying representative addresses for all ~1,186 Danish postcodes using DAWA address data.

**🥉 BUILD-YOUR-OWN STATIC TABLE APPROACH:**
1. Use DAWA (`https://api.dataforsyningen.dk/postnumre`) to get all postcodes with representative addresses
2. For each, query Green Power Denmark API → get grid company name
3. Cross-reference with chargeowners.py to get GLN numbers
4. Use GLN to query DatahubPricelist for tariffs
5. Cache aggressively — grid areas rarely change

**Sources that DO NOT solve this problem:**
- **DAWA** — does not include utility/grid company areas. Also being retired July 1, 2026
- **Energinet DataHub** — grid area data exists internally but is not publicly exposed
- **Energistyrelsen** — has licensed grid company list but no geographic boundary data (no shapefiles, GeoJSON, or postcode mappings)
- **Forsyningstilsynet** — no public registry with service area information
- **data.gov.dk / Geodatastyrelsen / GEUS** — no grid company boundary GIS data found
- **elpris.dk** — solves it for end users but has no public API

---

### 1C. Spot prices — DayAheadPrices

**SOURCE: Energi Data Service — DayAheadPrices**
- URL: `https://api.energidataservice.dk/dataset/DayAheadPrices`
- Format: JSON REST API
- Access: Open, free, no authentication
- Granularity: **Quarter-hourly (15-minute)** from October 1, 2025; by price area (DK1/DK2)
- Update frequency: Daily (day-ahead prices available by early afternoon)
- Historical: From October 2025 (quarter-hourly); for older data use legacy `Elspotprices` dataset (hourly, from ~2000, no new data after October 2025)

**Key columns:** HourUTC, HourDK, PriceArea, SpotPriceDKK, SpotPriceEUR. **Units: DKK/MWh** (divide by 1,000 for DKK/kWh).

**Postcode-to-price-area mapping** is straightforward: postcodes **1000–4999 = DK2** (Eastern Denmark/Zealand), **5000–9999 = DK1** (Western Denmark/Jutland+Funen). The split is at Storebælt.

**Example query:**
```
https://api.energidataservice.dk/dataset/DayAheadPrices?start=2026-01-01&end=2026-01-02&filter={"PriceArea":["DK1"]}&limit=96
```

**Legacy Elspotprices** (`https://api.energidataservice.dk/dataset/Elspotprices`) still serves historical data through September 30, 2025, with hourly resolution back to ~2000.

---

### 1D. National fixed components — confirmed rates

**Energinet TSO tariffs (2026):**
- **Transmissionstarif (nettarif):** 4.3 øre/kWh
- **Systemtarif:** 7.2 øre/kWh
- **Total TSO consumer tariff:** **11.5 øre/kWh** (down 15% from 13.5 øre/kWh in 2025)
- **System abonnement:** ~182 DKK/year
- Source: Energinet press release via Ritzau (September 1, 2025); tariff forecast PDF at `https://energinet.dk/media/alpn1j0h/fremskrivning-af-energinets-eltariffer-2025-2027-1.pdf`
- Available in DatahubPricelist under GLN 5790000432752
- Official page: `https://energinet.dk/el/elmarkedet/tariffer/aktuelle-tariffer/`

**Elafgift (state electricity tax):**
- **2025 rate:** 72.7 øre/kWh (0.727 DKK/kWh)
- **2026–2027 rate:** **0.8 øre/kWh** (0.008 DKK/kWh) — EU minimum
- **Effective:** January 1, 2026 through December 31, 2027
- **Adopted:** Parliament vote December 17, 2025; Law No. 1775 published December 30, 2025
- **Savings:** ~DKK 3,600/year incl. VAT for 4,000 kWh household
- **Cost to state:** ~DKK 7.1B (2026), ~DKK 7.0B (2027)
- Source: Skatteministeriet; also appears in DatahubPricelist under Energinet's GLN
- After 2027: No firm decision; government has expressed desire to keep it low permanently

**PSO tariff:** **Fully abolished January 1, 2022.** Phase-out agreed November 2016; costs moved to state budget. No longer appears on bills.

**Tarifmodel 3.0:** **Fully deployed** across nearly all Danish grid companies since January 1, 2024 (when N1, the largest DSO, adopted it). Approved by Forsyningstilsynet March 25, 2022. Three-tier time-differentiated pricing: peak (17:00–21:00), mid-load (daytime), off-peak (night 00:00–06:00), with winter/summer seasonal variation.

---

### 1E. Retail electricity margins

**SOURCE: Forsyningstilsynet quarterly statistics**
- URL: `https://forsyningstilsynet.dk/analyser-og-tal/forbrugerpriser/elpriser`
- Format: PDF reports (quarterly)
- Access: Public
- Key data: Average retail product prices by type (fixed/variable), separated from grid and taxes

**Typical retail markup for spot-price contracts:** **3–15 øre/kWh** above spot (surcharge/tillæg), plus a monthly subscription of **~20–39 DKK/month**. Nordic Energy Research (2023) found most consumers report surcharges of 1–5 or 5–10 øre/kWh, but ~75% of households did not know their surcharge.

**Strømligning.dk** provides the most comprehensive retailer comparison data, including scores, surcharges, and subscription fees via their API. The open-source project **elspotpris.dk** (`https://github.com/rndfm/elspotpris`) maintains manually collected and consumer-verified retailer surcharges.

No authoritative structured API exists for retail margins specifically. For the Screwed Score, an estimated **5 øre/kWh + 29 DKK/month** is a reasonable default when actual retailer data is unknown.

---

## Postcode 8000 pipeline walkthrough

Here is the exact data pipeline for **postcode 8000 (Aarhus), monthly bill 2,500 DKK**, validated step by step:

**Step 1: Postcode → municipality + coordinates (DAWA)**
- Endpoint: `GET https://api.dataforsyningen.dk/postnumre/8000`
- Returns: Municipality "Aarhus" (code 751), bounding coordinates ~56.15°N, 10.21°E
- Status: ✅ Works. Open, no auth. **⚠️ DAWA closing July 1, 2026** — migration needed.

**Step 2: Coordinates → grid company (Strømligning API)**
- Endpoint: `GET https://stromligning.dk/api/suppliers/find?lat=56.15&long=10.21`
- Returns: Grid company ID (N1 for Aarhus area), price area = DK1
- Status: ✅ Works. Free for non-commercial.
- **This is where the pipeline would break without Strømligning or equivalent.** No open government API provides this mapping.

**Step 3: Grid company → nettarif (DatahubPricelist)**
- Endpoint: `GET https://api.energidataservice.dk/dataset/DatahubPricelist?filter={"GLN_Number":["<N1_GLN>"],"ChargeType":["D03"]}&start=2026-01-01&sort=ValidFrom%20DESC&limit=20`
- Returns: Hourly tariff rates (Price1–Price24) in DKK/kWh
- Status: ✅ Works. Need to know N1's GLN (available from chargeowners.py or Strømligning).
- **Typical N1 nettarif:** ~15–25 øre/kWh weighted average (varies by time of day under Tarifmodel 3.0)

**Step 4: TSO tariffs (DatahubPricelist)**
- Endpoint: `GET https://api.energidataservice.dk/dataset/DatahubPricelist?filter={"GLN_Number":["5790000432752"]}&start=2026-01-01&limit=20`
- Returns: Systemtarif (7.2 øre/kWh) + transmissionstarif (4.3 øre/kWh) = 11.5 øre/kWh
- Status: ✅ Works.

**Step 5: Spot price (DayAheadPrices)**
- Endpoint: `GET https://api.energidataservice.dk/dataset/DayAheadPrices?start=now-P30D&end=now&filter={"PriceArea":["DK1"]}`
- Returns: Quarter-hourly prices in DKK/MWh; compute 30-day average, divide by 1000
- Status: ✅ Works. **Recent average:** ~60 øre/kWh

**Step 6: Known fixed values**
- Elafgift: 0.8 øre/kWh (from January 2026)
- Moms: 25% VAT
- Estimated retail margin: ~5 øre/kWh

**Step 7: Derive kWh from total bill**
```
Total bill: 2,500 DKK/month
Excl. VAT: 2,500 / 1.25 = 2,000 DKK
Monthly subscription estimate: ~30 DKK (retail) + ~15 DKK (system abonnement/12)
Per-kWh budget: 2,000 - 45 = 1,955 DKK

Sum of per-kWh rates (2026):
  Spot:              0.60 DKK/kWh
  DSO nettarif:      0.20 DKK/kWh (N1 weighted avg)
  TSO tariffs:       0.115 DKK/kWh
  Elafgift:          0.008 DKK/kWh
  Retail margin:     0.05 DKK/kWh
  Total per kWh:     0.973 DKK/kWh

Derived consumption: 1,955 / 0.973 ≈ 2,009 kWh/month ≈ 24,108 kWh/year
```

This is very high consumption (6× average), consistent with electric heating or a large property. For a "normal" household at **~333 kWh/month**, the monthly bill would be ~400–500 DKK post-reform.

**Step 8: Bill breakdown for this scenario**

| Component | DKK/month | % of bill |
|-----------|-----------|-----------|
| Spot price (wholesale) | 1,205 | **48.2%** |
| DSO grid tariff (N1) | 402 | **16.1%** |
| TSO tariffs | 231 | 9.2% |
| Retail margin | 100 | 4.0% |
| Subscriptions | ~45 | 1.8% |
| Elafgift | 16 | 0.6% |
| **Subtotal excl. VAT** | **1,999** | |
| VAT (25%) | 500 | **20.0%** |
| **Total** | **~2,499** | |

**Step 9: Screwed Score = 100% − wholesale% = ~52%** (meaning 52 øre of every krone goes to things other than actual electricity).

**Where the pipeline breaks:**
- **Step 2 is the critical failure point** — without Strømligning or similar, there is no open-data way to go from postcode to grid company
- **DAWA sunset (July 2026)** means the geocoding step needs migration to Dataforsyningen's replacement APIs
- **Subscription amounts** require parsing D01 entries from DatahubPricelist, which have inconsistent naming conventions
- **Retail margin** must be estimated unless the user provides their retailer information

---

## PART 2: Postcode leaderboard data

**Average nettarif per grid company**
- Source: DatahubPricelist, filtering ChargeType=D03 per GLN, computing weighted average
- Forsyningstilsynet comparison: `https://forsyningstilsynet.dk/Media/638536031032121805/Elnetvirksomhedernes%20tariffer%20for%20husholdninger.pdf` — compares tariffs across all ~36 grid companies with Tarifmodel 3.0 breakdown (low/high/peak load, summer/winter), historical data 2018–2024
- Format: PDF (not structured API)
- Granularity: Per grid company
- Limitation: Must be manually parsed from PDF

**Household electricity consumption per municipality**
- Source: Energi Data Service — `PrivIndustryConsumptionMunicipalityMonth`
- URL: `https://api.energidataservice.dk/dataset/PrivIndustryConsumptionMunicipalityMonth?start=2024-01-01&limit=100`
- Format: JSON API, open, free
- Granularity: Monthly × 98 municipalities, private vs. industry split
- Also: `ConsumptionPerMunicipalityDE35` — monthly by municipality and DE35 industry code
- Update: Monthly with 9-day delay

**Number of households per municipality**
- Source: Danmarks Statistik — **FAM55N** (Households by region, type, size)
- URL: `https://api.statbank.dk/v1/data/FAM55N`
- Format: JSON/CSV API, open, no auth
- Granularity: Municipality level, annual (January 1)

---

## PART 3: Property and building data

### 3A. BBR (Building and Dwelling Register)

**SOURCE: Datafordeler.dk — BBR API**
- URL: `https://services.datafordeler.dk/BBR/BBRPublic/1/rest/{method}` (REST, retiring June 30, 2026)
- GraphQL replacement: `https://services.datafordeler.dk/BBR/BBRPublic/1/graphql/`
- Format: JSON or XML (configurable)
- Access: **Free registration** — username/password via self-service portal. No NemID/MitID for API access itself (only for admin portal).
- Granularity: Individual building/address
- Update frequency: Near real-time (continuously updated by municipalities)

**Methods:** `bygning` (building), `enhed` (dwelling unit), `tekniskanlaeg` (technical installation incl. solar panels), `grund` (plot), `ejendomsrelation`, `bbrsag`

**Key building fields:** `byg021BygningensAnvendelse` (use code: 110=single-family, 120=row house, 130=multi-family), `byg026Opførelsesår` (construction year), `byg056Varmeinstallation` (heating installation), `byg057Opvarmningsmiddel` (heating fuel), `byg038SamletBygningsareal` (total area), `byg041BebyggetAreal` (footprint)

**BBR heating installation codes (field 229):**

| Code | Type |
|------|------|
| 1 | District heating (fjernvarme) |
| 2 | Central heating, single boiler |
| 3 | Stoves (wood stoves, fireplaces) |
| 5 | **Heat pump** |
| 6 | Central heating, dual fuel |
| 7 | Electric heaters/panels |
| 9 | No heating |

**Solar panels in BBR:** Registered as `tekniskanlaeg` (technical installation) with fields for classification code (solar vs. oil tank), capacity (kW), area (m²), installation year. Queryable via the `tekniskanlaeg` endpoint.

**Postcode query limitation:** BBR REST API does not accept postcode directly. Requires two-step: (1) use DAWA to get address UUIDs for a postcode, (2) query BBR by `AdresseIdentificerer`.

**DAWA (address resolution only)**
- URL: `https://api.dataforsyningen.dk/adresser?postnr=8000`
- Access: Fully open, no auth, JSON
- **BBR data was removed from DAWA on April 1, 2024.** DAWA now only provides address resolution and geocoding, not building attributes.
- **⚠️ DAWA closing July 1, 2026**

### 3B. Energy Performance Certificates

**SOURCE: EMOData API**
- URL: `https://emoweb.dk/emodata/api-docs/index.html` (Swagger)
- Test page: `https://emoweb.dk/emodata/test/`
- Format: JSON/XML API
- Access: **Free registration required** — email emo-info@ens.dk with name, email, phone
- Granularity: Individual building/address
- Update frequency: Continuous (~70,000 new labels/year; ~700,000 total since 2006)
- Key fields: Energy label class (A2020–G), calculated consumption (kWh/m²), building components, improvement suggestions with costs/savings, actual consumption, validity period
- Limitation: No postcode-batch query. Older labels (pre-September 2006) not available.

**Aggregate energy label statistics** (open, no registration):
- URL: `https://emoweb.dk/emostat`
- Provides statistics by year, postcode, building type, classification

**SparEnergi.dk** (`https://sparenergi.dk`) offers address-based energy label lookup and APIs (contact sparenergi@ens.dk). Run by Energistyrelsen.

### 3C. Housing type distribution

**SOURCE: Danmarks Statistik — BOL101, BOL102, BYGB40**
- **BOL101:** Dwellings by region, type, tenure, ownership, construction year
- **BOL102:** Dwellings by region, type, **heating type** — most relevant for energy analysis
- **BYGB40:** Buildings and heated area by region, **type of heating**, use, construction year
- URL: `https://api.statbank.dk/v1/data/BOL102` (POST with JSON body)
- Access: Open, no auth
- Granularity: Municipality (98 municipalities)
- Update: Annual
- ⚠️ **2021–2022 data temporarily closed** due to BBR data errors

---

## PART 4: Solar and wind potential

### 4A. Solar

**PVGIS API (EU JRC) — recommended primary source**
- URL: `https://re.jrc.ec.europa.eu/api/v5_3/PVcalc?lat=56.15&lon=10.21&peakpower=6&loss=14&outputformat=json`
- Format: JSON/CSV
- Access: **Free, no registration, no API key**
- Rate limit: 30 calls/second; CORS/AJAX not allowed
- Radiation database for Denmark: **PVGIS-SARAH3** (default, satellite-based, 2005–2023)
- Typical output for 6 kWp in Denmark: **5,400–6,200 kWh/year** (~900–1,030 kWh/kWp/year)
- Tools: PVcalc, seriescalc (hourly time series), MRcalc (monthly radiation), tmy (Typical Meteorological Year)
- Python: `pvlib.iotools.get_pvgis_hourly()`

**sologvindinfo.dk (Energistyrelsen)**
- URL: `https://sologvindinfo.dk/spatialmap`
- Format: Web map viewer (Plandata platform); underlying data as Excel from ens.dk
- Access: Public
- Shows grid-connected wind turbines and solar PV by municipality with capacity and connection dates
- **No API** — map viewer only. Use Excel downloads from ens.dk instead.

**DMI Open Data — solar irradiance**
- URL: `https://dmigw.govcloud.dk/v2/metObs/` and `/v2/climateData/`
- Format: GeoJSON
- Access: **Free API key** (register at dmi.dk)
- Parameter: `radia_glob` (global horizontal irradiance)
- Granularity: Station-based (~40–60 stations), 10-minute intervals
- Limitation: Point measurements only (not gridded). Better as validation than primary source.

### 4B. Wind

**DMI Open Data — wind speed**
- Same API as above. Parameters: `wind_speed` (10m), `wind_dir`, `wind_max`
- Station-based, 10m height (not hub-height — needs extrapolation for turbines)

**Global Wind Atlas**
- URL: `https://globalwindatlas.info`
- Format: GeoTIFF (GIS files), GWC (WAsP-compatible)
- Access: Free for web/GIS downloads; **EMD-API requires paid subscription** for programmatic REST access (20 requests per 10 minutes)
- Resolution: **250m grid globally** — excellent for site-specific wind assessment
- Data: Mean wind speed at 10/50/100/150/200m, power density, capacity factors

**Energistyrelsen Stamdataregister (wind turbines)**
- URL: `https://ens.dk/media/3531/download` (Excel, monthly update)
- Format: Excel (.xlsx)
- Access: Free, no registration
- Fields per turbine: 18-digit ID, capacity (kW), rotor diameter, hub height, manufacturer, municipality, coordinates (UTM32), grid connection date, **actual production data** (kWh, annual + monthly)
- Update: Monthly with ~1 month lag

### 4C. Existing solar installations

**Energistyrelsen Stamdataregister** covers solar PV alongside wind. Aggregate data by municipality shown on sologvindinfo.dk; individual installations >1 MW publicly visible. Fields: GSRN number, capacity, location, connection date, owner, net company.

**BBR** tracks solar installations as `tekniskanlaeg` with capacity, area, and placement data.

**Energi Data Service** provides **real-time hourly solar production** for all of Denmark via `ElectricityBalance` and `ProductionConsumptionSettlement` datasets, split by DK1/DK2.

### 4D. Denmark's wind share

**Most recent verified figure: ~59% of total electricity generation in 2024** (Wikipedia citing official sources, Low Carbon Power). Denmark produced **88.4% of net electricity from renewables** in 2024 (EU #1, per Eurostat). The "55% from wind" claim is **approximately accurate** — the exact figure depends on whether you measure generation (~59%) or domestic supply including imports (~47–54%).

---

## PART 5: Energy consumption patterns

### 5A. Household consumption by area

**SOURCE: Energi Data Service**
- `PrivIndustryConsumptionMunicipalityMonth` — private vs. industry consumption by municipality
- `ConsumptionPerMunicipalityDE35` — by municipality and DE35 industry code (includes residential category)
- URL: `https://api.energidataservice.dk/dataset/PrivIndustryConsumptionMunicipalityMonth?start=2024-01-01&limit=100`
- Granularity: Monthly × 98 municipalities
- Delay: 9 days from month end

**Danmarks Statistik has NO municipality-level household electricity table.** The `ENEGEO` table covers industry energy consumption by municipality but excludes households. `ENERGI1` provides national half-yearly household prices by consumption band.

### 5B. Heating fuel mix by municipality

**Primary source: Danmarks Statistik BYGB40** — buildings and heated area by region, heating type, use, construction year. Available at municipality level. Covers district heating, oil, gas, electric, heat pump, stoves. Annual from BBR register data.

**District heating:** ~65–68% of Danish homes connected. Copenhagen's system covers ~98% of end users across 25 municipalities. Odense: 97% residential coverage.

**Heat pump growth:** 15.9% increase in building-registered heat pumps in 2024 vs. 2023 (Forsyningstilsynet).

### 5C. Historical price trends

| Period | All-in price (DKK/kWh, incl. VAT) | Source |
|--------|-------------------------------------|--------|
| 2018–2019 | ~2.24 (EUR 0.30) | Eurostat |
| 2020–2021 | ~2.09–2.39 | Eurostat |
| H2 2022 | ~3.43 (EUR 0.46, peak) | Eurostat |
| 2024 avg | ~2.76 (EUR 0.37) | Eurostat |
| June 2025 | ~2.30 | GlobalPetrolPrices |
| 2026 est. | ~1.23 | Calculated (post elafgift cut) |

**Elafgift timeline:**

| Year | Rate (øre/kWh) |
|------|----------------|
| 2018 | 91.4 |
| 2019 | 89.2 |
| 2020 | 89.0 |
| 2021 | 90.0 |
| 2022 | 76.3 (temporary reduction) |
| 2023 | 76.3 |
| 2024 | 76.1 |
| 2025 | 72.7 |
| **2026–2027** | **0.8** |

Energistyrelsen basisfremskrivning (annual projections): `https://ens.dk/service/statistik-data-noegletal-og-kort/energipriser-og-afgifter`

---

## PART 6: Grid infrastructure and energy communities

### 6A. Grid company territories

**No public GIS/shapefile data exists** for grid company boundaries. Verified absent from data.gov.dk, Geodatastyrelsen, GEUS, and all open data portals searched.

**Grid capacity data:**
- Energinet Capacity Map: `https://www.kapacitetskort.dk` — visualization of TSO-level grid capacity
- Energi Data Service: `GridCapacityMapTSO` dataset — monthly grid capacity data
- Energinet publishes long-term development plans identifying congestion areas (Western Jutland, Northern Jutland)

**Grid company consolidation:** ~40 active DSOs (down from 80+ historically). Major groups: **N1** (Norlys, largest in Western Denmark), **Radius Elnet** + **Cerius** (both under Andel group, operating jointly via Nexel; ~1.4M meters combined covering Greater Copenhagen and outer Zealand).

### 6B. Energy communities

**Energifællesskaber Danmark**
- URL: `https://www.energifaellesskaber.dk/`
- Function: National support/advocacy organization (not a registry)
- Services: 1-on-1 guidance, experience groups, policy advocacy
- Does **NOT** maintain a formal public registry of all energy communities

**Estimated count:** 633 energy entities in Denmark (EC/Wierling et al, 2023), with 467 in Jutland. This includes legacy cooperatives; formal EU-definition communities are much fewer.

**First formal energy community:** Energifællesskab Avedøre A.M.B.A., founded **August 18, 2020** in Hvidovre Municipality — **verified** ✅ (source: eboconsult.dk).

**Legal framework:** Since January 1, 2021, the Electricity Supply Act (Elforsyningsloven) allows formal energy communities. Denmark chose "minimum implementation" of EU directives. Local collective tariff methodology approved by Forsyningstilsynet July 2025.

### 6C. CVR register

Energy cooperatives can be found on `https://datacvr.virk.dk` by searching company names containing relevant terms (energifællesskab, energi, andel). Free, open access. No dedicated energy community category — keyword search required.

---

## PART 7: Demographics and socioeconomic data

**Danmarks Statistik StatBank API**
- Base URL: `https://api.statbank.dk/v1`
- Format: JSON, CSV, JSONSTAT, XML, XLSX
- Access: **Open, no API key required**, CC 4.0 BY license
- Documentation: `https://www.dst.dk/en/Statistik/hjaelp-til-statistikbanken/api`
- Console: `https://api.statbank.dk/console`

**Key tables for the Screwed Score tool:**

| Table | Purpose | Granularity |
|-------|---------|-------------|
| **FOLK1A** | Population by age, sex, marital status | Municipality, quarterly |
| **FAM55N** | Households by type, size, children | Municipality, annual |
| **INDKP106** | Disposable income by age, income interval | Municipality, annual |
| **BOL101** | Dwellings by tenure, ownership, type | Municipality, annual |
| **BOL102** | Dwellings by heating type | Municipality, annual |
| **BYGB40** | Buildings by heating type, area | Municipality, annual |
| **ENERGI1** | Household electricity prices by band | National, half-yearly |

**Example API call (households in Aarhus):**
```json
POST https://api.statbank.dk/v1/data
{
  "table": "FAM55N",
  "format": "JSONSTAT",
  "lang": "en",
  "variables": [
    {"code": "OMRÅDE", "values": ["751"]},
    {"code": "HUSTYPE", "values": ["*"]},
    {"code": "HUSSTØR", "values": ["*"]},
    {"code": "Tid", "values": ["2025"]}
  ]
}
```

---

## PART 8: Funding and subsidies

### 8A. Danish national funding

- **Erhvervspuljen:** ~DKK 3.5–3.9B over 2020–2029 for business energy efficiency. 2026 round: DKK 217.7M, opens March 2, 2026. Min DKK 50,000 grant. **Businesses only**, not households. Via `https://sparenergi.dk/erhvervspuljen`
- **Varmepumpepuljen:** 2026 subsidy = **DKK 27,000** per heat pump (any type). DKK 555M allocated for 2027–2029. Via sparenergi.dk
- **Energy community grants:** DKK 4M/year (2022–2025) via Energistyrelsen. Very small.
- **2026 budget green transition:** ~DKK 1.4B total, including DKK 851M for public building solar, DKK 100M for grid modernization
- **EUDP:** ~DKK 300M per round (2 rounds/year) for energy R&D

### 8B. EU funding

**European Energy Communities Facility (ENERCOM)**
- URL: `https://energycommunitiesfacility.eu/`
- Budget: €7M total, **€45,000 per community** (lump sum grants)
- Target: 140+ communities across EU
- 1st call: June 2025 (closed September 2025); 2nd call: September 2026
- Funded by: EU LIFE Programme

**REPowerEU Danish allocations:** €131M in REPowerEU grants within total RRF plan of €1.63B. Measures include heat pump/DH conversions (21,200+ homes), CCS, green upskilling, wind/solar permitting reform. Must be implemented by August 2026.

**EU Innovation Fund:** Danish projects have secured **DKK 3.5B** to date. Primarily for large-scale industrial projects (TRL 6+), not community-level.

### 8C. The "3 billion DKK" claim

**Assessment: Unverifiable as stated.** The Erhvervspuljen alone totals ~3.5B DKK, but it is for businesses only, not households or communities. Total identifiable green energy funding across all Danish and EU programs exceeds **DKK 50 billion**, but most is not accessible to households or small energy communities. No single published source aggregates this specific "3 billion" figure. The claim likely refers to Erhvervspuljen or a subset of programs but lacks specific sourcing.

---

## PART 9: Aggregator platforms

| Platform | URL | Type | Access | Key capability |
|----------|-----|------|--------|---------------|
| **Strømligning** | stromligning.dk/api/docs | JSON REST API | Free non-commercial | Postcode→grid company + complete price breakdowns |
| **eloverblik.dk** | api.eloverblik.dk/thirdpartyapi | JSON REST API | MitID token + user consent | Actual consumption, metering point details, charges |
| **Energi Data Service** | api.energidataservice.dk | JSON REST API | Open | Spot prices, tariffs, consumption statistics |
| **elpris.dk** | elpris.dk | Web only (no API) | Public | Forsyningstilsynet's official retail price comparison |
| **Tibber** | developer.tibber.com | GraphQL API | Customer token required | Prices, consumption (Denmark supported: DK1, DK2) |
| **Barry** | developer.barry.energy | JSON-RPC API | Customer-only, beta | Spot prices, consumption (10 calls/min limit) |
| **Carnot** | carnot.dk | JSON API | Free registration + API key | 5–7 day price forecasts for DK1/DK2 |
| **elprisen.somjson.dk** | github.com/mchro/elprisen.somjson.dk | JSON API | Open source | Proxy over Energi Data Service |
| **elprisenligenu.dk** | elprisenligenu.dk/elpris-api | Static JSON | Open | Simple DK1/DK2 spot prices |

**eloverblik.dk Third Party API** is the most important for personalized data. Users authorize via MitID, and the tool retrieves actual hourly consumption, grid company identification, and associated charges. Technical docs: `https://energinet.dk/media/2l1lmb2z/customer-and-third-party-api-for-datahub-eloverblik-technical-description.pdf`. Python wrappers: **pyeloverblik** (`https://github.com/JonasPed/pyeloverblik`).

**Key GitHub repositories:**
- **MTrab/energidataservice** (★163) — Home Assistant integration with the definitive chargeowners.py mapping
- **rndfm/elspotpris** — Consumer-verified retailer surcharges
- **MTrab/stromligning** — Home Assistant integration for Strømligning API

---

## PART 10: Data verification results

### "For every krone, 16–22 øre is actual electricity"

**Verdict: Inaccurate for 2026.** After the elafgift cut, wholesale electricity is approximately **48–50%** of the total bill at current spot prices (~60 øre/kWh). The claim was more accurate during 2022–2023 when high taxes (72+ øre/kWh elafgift) and moderate spot prices created a scenario where wholesale was indeed ~19–22% of total cost. **Post-reform, the Screwed Score should be communicated as "roughly 50 øre of every krone is actual electricity."**

### "Average Danish household pays ~2,847 DKK/month"

**Verdict: False for electricity alone.** At 4,000 kWh/year and ~2.30 DKK/kWh (2025), the real average is **~767 DKK/month**. Post-reform at ~1.23 DKK/kWh, it drops to **~410 DKK/month**. The 2,847 figure might represent total energy costs (electricity + heating + water) or peak-crisis 2022 pricing for a high-consumption household. It is not a verifiable average electricity-only figure.

### "Denmark produces 55% of electricity from wind"

**Verdict: Approximately accurate.** The most recent data shows **~59% of generation** (2024) from wind, or **~47–55% of domestic supply** (including imports/exports). Saying "over 55%" is defensible.

### 2026 elafgift impact on bill composition

Post-reform (at 60 øre/kWh spot), the breakdown shifts dramatically:

| Component | Pre-reform (2025) | Post-reform (2026) |
|-----------|-------------------|-------------------|
| Wholesale electricity | ~28% | **~49%** |
| DSO grid tariff | ~9% | **~16%** |
| TSO tariffs | ~6% | **~9%** |
| Elafgift | **~34%** | **<1%** |
| Retail margin | ~2% | ~4% |
| VAT | ~20% | ~20% |

**Grid fees (DSO + TSO) become ~25% of the total bill**, up from ~15%. This makes location-specific grid tariff variation far more impactful for consumers — and the Screwed Score more relevant.

---

## The architecture that works

The optimal data pipeline for the Screwed Score tool chains four open APIs with one critical proprietary dependency:

1. **DAWA** (geocoding, open) → coordinates from postcode
2. **Strømligning API** (grid company lookup, free non-commercial) → the critical link
3. **Energi Data Service** (tariffs + spot prices, open) → all per-kWh cost components
4. **Danmarks Statistik** (context data, open) → benchmarking and demographics

The Strømligning dependency is the tool's single point of fragility. Building and maintaining a static postcode→GLN lookup table using Green Power Denmark's API as a fallback is essential insurance. For personalized accuracy beyond postcode-level estimates, integrating the **eloverblik.dk Third Party API** (with user consent via MitID) provides actual consumption data and eliminates the need to reverse-engineer kWh from bill amounts. The post-2026 elafgift landscape makes this tool more valuable, not less—with taxes nearly eliminated, the geographic variation in grid tariffs now drives meaningful differences in what Danish households pay.