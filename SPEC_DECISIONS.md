# Confirmed MVP decisions

## Unavailable prices

Use `null` for a tariff price that is unavailable or cannot be represented as a simple energy price. Do not use `0` as an unavailable placeholder because zero may represent a genuinely free tariff.

Example:

```json
{
  "kind": "UNKNOWN",
  "pricePencePerKWh": null,
  "currency": "GBP",
  "summary": "Price unavailable"
}
```

Complex tariffs keep their raw components and a human-readable summary. Their simple energy price should be `null` unless a valid, unambiguous pence-per-kWh value can be calculated.

## Additional detour distance

Use `additionalDetourKm` for the additional road distance travelled from the selected route to the charging point. The same definition and field name should be used by filters, API responses, result cards, tests, and analytics.

The MVP filter is therefore:

```text
candidate.additionalDetourKm <= filters.maxAdditionalDetourKm
```

Reachability should include the road distance required to reach the charger entrance. Straight-line distance may be used only for the first spatial pre-filter, never for the final safety decision.

## Administrative synchronization endpoints

`/api/admin/sync/*` can start charger, tariff, or status imports. These jobs consume upstream quota and write large amounts of data, so the endpoints must not be public.

Recommended MVP controls:

- Require a server-held bearer secret or authenticated administrator role.
- Reject missing or invalid credentials with `401` or `403`.
- Permit only HTTPS in hosted environments.
- Use a job lock so the same import cannot run concurrently.
- Make imports idempotent so safe retries do not duplicate records.
- Record who or what started the job, its timestamps, outcome, and counts without logging API credentials.

## Public endpoint throttling

Rate-limit `/api/geocode` and `/api/route-search` because they can consume routing-provider quota. Initial limits should be configuration, not permanent constants. A reasonable trial starting point is approximately 30 geocode requests and 10 route searches per minute per client, plus an application-wide provider budget. Return HTTP `429` with `Retry-After` when a limit is reached.

Cache repeated geocodes and routes, debounce autocomplete, and never expose provider API keys to browser code.

## Live provider integration checklist

Immediately before connecting live services, confirm from each provider's current documentation:

- API hostname and version
- Authentication method and credential provisioning
- Daily and per-minute quotas
- Route, matrix, and geocoding request-size limits
- Pagination and incremental-sync behaviour
- Caching and data-retention terms
- Attribution and licence requirements
- Status refresh restrictions and error responses

Keep these values configurable. An upstream failure should retain the last successful charger snapshot and show its freshness rather than producing misleading live results.

## Network exclusion control

Present network exclusions as a compact dropdown containing a checkbox list. The control must support multiple selections, show a concise selection summary when closed, and provide **Select all** and **Clear** actions. Newly discovered networks can be added to the list without expanding the main filter layout.

## Map and additional routing features

The production version should replace the schematic fixture with a real interactive map. Render the selected route as GeoJSON, display charger markers from live data, keep marker and result-card selection synchronized, fit the viewport to the route, and show the required map-data attribution. Map tiles and routing are separate provider concerns and must be configurable.

Add **Avoid tolls** with the first live routing integration. It is a route-generation preference, so changing it must request a new route and rerun charger reachability rather than filtering the existing results.

Treat GPX import as a later-version enhancement after live routing, maps, and route-corridor search. It is explicitly outside this initial implementation. Its first mode should accept a valid driving track as preferred/locked route geometry, validate and simplify it, and calculate charger reachability along it. A later mode may convert the track to via points or use map matching. Uploaded route files should not be persisted by default.

## Initial live-routing prototype scope

- Accept masked MapTiler and ORS API-key inputs immediately above the search action.
- Allow a local API text file with exactly `Maptile: <key>` on line one and `ORS: <key>` on line two. Reject every other layout with **API file error**, clear any existing key values, and never upload or persist the file.
- Keep both keys in page memory only; do not use local storage, cookies, query strings, or checked-in configuration.
- Use the MapTiler `streets-v4` style with the bundled MapLibre GL JS renderer.
- Geocode UK start and destination text and request a `driving-car` route from the HeiGIT-hosted ORS API.
- When **Avoid toll roads** is selected, send `options.avoid_features: ["tollways"]` and recalculate the route.
- Keep charger, tariff, availability, chainage, and detour information as clearly labelled fixture data until live OCPI ingestion and spatial matching are implemented.
- In production, proxy ORS through the backend and apply provider throttling; do not ship a privileged ORS key to browsers.

## Map interaction and range wording

When the live map activates, the schematic SVG must be hidden and made non-interactive so its straight route, endpoints, and fixture marker elements cannot appear above or intercept MapLibre. Route and journey metrics occupy a dedicated strip above the map rather than overlaying the canvas. The large selected-charger card and planning disclaimer are hidden when live-map mode first opens. Selecting a charger marker or result opens the detail card; selecting the same charger again hides it, and selecting another charger switches and opens the card. A close control also dismisses it. MapLibre's navigation controls remain unobstructed at the top right. Hovering a sample charger marker shows its name, network, and tariff; clicking it synchronizes the marker with the selected result card.

Label the fixture calculation as, for example, **4 of 8 sample sites qualify · 126 mi estimated usable range**. Here, `4 of 8` means four fixture chargers pass the current power, detour, network, availability, price, and arrival-reserve filters. The range is calculated from battery capacity, current SOC, reserve SOC, and configured efficiency; it is not the route distance.

The maximum additional-detour options for this version are 1 km, 2 km, 5 km, and 10 km.
