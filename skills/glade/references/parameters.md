# Glade search parameters

Accepted values for `search_locations` filters. Send slugs and values exactly as written.

## `activity` (one slug)

| Slug | Meaning |
|---|---|
| `forest-walks` | Forest / woodland walks |
| `dog-walks` | Dog walks |
| `family-friendly` | Family friendly |
| `hiking` | Hiking |
| `bird-watching` | Bird watching |
| `wildlife` | Wildlife watching |
| `heritage` | Heritage / historic interest |
| `cycling` | Cycling |
| `running` | Running / trail running |
| `wild-swimming` | Wild swimming |
| `foraging` | Foraging |
| `mushroom-foraging` | Mushroom foraging |

## `location_types` (array, any of)

| Group | Values |
|---|---|
| Woodland | `broadleaved`, `conifer`, `mixed`, `coppice`, `plantation`, `low_density`, `assumed_woodland` |
| Parks & reserves | `nature_reserve`, `country_park`, `urban_park` |
| Water | `open_water`, `river`, `reservoir` |
| Historic | `historic_site` |
| Community & recreation | `religious_grounds`, `cemetery`, `allotments`, `playing_field`, `golf_course`, `bowling_green`, `visitor_farm` |
| Land cover | `grassland`, `other_vegetation`, `urban`, `quarry` |

For "any woodland" send `["broadleaved", "conifer", "mixed", "assumed_woodland"]`, or use
`activity: "forest-walks"`, which is usually better.

## `facilities` (array, all required)

`car_park`, `toilets`, `cafe`, `playground`, `picnic_area`, `visitor_centre`, `campsite`

Every facility listed must be present, so each one you add narrows the results sharply.

## Limits

| Parameter | Default | Max |
|---|---|---|
| `radius_km` | 20 | 50 (larger values are clamped) |
| `limit` | 10 | 25 (larger values are clamped) |
