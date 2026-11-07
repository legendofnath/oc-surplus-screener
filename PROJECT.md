# OC Surplus Land Screener

Goal: rank Orange County surplus-land notices as affordable-housing site CANDIDATES, shown on a map and a ranked table. Never claim a site is legally eligible. Label every assumed number as an assumption. Show a source link for every number. Public data only. No API keys in the repo.

IMPORTANT DATA FACT: every Orange County notice in the HCD dataset has status "Archive" (the notice window has closed; 0 active OC notices today). Archive does NOT mean the land is gone, but it may have been sold or committed. The app must label every site "Historical notice - verify current status and ownership". There is no "availability" score.

## Team and roles (each person edits ONLY their own files)
- Integrator (legendofnath): data/sites.json, README.md, PROJECT.md, final submission
- Checks and scoring (tanbui): data/scored_sites.json
- App (carcollector25): index.html (reads data/scored_sites.json)

## Contract 1: data/sites.json (DONE, 235 sites, array of objects)
Raw link: https://github.com/legendofnath/oc-surplus-screener/raw/main/data/sites.json
Fields: id, apn, address, city, city_inferred, agency, lat, lon, acres, acres_raw, general_plan (always null), zoning_listed, existing_use, status (always "Archive"), notice_issued (YYYY-MM-DD), source_url, notice_count
Notes: 4 sites have acres null, 2 have city null, 11 have zoning_listed null. Handle blanks without crashing. zoning_listed is the city's own free-text zoning/general plan note.
Source: https://data.ca.gov/dataset/surplus-land-act-notices-of-availability-received

## Contract 2: data/scored_sites.json (every sites.json field plus:)
checks: zoning_scag, housing_element_site (true/false), flood_zone, fire_zone, in_hqta (true/false)
score: total, zoning (25), size (20), existing_use (15), hazards (15), transit (15), recency (10)
sources: object of check name to URL

## Scoring (100 points, ALL WEIGHTS ARE OUR ASSUMPTIONS and must be shown as such)
- zoning: 25 (SCAG zoning allows housing, or site is in a housing-element list)
- size: 20 (more acres scores higher)
- existing_use: 15 (vacant or parking lot scores higher; occupied or open space scores lower)
- hazards: 15 (deduct for FEMA flood zone or high CAL FIRE severity)
- transit: 15 (inside a Caltrans high-quality transit area)
- recency: 10 (newer notice_issued scores higher, because it is more likely still public-held)

## Data services (use inSR=4326 and outSR=4326; cache everything; do not hammer servers)
- SCAG zoning: https://maps.scag.ca.gov/scaggis/rest/services/LDX/Zoning_poly_OR/MapServer
- SCAG housing-element sites: https://maps.scag.ca.gov/scaggis/rest/services/LDX/HousingElementSite_poly_scag/MapServer
- FEMA flood (layer 28, retry on timeout): https://hazards.fema.gov/arcgis/rest/services/public/NFHL/MapServer
- CAL FIRE: https://services.gis.ca.gov/arcgis/rest/services/Environment/Fire_Severity_Zones/MapServer
- Caltrans HQTA: https://caltrans-gis.dot.ca.gov/arcgis/rest/services/CHrailroad/CA_HQ_Transit_Areas/FeatureServer/0

## App behavior
Leaflet map plus sortable table. Click a row zooms the map. Click a pin highlights the row. Show the score breakdown and source links for each site. State that all weights are assumptions and every site is a historical notice to verify.

## Until real files exist
Build against a 3-row sample file with "SAMPLE": true on each row. Remove the sample before submitting.
