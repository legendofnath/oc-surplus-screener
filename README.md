# OC Surplus Land Screener

> A map and ranked table that helps a small housing nonprofit decide which Orange County surplus-land notices are worth a phone call, with the zoning, hazard and transit checks shown next to each site.

**Team:** OC Surplus Land Screener
**Members:** legendofnath (integrator, data, README), tanbui (checks and scoring), carcollector25, also known as Sam (app, Newport Beach lot data)
**Lane:** 1 Find it ("Where can the next affordable home go?")
**Built at:** Claude Impact Lab: UCI × SoCal Affordable Housing, November 7, 2026

## The real user

This is a made-up user based on scenario F02 in `lanes/1-find-it/scenarios.json`: a one-person acquisitions team at a small housing nonprofit with no GIS staff. They ask, "Give me ten sites in one city I should call about this month, and why each one." No real person or organization is described.

**Before:** Public surplus-land notices sit in a state spreadsheet. To judge one site they open the notice, look up the zoning on a city map, check a flood map, check a fire-hazard map and look for nearby transit, one parcel at a time. Doing that for hundreds of notices takes days, so most are never looked at.

**After:** They open one page, see all 235 Orange County notices on a map and in a ranked table, filter to a city, and click a site to see its score broken into six parts, with a source link for every check. They leave with a short shortlist of sites to verify and call about.

## What we built

A static web page with a Leaflet map and a sortable, filterable table of 235 Orange County surplus-land notices. Each site gets a 0-100 screening score from public data:

| Part | Max | Basis |
|---|---|---|
| Zoning | 25 | On a SCAG housing-element site, or SCAG regional zoning category |
| Size | 20 | Acres (2 acres or more gets full marks) |
| Existing use | 15 | Vacant or parking lot scores higher |
| Hazards | 15 | Deduct for FEMA special flood hazard area and CAL FIRE severity |
| Transit | 15 | Inside a Caltrans high-quality transit area |
| Recency | 10 | Newer notices score higher |

Every site is a **candidate**, not an eligible, approved or buildable site. Every site carries the label "Historical notice - verify current status and ownership".

**Demo video:** uploaded with our submission on the event hub (claudesocal.com/uci-impact-lab)
**Live demo:** https://legendofnath.github.io/oc-surplus-screener/

## How Claude is used

- **Building:** Claude pulled the notices from the state open-data portal, queried the public map services for each of the 235 sites, wrote the scoring, and built the data files and page. Three teammates with no coding experience each worked through their own Claude.
- **Inside the tool:** nothing runs live. The checks were run once and saved to `data/scored_sites.json`; the page only reads that file.
- **What a person checks:** the user must confirm zoning with the city, and ownership and current status of any site, before acting. We plan to hand-check the top-ranked sites against the source maps.

## Try it

The page loads a local JSON file, so it needs a local web server.

```bash
git clone https://github.com/legendofnath/oc-surplus-screener.git
cd oc-surplus-screener
python3 -m http.server 8000
```

Then open http://localhost:8000/index.html in a browser. Or skip the install and open the live demo above. You need Python 3 (any recent version) and an internet connection for the map tiles. No keys or environment variables are used.

The page reads `data/scored_sites.json`. It does not call any government server when you open it.

## Sources and assumptions

| Item | Source or assumption | Date checked |
|---|---|---|
| Surplus land notices (235 Orange County rows) | HCD Surplus Land Act notices of availability, https://data.ca.gov/dataset/surplus-land-act-notices-of-availability-received. Self-reported by local agencies; HCD does not verify it | 2026-11-07 |
| Zoning | SCAG Zoning_poly_OR, regional category `ZN24_SCAG`, matched by parcel number, else by point | 2026-11-07 |
| Housing-element sites | SCAG HousingElementSite_poly_scag | 2026-11-07 |
| Flood | FEMA National Flood Hazard Layer, layer 28 | 2026-11-07 |
| Fire | CAL FIRE Fire Hazard Severity Zones (state and local responsibility areas). Older vintage, as published on the state GIS service | 2026-11-07 |
| Transit | Caltrans High Quality Transit Areas (HQTA), https://caltrans-gis.dot.ca.gov/arcgis/rest/services/CHrailroad/CA_HQ_Transit_Areas/FeatureServer/0, point-in-polygon on each site. The service was unreachable from one of our browsers, so the results come from a second run by a teammate and were not independently re-run | 2026-11-07 |
| Notice status | Every Orange County notice in the dataset is "Archive" (the notice window closed). That does not mean the land is gone, only that availability must be verified | 2026-11-07 |
| Zoning weight 25, size 20, existing use 15, hazards 15, transit 15, recency 10 | ASSUMPTION: our own weights, not from any law or program | n/a |
| Zoning points by SCAG category | ASSUMPTION: residential 25, commercial and office 15, public, industrial and open space 5, unknown 0 | n/a |
| Specific Plan zoning scores 15 | ASSUMPTION: a specific plan may or may not allow housing, so we gave it a middle score. 116 sites fall in this group | n/a |
| Size score reaches full marks at 2 acres | ASSUMPTION | n/a |
| Existing-use points (vacant 15, parking 12, gas 8, other 3, and so on) | ASSUMPTION | n/a |
| Recency: 10 points for a 2026 notice, minus 2 per year older | ASSUMPTION | n/a |

Every number we made up is labelled ASSUMPTION here and in the tool.

## What is real and what is mocked

| Part | Status |
|---|---|
| 235 Orange County notices (`data/sites.json`) | Real, from the state dataset |
| Zoning, housing-element, flood, fire and transit checks | Real queries to public map services, run once on 2026-11-07 |
| Scores and weights | Working, but the weights are assumptions |
| Map and table page | Working |
| Live government lookups when the page opens | Not built; results are saved in the data file |
| Current ownership or availability | Not checked. Not available in the data |
| Newport Beach lot dataset (`Newport_Beach_lots_master.xlsx`, 29,355 lots with general plan, zoning, lot size and census-block data; no owner names) | Real public data added by a teammate to widen our scope. Not yet scored or shown in the app |

## Where it gets things wrong

- **Transit is a yes/no check** against Caltrans HQTA polygons. It does not measure walking distance, service frequency or whether service has changed since the layer was published.
- **The fire layer is an older vintage**, so recent hazard changes may be missing.
- **Historical notices.** All notices are Archive status. A site may since have been sold or committed.
- **Self-reported data.** Local agencies filled in the notices, and HCD does not verify them.
- **Missing fields.** 4 sites have no acreage (they score 0 on size), 2 have no city, about 11 have no zoning note from the city, and 7 did not match a SCAG zoning record (they score 0 on zoning). These are flagged in each site's `data_gaps`.
- **Housing-element match is narrow.** Only 1 of the 235 sites matched a housing-element site, and 2 could not be checked.
- **Zoning is a regional category, not the city's legal zoning.** A site flagged as residential still needs a planning check.
- **Weights are our assumptions**, so ranking is a screening aid, not a legal or financial determination.
- **Duplicate parcels.** Some parcels appear next to each other with the same address or lot, and may be one larger site.

## Next steps

For a nonprofit or city to use this next month: confirm current ownership and status for the top sites, re-run the transit check from a second source, add slope and contamination checks, let users change the weights, and refresh the notice data on a schedule. With one more week we would score the Newport Beach lots with the same checks and test the ranking with a real acquisitions team.

## Data and privacy

This project uses only public or provided data. It contains no personal information about real tenants, buyers or residents. It does not file applications or contact any city or person.

## License

MIT, see the LICENSE file in this repo.
