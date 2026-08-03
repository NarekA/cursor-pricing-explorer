# Cursor Model Pricing Explorer

Interactive pricing dashboard for Cursor API models — simulators, comparison deck, and advisor presets.

**Live site:** https://nareka.github.io/cursor-pricing-explorer/

Pricing data is sourced from [Cursor Models & Pricing](https://cursor.com/docs/models-and-pricing).

## Run locally

The site is a single static HTML file:

```sh
python3 -m http.server 8000
```

Open http://localhost:8000.

## Updating pricing

1. Compare the model table against the official [Cursor pricing reference](https://cursor.com/docs/models-and-pricing) and the linked model pages.
2. Update `baseModelDataset` in `index.html`.
   - Keep one row per standard (non-fast) model.
   - Store prices in dollars per million tokens.
   - Use `0` when Cursor publishes no cache-write or cache-read rate; the simulator falls back to normal input pricing when no cache-read rate exists.
   - Treat `bench` and `score` as editorial estimates, not vendor benchmarks.
3. Search `runPickerAdvisor()` for renamed or removed models. Every recommendation must match a dataset `name`.
4. Update the visible sync date and any time-limited pricing note.
5. Validate the page:

```sh
python3 -m http.server 8000
```

Check the simulator, every provider filter, search, sorting, advisor presets, and the three-model comparison deck.

6. Commit and push `main`. GitHub Pages publishes automatically from the branch.
