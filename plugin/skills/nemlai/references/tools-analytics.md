# Analytics tools

Spending reports over the household's order history. All seven tools are read-only and give the same
numbers as the web dashboard (`/analytics`) and `nemlai analytics …`.

**Which report answers which question**

| Question | Tool |
|---|---|
| "How much did we spend this month / last quarter?" (start here) | `analytics_overview` |
| "How much on milk / on Mejeri / on strawberries?" | `analytics_spend` |
| "What do we spend most on?" / "Which products do we buy most?" | `analytics_top` |
| "Is our spending on X going up or down?" | `analytics_trend` |
| "When did we last buy X, and at what price?" | `analytics_drill` |
| "Give me the receipts / orders for X" (expense documentation) | `analytics_receipts` |
| "What categories exist, and what are their exact names?" | `analytics_categories_tree` |

**Rules that hold for every tool**

- Amounts are **DKK (kr)** and are what was **paid, after campaign discounts**. The saving is reported
  separately and is always positive: `total_discount` (spend, top, overview) or `discount_amount` per line
  (drill, receipts). Every response repeats this in `data.units`.
- **Period:** `period` (`7d`, `30d`, `3m`, `6m`, `12m`, `2y`, `this-month`, `last-month`, `ytd`, `this-year`,
  `last-year`, `since-first-order`) **or** `from_date` + `to_date` (`YYYY-MM-DD`, both inclusive, Copenhagen
  days), never both. With neither: the last 30 days. Maximum range 5 years; `since-first-order` is exempt.
- **Scope:** `household` (default, everyone's orders) or `me` (only the caller's own orders). `shop` narrows to
  `nemlig`, `bilka` or `foetex`; omit it for all shops combined. `hide_excluded` leaves out what the household
  hid in the dashboard.
- **Filters** (spend, top, trend, drill, receipts; they combine with AND):
  - `product` and `exclude` take **Danish names as Nemlig writes them** (`mælk`, `jordbær`, `leverpostej`),
    fuzzy-matched. A product always covers the rotated SKUs Nemlig gave the same product, so totals include its
    full history. `exclude` wins over `product`. `exclude_sku` takes exact `product_numbers`.
  - `category_main`, `category_level`, `category_sub` (and their `exclude_…` forms) take category names
    **exactly as returned** by `analytics_top` or `analytics_categories_tree`.
- **Ids you can pass on:** `product_name`, `product_numbers`, `canonical_id` (one product across SKU
  rotations) and category `name`s from `analytics_top` / `analytics_spend`; `order_number` and `order_id` from
  `analytics_receipts` / `analytics_drill`. There are no numeric category ids: category names are the ids.
- A product or category filter that matches nothing returns **0 kr, not an error**. Read `next`: it says so.
  A mistyped order number is a `not_found` error.

### analytics_overview
When: "How much did we spend?", a summary of a period. The one call to start with.
Arguments: `bucket` (auto), `top_limit` (6), `recent_orders` (4, each with all its lines), `compare` (true:
against the previous equally long period; ignored for `since-first-order`). No product/category filters.
Returns: `spend` (total, total_discount, order_count, comparison), `top_categories`, `trend`, `recent`
(newest orders with lines).
Next: `analytics_top` for a longer ranking, `analytics_spend` / `analytics_trend` with filters,
`analytics_receipts`.
Pitfalls: it ignores filters on purpose. For "milk only", use `analytics_spend`.
Fallback: CLI `nemlai --json analytics overview --last 3m`; web https://nemlai.milops.org/analytics (Open the page. The Forbrug tab shows the total for the chosen period. Use the Top and Trend tabs for the ranking and the trend.)

### analytics_spend
When: the total for a product or category, optionally compared with the previous period.
Arguments: `compare` (false), `limit` (all rows; totals always cover everything).
Returns: `total`, `total_discount`, `order_count`, `matches` (one row per canonical product: `canonical_id`,
`product_name`, `product_numbers`, `quantity`, `total`, `share_pct`, `variants`), `comparison`
(`previous_total`, `delta_absolute`, `delta_pct`; `delta_pct` is null when the previous period had no spend).
Next: `analytics_trend` (same filters), `analytics_drill`, `analytics_receipts`.
Pitfalls: Danish spelling; `øko` and non-øko are different products. A 0 total usually means a name that
matches nothing: use `analytics_top(kind='products')` to see real names. With `product` AND `category_main`
both set, only products inside that category count.
Fallback: CLI `nemlai --json analytics spend --last 30d`; web https://nemlai.milops.org/analytics (Open the page. Stay on the Forbrug tab and set the period and the Produkt / Kategori filters.)

