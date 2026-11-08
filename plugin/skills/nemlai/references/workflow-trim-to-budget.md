# Trim to budget

Bring a basket down to a budget in kr by removing or reducing the least needed items, with the user's approval.

**User says:** "max 1000 kr", "it's too expensive", "trim it to 800".

1. `get_basket()`: the total in kr, the budget, and each item's price, quantity and campaign info.
2. Work out how much to cut (total minus the target).
3. Propose cuts, cheapest pain first:
   - items the prediction is least sure about: `explain_item()` reasons with category `fills_budget`
     (added only to reach the budget) or `less_likely` (often skipped);
   - big quantities that can go down by one;
   - expensive items with a cheaper similar product (`search_products()`; keep øko if the user buys
     øko);
   - treats and extras before staples (milk, bread, eggs).
4. Mind campaigns: lowering a quantity below `campaign_min_quantity` raises the unit price
   ("Mix 3 stk 79,00": 3 cost 79 kr, 2 cost the regular price each). Sometimes removing the whole
   campaign line saves more.
5. Show the proposal as a list with what each change saves and the new total. Change nothing until
   the user agrees.
6. Apply: `set_quantity()`, `remove_from_basket()`, or `add_to_basket()` for a swap.
7. `get_basket()` and show the final total against the budget.
8. Next time, `generate_basket()` with a smaller budget preset may fit better.

**Pitfalls:** never cut silently; recompute the total from `get_basket()` instead of doing the sum
yourself; pant on drinks is not part of the product price.
