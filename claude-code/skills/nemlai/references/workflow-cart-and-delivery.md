# Shop cart and delivery times

See what is already in the user's cart at Nemlig or BilkaToGo, and which delivery times are free and cheapest, without changing anything at the shop.

**User says:** "hvad ligger der i min Nemlig-kurv?", "hvilke tider er billigst i morgen?",
"hvad koster leveringen?", or you are about to `send_basket()`.

1. `get_shop_cart()` (or `shop="nemlig"`/`"bilkatogo"`): the lines, fees, total and whether the
   minimum order is met. Lines with `basket_sku` are also in the NemlAI basket.
2. Sending: call `send_basket(shop=…)` without `cart_mode`. If the shop's cart already holds items,
   nothing is sent and the answer is `cart_not_empty`: ask the user `details.ask_user` ("Skal jeg
   beholde dem og lægge de nye varer oveni, eller tømme kurven og kun sende de nye?") and send again
   with `cart_mode="keep"` or `"replace"` as they answer. Never choose for them.
3. Times: `list_delivery_slots(shop=…, days=2)` for today and tomorrow, cheapest first. Say the
   price and the kind (Fleksibel is unattended and often cheaper at Nemlig).
4. `get_delivery_info(shop=…)` for the chosen time and the deadline to change the order.
5. The user books the time and pays on the shop's site (`cart_url`). Never offer to do it.

**Pickup (BilkaToGo):** a cart set to pickup answers `delivery_mode: "pickup"` and
`pickup_store` ("Bilka Vejle"); the times are pickup times and the fee is the cart's (often 0). With
`delivery_mode: null` BilkaToGo doesn't say: ask the user to choose home delivery or pickup on
bilkatogo.dk.

**Pitfalls:** a missing or expired shop login answers `not_linked`/`reauth_required`: give the link
from `account_links()`, never ask for a password.
