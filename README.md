# Glade MCP Server

Find woodland, nature reserves, parks, green and outdoor spaces across the UK — built for AI agents.

Glade indexes 443,000+ named destinations across England, Scotland, and Wales — woodlands, parks, nature reserves, historic sites, and more — and exposes them through a [Model Context Protocol](https://modelcontextprotocol.io) server and a REST API. Ask it what's near a postcode, filter by activity, and get back structured location data including trails, facilities, and upcoming events.

**Endpoint:** `https://getglade.co.uk/api/mcp`  
**Transport:** Streamable HTTP (JSON-RPC 2.0)  
**Auth:** Bearer API key (`glade_...`)

---

## Quick start

### 1. Request an API key

Visit [getglade.co.uk/developers](https://getglade.co.uk/developers) and fill in the short form. Free tier keys are issued within 24 hours.

### 2. Add to your client

**Claude Code (plugin — recommended)** — installs the MCP server plus the Glade [skill](#agent-skill), and prompts for your key (stored in your OS keychain):

```
/plugin install glade --marketplace jamesmorgan/glade-agent-api
```

**Claude Code (server only):**

```bash
claude mcp add --transport http glade https://getglade.co.uk/api/mcp \
  --header "Authorization: Bearer glade_YOUR_KEY_HERE"
```

**Claude Desktop** — edit `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "glade": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://getglade.co.uk/api/mcp",
        "--header",
        "Authorization:Bearer glade_YOUR_KEY_HERE"
      ]
    }
  }
}
```

**Cursor** — edit `.cursor/mcp.json` in your project root (or `~/.cursor/mcp.json` globally):

```json
{
  "mcpServers": {
    "glade": {
      "url": "https://getglade.co.uk/api/mcp",
      "headers": {
        "Authorization": "Bearer glade_YOUR_KEY_HERE"
      }
    }
  }
}
```

**Cline / other clients** — use the same URL and header. Any client that supports Streamable HTTP transport works.

---

## Agent skill

[`skills/glade/SKILL.md`](https://github.com/jamesmorgan/glade-agent-api/blob/main/skills/glade/SKILL.md) teaches an agent when to reach for Glade and how to use it well: turning "woods near Leeds with a café" into the right filters, searching before fetching detail to save credits, and recovering when a place isn't found. It follows the open [Agent Skills](https://agentskills.io) format, so it works in Claude Code, Codex, Cursor, Gemini CLI and other compatible agents:

```bash
npx skills add jamesmorgan/glade-agent-api
```

The Claude Code plugin above includes it automatically.

---

## Tools

### `search_locations`

Search for outdoor locations near a UK place name, postcode, or coordinates. Supports filtering by activity, location type, and facilities.

**Cost:** 1 credit per call

| Parameter | Type | Required | Description |
|---|---|---|---|
| `place` | string | one of place, postcode, or lat+lng | UK place name e.g. `"Manchester"`, `"Leeds"`, `"Lake District"`. **Preferred for city/town searches.** |
| `postcode` | string | one of place, postcode, or lat+lng | UK postcode e.g. `"WA14 4SA"`, `"M1 1AA"` |
| `lat` | number | one of place, postcode, or lat+lng | Decimal latitude |
| `lng` | number | one of place, postcode, or lat+lng | Decimal longitude |
| `activity` | string | no | Activity slug — see [activity slugs](#activity-slugs) |
| `location_types` | array | no | Filter by location type — see [location types](#location-types) |
| `facilities` | array | no | Filter by required facilities — see [facilities](#facilities) |
| `radius_km` | number | no | Search radius in km. Default `20`, max `50` |
| `limit` | integer | no | Number of results. Default `10`, max `25` |

**Example prompts:**
- *"Find places for a dog walk near WA14 4SA"*
- *"Woodlands near Manchester"* → `place: "Manchester"`, `activity: "forest-walks"`
- *"Park with cafe in Leeds"* → `place: "Leeds"`, `location_types: ["urban_park", "country_park"]`, `facilities: ["cafe"]`
- *"Nature reserve with toilets near Nottingham"* → `place: "Nottingham"`, `location_types: ["nature_reserve"]`, `facilities: ["toilets"]`

**Response shape:**
```json
{
  "locations": [...],
  "total": 10,
  "resolved_place": "manchester",
  "explore_url": "/explore?lat=53.48&lng=-2.24&activity=forest-walks"
}
```

When no results are found (empty or place not recognized):
```json
{
  "locations": [],
  "total": 0,
  "status": "place_not_found",
  "message": "Could not find \"South Manchester\". Try a major city name, town, or UK postcode.",
  "explore_url": "/explore?lat=53.48&lng=-2.24"
}
```

**Note:** The `explore_url` can be appended to `https://getglade.co.uk` to link users to the Explore map with the same search.

**Example location in results:**
```json
{
  "id": "3036a816-038f-46f1-8396-af7e6eb5d3bb",
  "name": "Wythenshawe Park",
  "location_type": "urban_park",
  "lat": 53.4042,
  "lng": -2.2797,
  "distance_km": 6.8,
  "top_activities": [
    {"slug": "family-friendly", "label": "Family friendly", "confidence": 0.97},
    {"slug": "picnic-day-out",  "label": "Picnic day out",  "confidence": 0.95},
    {"slug": "playgrounds",     "label": "Playgrounds",     "confidence": 0.93}
  ],
  "has_trails": false,
  "has_parking": true,
  "glade_url": "https://getglade.co.uk/locations/wythenshawe-park-park-osm-73a6ca01"
}
```

---

### `get_location_detail`

Fetch full detail for a specific location by ID, including activities, trails, facilities, and upcoming events.

**Cost:** 3 credits per call

| Parameter | Type | Required | Description |
|---|---|---|---|
| `location_id` | string | yes | `id` from a `search_locations` result |

**Example result:**
```json
{
  "id": "e18cd089-a994-4d96-b5e8-989bebe599af",
  "name": "Dunham Massey",
  "location_type": "historic_site",
  "lat": 53.3816,
  "lng": -2.3899,
  "activities": [
    {"slug": "wildlife",       "label": "Wildlife",       "confidence": 0.95, "evidence": []},
    {"slug": "bird-watching",  "label": "Bird watching",  "confidence": 0.91, "evidence": []},
    {"slug": "heritage",       "label": "Heritage",       "confidence": 0.86, "evidence": []},
    {"slug": "foraging",       "label": "Foraging",       "confidence": 0.82, "evidence": []},
    {"slug": "insect-spotting","label": "Insect spotting","confidence": 0.81, "evidence": []}
  ],
  "trails": [],
  "facilities": [],
  "events": [
    {
      "name": "The Brit Fest 2026",
      "date": "2026-07-02T00:00:00+00:00",
      "provider": "skiddle",
      "url": "https://www.skiddle.com/festivals/the-brit-fest/"
    }
  ],
  "address": "Trafford, WA14 4SA",
  "getting_here": {
    "google_maps_url": "https://www.google.com/maps/dir/?api=1&destination=53.3816,-2.3899",
    "has_parking": false
  },
  "glade_url": "https://getglade.co.uk/locations/dunham-massey-rpg-1000853"
}
```

---

## Rate limits

| Tier | Per minute | Per day |
|---|---|---|
| Free | 10 credits | 200 credits |
| Standard | 60 credits | 5,000 credits |

When a limit is exceeded the server returns HTTP `429` with a `Retry-After` header (seconds until the window resets — daily quotas reset at 00:00 UTC).

Credit costs: `search_locations` = 1 credit, `get_location_detail` = 3 credits.

---

## Facilities

Pass these as the `facilities` array parameter to filter results to locations with specific amenities:

| Value | Description |
|---|---|
| `car_park` | Has parking |
| `toilets` | Has toilets |
| `cafe` | Has a café |
| `playground` | Has a playground |
| `picnic_area` | Has picnic facilities |
| `visitor_centre` | Has a visitor centre |
| `campsite` | Has camping facilities |
| `cycle_hire` | Offers cycle hire |
| `walking_trails` | Has marked walking trails |
| `cycling_trails` | Has cycling trails |
| `horse_riding` | Allows horse riding |

**Example:** To find parks with a café and toilets: `facilities: ["cafe", "toilets"]`

---

## Activity slugs

Pass these as the `activity` parameter to filter results:

| Slug | Activity |
|---|---|
| `forest-walks` | Forest walks / woodland walks |
| `dog-walks` | Dog walks |
| `family-friendly` | Family friendly |
| `hiking` | Hiking |
| `bird-watching` | Bird watching |
| `wildlife` | Wildlife watching |
| `heritage` | Heritage / historic sites |
| `cycling` | Cycling |
| `mountain-biking` | Mountain biking |
| `running` | Running / trail running |
| `wild-swimming` | Wild swimming |
| `easy-access` | Easy access / accessible |
| `fishing` | Fishing |
| `horse-riding` | Horse riding |
| `foraging` | Foraging |
| `mushroom-foraging` | Mushroom foraging |
| `photography` | Photography |

---

## Location types

Every location has a `location_type` field describing what kind of place it is. The value may be `null` where the type has not yet been classified.

### Woodland

Sourced from the National Forest Inventory (NFI).

| Value | Label | Notes |
|---|---|---|
| `broadleaved` | Broadleaved woodland | Oak, ash, beech and other deciduous trees |
| `conifer` | Conifer forest | Spruce, pine, larch and similar evergreens |
| `mixed` | Mixed woodland | Combination of broadleaved and conifer |
| `coppice` | Coppiced woodland | Managed rotational cutting woodland |
| `plantation` | Plantation forest | Commercial or managed tree planting |
| `low_density` | Open woodland | Widely spaced trees with open understorey |
| `assumed_woodland` | Woodland | Wooded area; specific type not classified |

### Parks & reserves

| Value | Label | Notes |
|---|---|---|
| `nature_reserve` | Nature reserve | Local or national nature reserve |
| `country_park` | Country park | Managed countryside open to the public |
| `urban_park` | Urban park | City or town park |

### Water

| Value | Label | Notes |
|---|---|---|
| `open_water` | Open water | Lake, loch, pond, or tarn |
| `river` | River | Named river or significant watercourse |
| `reservoir` | Reservoir | Man-made water storage — may or may not have public access |

### Historic

Indexed from Historic England and Cadw records.

| Value | Label | Notes |
|---|---|---|
| `historic_site` | Historic site | Monument, scheduled site, battlefield, or listed landscape |

### Community & recreation

| Value | Label | Notes |
|---|---|---|
| `religious_grounds` | Religious grounds | Churchyard, graveyard, or place of worship grounds |
| `cemetery` | Cemetery | Municipal cemetery or burial ground |
| `allotments` | Allotments | Community growing plots |
| `playing_field` | Playing field | Sports field or recreation ground |
| `golf_course` | Golf course | Golf course (often partially publicly walkable) |
| `bowling_green` | Bowling green | Lawn bowls green |
| `visitor_farm` | Visitor farm | Farm open to the public; may include pick-your-own |

### Land cover

Broad classifications where a more specific type is not available.

| Value | Label | Notes |
|---|---|---|
| `grassland` | Grassland | Open grass — meadow, heath, or downland |
| `other_vegetation` | Greenspace | Vegetated land not matching another category |
| `urban` | Urban greenspace | Green areas within built-up settlements |
| `quarry` | Quarry | Former or active quarry — some have public access |

---

## REST API

The same data is available over a conventional REST API if you prefer that over MCP.

Authenticate with an `Authorization: Bearer glade_YOUR_KEY` header, or pay per-call using [x402 USDC payments](#x402-payments).

### Search

```
GET https://getglade.co.uk/api/v1/locations/search
```

Query parameters match `search_locations` tool arguments:

| Parameter | Type | Description |
|---|---|---|
| `place` | string | UK place name (preferred for city/town searches) |
| `postcode` | string | UK postcode |
| `lat` | number | Decimal latitude |
| `lng` | number | Decimal longitude |
| `activity` | string | Activity slug |
| `location_types` | string | Location type(s) — repeat param or comma-separated |
| `facilities` | string | Required facility(s) — repeat param or comma-separated |
| `radius_km` | number | Search radius in km (default 20, max 50) |
| `limit` | integer | Number of results (default 10, max 25) |

**Examples:**

```bash
# Search by postcode
curl "https://getglade.co.uk/api/v1/locations/search?postcode=WA14+4SA&activity=dog-walks&limit=5" \
  -H "Authorization: Bearer glade_YOUR_KEY"

# Search by place name
curl "https://getglade.co.uk/api/v1/locations/search?place=Manchester&activity=forest-walks" \
  -H "Authorization: Bearer glade_YOUR_KEY"

# Search with location type filter (woodland near Leeds)
curl "https://getglade.co.uk/api/v1/locations/search?place=Leeds&location_types=broadleaved,conifer,mixed" \
  -H "Authorization: Bearer glade_YOUR_KEY"

# Search with facility filter (park with cafe)
curl "https://getglade.co.uk/api/v1/locations/search?place=Wakefield&location_types=urban_park,country_park&facilities=cafe" \
  -H "Authorization: Bearer glade_YOUR_KEY"
```

**Response:**
```json
{
  "locations": [...],
  "total": 10,
  "resolved_place": "manchester",
  "explore_url": "/explore?lat=53.48&lng=-2.24&activity=forest-walks"
}
```

**Note:** The phrase→filter lexicon (e.g. "woodlands" → `forest-walks` activity only) is **chat-only**. API callers should send structured `activity`, `location_types`, and `facilities` parameters directly. "Near me" geolocation is also chat-only — API callers should pass explicit `lat`/`lng` coordinates.

### Detail

```
GET https://getglade.co.uk/api/v1/locations/:id
```

```bash
curl "https://getglade.co.uk/api/v1/locations/e18cd089-a994-4d96-b5e8-989bebe599af" \
  -H "Authorization: Bearer glade_YOUR_KEY"
```

---

## x402 payments

REST endpoints support [x402](https://x402.org) — a standard for per-call USDC payments on Base. No API key or account required. x402-compatible agents receive a `402 Payment Required` response, pay automatically, and retry.

| Endpoint | Cost |
|---|---|
| `GET /api/v1/locations/search` | $0.0010 USDC (1 credit) |
| `GET /api/v1/locations/:id` | $0.0030 USDC (3 credits) |

**Flow:**

```bash
# 1 — request without auth returns 402 with payment instructions
curl -s https://getglade.co.uk/api/v1/locations/search?postcode=WA14+4SA
# → HTTP 402  { "x402Version": 1, "accepts": [...payment requirements...] }

# 2 — x402 client pays and retries with signed X-PAYMENT header
# (handled automatically by x402-compatible agents and SDKs)
```

**TypeScript / Node:**

```typescript
import { wrapFetchWithPayment } from 'x402-fetch';
import { createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';

const account = privateKeyToAccount(process.env.PRIVATE_KEY);
const walletClient = createWalletClient({ account, chain: base, transport: http() });

// Wraps fetch — handles 402 → pay → retry automatically
const fetch402 = wrapFetchWithPayment(fetch, walletClient);

const res = await fetch402(
  'https://getglade.co.uk/api/v1/locations/search?postcode=WA14+4SA'
);
const { locations } = await res.json();
```

```
npm install x402-fetch viem
```

Payments settle on Base mainnet. For testing use `base-sepolia` (configured server-side).

---

## Coverage

Glade covers England, Scotland, and Wales. Location data is sourced from Ordnance Survey Open Data, OpenStreetMap, Natural England, and NatureScot, and is updated continuously.

**Destinations**
- 443,000+ named destinations
- 249,000+ public access places
- 431,000+ habitat records
- 325,000+ woodlands
- 41,000+ historic sites
- 28,000+ parks
- 20,000+ churchyards
- 1,000+ nature reserves

**Activities indexed**
- 328,000+ forest walks
- 249,000+ dog walks
- 124,000+ heritage
- 90,000+ hiking
- 67,000+ cycling
- 63,000+ family friendly
- 15,000+ mountain biking
- 8,000+ horse riding
- 3,683,000+ activity signals total

**Facilities**
- 184,000+ parking
- 50,000+ playgrounds
- 25,000+ cafés
- 20,000+ toilets
- 13,000+ picnic areas
- 7,000+ campsites

---

## Errors

| HTTP status | Meaning |
|---|---|
| `401` | Missing or invalid API key |
| `400` | Invalid parameters (bad postcode format, lat/lng out of UK bounds) |
| `404` | Location not found |
| `429` | Rate limit exceeded — check `Retry-After` header |
| `500` | Internal error |

MCP JSON-RPC errors use standard codes: `-32601` method not found, `-32603` internal error.

---

## Links

- [Glade](https://getglade.co.uk) — the main app
- [Developer page & API key request](https://getglade.co.uk/developers)
