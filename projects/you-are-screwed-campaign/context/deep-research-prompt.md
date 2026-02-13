# Deep Research Prompt: Public Data Sources for "How Screwed Are You?"

## Context

The "How Screwed Are You?" campaign by Orklys needs to replace mocked calculations with real, verifiable data. The tool takes a Danish postcode + monthly bill amount and produces a "Screwed Score" (0-10) showing how much of a user's bill is actual electricity vs. tariffs/fees/taxes. Currently every number is hardcoded fiction.

This plan delivers a **ready-to-use deep research prompt** designed for Claude Deep Research, Perplexity, or similar tools. The prompt was assembled by three specialized agents focusing on: (1) Danish energy data infrastructure, (2) viral product features powered by data, and (3) technical API specifications.

**Key discovery during research:** Denmark is cutting elafgift from 72.7 øre/kWh to 0.8 øre/kWh in 2026 — the single biggest component of "being screwed" is about to disappear, shifting the narrative to grid fees (nettarif) which Energy Communities directly address.

---

## The Prompt (copy everything below this line)

---

# Deep Research: Public Data Sources for a Danish Energy Bill Transparency Tool

## What I'm Building

A consumer-facing web tool for the Danish energy market. Users enter their **postcode** and **monthly electricity bill amount** (DKK). The tool calculates a "Screwed Score" showing what percentage of their bill is actual electricity versus tariffs, fees, and taxes — broken down by component, personalized to their location.

The tool currently runs on mock data. I need to replace every mock calculation with real, verifiable, publicly available data.

**Be specific.** For every data source you find, provide:

- Exact URL (not just the domain)
- Data format (JSON API, CSV download, XML, SOAP, etc.)
- Access requirements (open, free registration, paid, API key, NemID/MitID)
- Granularity (address / postcode / municipality / grid area / price zone / national)
- Update frequency (real-time, hourly, daily, monthly, annually)
- Key fields/endpoints relevant to my use case
- Known limitations (rate limits, coverage gaps, deprecation warnings)

Do not list sources you cannot verify exist. If a portal has been restructured or deprecated, note that.

---

## PART 1: ELECTRICITY BILL COMPONENTS (HIGHEST PRIORITY)

A Danish residential electricity bill consists of:

- **Spotpris** — wholesale electricity cost, set hourly by Nord Pool (DK1 = West, DK2 = East)
- **Nettarif** — grid tariff charged by the local netselskab (~40 grid companies, each with different rates)
- **Systemtarif** — charged by Energinet for system balancing
- **Transmissionstarif** — charged by Energinet for high-voltage transmission
- **Elafgift** — state electricity tax per kWh (currently ~0.761 DKK/kWh, dropping to 0.8 øre/kWh in 2026-2027)
- **Moms** — 25% VAT on everything above

### 1A. Grid Company Tariff Data (The Variable That Differs by Location)

Research **Energi Data Service** (energidataservice.dk) — Energinet's open data platform:

The `DatahubPricelist` dataset at `https://api.energidataservice.dk/dataset/DatahubPricelist` contains per-supplier tariff data. Document:

- What do `ChargeType` codes D01, D02, D03 mean? Find the official enumeration.
- What is the `GLN_Number` field? Is this the unique identifier for each grid company?
- The `Price1` through `Price24` fields — do these represent hourly tariff rates or something else?
- How does `ResolutionDuration` work? (P1M = monthly, PT1H = hourly?)
- Example filter query for a specific grid company: `?filter={"GLN_Number":["5790000392261"]}&start=now-P1M`
- How many distinct grid companies (unique GLN_Numbers) exist in this dataset?
- Is elafgift included in DatahubPricelist or in a separate dataset?

Where is the **master list** mapping GLN numbers to grid company names and geographic service areas?

### 1B. Postcode-to-Grid-Company Mapping (THE CRITICAL MISSING LINK)

This is the most important lookup: given a 4-digit Danish postcode, which netselskab serves that area?

Research ALL of these potential sources:

- Does Energinet's DataHub provide a postcode-to-grid-area mapping?
- Does Energistyrelsen (ens.dk) publish grid company boundary data as downloadable GeoJSON/shapefile? (They have a PDF map at `ens.dk/sites/ens.dk/files/Statistik/elnetgraenser.pdf` — is there a machine-readable version?)
- Does Forsyningstilsynet maintain a registry of licensed grid companies with geographic coverage?
- Can `elpris.dk` or `stromligning.dk/tariffer` postcode lookups be used or reverse-engineered?
- Can the mapping be derived by cross-referencing `DatahubMeasuringPointStatisticsMunicipality` (which has municipality codes) with `DatahubPricelist` (which has GLN numbers)?
- Does DAWA (Danmarks Adressers Web API at `dawadocs.dataforsyningen.dk`) include any utility/grid information?

