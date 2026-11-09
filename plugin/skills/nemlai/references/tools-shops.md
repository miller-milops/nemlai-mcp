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

### list_offers
When: "hvad er på tilbud?", "er noget af det vi plejer at købe på tilbud?", "ugens tilbud hos Bilka",
Dagstilbud or free gifts at nemlig.
Arguments: `mine` (default true: only the household's usual items on offer; false: every offer),
`shops` (`nemlig`, `bilkatogo`; default nemlig, plus bilkatogo when Bilka is linked), `category`
(Danish text matched against the category or label: "kød", "mejeri", "dagstilbud", "gratis"),
`limit`/`cursor`.
Returns: `offers[]` with `shop`, `product_id`, `name`, `size`, `label` (Skarp pris | Priskup |
Dagstilbud | Undgå madspild | Gratis | Mix | Tilbud), `offer_text`, `price`, `before_price`,
`campaign_min_quantity`, `campaign_unit_price`, `max_per_order`, `offer_until`, `available`,
`category`; with `mine`: `why` (a Danish note to show the user), `usual` (`name`, `sku`, `times_bought`,
`last_bought`, `usual_quantity`), `in_basket`, `in_prepared_basket`. Also `total`, `next_cursor`,
`errors` (one entry per shop that failed) and `notes`.
Next: for a nemlig usual item the user wants: `add_to_basket(items=[{"sku": product_id}])`; if it is
already in the basket below the campaign minimum: `set_quantity(sku=…, quantity=campaign_min_quantity)`;
for BilkaToGo: `compare_shop_prices(shops=["bilkatogo"])`.
Pitfalls: a campaign price applies only from `campaign_min_quantity` ("Mix 2 stk. 25,-"): say so,
and say how many the household usually buys (the `why` note does). `offer_until` is null when the
shop doesn't say; never invent an end date. Free gifts have price 0 and `max_per_order`. A
BilkaToGo `product_id` is not a nemlig sku: never pass it to `add_to_basket`. When `errors` lists a
shop, tell the user that shop's offers are missing instead of saying it has none. Add nothing
without the user's yes.
Fallback: CLI `nemlai --json offers [--all] [--shop <nemlig|bilkatogo>] [--category <text>]`

### send_basket
When: only after the user has seen the basket and its total and explicitly asks to send it.
Arguments: `shop`: `nemlig` or `bilkatogo`. `cart_mode`: leave it out on the first call; after the
user has chosen, `keep` (leave the items already in the shop's cart and add the basket; a product in
both gets the higher quantity, never the sum) or `replace` (empty the shop's cart first, then add the
basket). `overrides` (bilkatogo only) as in `compare_shop_prices`. `wait_seconds` (default 20).
Returns: a job; completed `result`: `item_count`, `basket_total_kr`, `cart_mode`, `cart_url`
(nemlig: also `loaded_item_count`, `already_in_cart` and `cart_link_text`; bilkatogo: `status` sent |
partial | nothing_to_send, `sent`, `skipped`, `missing`, `removed`, `unavailable`, and
`pickup_store` for a pickup cart). Each item goes into the cart through the shop's API. For nemlig,
`cart_url` is the plain Nemlig cart link (it opens the Nemlig app if installed). A failed send has no
`cart_url`.
Without `cart_mode`, an empty shop cart is filled straight away. A cart that already holds items
sends **nothing** and answers the error `cart_not_empty` with `details`: `item_count`,
`line_count`, `products_total_kr`, `names` (up to 5) and `ask_user`, the question in Danish ("Du har
allerede 6 varer for 214,50 kr. i din Nemlig-kurv (bl.a. mælk, rugbrød og bananer). Skal jeg beholde
dem og lægge de nye varer oveni, eller tømme kurven og kun sende de nye?").
Next: on `cart_not_empty`, ask the user and call `send_basket` again with their `cart_mode`. After
the send, give the user `cart_url` (nemlig: as "Åbn kurven i Nemlig (åbner appen, hvis den er
installeret)"): they check out and pay at the shop themselves. Say what happened to what was
already in the cart (kept, or emptied first). A BilkaToGo pickup cart answers `pickup_store`: say
"Afhentning i <butik>".
Pitfalls: destructive: it changes the user's real cart. Never choose `cart_mode` yourself, and never
send `replace` without the user's answer: it removes things they put there. It is the only way to
fill the cart: there is no link that adds items, so never offer one before the send. Never send
twice: a send that is running returns `job_running` with its job id; after a timeout or unclear
result, call `job_status` first. With `keep` a second send changes nothing (the higher quantity
wins), but don't count on it. `not_linked`/`reauth_required` point to the web app; never ask for a
password. Shops other than nemlig and bilkatogo can't be sent to. CLI fallback:
`nemlai basket send --keep|--replace` (Nemlig) and
`nemlai --json basket send-shop bilkatogo --keep|--replace`.

## The shop's own cart and delivery times

Three read-only tools look at the user's cart **at the shop** (nemlig, bilkatogo), not the NemlAI
basket. They use the stored shop logins and never change anything: no cart change, no delivery time
booked, no pickup store, mode or address chosen. The user does all of that on the shop's site.
Fallback: CLI `nemlai --json basket send --keep|--replace`; web https://nemlai.milops.org/kurv/ (Open the page (log in if asked): it shows the same basket as MCP and the CLI. Check the basket and the total with the user. Nemlig: click 'Send til nemlig'. BilkaToGo: open the 'Bilka' tab and send from there. If the shop's cart already holds items, the page asks: 'Behold dem og læg de nye oveni' or 'Tøm kurven og send kun de nye'. Let the user choose. Open the cart link and let the user check out themselves.)

### get_shop_cart
When: the user asks "hvad ligger der i min kurv hos Nemlig/BilkaToGo?", or wants to see the whole
cart before choosing `keep` or `replace` after `send_basket()` answered `cart_not_empty`.
Arguments: `shop` (`nemlig` or `bilkatogo`; omit for every linked shop), `limit`/`cursor` for the
lines (default 100).
Returns: `shops[]`, per shop `lines[]` (`product_id`, `name`, `quantity`, `unit_price_kr`,
`line_total_kr`, `canonical_id`, `basket_sku` when the line is also in the NemlAI basket),
`line_count`, `item_count`, `products_total_kr`, `fees` (`delivery_kr`, `service_kr`,
`packaging_kr`, `deposit_kr`), `total_kr`, `minimum_order_kr`, `minimum_met`, `minimum_left_kr`,
`delivery_mode` (`home`, `pickup` or null), `pickup_store`, `chosen_time`, `notes` (Danish, for the
user), `cart_url`; or `{shop, error}` for a shop that failed. `not_linked[]` lists shops without a
login.
Next: `send_basket()` only after the user's yes; `get_delivery_info()` or `list_delivery_slots()`
for times and fees.
Pitfalls: money the shop doesn't show is null, not 0 (a BilkaToGo pickup fee of 0 is a real 0).
`not_linked`/`reauth_required` point to `account_links()`: relinking is web-only.
Fallback: CLI `nemlai --json shop cart [nemlig|bilkatogo]`

### get_delivery_info
When: "hvornår kommer varerne?", "hvad koster leveringen?", "kan jeg nå at ændre ordren?".
Arguments: `shop`: `nemlig` or `bilkatogo`.
Returns: `chosen_time` for the next order (`start`, `end`, `text`; nemlig also `kind` and `reserved`;
null when none is chosen or it has passed), `upcoming_order` (nemlig: an order already placed and
still ahead, with its `edit_deadline`; else null), `edit_deadline`, `next_free_day` and
`next_free_day_cheapest_fee_kr`, `fees`,
`minimum_order_kr`, `minimum_met`, `delivery_mode`, `pickup_store`, `notes`, `cart_url`.
Next: `list_delivery_slots()` for the free times.
Pitfalls: `reserved: false` means nemlig only holds the time once the user orders; it can sell out.
nemlig always delivers home. A BilkaToGo cart can be set to pickup in a Bilka store
(`delivery_mode: pickup`); with `delivery_mode: null` BilkaToGo doesn't say which: tell the user to
choose it on bilkatogo.dk. Never offer to book or change the time.
Fallback: CLI `nemlai --json delivery <nemlig|bilkatogo>`

### list_delivery_slots
When: "hvilke leveringstider er billigst i morgen?", "hvornår kan jeg få leveret?".
Arguments: `shop` (omit for every linked shop), `days` (calendar days from today, today included;
default 3, max 14; "tomorrow" = `days=2`), `limit`/`cursor` (default 20 per shop).
Returns: per shop `slots[]` cheapest first (`day`, `start`, `end`, `time`, `fee_kr`, `kind`,
`order_before`), `slot_count`, `cheapest` (top 3), `sold_out_days`, `unavailable_count`,
`delivery_mode`, `pickup_store`, `notes`; `first_day`, `last_day`.
Next: tell the user the cheapest times; they choose and book on the shop's site.
Pitfalls: `kind` is Personlig/Fleksibel at nemlig (Fleksibel = unattended, often cheaper) and
Afhentning (pickup), Nat or Hjemlevering at BilkaToGo. BilkaToGo has one flat fee for every slot (the
cart's delivery fee). A BilkaToGo cart with an unknown mode refuses (`forbidden`): the user chooses
home or pickup on bilkatogo.dk. Only free slots are listed.
Fallback: CLI `nemlai --json delivery slots [nemlig|bilkatogo] [--days <n>]`
