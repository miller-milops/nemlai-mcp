# Basket

Make, read, explain and change the user's basket (kurv): items with quantity, prices in kr, campaign prices and a total against a budget.

The basket belongs to the NemlAI user (each household member has their own) and is saved on the
server, so it survives restarts and deploys. It is the same basket `nemlai basket items` shows in
the CLI and SDK and the web page `/kurv/` shows: a change made here appears there, and the other
way round. Items are identified by `sku` (Nemlig's product number), always taken from a tool
result.

Every read and write returns the basket's `version`. If the user changes the basket on `/kurv/` (or
another agent or the CLI does) while you change it, your write fails with `conflict` and
`fix: get_basket`: read the basket again, check the change still makes sense, and redo it.

Every item carries `unit_price` (what one unit costs at this quantity), `regular_unit_price`,
`campaign_unit_price`, `campaign_min_quantity`, `campaign_label` (e.g. "Mix 3 stk 79,00"),
`on_sale`, `line_total`, `is_eco`, `category`, `source` (`prediction`, `substitute`, `manual`,
`dish`) and `hint`. A campaign price only applies from `campaign_min_quantity` units: every full
bundle costs the campaign price and the rest the regular price (Rugbrød 29 kr at "Mix 3 stk 79,00":
2 = 58.00 kr, 3 = 79.00 kr, 4 = 108.00 kr). `hint` says what to add to reach the next bundle.

### generate_basket
When: "make my weekly basket", "start a new basket", after `refresh_orders()`, or when `get_basket()` is empty or old.
Arguments: `size` (`small` ≈ 800 kr, `medium` ≈ 1500 kr (default), `large` ≈ 2500 kr); `budget_kr` (100–20000) sets a custom budget instead of the preset; `delivery_date` (YYYY-MM-DD, default the next delivery); `wait_seconds` (default 20, max 50).
Returns: a job: `status` `completed` with `result` (`item_count`, `total_kr`) and `basket` (totals, budget, `campaign_hints`), or `status` `running` with `job_id`.
Next: `get_basket()` to review it with the user; while running, `job_status(job_id)`.
Pitfalls: it **replaces** the current basket, including the user's manual changes, so warn first if they have edited it. A second call while one runs returns `job_running` with the running job's id: poll that instead. Generating refreshes orders with the stored Nemlig login; when that login has expired it still generates from the existing history. The budget is a target, not a hard cap: compare `total_kr` with `budget.budget_kr`.
Fallback: CLI `nemlai --json basket generate --size <small|medium|large> [--budget <kr>]`; web https://nemlai.milops.org/kurv/ (Open the page (log in if asked): it shows the same basket as MCP and the CLI. If a basket is shown, click 'Ny kurv'. Pick the size and a delivery date and generate the basket; it replaces the saved one.)

### get_basket
When: before talking about the basket, before `send_basket()`, after changes, or when the user asks what's in it.
Arguments: `limit` (default 50, max 200) and `cursor` (`next_cursor` from the previous page).
Returns: `version`, `items` (see above), `item_count`, `total_kr`, `budget` (`preset`, `budget_kr`, `remaining_kr`, `over_budget`) or null, `generated_at`, `campaign_hints`, `next_cursor`.
Next: `explain_item()`, `set_quantity()`, `remove_from_basket()`, `add_to_basket()`; `send_basket()` only when the user says so.
Pitfalls: an empty basket means "generate one", not an error. Read every page before quoting a total of items; `total_kr` already covers the whole basket. Prices can change at the shop; the shop's cart total is final.
Fallback: CLI `nemlai --json basket items`; web https://nemlai.milops.org/kurv/ (Open the page (log in if asked): it shows the same basket as MCP and the CLI. Read the items and the total ('Anslået beløb'); an empty basket opens on the generate form.)