### 1C. Spot Prices

- The `DayAheadPrices` dataset on Energi Data Service (replaces deprecated `Elspotprices` as of September 2025). Document:
  - Exact column names (PriceArea, SpotPriceDKK, SpotPriceEUR, HourUTC, HourDK?)
  - How far back does historical data go?
  - What unit are prices in — EUR/MWh or DKK/kWh?
  - Example query for trailing 12-month average DK1 price
- Is the postcode-to-price-area mapping simply: 1000-4999 = DK2 (East), 5000-9999 = DK1 (West)? Verify this.

### 1D. National Fixed Components

Confirm current rates and their official sources:

- Systemtarif (Energinet) — current rate, where published
- Transmissionstarif (Energinet) — current rate, where published
- Elafgift — current rate AND the 2026-2027 reduction timeline to 0.8 øre/kWh (exact effective date, source from Skatteministeriet)
- PSO — is this still applicable or has it been phased out?
- Is Tariff Model 3.0 (time-differentiated nettarif) now mandatory, and how does it affect average calculations?

### 1E. Retail Electricity Margin

- What is the typical retail markup by Danish elhandelsselskaber?
- Does Forsyningstilsynet publish data on average retail margins?
- Does `elpris.dk` have an API or structured data?

### CRITICAL QUESTION FOR PART 1:

Given postcode 8000 and a monthly bill of 2,500 DKK, walk through the exact data pipeline: which APIs do I call, in what order, with what parameters, to produce an accurate bill breakdown? Where does the pipeline break — what mapping or data is missing?

---

## PART 2: POSTCODE LEADERBOARD — "Your Area Is the Nth Most Screwed"

Danes are competitive about geography. Ranking postcodes/municipalities by how "screwed" residents are creates the most shareable feature.

### Data Needed:

- Average nettarif per grid company (from DatahubPricelist — extract and rank all ~40 companies)
- Average household electricity consumption per municipality (Danmarks Statistik tables — which specific codes? `ENE` series? `ENEGEO`? `PrivateConsumptionHeatingMonth` on Energi Data Service?)
- Number of households per postcode or municipality (Danmarks Statistik `BOL` series or BBR aggregates)
- Grid company territory mapping (see Part 1B)

### Research:

- Does Forsyningstilsynet publish an annual benchmarking comparison of grid company tariffs in tabular format? (Their 2024 National Report exists as PDF — is there downloadable data?)
- What Danmarks Statistik tables contain residential energy consumption by municipality? Provide exact table codes and API calls.
- The `PrivateConsumptionHeatingHour` / `PrivateConsumptionHeatingMonth` datasets on Energi Data Service — do these break down by municipality? What housing/heating categories exist?

---

## PART 3: PROPERTY AND BUILDING DATA

### 3A. BBR (Building and Dwelling Register)

- Is BBR data accessible via API through Datafordeler.dk? What authentication is required?
- Can BBR be queried by postcode to get: building count, building types, construction years, heating types (varmeinstallation), roof types/areas, floor areas?
- Does BBR record whether a building has registered solar panels (under "teknisk anlaeg")?
- What are the BBR codes for heating types? (fjernvarme, naturgas, varmepumpe, oliefyr, etc.)
- Does DAWA include some BBR data via its building endpoints? Is this a simpler alternative?

### 3B. Energy Performance Certificates (Energimaerkning)

- The EMOData API requires registration via `emo-info@ens.dk`. What fields does `FindEnergyLabelInput` return? (Energy label grade A-G, estimated kWh/m2/year, primary heating type?)
- Can it be queried by postcode for aggregate statistics, or only individual addresses?
- Is there a bulk download of all ~700,000 energy labels?
- Does sparenergi.dk have a simpler public API for energy label lookups?

### 3C. Housing Type Distribution

- What Danmarks Statistik table contains dwelling type distribution (detached house, apartment, terraced house) per municipality? Provide exact table code and API call.

---

## PART 4: SOLAR AND WIND POTENTIAL

### 4A. Solar Potential Per Building

