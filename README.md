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
- **Forecast mode from the previous election**
  - The button **"ใช้ผลการเลือกตั้งครั้งก่อน (2026)"** (in the header and in the forecast card) loads the embedded previous-election constituency results as the baseline. Every district is filled in vote mode, so the map shows real winners and margins.
  - **Swing sliders:** one slider per party, from -15 to +15 percentage points, applied nationally or to a single region (regional values add on top of the national value). The party's vote share changes in every district where it ran a candidate, the other parties are rescaled so each district still totals 100%, and winners are recalculated immediately.
  - A comparison table shows each party's district seats in the previous election, the forecast, and the change, plus total seats including party-list.
  - Party-list seats are estimated from the summed district votes (the previous-election data has no party-list votes). This can be switched off to enter party-list votes yourself.
  - Moving a slider overwrites any districts you edited by hand. "รีเซ็ตการปรับ" clears all swings and "ออกจากโหมดพยากรณ์" leaves forecast mode while keeping the current map.
- **Random fill** to try the tool quickly.

## Quick start

No build step or dependencies to install. Open `index.html` in a modern browser.

An internet connection is needed for two external resources:

- Google Fonts (Sarabun)
- [lz-string](https://github.com/pieroxy/lz-string) from jsDelivr (used for share links)

If lz-string fails to load, the app still works, but the share-link button will not.

## Previous-election data

The previous election's constituency results are embedded in `index.html` as the `PRIOR` constant, built from `constituency.csv` (one row per candidate: province code, province, district, candidate number, name, party, votes).

- It covers 397 of the 400 districts. Nan 1, Bangkok 15 and Prachinburi 2 are not in the source file and stay undecided in forecast mode.
- Parties with meaningful votes or seats are kept as separate parties; the rest are grouped as "อื่น ๆ".
- Obvious typos in party names were corrected (for example `กลาธรรม` to `กล้าธรรม`), and the English province name "Sakon Nakhon" was mapped to `สกลนคร`.
- Parties in the data that were not in the default list (ไทรวมพลัง, เศรษฐกิจ, โอกาสใหม่, ไทยสร้างไทย) were added to `DEFAULT_PARTIES`. ชาติไทยพัฒนา had no candidates in the data and shows 0.
- Data keys are `"province|district"`, with party order stored in `PRIOR.p`. To use a different election, regenerate `PRIOR` in the same shape.

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

Columns are province, district number, party name (or party id), and votes. (This is a different layout from the source `constituency.csv` used for the previous-election data.) Unknown party names are added automatically with a random color. Rows that do not match a known province and district are skipped, and the import reports how many.

## JSON scenarios

"Save file" writes the full state (parties, district results, party-list votes, coalition choices, view settings). "Open file" only accepts files whose district array matches the current number of districts. If you change the district layout, older scenario files will be rejected. Files and share links from before the forecast update (state version 1) are also rejected, because the district counts changed.

## Customizing

Everything is in `index.html`:

- `CONFIG`: seat counts, majority, PM nomination threshold, stable-majority threshold.
- `REGIONS`: districts per province, grouped by region. The total must equal `CONFIG.DISTRICT_SEATS`; otherwise a warning banner appears. Counts follow the previous-election file: Chiang Mai 10, Pathum Thani 8, Phang Nga 2, Nakhon Si Thammarat 9.
- `PROV_POS`: approximate latitude and longitude per province, used to lay out the hex map.
- `GEO`: layout tuning for the hex map (`k` for geographic spread, `pull` for how strongly provinces stay near their real position, `iter` for relaxation passes).
- `DEFAULT_PARTIES`: starting parties, ids, and colors.
- `PRIOR`: embedded previous-election results used by forecast mode.
- `LEVELS` and the margin thresholds in `result()`: how vote margins map to shades.

## How it works (briefly)

- **Hex map:** provinces start at their geographic positions, are pushed apart so they do not overlap (similar to a Dorling cartogram), and then each province grows cell by cell around its center until it has one hexagon per district.
- **Coalition search:** brute-forces subsets of up to the 16 largest parties and keeps minimal winning sets, up to the party-count limit you set. At most 40 results are shown.
- **Forecast:** for each district, previous vote shares are shifted by the party's national plus regional swing. Parties with a swing get exactly the adjusted share (floored at 0) and the unswung parties share the remainder in proportion to their previous shares. Votes are then rounded and fed through the normal vote-mode winner logic.
- **State:** held in a single object (version 2), saved to `localStorage` under the key `th-election-sim-v2`.

## Notes and limitations

- The hex map is a schematic with one equal-size cell per district. Positions are approximate and are not real constituency boundaries.
- The district counts per province are defaults. Verify them against the latest announcement from Thailand's Election Commission (กกต.) before relying on them.
- Forecasts are simple swing models on last election's district results. They ignore candidate effects, turnout changes, and parties that did not run in a district.
- Party-list allocation is a simplified largest remainder calculation on the votes you enter. It does not model every detail of the legal formula.
