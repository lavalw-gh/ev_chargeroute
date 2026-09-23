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

