# Glade REST API

The same data as the [Glade MCP server](../README.md), over plain HTTPS. Parameters, credit costs, rate limits and value lists (activity slugs, location types, facilities) are shared with the MCP tools. See the [README](../README.md) for those.

Authenticate with an `Authorization: Bearer glade_YOUR_KEY` header ([request a key](https://getglade.co.uk/developers)), or pay per call with [x402](x402.md) and no key.

## Search

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

## Detail

```
GET https://getglade.co.uk/api/v1/locations/:id
```

```bash
curl "https://getglade.co.uk/api/v1/locations/e18cd089-a994-4d96-b5e8-989bebe599af" \
  -H "Authorization: Bearer glade_YOUR_KEY"
```

---

## Errors

| HTTP status | Meaning |
|---|---|
| `400` | Invalid parameters (bad postcode format, lat/lng out of UK bounds) |
| `401` | Missing or invalid API key |
| `402` | Payment required — no API key sent; see [x402](x402.md) |
| `404` | Location not found |
| `429` | Rate limit exceeded — check the `Retry-After` header |
| `500` | Internal error |
