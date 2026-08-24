# Data Fix Notes — 2026-08-24

The app's `RemoteWinery` decoder is a strict mirror of `wineries.json`.
Any `null` in a non-optional field makes **the entire payload fail to decode**,
silently falling back to the ~61 bundled seed wineries. That had been happening
since 116 records were added without coordinates.

## Fixed in this commit

| Issue | Records | Action |
|---|---|---|
| `latitude`/`longitude` null | 116 | backfilled (see precision below) |
| `priceTier: "$$$$"` | 32 | → `"$$$"` (no `$$$$` case in `PriceTier`) |
| `dogPolicy: "outsideOnly"` | 118 | → `"designatedAreaOnly"` |
| `familyPolicy: "allAges"` | 1 | → `"familyFriendly"` |
| Mermaid Winery empty `address` | 1 | restored VB location; Norfolk noted |

## Follow-ups

**1. Restore the luxury tier.** These 32 were collapsed `$$$$` → `$$$`. Add `case luxury = "$$$$"` to `PriceTier` in `Winery.swift`, then revert them:

- Alexander Valley Vineyards
- Chateau Montelena
- Cline Family Cellars
- Corison Winery
- Cristom Vineyards
- Domaine Serene
- Doubleback Winery
- Duckhorn Vineyards
- Far Niente Winery
- Ferrari-Carano Vineyards
- Flowers Vineyards & Winery
- Freemark Abbey Winery
- Gary Farrell Vineyards & Winery
- Hawkes Wine
- Imagery Estate Winery
- Jordan Vineyard & Winery
- Leonetti Cellar
- MacRostie Winery Estate House
- Medlock Ames
- Opus One
- Pine Ridge Vineyards
- Ponzi Vineyards
- Quivira Vineyards & Winery
- RdV Vineyards
- Ridge Vineyards - Lytton Springs
- Rogers Ford Farm Winery
- Saintsbury
- Smith-Madrone Vineyards
- Spottswoode Estate Vineyard & Winery
- Trione Vineyards & Winery
- Valley Road Vineyards
- Vint Hill Craft Winery

**2. Approximate coordinates.** These 39 resolved only to a town centroid, not the street address — map pins are off by up to a few miles:

- 12 Ridges Vineyard
- Alexander Valley Vineyards
- Arterra Wines
- Bella Vineyards and Wine Caves
- Bridlewood Estate Winery
- Cardinal Point Vineyard & Winery
- Chateau Montelena
- Cline Family Cellars
- Creek's Edge Winery
- Domaine Fortier Vineyards
- Duckhorn Vineyards
- Eastwood Farm and Winery
- Frank Family Vineyards
- General's Ridge Vineyard
- Good Luck Cellars
- Halter Ranch
- Hark Vineyards
- Hawkes Wine
- Hazy Mountain Vineyards & Brewery
- Hill Top Berry Farm, Winery & Meadery
- Iron Heart Winery
- JUSTIN Vineyards & Winery
- Mediterranean Cellars
- Monroe Bay Winery
- Moss Vineyards
- Mountain & Vine Vineyards
- Pollak Vineyards
- Quivira Vineyards & Winery
- Saude Creek Vineyards
- Shenandoah Vineyards
- Silver Oak Alexander Valley
- Tamber Bey Vineyards
- The Barns at Hamilton Station Vineyards
- The Winery at Kindred Pointe
- Tranquility Farm & Winery
- Tres Sabores Winery
- Veramar Vineyard
- West Wind Farm Vineyard & Winery
- Wolf Gap Vineyard & Winery

**3. Harden the decoder** so this cannot recur:
- Make `latitude`/`longitude` optional in `RemoteWinery` and drop the record in `toWinery()` instead of throwing for the whole file.
- Decode `[RemoteWinery]` losslessly (per-element try/catch) so one bad record never blanks the app.
- Surface `dataSource` (`.bundled` vs `.remote`) in Settings → Data so a silent fallback is visible instead of invisible.

**4. Unknown attributes** `historic` and `sustainable` appear in the data but have no `VibeAttribute` case — they are silently ignored (non-fatal).
