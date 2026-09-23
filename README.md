# ChargeRoute UK interactive prototype

This browser prototype demonstrates route-search controls, reachability calculation, filtering, price ordering, map and result synchronization, empty states, and responsive layout. With user-supplied API keys it loads a MapTiler basemap, geocodes the two UK locations through ORS, and requests a driving route with optional toll avoidance.

Charger locations, tariffs, status and reachability distances remain illustrative fixtures. API keys are held only in page memory and are not stored by the prototype.

## Run locally

Because MapLibre is bundled as a browser module, run the prototype through a local web server rather than double-clicking `index.html`. From this folder, run one of these commands:

```sh
python -m http.server 8000
```

or:

```sh
npx serve .
```

Then open `http://localhost:8000` or the address shown by the selected server.

## Publish with GitHub Pages

1. Create a GitHub repository and upload `index.html`, `README.md`, `SPEC_DECISIONS.md`, and the complete `assets` folder to the repository root.
2. Open the repository's **Settings** page, then select **Pages**.
3. Under **Build and deployment**, select **Deploy from a branch**.
4. Select the `main` branch and `/(root)`, then save.
5. GitHub will show the public site URL when deployment finishes.

GitHub Pages publishes static sites publicly. Do not add API keys or other secrets to this repository or to `index.html`.

## API setup

1. Obtain a MapTiler API key and an openrouteservice/HeiGIT API key.
2. Create a plain-text file containing exactly two lines, with no blank lines:

```text
Maptile: YOUR_MAPTILER_KEY
ORS: YOUR_ORS_KEY
```

3. Open the running prototype and choose **Load API file**. A malformed file displays **API file error** and clears both key fields.
4. Alternatively, enter the two keys manually. The inputs are masked and are not saved.
5. Optionally select **Avoid toll roads**, then choose **Find reachable chargers**.

For this static prototype, API calls run from the browser. Restrict the MapTiler key to the deployed domain where possible. A production implementation must send ORS requests through a backend so the ORS credential is never exposed to browser users.

The selected API file is read locally by the browser. It is not uploaded, retained, or added to the GitHub project. Never commit the API text file to the repository.

## Suggested checks

- Change current SOC from 60% to 20%. No charger should remain reachable with the default reserve.
- Restore SOC to 60%, select `150+ kW`, and run the search. Lower-powered sites should disappear.
- Set maximum additional detour to `0.5 km`. Only sites within that extra travel distance should remain.
- Open **Exclude networks**, select one or more networks, run the search, and confirm their sites disappear.
- Enter valid MapTiler and ORS keys and confirm the schematic changes to a real map with the calculated route.
- Confirm the live map contains only the ORS road route; the old straight schematic line and schematic markers must disappear.
- Pan and zoom the live map using the canvas or the unobstructed controls at the top right, and hover a charger marker for its summary.
- Select a charger marker or result to show its detail card. Select the same charger again to hide the card, select it again to restore it, or use the card's close button.
- Select **Avoid toll roads**, run the route again, and confirm the success message identifies a toll-free route.
- Turn on **Hide chargers with unknown price** and confirm the price-unavailable result disappears.
- Select a result card and a map marker. Both views should show the same selected charger.
- Resize the browser to a phone-width window and confirm there is no horizontal page scrolling.

## Prototype limitations

- A live route requires valid MapTiler and ORS keys plus internet access.
- The route geometry can be live, but charger positions and route-relative charger distances are still fixtures.
- The result summary identifies how many of the eight sample sites pass the current filters and displays the battery-modelled usable range.
- Prices and availability are illustrative, not current public-charger information.
- The energy model is constant-efficiency and does not include weather, elevation, traffic, speed, battery temperature, or charging curves.
- The prototype is static and has no backend, database, ingestion worker, or administrative endpoints.

## Planned implementation path

1. Build the production application foundation, database schema, provider adapters, configuration, and automated tests.
2. Import live OCPI location, connector, tariff, and status data.
3. Move ORS behind the application backend and turn the current live routing/map integration into a production service.
4. Implement route-corridor search, chainage, reachability, exclusions, and final road-distance detour checks.
5. Add optional GPX preferred-route import in a later version, after the live routing and spatial-search path is stable.
6. Complete the energy model, price ranking, caching, monitoring, accessibility, security, and private-trial deployment.
