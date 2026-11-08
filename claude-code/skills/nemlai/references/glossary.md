# Glossary

The Danish words users, products and the web app use, and how to read campaign texts.

## Words

| Danish | Meaning | Notes for agents |
|---|---|---|
| **kurv** / indkøbskurv | basket, shopping cart | "min kurv" = the user's saved NemlAI basket (`get_basket()`, the web page `/kurv/`; per user, not per household); the shop's own cart is filled by `send_basket()`. |
| **tilbud** | offer, special price | A campaign price; see the formats below. "på tilbud" = on sale. |
| **kampagne** | campaign | Same as tilbud in product data (`campaign_*` fields). |
| **øko** / økologisk | organic | Part of the product, not a detail: øko and non-øko are different products. Keep the "ø". |
| **pant** | deposit on bottles and cans | Charged on top of drinks, refunded on return. Not part of the product price. |
| **stk** (stykker) | pieces, items | "3 stk" = 3 items. Quantities in the basket are counted in stk unless a product is sold by weight. |
| **kg**, **g**, **l**, **cl**, **ml** | units | Part of the product name and size ("Letmælk 1,5% 1 l"). A size, not a quantity. |
| **kr** (kroner, DKK) | Danish currency | Decimal comma: `79,00 kr` = 79.00 kr. |
| **pr. kg** / **kr/kg**, **pr. l** | price per kilo / litre | The comparison price; use it to compare sizes. |
| **vare**, **varer** | product(s), item(s) | |
| **ret**, **retter**, **aftensmad** | dish(es), dinner | `find_dishes()`. |
| **opskrift** | recipe | `dish_recipe()`. |
| **husstand** | household | Settings page section "Husstand". |
| **medlem** / "andre i husstanden" | member / "others in the household" | |
| **ordre**, **ordrer** | order(s) | Past purchases; `refresh_orders()` fetches new ones. |
| **levering**, **leveringsdato** | delivery, delivery date | Picked in the shop's checkout, not in NemlAI. |
| **budget** | budget | Basket presets `small` / `medium` / `large` (kr limits shown by the tools). |
| **ekskluder** / **skjul** | exclude / hide | Leave an order or a product out of predictions (`update_exclusions()`). |
| **indstillinger** | settings | `/settings` in the web app. |
| **butik**, **butikker** | shop(s) | `list_shops()`. |
| **tilknyt butik** | link a shop | Web app only (takes the shop password). |
| **adgangskode** | password | Never ask for one in chat. |
| **send til nemlig** | send to Nemlig | The web app's send button: fills the Nemlig cart. |
| **mejeri**, **frugt & grønt**, **kød**, **brød**, **frost**, **drikkevarer** | dairy, fruit & veg, meat, bread, frozen, drinks | Category names; search in Danish. |

## Campaign text formats

Shops describe a campaign in text. NemlAI parses it into `campaign_min_quantity`,
`campaign_unit_price` and `regular_unit_price`; when the quantity is at least the minimum the
basket uses the campaign price and marks the item `on_sale`.

| Text | Means | Unit price | Minimum |
|---|---|---|---|
| `Mix 3 stk 79,00` | any 3 of the campaign's products for 79 kr together | 26,33 kr | 3 |
| `Ta' 2 for 50` | 2 for 50 kr | 25,00 kr | 2 |
| `3 for 57` | 3 for 57 kr | 19,00 kr | 3 |
| a single lower price, no count | a plain discount | the shown price | 1 |

"Mix" means the products can be mixed: 2 of one flavour and 1 of another still reach 3. Below the
minimum, each item costs `regular_unit_price`. If the text can't be parsed, NemlAI falls back to the
regular price; say "the campaign may not apply" rather than guessing.

## Shop names

| Id | Shop | Notes |
|---|---|---|
| `nemlig` | Nemlig.com | The main shop: orders come from here, the basket is sent here. |
| `bilkatogo` | BilkaToGo (shown as "Bilka") | Salling Group. Can receive the basket after linking. |
| `bilka`, `foetex`, `netto` | Bilka, Føtex, Netto (Salling Group) | Receipts can be linked for analytics; prices for comparison. |
| `rema1000` | REMA 1000 | Prices for comparison only. |

Use the ids `list_shops()` returns; they are the only valid values.
