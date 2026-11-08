# Dishes

Find the Nemlig products a dish or meal needs (AI search), show its recipe and add the ingredients to the basket.

A dish search is remembered under a `dish_id` for the user's last 10 searches (in server memory,
so a restart forgets them: search again). The CLI uses the same search but numbers the ingredients
of the latest dish instead of using ids.

### find_dishes
When: "what do I need for butter chicken?", "add a fish dinner", "ideas for lasagne til 4".
Arguments: `query`: the dish or meal, Danish or English, with the number of people if the user said it; `wait_seconds` (default 30, max 50).
Returns: when the search finishes in time, the dish: `dish_id`, `query`, `ingredients` (`sku`, `name`, `price_kr`, `price_label`, `quantity`, `is_eco`, `available`, `low_stock`, `previously_purchased`, `purchase_count`, `matched_term`, `in_basket`), `ingredient_count`, `estimated_total_kr`, `recipes` (titles), `has_recipe`, `search_terms`. Otherwise `{job_id, status: "running"}`: poll `job_status(job_id)`, whose `result` is the same dish.
Next: show the ingredients (and the estimated total); `add_dish_to_basket(dish_id)` with all or the chosen ones; `dish_recipe(dish_id)` if they want to cook it.
Pitfalls: the AI search takes from a few seconds up to 1–2 minutes; while it runs, poll `job_status()` and don't start the same search again. It can return only a recipe and no products, or products that are only roughly right (it picks one product per ingredient): let the user check before adding. `in_basket` marks ingredients the basket already has. A `not_found` means the AI found nothing: rephrase.
Fallback: CLI `nemlai --json dish <query>`; web https://nemlai.milops.org/kurv/ (Open the page. Pick a dish suggestion (e.g. '+ Fisk en hverdag').)

### dish_info
When: going back to a dish found earlier in the conversation, or to see which of its ingredients are already in the basket.
Arguments: `dish_id` from `find_dishes()`.
Returns: the same as `find_dishes()`.
Next: `add_dish_to_basket()`.
Pitfalls: ids are only valid for the last 10 searches of this user; an unknown id returns `not_found` with the recent `dish_id`s as candidates, or tells you to search again.
Fallback: CLI `nemlai --json dish info <#>`

### dish_recipe
When: the user wants to know how to cook the dish.
Arguments: `dish_id` from `find_dishes()`.
Returns: `recipes`: `[{title, markdown}]` (usually Danish Markdown); empty when the search had no recipe.
Next: `add_dish_to_basket()`.
Pitfalls: the recipe is AI-written: quantities in it may not match the suggested product quantities.
Fallback: CLI `nemlai --json dish recipe`

### add_dish_to_basket
When: the user wants the dish's ingredients in the basket (all or some).
Arguments: `dish_id`; `skus` (only these ingredients; default all); `quantities` (`{sku: n}` to override the suggested quantity); `include_unavailable` (default false).
Returns: `added` (basket items), `already_in_basket`, `skipped_unavailable`, `basket` totals.
Next: `get_basket()`; tell the user what was skipped; `search_products()` for a replacement of an unavailable ingredient.
Pitfalls: ingredients already in the basket are left unchanged, not doubled: use `set_quantity()` if the user wants more. Unavailable ingredients are skipped unless the user insists. A sku that isn't one of this dish's ingredients adds nothing and returns `not_found` with the dish's skus.
Fallback: CLI `nemlai --json dish add all`; web https://nemlai.milops.org/kurv/ (Open the page (log in if asked): it shows the same basket as MCP and the CLI. Pick a dish and add its ingredients.)
