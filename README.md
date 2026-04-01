# BLOCK

**Blueprint for Local Outreach & Canvassing Kit**

**Live tool:** https://phillyblock.netlify.app/

Many City of Philadelphia processes — block party permits, block captain applications, and others — require you to collect signatures from neighbors on your block. The city provides paper forms with blank lines but no address list, no household counts, and no map. You're on your own to figure out which addresses on your block are residential, go door to door, and mail the completed form to a city office.

BLOCK fills that gap. Enter any Philly address to get a printable block map and checklist built from City data. Use it to plan your door-to-door outreach and track your progress as you collect signatures on the City's form.

BLOCK does not replace the City's official forms or requirements — it is a planning and tracking tool to help you stay organized while using them. Always consult the [City of Philadelphia](https://www.phila.gov/) for the official rules and requirements for your specific permit or request.

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
| **MF** | Multi-Family (smaller multi-unit residential) |
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

OPA treats an entire apartment building as a single property record regardless of how many units it contains. A 50-unit building and a row home each count as one address. For Philadelphia block party permits specifically, the City requires signatures from 75% of residential households on a block — so knowing the actual unit counts matters.

To get actual unit counts, BLOCK queries the LHHP lead certification dataset, which includes unit counts from L&I rental licenses. For each property tagged **APT** or **MF** on the block, BLOCK looks up the parcel number in `lhhp_lead_certifications` and uses `COALESCE(li_rl_units, lhhp_certified_units)` to get the best available unit count.

- **If a match is found:** The exact unit count is used and displayed, e.g., "3408-16 Rhawn St (27 units)"
- **If no match is found:** BLOCK falls back to the OPA `building_code_description` field, which contains coarse ranges like "APTS 5-50 UNITS MASONRY" or "APTS 100+ UNITS MASONRY." The midpoint of the range is used as an estimate, and a warning is shown that the count is approximate.

The household total and 75% signature threshold reflect these unit counts rather than simply counting addresses.

### Signature goals and city requirements

BLOCK displays a 75% signature threshold based on residential households. This is specifically the requirement for **Philadelphia block party permits**. Other types of community requests, petitions, or permits may have different signature requirements — including non-residential signatories. BLOCK is a planning tool, not an authority on what any particular process requires.

**Always check with the City of Philadelphia for the official requirements for your specific request.** The household counts and signature goals shown in BLOCK are estimates to help you plan your outreach, not a substitute for the City's official guidelines.

### How different property types count toward household totals

Not every property type counts as a household. Only residential properties contribute to the household total used for the 75% signature goal:

- **Single-family homes** (no tag): counted as 1 household each
- **Duplexes, triplexes, and other small multi-family buildings**: if OPA categorizes the property as "MULTI FAMILY," it gets tagged **MF** and BLOCK will look up the actual unit count. However, some smaller multi-unit properties may be categorized as standard residential in OPA's data, in which case they would count as 1 household. This may slightly undercount the true number of households on a block.
- **Larger multi-family (MF) and apartment (APT) buildings**: counted by their actual unit count (from LHHP or the building code fallback)
- **Mixed use (MX)**: counted as 1 household (the residential portion)
- **Non-residential properties** (COM, OFC, IND, HTL, SP, GAR, VL): excluded from the household count entirely

### How multi-unit buildings appear on the checklist

On the printed checklist, single-address properties get one row with checkboxes for Empty / Signed / Declined. Multi-unit buildings (APT and MF) get a different layout: a header row showing the address, tag, and unit count, followed by a grid of individual unit cells (3 per row) so you can track each unit separately. If a building spans a page break, a "(cont.)" header carries over to the next page.

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
