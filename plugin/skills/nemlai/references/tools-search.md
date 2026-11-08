# Search

Find Nemlig products by Danish name, look one up by SKU, and list the product categories.

### search_products
When: the user wants a product that isn't in the basket, wants a cheaper or øko alternative, or `add_to_basket()` returned candidates and you need more.
Arguments: `query` in Danish, one product per search ("letmælk", "rugbrød", "øko bananer"); `limit` (default 10, max 50); `cursor` (`next_cursor`).
Returns: `products` (`sku`, `name`, `price_kr`, `unit_price` + `unit_price_label` (e.g. "kr/l"), `is_eco`, `availability` (`status` ok | low_stock | unavailable, `reason`), `campaign_label`, `description`), `total`, `next_cursor`.
Next: show the matches; `add_to_basket(items=[{sku, quantity}])` with the one the user picks; `product_info()` for details.
Pitfalls: search one product at a time ("mælk og brød" finds neither well). Names are Danish: translate the user's words ("milk" → "mælk"). An empty list is a normal result, not an error: try a shorter word. `campaign_label` is the shop's text ("3 for 50", "Mix 3 stk 79,00"): the price shown is the single-unit price. Don't offer `unavailable` products.
Fallback: CLI `nemlai --json search <query> [--limit <n>]`; web https://nemlai.milops.org/kurv/ (Open the page. Use the search field.)

### product_info
When: the user asks about one product: brand, weight, origin, whether it's øko or in stock, or the campaign details.
Arguments: `sku` from `search_products()`, `get_basket()` or `find_dishes()`.
Returns: `name`, `price_kr`, `unit_price`, `unit_price_label`, `label`, `description`, `brand`, `weight_grams`, `category`, `origin`, `is_eco`, `availability`, and `live` (`in_stock`, `stock_quantity`, `price_kr`, `campaign` (`label`, `min_quantity`, `unit_price`)) or null when the live lookup failed.
Next: `add_to_basket()`.
Pitfalls: an unknown sku returns `not_found` (the product may be discontinued): search by name instead. Fields are null when the catalogue has no value; don't invent them.
Fallback: CLI `nemlai --json search info <sku>`; web https://nemlai.milops.org/kurv/ (Open the page. Search for the product and open it.)

### list_categories
When: the user asks what kinds of products there are, or you need a category name (for browsing ideas or for the `analytics_*` category filters).
Arguments: none.
Returns: `categories` (`name` and `subcategories`, Danish names) and `count`.
Next: `search_products(query=<sub category>)`.
Pitfalls: the list comes from NemlAI's product catalogue, not live from Nemlig, so it changes rarely. It is a navigation aid: searching a category name finds products whose names match it, not everything in the category.
Fallback: CLI `nemlai --json categories`
