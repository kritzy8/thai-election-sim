# Thai Parliamentary Election Simulator

A single-file, browser-based simulator for Thailand's House of Representatives elections (400 constituency seats + 100 party-list seats). Paint a map, enter vote counts, and see how the seats and possible coalition governments would shake out.

The interface is in Thai. This tool is for scenario exploration only. It is not a poll or an official forecast.

## Features

- **Two ways to fill in results**
  - **Rating mode:** pick a party and a strength (safe / likely / lean), then click or drag across districts to paint them. A "toss-up" brush and an eraser are included.
  - **Vote mode:** click a district and enter votes per party. The winner and shade are derived from the margin (safe at 15% or more, likely at 7% or more, lean below that; an exact tie is shown as a tie).
- **Two map views**
  - **Country hex map:** one hexagon per district, all the same size, arranged roughly by geography. Includes zoom, optional province labels, and region and province borders.
  - **Province grid:** districts grouped by region and province. Click a province name to paint the whole province.
- **Seat summary:** hemicycle chart, seat bar with the majority line, a per-party table (district / party-list / total), and per-region bars.
- **Party-list seats:** enter nationwide votes (or percentages) per party. The 100 seats are allocated with the largest remainder method. Helpers can fill totals from district votes or approximate them from district seat shares.
- **Coalition builder**
  - Tick parties to see whether the bloc reaches a majority, how many seats short or over it is, and whether any party in it can nominate a PM candidate.
  - **Auto-find coalitions:** searches for minimal winning combinations in which every member is necessary (if any one leaves, the majority is lost). You can require a party, exclude parties, and cap the number of parties.
- **Customizable parties:** rename, recolor, add, or delete parties.
- **Save and share**
  - Autosaves to the browser's `localStorage`.
  - **Share link:** the full scenario is compressed into the URL hash.
  - Export or import a scenario as JSON.
  - Export or import district votes as CSV.
- **Random fill** to try the tool quickly.

## Quick start

No build step or dependencies to install. Open `index.html` in a modern browser.

An internet connection is needed for two external resources:

- Google Fonts (Sarabun)
- [lz-string](https://github.com/pieroxy/lz-string) from jsDelivr (used for share links)

If lz-string fails to load, the app still works, but the share-link button will not.

## Defaults and rules

| Setting | Value |
| --- | --- |
| Constituency seats | 400 |
| Party-list seats | 100 |
| Total seats | 500 |
| Majority | 251 |
| Minimum seats to nominate a PM candidate | 25 |
| "Comfortable majority" threshold | 300 |

All of these live in the `CONFIG` object near the top of the script.

## CSV format

Import and export use one row per district per party, with this header (Thai column names):

```
จังหวัด,เขต,พรรค,คะแนน
เชียงใหม่,1,ประชาชน,45000
เชียงใหม่,1,เพื่อไทย,38000
```

Columns are province, district number, party name (or party id), and votes. Unknown party names are added automatically with a random color. Rows that do not match a known province and district are skipped, and the import reports how many.

## JSON scenarios

"Save file" writes the full state (parties, district results, party-list votes, coalition choices, view settings). "Open file" only accepts files whose district array matches the current number of districts. If you change the district layout, older scenario files will be rejected.

## Customizing

Everything is in `index.html`:

- `CONFIG`: seat counts, majority, PM nomination threshold, stable-majority threshold.
- `REGIONS`: districts per province, grouped by region. The total must equal `CONFIG.DISTRICT_SEATS`; otherwise a warning banner appears.
- `PROV_POS`: approximate latitude and longitude per province, used to lay out the hex map.
- `GEO`: layout tuning for the hex map (`k` for geographic spread, `pull` for how strongly provinces stay near their real position, `iter` for relaxation passes).
- `DEFAULT_PARTIES`: starting parties, ids, and colors.
- `LEVELS` and the margin thresholds in `result()`: how vote margins map to shades.

## How it works (briefly)

- **Hex map:** provinces start at their geographic positions, are pushed apart so they do not overlap (similar to a Dorling cartogram), and then each province grows cell by cell around its center until it has one hexagon per district.
- **Coalition search:** brute-forces subsets of up to the 16 largest parties and keeps minimal winning sets, up to the party-count limit you set. At most 40 results are shown.
- **State:** held in a single object, saved to `localStorage` under the key `th-election-sim-v1`.

## Notes and limitations

- The hex map is a schematic with one equal-size cell per district. Positions are approximate and are not real constituency boundaries.
- The district counts per province are defaults. Verify them against the latest announcement from Thailand's Election Commission (กกต.) before relying on them.
- Party-list allocation is a simplified largest remainder calculation on the votes you enter. It does not model every detail of the legal formula.
