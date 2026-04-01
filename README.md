# BLOCK

**Blueprint for Local Outreach & Community Knowledge**

A tool for Philadelphia neighborhood outreach, signatures, and community requests. Enter any Philly address to get a printable block map and neighbor checklist — then check off each address as you talk with your neighbors.

BLOCK does not replace the City's official forms. It simply helps you stay organized as you fill them out.

## How the data works

BLOCK pulls from two City of Philadelphia open data sources, both queried in real time through public APIs. Nothing is stored locally — every search hits the live data.

### Property records: OPA Properties Public

**Source:** [Office of Property Assessment (OPA)](https://opendataphilly.org/datasets/opa-property-assessments/)
**API:** Philadelphia CARTO SQL API (`phl.carto.com/api/v2/sql`)
**Table:** `opa_properties_public`

This is the primary dataset. When you search an address, BLOCK:

1. Parses your address into a street name, direction, and block range (e.g., "1342 Spruce St" becomes the 1300-1399 block of Spruce)
2. Queries OPA for every property on that block, pulling the address, house number, parcel number, property category, building type, and geographic coordinates
3. Deduplicates by address (some properties have multiple records for individual condo units — these share the same `location` string and are collapsed into one entry)

OPA provides the property category codes that BLOCK uses to tag each address:

| Tag | Category |
|-----|----------|
| **APT** | Apartments (5+ units) |
| **MX** | Mixed Use |
| **COM** | Commercial / Retail / Store |
| **OFC** | Offices |
| **IND** | Industrial |
| **HTL** | Hotel |
| **SP** | Special Purpose (churches, schools, etc.) |
| **GAR** | Garage (commercial or residential) |
| **VL** | Vacant Land |

Properties with "VACANT" in the category description are flagged as possibly vacant on the map and checklist.

### Apartment unit counts: Lead and Healthy Homes Program (LHHP)

**Source:** [Lead and Healthy Homes Program Certifications](https://opendataphilly.org/datasets/lead-and-healthy-homes-program-certifications/)
**API:** Philadelphia CARTO SQL API (`phl.carto.com/api/v2/sql`)
**Table:** `lhhp_lead_certifications`

OPA treats an entire apartment building as a single property record regardless of how many units it contains. A 50-unit building and a row home each count as one address. This matters because Philadelphia block party permits (and other community requests) typically require signatures from 75% of households on a block.

To get actual unit counts, BLOCK queries the LHHP lead certification dataset, which includes unit counts from L&I rental licenses. For each apartment building on the block, BLOCK looks up the parcel number in `lhhp_lead_certifications` and uses `COALESCE(li_rl_units, lhhp_certified_units)` to get the best available unit count.

- **If a match is found:** The exact unit count is used and displayed, e.g., "3408-16 Rhawn St (27 units)"
- **If no match is found:** BLOCK falls back to the OPA `building_code_description` field, which contains coarse ranges like "APTS 5-50 UNITS MASONRY" or "APTS 100+ UNITS MASONRY." The midpoint of the range is used as an estimate, and a warning is shown that the count is approximate.

The household total and 75% signature threshold reflect these unit counts rather than simply counting addresses.

### What the data does not include

- **Exact unit counts for all buildings.** About 91% of apartment records in the LHHP table have unit counts. The rest fall back to OPA's coarse ranges.
- **Recent construction or demolitions.** Both datasets are updated periodically by the City but may lag behind real-world changes.
- **Cross-street addresses.** Properties that face a cross street but share a parcel on your block may not appear, or may appear under a different address.
- **Occupancy status.** The "possibly vacant" flag is based on OPA's category description, not a real-time occupancy check. Always verify in person.

## How to use

1. Open `index.html` in a browser
2. Type any Philadelphia address
3. If the street has both North/South (or East/West) sections, pick your side
4. View the block map and checklist on screen
5. Click "Print / Save as PDF" for a printable version to take door-to-door

## Credits

A passion project by [Carmen Daley](https://www.linkedin.com/in/carmendaley/).

Built with [Claude Code](https://claude.ai/code), [Leaflet](https://leafletjs.com/) for maps, and the [City of Philadelphia's open data](https://opendataphilly.org/) APIs.
