# Dinners via dishes

Plan the week's dinners from NemlAI's dishes and add their ingredients to the basket.

**User says:** "find 4 aftensmåltider", "plan dinners for the week", "add what I need for lasagne".

1. `find_dishes()` with what the user wants ("fisk", "vegetar", "hurtig", "lasagne"). Search in
   Danish; keep æ, ø, å.
2. Show a short list (name, a line about it). Let the user pick; don't pick for them unless asked.
3. For a picked dish: `dish_info()` shows its ingredients and which are already in the basket;
   `dish_recipe()` gives the recipe if the user wants it.
4. `get_basket()` to see what is already there, so you don't add a second pack of something.
5. `add_dish_to_basket()` with the dish id from step 1 and either all ingredients or the ones the
   user wants (skip what they have at home: "vi har løg og hvidløg").
6. `get_basket()` and show the new total in kr. Repeat for the next dish.
7. Send only when the user asks (see `weekly-basket`, steps 7 to 9).

**Pitfalls:** dish ids come from `find_dishes()`, never from the dish name; ingredients may map to
øko or non-øko products, so say which if the user cares; quantities are for the dish's servings.
