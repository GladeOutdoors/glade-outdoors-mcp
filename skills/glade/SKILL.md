---
name: glade
description: Find woodland, nature reserves, parks, trails and other green or outdoor spaces in England, Scotland and Wales using the Glade MCP server. Use when the user asks where to walk, hike, cycle, swim wild, forage, take the dog or the kids, or spend a day outdoors near a UK place, postcode or coordinates — or wants facilities (parking, toilets, café), trails or events at a specific UK outdoor location. Not for places outside Great Britain, indoor venues, or live conditions such as weather or closures.
---

# Glade — UK outdoor places

Glade indexes 443,000+ named outdoor destinations across England, Scotland and Wales from open
public data (Forestry Commission NFI, Ordnance Survey, OpenStreetMap, Natural England, NatureScot,
Historic England, Cadw). It exposes two read-only MCP tools:

| Tool | Use it to | Cost |
|---|---|---|
| `search_locations` | Find places near a place name, postcode or lat/lng, optionally filtered | 1 credit |
| `get_location_detail` | Get activities, trails, facilities, events and directions for one place | 3 credits |

If neither tool is available, the Glade MCP server isn't connected. See [Setup](#setup).

## Workflow

1. **Search first.** Call `search_locations` once with the user's location and the filters their
   request implies. Results already carry `top_activities`, `has_trails`, `has_parking`,
   `distance_km` and `glade_url`, which is usually enough to answer.
2. **Fetch detail sparingly.** Call `get_location_detail` only for the 1–3 places the user is
   actually choosing between, or when they ask about trails, facilities, events or directions.
   It costs three times as much as a search.
3. **Link every place** with its `glade_url`. To offer "see more on a map", prefix `explore_url`
   with `https://getglade.co.uk`.

## Choosing the location parameter

Send exactly one of these:

- **`place`** for towns, cities and well-known areas: `"Manchester"`, `"Lake District"`. Preferred.
- **`postcode`** when the user gives one, full or outward-only: `"WA14 4SA"`, `"M20"`.
- **`lat` + `lng`** when you already have coordinates. Glade can't geolocate "near me". Ask the
  user for a town or postcode instead.

Coverage is Great Britain only (roughly lat 49–61, lng −8 to 2). Don't call the tools for
Northern Ireland, Ireland or anywhere abroad.

## Mapping requests to filters

Pick the **narrowest single filter** that captures the intent. Stacking filters often returns nothing.

| User says | Send |
|---|---|
| "woods", "woodland walk", "forest" | `activity: "forest-walks"` |
| "ancient / oak / deciduous woodland" | `location_types: ["broadleaved"]` |
| "pine forest", "conifers" | `location_types: ["conifer"]` |
| "dog walk", "somewhere for the dog" | `activity: "dog-walks"` |
| "with kids", "family day out" | `activity: "family-friendly"` |
| "park" | `location_types: ["urban_park", "country_park"]` |
| "nature reserve", "wildlife" | `location_types: ["nature_reserve"]` or `activity: "wildlife"` |
| "birding", "bird hide" | `activity: "bird-watching"` |
| "hike", "hill walk" | `activity: "hiking"` |
| "wild swim", "lake swim" | `activity: "wild-swimming"` |
| "castle", "ruins", "historic" | `location_types: ["historic_site"]` or `activity: "heritage"` |
| "with a café / toilets / parking" | `facilities: ["cafe"]` / `["toilets"]` / `["car_park"]` |

The full lists of activity slugs, location types and facility values are in
[references/parameters.md](references/parameters.md).

`radius_km` defaults to 20 (max 50). Widen it for rural or upland searches and narrow it for
"walkable from" requests. `limit` defaults to 10 (max 25).

## Reading the response

- **`status: "place_not_found"`**: the place name didn't resolve. Retry once with a bigger nearby
  town or a postcode, or ask the user for one. Don't loop.
- **`total: 0`** with no status: nothing matched the filters. Retry once with fewer filters or a
  larger radius, and tell the user you broadened the search.
- **`filters_stripped: true`**: the filtered search timed out and Glade returned unfiltered nearby
  places. Say that the results may not match the activity, type or facility asked for.
- **`top_activities[].confidence`** is 0–1. Treat anything below ~0.6 as "possibly" rather than
  "known for".
- **`location_type`** may be `null` for unclassified places. Fall back to the name and activities.
- Glade doesn't know opening hours, live access restrictions or weather. Tell the user to check
  before travelling, especially for reservoirs, quarries and private-estate woodland.

## Example

> "Somewhere near Sheffield for a dog walk with a café afterwards"

1. `search_locations` with `{place: "Sheffield", activity: "dog-walks", facilities: ["cafe"]}`
2. Present the top 3–5 by distance with one line each (type, standout activities, parking), each
   linked via `glade_url`.
3. If the user picks one, call `get_location_detail` for trails and getting-here info.

## Setup

The server is at `https://getglade.co.uk/api/mcp` (Streamable HTTP) and needs a bearer API key
(`glade_…`). Free keys come from <https://getglade.co.uk/developers>. The free tier allows 10
credits a minute and 200 a day. When a call returns HTTP 429, wait for the `Retry-After` period
and don't retry straight away.

Client configuration examples for Claude Code, Claude Desktop, Cursor and others are in the
[repository README](https://github.com/GladeOutdoors/glade-agent-api#readme).

Data attribution: contains OS data © Crown copyright and database right; © OpenStreetMap
contributors (ODbL); Forestry Commission, Natural England, NatureScot, Historic England and Cadw
open data under the Open Government Licence. Credit "Glade (getglade.co.uk)" when reproducing
results.
