# Shops

Two different shop lists; don't mix their ids:

- **Price shops** (`list_shops.price_shops`): where the basket can be priced. `nemlig` is the source
  shop; `bilkatogo` is shown to users as "Bilka" (BilkaToGo's national prices); then `netto`,
  `foetex`, `rema1000`. A shop only appears when it has recent prices.
- **Linked shops** (`list_shops.linked_shops`): `bilka` and `foetex` receipt links (Salling Plus),
  per member. Linking takes the shop password, so it is web-only (`/settings/shops`).

### list_shops
When: before comparing prices or sending, or when the user asks which shops they can use.
Arguments: none.
Returns: `price_shops[]` (`shop_id`, `name`, `send_mode`, `can_send`, `prices_as_of`, `beta`,
`note`), `linked_shops[]` (`shop_id`, `label`, `available`, `members[]` with `state` ok |
not_linked | needs_relink | not_member | error, `order_count`, `last_order_date`, `is_me`),
`link_url`.
Next: `compare_shop_prices`; `refresh_shop` for a linked shop; `send_basket` for a shop with
`can_send`.
Pitfalls: `send_mode: manual_list` means NemlAI can't fill that shop's cart; the user shops from the
price list. `bilkatogo` can only be sent to when `can_send` is true (it is behind a gate). CLI
fallback for the price shops: `nemlai --json basket shops`.
Fallback: CLI `nemlai --json shops status`; web https://nemlai.milops.org/settings/shops (Open the page; linked shops and each member's status are shown. Price shops are the tabs above the basket on /kurv/.)

### refresh_shop
When: the user shopped at Bilka or Føtex and wants those receipts in the history or analytics.
Arguments: `shop`: `bilka` or `foetex` (linked-shop ids, not price-shop ids). `wait_seconds` as for
every job.
Returns: a job; completed `result`: `orders_imported`, `users[]` (per member `status`, `reason`,
`orders_written`), `needs_relink[]` (member ids), `message`.
Next: `list_shops` to show the states; `job_status` while running.
Pitfalls: `not_linked` means nobody linked the shop: give the link in the hint. Members in
`needs_relink` must link again in the web app (password; never in chat). At most 3 refreshes a
minute per shop; a second call while one runs returns `job_running` with the job id.
Fallback: CLI `nemlai --json shops refresh <bilka|foetex>`; web https://nemlai.milops.org/settings/shops (Open the page. Click 'Hent kvitteringer' for the shop.)

### compare_shop_prices
When: the user asks "hvad koster kurven i Netto/Bilka?", which shop is cheapest, or wants a shopping
list for a shop without a cart.
Arguments: `shops` (price-shop ids; default every shop except Nemlig), `detail_shop` to list one
shop's lines, `limit`/`cursor` for those lines (default 50), `overrides`
`{sku: {shop_id: product_number}}` from `list_alternatives` (sku = the basket item's sku from `get_basket`).
Returns: `basket` (`item_count`, `total_kr` at Nemlig), `shops[]` with `total` (kr),
`total_is_partial`, `counts` (matched/equivalent/suggestion/unavailable/unpriced/stale), `note` (small
print for the shop, e.g. that "Bilka" shows BilkaToGo's prices), `prices_as_of` (the oldest price in
that total), `not_matched`
(first 20) or, for `detail_shop`, `items[]` (`key` = the basket sku, `basket_name`, `status`, `product_number`,
`name`, `quantity`, `price`, campaign fields) and `next_cursor`; `cheapest_shop`. With an empty basket:
`source: "pregenerated"` and the old comparison of the basket NemlAI prepared.
Next: `list_alternatives` for unavailable or `suggestion` lines; `send_basket` if the user asks.
Pitfalls: a partial total leaves out unpriced/unavailable items, so don't call that shop cheapest
without saying so. `suggestion` lines are a different product (see `differs`); tell the user.
Campaign prices only apply from `campaign_min_quantity`. Overrides are not stored: pass them again on
every call, including `send_basket`. CLI fallback for the latest prepared basket:
`nemlai --json compare`.
Fallback: CLI `nemlai --json basket prices [--shop <shop_id>]`; web https://nemlai.milops.org/kurv/ (Open the page. Click the shop tabs above the basket (Bilka, Netto, …) to see each shop's total.)

### list_alternatives
When: a shop lacks a basket item, matched it loosely, or the user wants another brand/size there.
Arguments: `shop` (price-shop id), `sku` (the basket item's sku from `get_basket`), `query` (2–60 chars, Danish) to search that shop instead.
Returns: `items[]` (`product_number`, `name`, `brand`, `size`, `source` match | suggestion |
candidate | search, `differs`, `quantity`, `price`, campaign fields), `sku`, `basket_name`.
Next: pass the pick as `overrides={sku: {shop: product_number}}` to `compare_shop_prices` or
`send_basket`.
Pitfalls: items added by search have no product match, so they need `query`. Keep "øko" in the
search text when the basket item is organic.
Fallback: CLI `nemlai --json basket alternatives <item> --shop <shop_id> [--search <text>]`; web https://nemlai.milops.org/kurv/ (Open the page. Open the shop's tab. Click 'Vælg alternativ' on the item.)

### send_basket
When: only after the user has seen the basket and its total and explicitly asks to send it.
Arguments: `shop`: `nemlig` or `bilkatogo`. `overrides` (bilkatogo only) as in
`compare_shop_prices`. `wait_seconds` (default 20).
Returns: a job; completed `result`: `item_count`, `basket_total_kr`, `cart_url` (nemlig: also
`loaded_item_count`; bilkatogo: `status` sent | partial | nothing_to_send, `sent`, `skipped`,
`missing`, `unavailable`).
Next: give the user `cart_url`: they check out and pay at the shop themselves.
Pitfalls: destructive: it changes the user's real cart. Never send twice: a send that is running
returns `job_running` with its job id; after a timeout or unclear result, call `job_status` first.
BilkaToGo only raises quantities and never removes lines, so a second send can double items.
`not_linked`/`reauth_required` point to the web app; never ask for a password. Shops other than
nemlig and bilkatogo can't be sent to. CLI fallback for BilkaToGo:
`nemlai --json basket send-shop bilkatogo`.
Fallback: CLI `nemlai --json basket send`; web https://nemlai.milops.org/kurv/ (Open the page (log in if asked): it shows the same basket as MCP and the CLI. Check the basket and the total with the user. Nemlig: click 'Send til nemlig'. BilkaToGo: open the 'Bilka' tab and send from there. Open the cart link and let the user check out themselves.)
