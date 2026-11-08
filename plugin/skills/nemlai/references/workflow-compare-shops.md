# Compare shops

Price the basket at other shops, pick alternatives where a shop lacks a product, and send it to a shop that accepts a basket.

**User says:** "er det billigere i Bilka?", "compare shops", "send it to BilkaToGo instead".

1. `get_basket()`: there must be a basket. Empty → `generate_basket()` first.
2. `list_shops()`: the shops, whether each is linked, its status and whether it can receive a
   basket (`send_mode`). Use only these shop ids.
3. `compare_shop_prices()`: the basket's total at each shop, how many items matched, which are
   suggestions (a similar product, with how it differs) and which are missing. Prices are a
   snapshot; old prices are marked.
4. Show the user a short table: shop, total in kr, matched / suggested / missing.
5. For a suggested or missing item at the shop the user likes: `list_alternatives()` for that shop
   and item, let the user pick, and use the pick for that shop.
6. To send to another shop: it must be linked and able to receive a basket. Not linked → see
   `shop-not-linked`. Shops that can't receive a basket: give the user the list to shop by hand.
7. Show the items and the total at that shop and wait for a clear yes; then
   `send_basket()` with that `shop`. Poll `job_status()` if it returns a `job_id`.
8. Tell the user to check out on that shop's site.

**Pitfalls:** "Bilka" in the basket tabs is BilkaToGo (`bilkatogo`); prices at physical shops can
differ from online prices; a suggestion is a different product, so say how it differs (size, øko,
brand); never send twice without checking `job_status()`.
