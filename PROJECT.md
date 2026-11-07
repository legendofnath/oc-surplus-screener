# OC Surplus Land Screener

Goal: rank Orange County surplus-land notices as affordable-housing site CANDIDATES, shown on a map and a ranked table. Never claim a site is legally eligible. Label every assumed number as an assumption. Show a source link for every number. Public data only. No API keys in the repo.

## Team and roles (each person edits ONLY their own files)
- Integrator (legendofnath): data/sites.json, README.md, PROJECT.md, final submission
- Checks and scoring (tanbui): data/scored_sites.json
- App (carcollector25): index.html (reads data/scored_sites.json)

## Contract 1: data/sites.json (array of objects)
id, apn, address, city, lat, lon, acres, general_plan, zoning_listed, existing_use, status, source_url
Source: https://data.ca.gov/dataset/surplus-land-act-notices-of-availability-received (Orange County cities only, drop status "Archive", convert sizes like "9,005 SF" to acres).

## Contract 2: data/scored_sites.json (every sites.json field plus:)
checks: zoning_scag, housing_element_site (true/false), flood_zone, fire_zone, in_hqta (true/false)
score: total, available (30), size (20), zoning (20), hazards (15), transit (15)
sources: object of check name to URL

## Data services (use inSR=4326 and outSR=4326; cache everything; do not hammer servers)
- SCAG zoning: https://maps.scag.ca.gov/scaggis/rest/services/LDX/Zoning_poly_OR/MapServer
- SCAG housing-element sites: https://maps.scag.ca.gov/scaggis/rest/services/LDX/HousingElementSite_poly_scag/MapServer
- FEMA flood (layer 28, retry on timeout): https://hazards.fema.gov/arcgis/rest/services/public/NFHL/MapServer
- CAL FIRE: https://services.gis.ca.gov/arcgis/rest/services/Environment/Fire_Severity_Zones/MapServer
- Caltrans HQTA: https://caltrans-gis.dot.ca.gov/arcgis/rest/services/CHrailroad/CA_HQ_Transit_Areas/FeatureServer/0

## App behavior
Leaflet map plus sortable table. Click a row zooms the map. Click a pin highlights the row. Show the score breakdown and source links for each site. State that all weights are assumptions.

## Until real files exist
Build against a 3-row sample file with "SAMPLE": true on each row. Remove the sample before submitting.