- Energistyrelsen launched **sologvindinfo.dk** in February 2025, showing rooftop solar potential per building using laser scan data + BBR + meteorological data. Does it have a public API? Is the underlying data (from Klimadatastyrelsen's elevation model) available programmatically?
- **PVGIS** (EU Joint Research Centre) API at `https://re.jrc.ec.europa.eu/api/v5_3/`:
  - Example call for Aarhus: `PVcalc?lat=56.15&lon=10.21&peakpower=6&loss=14&angle=35&aspect=0&outputformat=json`
  - Which `raddatabase` for Denmark — `PVGIS-SARAH3` or `PVGIS-ERA5`?
  - For a standard 6 kWp Danish rooftop system, what annual production does it estimate?

### 4B. Wind Data

- DMI (Danish Meteorological Institute) Open Data — API portal for wind speed data by location?
- Global Wind Atlas (globalwindatlas.info) — API or download-only?
- Energistyrelsen Stamdataregister for wind turbines — open data with GPS coordinates? Does it distinguish community-owned (vindmøllelaug) from commercial?

### 4C. Existing Solar Installations

- Is there a public registry of installed solar PV in Denmark? Energistyrelsen or BBR teknisk anlæg?
- Can I query "how many buildings in postcode XXXX have solar panels"?

---

## PART 5: ENERGY CONSUMPTION PATTERNS

### 5A. Household Consumption by Area

- Danmarks Statistik — which specific table codes contain household electricity consumption by municipality? Is there per-postcode data?
- Energi Data Service `PrivateConsumptionHeatingMonth` — does this provide consumption by municipality and housing type?
- DataHub aggregated consumption — is any anonymized/aggregated data available publicly, or is DataHub access strictly for market participants?

### 5B. Heating Fuel Mix by Municipality

- District heating (fjernvarme) coverage maps — Dansk Fjernvarme member data?
- Gas grid coverage — where is this mapped?
- Heat pump adoption rates by area
- Can BBR heating type codes be aggregated by postcode to produce: "X% fjernvarme, Y% varmepumpe, Z% naturgas"?

### 5C. Historical Price Trends

- Time series for total consumer electricity price in Denmark, broken down by component, 2018-2026
- Energi Data Service: does `DatahubPricelist` contain historical entries going back to 2018?
- Energistyrelsen "basisfremskrivning" — do they publish 5-year tariff projections?
- Energinet System Plan — long-term tariff forecasts available?
- Critical timeline: compile exact elafgift rates from 2018 to 2027 including the announced 0.8 øre/kWh cut

---

## PART 6: GRID INFRASTRUCTURE AND ENERGY COMMUNITIES

### 6A. Grid Company Territories

- Definitive mapping of which netselskab serves which geographic area — GIS/shapefile, postcode lookup table, or only static PDF maps?
- Grid capacity/congestion data — where can new generation connect? Does Energinet publish available capacity by grid area?

### 6B. Existing Energy Communities in Denmark

- Is there a public registry? How many exist as of 2025/2026?
- Energifællesskaber Danmark (energifaellesskaber.dk) — do they maintain a public map/list?
- Energistyrelsen funding pool for local energy communities — are grant recipients in a public database?
- The first Danish energy community was "Energifællesskab Avedøre" in Hvidovre Municipality (August 2020). How many have been established since?
- REScoop.eu, communitypower.eu — do they have structured databases/APIs for EU-wide data?

### 6C. Danish Cooperative Registries

- Erhvervsstyrelsen CVR register — can I query for energy cooperatives (energifællesskaber, solcellelaug, vindmøllelaug)?

---

## PART 7: DEMOGRAPHICS AND SOCIOECONOMIC DATA

Using Danmarks Statistik StatBank API (`https://api.statbank.dk/v1/`):

- Average household income by municipality — which table code? (INDKP series?)
- Home ownership rates by area — which table?
- Household size distribution by area
- Population by age group by municipality
- Provide the exact API call to list all energy-related tables: `POST https://api.statbank.dk/v1/tables {"subjects":["energi"],"lang":"en"}`

---

## PART 8: FUNDING AND SUBSIDIES

### 8A. Danish National Funding

- Energistyrelsen "pulje til lokale energifællesskaber" — current allocation, eligibility, public list of past recipients?
- Erhvervspuljen at sparenergi.dk — applicable to Energy Community infrastructure?
- Varmepumpepuljen 2026 — higher subsidies for heat pumps, relevant for hybrid EC projects?

### 8B. EU Funding Programs

- European Energy Communities Facility — status, application process, Danish eligibility
- EU Innovation Fund — small-scale project eligibility for ECs
- LEADER / LAG programs — which Danish LAG regions exist and their current funding rounds?
- REPowerEU funds channeled through Danish national plans — allocation amounts?

### 8C. Verification of Campaign Claims

The campaign currently claims "over 3 billion DKK in available EU/national funding." Identify the specific programs and amounts that support or contradict this claim.

---

## PART 9: AGGREGATOR PLATFORMS AND EXISTING TOOLS

Don't just list raw data sources. Also investigate platforms that already aggregate Danish energy data:

- **elpris.dk** — price comparison tool. Does it have an API or structured data export?
- **sparenergi.dk** — energy advice portal. API access?
- **eloverblik.dk** — consumers' own DataHub consumption data. What does the Third Party API provide? (PDF docs at `energinet.dk/media/m2xo05he/...`)
- **tjekdinelpris.dk** — API or scrapable?
- **stromligning.dk/tariffer** — appears to have tariff lookup by postcode. How does it source data?
- **Barry, Tibber** — do these smart energy companies use public APIs that I could also access?
- **GitHub repositories** — are there open-source projects parsing Danish energy data? Search for repos related to Energi Data Service, Danish energy APIs, elafgift calculators.

---

## PART 10: DATA VERIFICATION REQUESTS

Calculate these from real data to verify or correct campaign claims:

1. **"For every krone on your energy bill, roughly 16-22 øre is actual electricity."** Using average household consumption of ~4,000 kWh/year, current average spot price, plus all tariffs and taxes — what is the actual percentage that goes to electricity? Show the math.

2. **"The average Danish household pays ~2,847 DKK/month for electricity."** What is the real figure? Source? Including or excluding heating?

3. **"Denmark produces 55% of its electricity from wind."** Most recent verified annual figure?

4. **Impact of the 2026 elafgift cut:** If elafgift drops from ~0.761 DKK/kWh to 0.008 DKK/kWh, how does the bill breakdown change? What percentage of the bill becomes grid fees (nettarif) — the exact fees that Energy Communities can reduce?

---

## PART 11: THE COMPLETE POSTCODE PIPELINE (VALIDATE THIS)

```
1. User enters postcode (e.g., 8000) and monthly bill (e.g., 2,500 DKK)
2. DAWA API: postcode 8000 → municipality code 0751 (Aarhus), coordinates (56.15, 10.21)
3. [MAPPING NEEDED]: municipality/postcode → grid company GLN number
4. Energi Data Service DatahubPricelist: GLN → nettarif rates
5. Energi Data Service: systemtarif + transmissionstarif (national, from Energinet)
6. Energi Data Service DayAheadPrices: DK1/DK2 → average spot price
7. Known fixed: elafgift rate, moms 25%
8. CALCULATION: derive kWh consumption from total bill, then break down per component
9. Screwed Score = percentage of bill that is NOT electricity
```

**For each step: confirm the API endpoint works, identify missing links, and suggest workarounds.**

---

## SEARCH STRATEGIES

To find sources that aren't immediately obvious:

1. Search Danish government open data portals: **data.gov.dk**, **Datafordeler.dk**, **virk.dk/data**
2. Search the Danish Energy Agency (Energistyrelsen) publication archives
3. Search Forsyningstilsynet databases and annual reports
4. Check Energinet's full API documentation beyond the main page
5. **Search Danish-language terms:** "åbne data energi," "eltarif API," "netselskab postnummer," "elafgift satser," "BBR API," "energimærkning API," "nettarif sammenligning"
6. Check Danish developer forums, GitHub issues, and Stack Overflow posts about Danish energy data
7. Look for hackathon projects built on Danish energy data (Energi Data Service has hosted hackathons)
8. Search academic papers citing Danish energy consumption microdata — they name their sources
9. Check if any Freedom of Information (aktindsigt) requests have released relevant datasets

---

## OUTPUT FORMAT

For each source, provide:

```
SOURCE NAME
- URL: [exact URL]
- Format: [JSON API / CSV / XML / etc.]
- Access: [Open / Registration / Paid / Restricted]
- Granularity: [Address / Postcode / Municipality / Grid area / Price zone / National]
- Update frequency: [Real-time / Hourly / Daily / Monthly / Annual]
- Key fields: [Most relevant data points]
- Limitations: [Rate limits, gaps, issues]
- How it feeds the Screwed Score: [Direct connection to the calculation]
```

Group by Part number. Within each Part, rank from most actionable (free API, postcode-level, well-documented) to least actionable (PDF report, national average only).

**What success looks like:** A user enters postcode 8000 and 2,500 kr/month. The system looks up their grid company, pulls real tariff rates, calculates real spot prices, and produces a bill breakdown where every number traces to a named, dated public source. Then it enriches with solar potential, neighborhood context, and available funding — all from verified data.

---

## Known API Endpoints (Quick Reference)

| Source              | Base URL                                | Auth              | Format       |
| ------------------- | --------------------------------------- | ----------------- | ------------ |
| Energi Data Service | `https://api.energidataservice.dk`      | None              | JSON/CSV     |
| DAWA                | `https://api.dataforsyningen.dk`        | None              | JSON         |
| Danmarks Statistik  | `https://api.statbank.dk/v1/`           | None              | JSONSTAT/CSV |
| PVGIS               | `https://re.jrc.ec.europa.eu/api/v5_3/` | None              | JSON/CSV     |
| Datafordeler (BBR)  | `https://services.datafordeler.dk`      | Registration      | JSON/GML     |
| ENTSO-E             | `https://web-api.tp.entsoe.eu/api`      | Free registration | XML          |
| Nord Pool (via EDS) | Use Energi Data Service mirror          | None              | JSON         |

---

_End of prompt._