### analytics_top
When: "what do we spend most on", by product or by category.
Arguments: `kind` (`products` or `categories`), `level` (`main`, `category_level`, `sub`; categories only),
`limit` (10, max 100), `compare`.
Returns: `matches` (products: as in `analytics_spend`; categories: `name`, `quantity`, `total`, `share_pct`),
`total`, `total_discount`, `comparison`.
Next: `analytics_trend` or `analytics_drill` with a returned `product_name`; for categories, filter by the
returned `name` with `category_main` / `category_level` / `category_sub` matching the `level` you asked for.
Pitfalls: `share_pct` is relative to the filtered total, not to all spend. Uncategorised lines are grouped
under the category `(ukendt)`.
Fallback: CLI `nemlai --json analytics top products --last 90d`; web https://nemlai.milops.org/analytics (Open the page. Open the Top tab, pick Produkter or Kategorier and set the period.)

### analytics_trend
When: spend over time, optionally for a product or category.
Arguments: `bucket` (`auto`, `day`, `week`, `month`). `auto` picks the finest one the period supports.
Returns: `buckets` (`label`, `period_start`, `period_end`, `total`, `count` of orders), `granularity` actually
used, `requested_granularity`, `bucket_adjusted`, `total`.
Next: `analytics_drill` with a bucket's dates as `from_date` / `to_date`.
Pitfalls: a bucket too coarse for the period is downgraded, not rejected (week needs 14+ days, month 60+):
check `bucket_adjusted`. Buckets without matching orders are returned with total 0 and count 0.
Fallback: CLI `nemlai --json analytics trend --bucket auto --last 12m`; web https://nemlai.milops.org/analytics (Open the page. Open the Trend tab and set the period and filters.)

### analytics_drill
When: the individual lines behind a number: when something was bought, at what price, in which order.
Arguments: `order` (an `order_number`, or `id:<n>` for an `order_id`; the order must be inside the period),
`limit` (50 per page, max 200), `cursor`.
Returns: `entries` newest first (`order_id`, `delivery_date`, `product_number`, `product_name`, `quantity`,
`amount`, `discount_amount`, `canonical_id`), `total` and `total_lines` for **all** matches, `returned`,
`has_more`, `next_cursor`, `truncated`.
Next: the next page with `next_cursor`; `analytics_receipts` to group per order.
Pitfalls: pass the same arguments with the cursor, only `limit` may change; a cursor from another request is
rejected. At most the 5000 newest matching lines can be paged (`truncated: true` beyond that): narrow the
period. An unknown `order` returns `not_found` with a `fix` pointing at `analytics_receipts`.
Fallback: CLI `nemlai --json analytics drill --product <name> --last 30d`; web https://nemlai.milops.org/analytics (Open the page. Open the Detaljer tab and set the filters to see the order lines.)

### analytics_receipts
When: expense documentation, or finding the `order_number` for `analytics_drill`.
Arguments: `detail` (`compact` or `full`; in this JSON output both return the same fields, it only changes the CLI's text tables), `limit` (10
**orders** per page, max 100), `cursor`, `order`, `names_only` (triage: which product names match each
`product` term, with `order_count` and `total_quantity`; needs `product`, no paging).
Returns: `orders` newest first (`order_id`, `order_number`, `delivery_date`, `matched_subtotal`,
`order_total`, `lines`), `total_orders`, `total_matched`, `has_more`, `next_cursor`.
Next: `analytics_drill(order=<order_number>)`; with `names_only`, pick a `product_name` for
`analytics_spend`.
Pitfalls: `matched_subtotal` covers only the lines that match your filters, `order_total` is the whole
order. `order_total` is always filled in. Saving to a file (the CLI's `--save`)
isn't available here: format the receipts yourself.
Fallback: CLI `nemlai --json analytics receipts --last 30d`; web https://nemlai.milops.org/analytics (Open the page. Open the Kvitteringer tab and set the period and filters.)

### analytics_categories_tree
When: you need the exact category names before filtering, or a full breakdown by category.
Arguments: only the shared period / scope / shop / `hide_excluded`.
Returns: `tree`: main nodes, each with `name`, `total` and `children` (middle level, then sub), biggest first.
Next: filter another tool by a node's `name`: main -> `category_main`, middle -> `category_level`,
leaf -> `category_sub`.
Pitfalls: the same name can occur under several parents; filter at the right level. An empty tree means no
orders in the period.
Fallback: CLI `nemlai --json analytics categories-tree --last 12m`; web https://nemlai.milops.org/analytics (Open the page. Open the Kategori filter dropdown; it lists the category tree.)
