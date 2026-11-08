# Exclude products

Stop products or whole orders from shaping the predictions (a one-off party order, a product the household no longer buys).

**User says:** "vi køber ikke længere sodavand", "ignore the Christmas order", "stop suggesting Y".

1. `get_exclusions()`: what is excluded or hidden today.
2. Find what to exclude:
   - a product: from `get_basket()` (it is in the basket now) or `search_products()`; use its SKU;
   - an order: `analytics_drill()` or `analytics_receipts()` show orders; `list_order_items()` shows
     one order's lines.
3. Say what will change ("the order from 20 December will be left out of predictions and spending
   reports") and get the user's yes.
4. `update_exclusions()`: exclude a whole order, include it again, or hide single products in an
   order.
5. If the product is in the current basket, also `remove_from_basket()` (exclusions shape future
   baskets; they don't edit this one).
6. For a lasting "never suggest this", also check `get_preferences()` / `update_preferences()`
   (e.g. excluded products, prefer øko).
7. The next `generate_basket()` uses the new exclusions.

**Pitfalls:** excluding an order also removes it from analytics; hiding a product in one order is
not the same as never suggesting it; use ids from tool results only.
