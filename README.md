# ChargeRoute UK interactive prototype

This is a self-contained browser prototype based on the ChargeRoute UK MVP specification. It demonstrates the route-search controls, reachability calculation, filtering, price ordering, map and result synchronization, empty states, and responsive layout.

The prototype uses a frozen Sutton-to-Manchester route and illustrative charger records. It does not call live routing, charger, tariff, status, or map-tile services. It stores no journey information.

## Run locally

The quickest option is to double-click `index.html` and open it in a modern browser.

For a local web server, run one of these commands from this folder:

```sh
python -m http.server 8000
```

or:

```sh
npx serve .
```

Then open `http://localhost:8000` or the address shown by the selected server.

## Publish with GitHub Pages

1. Create a GitHub repository and upload `index.html`, `README.md`, and `SPEC_DECISIONS.md` to the repository root.
2. Open the repository's **Settings** page, then select **Pages**.
3. Under **Build and deployment**, select **Deploy from a branch**.
4. Select the `main` branch and `/(root)`, then save.
5. GitHub will show the public site URL when deployment finishes.

GitHub Pages publishes static sites publicly. Do not add API keys or other secrets to this repository or to `index.html`.

## Suggested checks

- Change current SOC from 60% to 20%. No charger should remain reachable with the default reserve.
- Restore SOC to 60%, select `150+ kW`, and run the search. Lower-powered sites should disappear.
- Set maximum additional detour to `0.5 km`. Only sites within that extra travel distance should remain.
- Exclude a network and confirm its sites disappear.
- Turn on **Hide chargers with unknown price** and confirm the price-unavailable result disappears.
- Select a result card and a map marker. Both views should show the same selected charger.
- Resize the browser to a phone-width window and confirm there is no horizontal page scrolling.

## Prototype limitations

- Start and destination fields update labels but do not geocode or request a new route.
- Route geometry and charger data are fixtures.
- Prices and availability are illustrative, not current public-charger information.
- The energy model is constant-efficiency and does not include weather, elevation, traffic, speed, battery temperature, or charging curves.
- The prototype is static and has no backend, database, ingestion worker, or administrative endpoints.

