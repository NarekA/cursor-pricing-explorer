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

Two ways to see them in the dashboard:

- The **Mode** column in the rate matrix badges every row as `Std`, `Fast`, or `Auto` (Auto rows also show which model they bill as).
- The **Mode** filter chips sit directly above the table — `All modes` / `Standard` / `Fast` / `Auto` — next to the provider chips. The row counter on the right shows how many rows the current filters and search leave visible.

The dashboard’s **High simulation (1.5×)** toggle is what-if math only. It is not the same as Cursor Fast mode.

## Score columns

| Column | Source |
| --- | --- |
| **Coding Bench** | Editorial estimate of coding capability (0–100). Not a vendor benchmark. |
| **Speed** | Editorial estimate of relative responsiveness (0–100). Cursor publishes no speed metric, so read it as ordering, not tokens/sec. A `Fast` row always ranks above its standard twin. |
| **Overall** | Derived, not stored: `bench × 0.65 + (1 − min(input/1.25, 15) / 15) × 100 × 0.35`. Uses base input rates, so High simulation does not move it. Speed is excluded on purpose. |

Only the per-million-token rates come from Cursor docs. `bench` and `speed` are hand-maintained judgement calls; `overallScore()` computes Overall at render time so it cannot drift from the formula the footer documents.

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
   - Treat `bench` and `speed` as editorial estimates, not vendor benchmarks. Do not add a `score` field — Overall is computed by `overallScore()`.
   - Keep each Fast row's `speed` above its standard twin, and its `bench` equal to it (same model, faster serving).
3. Search `runPickerAdvisor()` for renamed or removed models. Every recommendation must match a dataset `name`.
4. Update the visible sync date and any time-limited pricing note (e.g. Sonnet 5 promo).
5. Validate the page:

```sh
python3 -m http.server 8000
```

Check the simulator, mode filters, provider filters, search, sorting, advisor presets, and the three-model comparison deck.

If you add or reorder table columns, update `dataKeys` in `sortTable()`, the `sortTable(n)` indices in `<thead>`, the string-vs-numeric cutoff (`colIndex > 1`), and the empty-state `colspan`. These must stay equal: `<th>` count, `<td>` count in the row template, and the `colspan`.

Footer explainer cards are capped at two columns (`lg:grid-cols-2`) because the display equations need roughly 350px; a four-column layout clipped them.

6. Commit and push `main`. GitHub Pages publishes automatically from the branch.
