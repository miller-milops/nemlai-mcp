# Compare shops

Price the basket at other shops, pick alternatives where a shop lacks a product, and send it to a shop that accepts a basket.

**User says:** "er det billigere i Bilka?", "compare shops", "send it to BilkaToGo instead".

1. `get_basket()`: is there a basket? Empty is fine for a price question: `compare_shop_prices()`
   then prices the basket NemlAI prepared for the household (`source: "pregenerated"`); tell the
   user that. Make a basket (`generate_basket()`) only when the user asks for one.
2. `list_shops()`: the shops, whether each is linked, its status and whether it can receive a
   basket (`send_mode`). Use only these shop ids.
3. `compare_shop_prices()`: the basket's total at each shop, how many items matched, which are
   suggestions (a similar product, with how it differs) and which are missing. Prices are a
   snapshot; check `prices_as_of` and say when a shop's prices are old.
4. Show the user a short table: shop, total in kr, matched / missing. A shop with
   `total_is_partial` lacks some items, so its total is not the basket's price there: compare it
   with `same_items_nemlig_kr` (the same items at Nemlig) and say how many items are missing.
   `cheapest_shop` only counts complete totals.
5. For a suggested or missing item at the shop the user likes: `list_alternatives()` for that shop
   and item, let the user pick, and use the pick for that shop.
6. To send to another shop: it must be linked and able to receive a basket. Not linked → see
   `shop-not-linked`. Shops that can't receive a basket: give the user the list to shop by hand.
7. Show the items and the total at that shop and wait for a clear yes; then
   `send_basket()` with that `shop`. Poll `job_status()` if it returns a `job_id`.
8. Tell the user to check out on that shop's site.

**Pitfalls:** "Bilka" in the basket tabs is BilkaToGo (`bilkatogo`). Bilka's stores use the same
price list, but a store can have local offers and Bilka Plus member prices, and some store items
aren't sold online; say so when the user shops in the store. If `counts.stale` is above 0, some
prices are old: say so; a suggestion is a different product, so say how it differs (size, øko,
brand); never send twice without checking `job_status()`.
