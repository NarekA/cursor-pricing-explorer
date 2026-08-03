# Cursor Model Pricing Explorer

Interactive pricing dashboard for Cursor API models — simulators, comparison deck, and advisor presets.

**Live site:** https://nareka.github.io/cursor-pricing-explorer/

Pricing data is sourced from [Cursor Models & Pricing](https://cursor.com/docs/models-and-pricing) and linked model pages.

## Modes in the table

Each billing mode is its **own row** so you can compare them side by side:

| Mode | What it means |
| --- | --- |
| **Standard** | Default / non-fast API rates |
| **Fast** | Published Fast-tier rates (often ~2×; Opus Fast can be higher) |
| **Auto Cost** | Fixed Auto rates ($1.25 / $6 / $0.25 cache read per 1M) |
| **Auto Balance** | Bills at the routed model’s API rate — simulator uses **Claude Sonnet 5** as a typical example |
| **Auto Intelligence** | Bills at the routed model’s API rate — simulator uses **Claude Opus 5** as a typical example |

Use the **Filter by mode** chips (All / Standard / Fast / Auto) to narrow the matrix.

The dashboard’s **High simulation (1.5×)** toggle is what-if math only. It is not the same as Cursor Fast mode.

## Run locally

The site is a single static HTML file:

```sh
python3 -m http.server 8000
```

Open http://localhost:8000.

## Updating pricing

1. Compare the model table against the official [Cursor pricing reference](https://cursor.com/docs/models-and-pricing) and linked model pages (including Fast tiers for Composer, Grok, Opus, GPT-5.x).
2. Update `baseModelDataset` in `index.html`.
   - One row per **mode** of a model (`mode: "standard" | "fast" | "auto"`).
   - Store prices in dollars per million tokens.
   - Use `0` when Cursor publishes no cache-write or cache-read rate; the simulator falls back to normal input pricing when no cache-read rate exists.
   - For Auto Balance / Intelligence, keep `variable: true` and `assumes: "<typical model>"` when the row is an example of routed pricing.
   - Treat `bench` and `score` as editorial estimates, not vendor benchmarks.
3. Search `runPickerAdvisor()` for renamed or removed models. Every recommendation must match a dataset `name`.
4. Update the visible sync date and any time-limited pricing note (e.g. Sonnet 5 promo).
5. Validate the page:

```sh
python3 -m http.server 8000
```

Check the simulator, mode filters, provider filters, search, sorting, advisor presets, and the three-model comparison deck.

6. Commit and push `main`. GitHub Pages publishes automatically from the branch.
