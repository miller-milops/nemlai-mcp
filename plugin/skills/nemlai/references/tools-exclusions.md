# Exclusions

Excluded orders and hidden lines are left out of predictions and analytics ("the party order", "the diapers I bought for a friend").

### get_exclusions
When: "what have I excluded?", or to find an order id before excluding it.
Arguments: `state` (all | excluded | hidden), `limit` (default 20, max 100), `cursor`.
Returns: orders newest first (`order_id`, `delivery_date`, `total_amount` in kr, `item_count`, `excluded`, `hidden_lines`), `excluded_order_ids`, `hidden_line_count`, `next_cursor`.
Next: `list_order_items(order_id)`; `update_exclusions`.
Pitfalls: ids come only from here. Never guess an order id.
Fallback: CLI `nemlai --json exclusions list`; web https://nemlai.milops.org/ekskluder (Open the page; excluded products and orders are listed.)

### list_order_items
When: the user wants to hide single products on an order.
Arguments: `order_id` from `get_exclusions`; `limit` (default 100, max 500), `cursor`.
Returns: lines with `product_name`, `quantity`, `amount` in kr, `hidden`, `exclusion_key`.
Next: `update_exclusions(add_lines=[exclusion_key, …])`.
Pitfalls: pass `exclusion_key`, not `line_key` or the product number. An unknown order returns `not_found` with recent order ids.
Fallback: CLI `nemlai --json exclusions items <order_id>`; web https://nemlai.milops.org/ekskluder (Open the page. Open the order to see its lines.)

### update_exclusions
When: exclude or include a whole order, or hide or unhide lines.
Arguments: `add_orders`, `remove_orders` (order ids); `add_lines`, `remove_lines` (`exclusion_key` values). Name only what changes: everything else stays. Max 200 changes per call; the same id can't be in both add and remove.
Returns: new totals, `changed`, `skipped` (already in the wanted state), `saved` (false when nothing changed).
Next: `get_exclusions` to review; `generate_basket` to apply.
Pitfalls: this is an explicit add/remove, never a replacement. Excluding an order removes it from predictions and from `analytics_*` with hide-excluded on, so tell the user. A concurrent edit is retried once; if it still conflicts you get `upstream_unavailable`: re-read and retry.
Fallback: CLI `nemlai --json exclusions exclude|include <order_id>  /  nemlai --json exclusions hide|unhide <order_id> <product_number>`; web https://nemlai.milops.org/ekskluder (Open the page. Exclude or include the product or order.)
