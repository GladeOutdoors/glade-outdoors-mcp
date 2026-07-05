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

## Tools

### `search_locations`

Search for outdoor locations near a UK postcode or coordinates.

**Cost:** 1 credit per call

| Parameter | Type | Required | Description |
|---|---|---|---|
| `postcode` | string | one of postcode or lat+lng | UK postcode e.g. `"WA14 4SA"` |
| `lat` | number | one of postcode or lat+lng | Decimal latitude |
| `lng` | number | one of postcode or lat+lng | Decimal longitude |
| `activity` | string | no | Activity slug — see [activity slugs](#activity-slugs) |
| `radius_km` | number | no | Search radius in km. Default `20`, max `50` |
| `limit` | integer | no | Number of results. Default `10`, max `25` |

**Example prompt:** *"Find places for a dog walk near WA14 4SA"*

**Example result (one location):**
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
| Free | 10 credits | 100 credits |
| Standard | 60 credits | 5,000 credits |

When a limit is exceeded the server returns HTTP `429` with a `Retry-After` header (seconds until the window resets).

Credit costs: `search_locations` = 1 credit, `get_location_detail` = 3 credits.

---

## Activity slugs

Pass these as the `activity` parameter to filter results:

| Slug | Activity |
|---|---|
| `walking` | Walking |
| `dog-walks` | Dog walks |
| `cycling` | Cycling |
| `mountain-biking` | Mountain biking |
| `running` | Running / trail running |
| `bird-watching` | Bird watching |
| `wild-swimming` | Wild swimming |
| `fishing` | Fishing |
| `horse-riding` | Horse riding |
| `rock-climbing` | Rock climbing |
| `orienteering` | Orienteering |
| `nature-study` | Nature study |
| `foraging` | Foraging |
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

Query parameters match `search_locations` tool arguments.

```bash
curl "https://getglade.co.uk/api/v1/locations/search?postcode=WA14+4SA&activity=dog-walks&limit=5" \
  -H "Authorization: Bearer glade_YOUR_KEY"
```

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