### explain_item
When: "why is X in my basket?", "why 3 of them?", or when trimming to a budget and you need to know which items the prediction is least sure about.
Arguments: `sku` from `get_basket()`.
Returns: `summary` (one Danish sentence), `reasons` (`[{category, text}]`, the text in Danish; categories: `purchase_history` (how often and when they buy it), `quantity` (why this many), `quantity_limited` (kept close to what they usually buy), `running_low` (probably running out), `season_or_holiday`, `often_together` (bought with something else), `fills_budget` (added to reach the budget), `less_likely` (often skipped when suggested), `replacement` (replaces a product that isn't available)), `predicted_quantity`, `typical_quantity`, `median_quantity`, `max_quantity_ever`, `quantity_limited`, `substitution` (`replaces`, `replaces_sku`) or null, `history` (`times_bought`, `orders_considered`, `last_bought`, `average_quantity`) or null, and the `item`.
Next: `set_quantity()` or `remove_from_basket()` if the user disagrees.
Pitfalls: items the user added themselves (`source` `manual` or `dish`) have no prediction reasons. A `fills_budget` reason means the item was added to reach the budget, so it is the first candidate to drop; `less_likely` items come next. `history` is null when the database can't be read; that's not an error.
Fallback: CLI `nemlai --json basket why <sku>`; web https://nemlai.milops.org/kurv/ (Open the page (log in if asked): it shows the same basket as MCP and the CLI. Tap the (i) next to the item.)

### add_to_basket
When: the user wants a product added ("add rugbrød", "2 liter øko mælk too").
Arguments: `items`: 1–50 of `{sku | query, quantity}` (quantity 1–99, default 1). Prefer `sku` from `search_products()`, `find_dishes()` or a previous error's candidates.
Returns: `added` (items as in `get_basket()`), `already_in_basket` (`sku`, `name`, current `quantity`), `basket` totals.
Next: `get_basket()`; `set_quantity()` for items that were already there.
Pitfalls: all or nothing. A `query` is added only when it matches exactly one product or exactly one product's name; otherwise **nothing** is added and the error (`invalid_argument`) lists `candidates` (`sku`, `name`, `price_kr`, `is_eco`, `available`): show them to the user and call again with the chosen `sku`. Never pick a candidate yourself unless the user's words name it exactly. Keep "øko" when the user said it. A product already in the basket is not doubled: change it with `set_quantity()`.
Fallback: CLI `nemlai --json basket add --product-number <sku> --quantity <n>`; web https://nemlai.milops.org/kurv/ (Open the page (log in if asked): it shows the same basket as MCP and the CLI. Type the product in 'Tilføj en vare eller søgning…' and click 'Tilføj'.)

### set_quantity
When: "more milk", "only one rugbrød", or to reach a campaign minimum.
Arguments: `sku` from `get_basket()`; `quantity` 1–99 (units, stk).
Returns: the `item` (with `line_total`, `on_sale`, `hint`) and `basket` totals.
Next: `get_basket()`.
Pitfalls: 0 is not allowed: use `remove_from_basket()`. Check `hint`: below `campaign_min_quantity` the regular price applies.
Fallback: CLI `nemlai --json basket adjust --product-number <sku> --quantity <n>`; web https://nemlai.milops.org/kurv/ (Open the page (log in if asked): it shows the same basket as MCP and the CLI. Change the item's quantity with − and +; it is saved at once.)

### remove_from_basket
When: the user doesn't want an item, or to trim the basket to the budget.
Arguments: `skus`: one or more from `get_basket()`.
Returns: `removed` (`sku`, `name`, `quantity`) and `basket` totals.
Next: `get_basket()`.
Pitfalls: all or nothing: an unknown sku removes nothing and returns `not_found` with the closest basket items.
Fallback: CLI `nemlai --json basket remove --product-number <sku>`; web https://nemlai.milops.org/kurv/ (Open the page (log in if asked): it shows the same basket as MCP and the CLI. Press − at quantity 1, then the bin next to the item.)

### clear_basket
When: only when the user explicitly asks to empty the basket or start over by hand.
Arguments: none.
Returns: `removed_count` and the (empty) `basket` totals.
Next: `generate_basket()` or `add_to_basket()`.
Pitfalls: destructive and can't be undone; clients ask the user first. To start over, `generate_basket()` already replaces the basket, so clearing first is unnecessary.
Fallback: CLI `nemlai --json basket clear`; web https://nemlai.milops.org/kurv/ (Open the page (log in if asked): it shows the same basket as MCP and the CLI. Remove the items (− at quantity 1, then the bin), or click 'Ny kurv' and generate a new basket, which replaces it.)
