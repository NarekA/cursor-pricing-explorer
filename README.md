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
| **Auto Cost** | Fixed Auto rates ($1.25 / $6 / $0.25 cache read per 1M), whatever the router picks |
| **Auto Balance** | Routed rate, no fixed model. Shown as **2× Auto Cost**. Teams/Enterprise only |
| **Auto Intelligence** | Routed rate, no fixed model. Shown as **4× Auto Cost**. Teams/Enterprise only |

### Why the Auto rows are a multiple, not a model

[Cursor Router](https://cursor.com/docs/cursor-router.md) picks among Composer 2.5, Grok 4.5, GPT-5.5, Claude Opus 5, and Claude Fable 5, hides which model served a request, and changes the pool over time — so no row can honestly quote one model's rates. Earlier versions of this dashboard claimed Auto Balance billed as Claude Sonnet 5 (which is **not in the pool**) and Auto Intelligence as Claude Opus 5 (one of five, and the docs say Intelligence costs *less* than running a single frontier model). Both were wrong.

The only cost figure Cursor publishes is that Balance and Intelligence "cost about twice as much as Cost, and up to two to four times as much depending on the mode." So `routedAutoRates(m)` scales `AUTO_COST_RATES` by `m`: `2` for Balance, `4` for Intelligence.

Two caveats the table does not price in:

- **Cursor Token Rate** — Teams/Enterprise add $0.25/1M tokens on third-party routes, on top of the model's API rate. Auto Cost and first-party Grok 4.5 / Composer 2.5 are exempt. Cursor does not say whether it applies to input, output, or both, so it is disclosed rather than baked into a rate.
- **Auto Cost's own rates** are verified at $1.25 / $1.25 / $0.25 / $6.00, scraped from the rendered docs page (see below), so the derived rows rest on a confirmed base.

Two ways to see modes in the dashboard:

- The **Mode** column in the rate matrix badges every row as `Std`, `Fast`, or `Auto`, plus a `Teams` chip and the routed multiple on Auto Balance / Intelligence.
- The **Mode** filter chips sit directly above the table — `All modes` / `Standard` / `Fast` / `Auto` — next to the provider chips. The row counter on the right shows how many rows the current filters and search leave visible.

The dashboard’s **High simulation (1.5×)** toggle is what-if math only. It is not the same as Cursor Fast mode.

## Score columns

| Column | Source |
| --- | --- |
| **Coding Bench** | Editorial estimate of coding capability (0–100). Not a vendor benchmark. |
| **Speed** | Editorial estimate of relative responsiveness (0–100). Cursor publishes no speed metric, so read it as ordering, not tokens/sec. A `Fast` row always ranks above its standard twin. |
| **Overall** | Derived, not stored: `bench × 0.65 + (1 − min(input/2.5, 15) / 15) × 100 × 0.35`. The `2.5` baseline is `BASELINE_INPUT`, which reads Auto Balance's input rate off `routedAutoRates(2)` rather than hard-coding it. Uses base input rates, so High simulation does not move it. Speed is excluded on purpose. |

Cost Multiplier \(M\) uses the same Auto Balance baseline, so Auto Balance is always `1.0x` (at Standard simulation) and Auto Cost is `0.5x`. Changing the Balance multiple moves `BASELINE_INPUT`, which shifts **every** row's Overall, not just the Auto rows.

Only the per-million-token rates come from Cursor docs. `bench` and `speed` are hand-maintained judgement calls; `overallScore()` computes Overall at render time so it cannot drift from the formula the footer documents.

## Run locally

The site is a single static HTML file:

```sh
python3 -m http.server 8000
```

Open http://localhost:8000.

## Updating pricing

1. Compare the model table against the official [Cursor pricing reference](https://cursor.com/docs/models-and-pricing) and linked model pages (including Fast tiers for Composer, Grok, Opus, GPT-5.x).

   **The `.md` version of the docs is not enough.** Auto Cost, Grok 4.5, and Composer 2.5 rates live in client-rendered React components (`modelId: "auto-cost"`, `"grok-4.5"`, `"composer-2.5"`), so they are missing from both `models-and-pricing.md` and the raw HTML, and there is no JSON endpoint in the page payload. Render the page in a real browser and read the tables out of the DOM, or you will silently miss three rows — that is how the Grok 4.5 cache-read rates ended up as `0`.

   Fast tiers are worse: some are a table row, some are only a note ("2x pricing", "3x lower than Opus 4.7 fast mode"), some only appear on the model's own page (Claude Opus 5 Fast is $10 / $50), and GPT-5.5 Fast has no published rate at all.

2. Update `baseModelDataset` in `index.html`.
   - One row per **mode** of a model (`mode: "standard" | "fast" | "auto"`).
   - Store prices in dollars per million tokens.
   - Use `0` **only** when Cursor publishes no cache-write or cache-read rate. It is a sentinel, not a zero price: the simulator bills cached tokens at the full input rate when `read` is `0`. Putting `0` on a model that does publish a cache-read rate overstates its cost.
   - Set `estimated: "<why>"` on any row whose rate is inferred rather than published. It renders an `Est` chip in the Mode column and a warning in the comparison modal.
   - For Auto Balance / Intelligence, keep `variable: true`, `teamsOnly: true`, and `routedMultiple` / `assumes` in sync, and let `routedAutoRates()` derive the rates. Do not paste a specific model's rates into these rows.
   - If Cursor publishes the routing pool differently, update `ROUTER_POOL`; it feeds both tooltips.
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
