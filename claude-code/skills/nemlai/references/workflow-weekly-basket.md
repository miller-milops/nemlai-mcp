# Weekly basket

Build the household's usual weekly basket, review it with the user and send it to Nemlig on their go-ahead.

**User says:** "lav ugens kurv", "do my weekly shop", "order the usual groceries".

1. `whoami()`: check the connection and that there is a household.
2. `get_basket()`: is there already a basket? If it has items from today or yesterday, offer to
   review it instead of generating a new one (generating replaces it).
3. If the orders look stale (the basket or `order_fetch_status()` says the last fetch is old), offer
   `refresh_orders()` first so the prediction knows about the latest order. It is a job: wait for it
   or poll `job_status()`. If it says `reauth_required`, see `order-refresh-reauth`.
4. `generate_basket()` with the user's budget preset (`medium` if they don't say). It waits up to
   `wait_seconds`; if it returns a `job_id`, poll `job_status()` until it is done.
5. `get_basket()` and show the user: the items (name, quantity, unit price), items on campaign, the
   total in kr and the budget. Group by category if the list is long.
6. Adjust what the user asks for:
   - "more milk": `set_quantity()` with the item's SKU.
   - "no crisps this week": `remove_from_basket()`.
   - "add rugbrød": `search_products()`, let the user pick if there are several, then
     `add_to_basket()` with the chosen SKU.
   - "why is X there?": `explain_item()`.
7. Show the new total. Ask "Shall I send it to Nemlig?" and wait for a clear yes.
8. `send_basket()` with `shop="nemlig"`. If it returns a `job_id`, poll `job_status()`; never send
   again without checking.
9. Tell the user it is in their Nemlig cart and that they choose the delivery time and pay on
   nemlig.com themselves. Give them the completed send's `cart_url` as "Åbn kurven i Nemlig (åbner
   appen, hvis den er installeret)". If the send failed there is no link: say what to do instead.

**Pitfalls:** don't send on "make a basket"; don't raise quantities to reach a campaign without
asking; a new `generate_basket()` throws away manual changes, so warn first.
