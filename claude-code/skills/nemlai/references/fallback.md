# Fallback

What to do when a tool or the whole MCP server fails: the ladder, its rules, and every tool's CLI command and web page.

## The ladder

Go one step down only when the step above fails.

1. **The tool returned an error with a `fix`:** do the `fix` call. Retry a timeout,
   `upstream_unavailable` or a 5xx **once** after a few seconds. For a job, poll `job_status()`
   with its `job_id`; don't start the job again. An HTTP 502/503 from the proxy (not a NemlAI
   error envelope) usually means a deploy is restarting the server: wait about 10 seconds and retry
   once. A job the server no longer knows after that (`not_found`) was lost in the restart: start it
   once more, except `send_basket()` (check the basket and the shop's cart first).
2. **MCP unreachable, auth failing, or a tool missing → the CLI** (agents with a shell):
   `pipx install nemlai`, then run the tool's command from the table below (or the error's
   `fallback.cli`) as written, with `NEMLAI_TOKEN` set. They are `nemlai --json <command>`.
3. **No shell, or the CLI fails too → the web app:** with a browser tool, open the page from the
   table (or `fallback.web.url`) and follow its steps; without one, give the user the link and the
   steps. The user logs in themselves.
4. **Everything down:** say so plainly, check `https://nemlai.milops.org/healthz`, never invent
   results. The user can still shop directly on nemlig.com.

## Rules on every step

- **Never send a basket twice blindly.** After a failed or unclear `send_basket()`, check
  `job_status()` or the basket (`get_basket()`, `nemlai --json basket items` or `/kurv/`) first.
  A second send can double the shop's cart.
- **The web page shows the same basket.** MCP, the CLI and the web page `/kurv/` share one saved
  basket per user, so falling back to the web for a basket action means opening `/kurv/`: the
  items you added are there. A change made on the page while you work makes your next write fail
  with `conflict`: read the basket again (`get_basket()`) before you change it.
- **Never ask the user to paste a password into chat**, whichever step you are on. Logins, shop
  linking and Nemlig reauth happen in the web app (or in the user's own terminal).
- **Tell the user which step you used** ("MCP is down, so I used the nemlai CLI").
- The same rules as with MCP apply: show the basket and total before sending, send only on the
  user's go-ahead, no checkout.

## Per tool

The table is generated from `nemligbot/mcp/coverage.py` (`TOOLS`): `CLI` is the command to run
with `NEMLAI_TOKEN` set; `Web` is the page and the steps there. Placeholders in `<…>` come from
earlier results (SKUs, ids) or from the user.

| Tool | CLI (`NEMLAI_TOKEN` set) | Web page and steps |
|---|---|---|
| `update_profile` | `nemlai --json profile update --name <name>` | https://nemlai.milops.org/settings: Open the page. Edit the 'Profil' section. Click 'Gem ændringer'. |
| `export_my_data` | `nemlai --json account export [--section <profile\|orders\|order_items\|preferences\|suggestions>]` | https://nemlai.milops.org/auth/export: Open the URL while logged in; the browser downloads a JSON file. |
| `account_links` | none | https://nemlai.milops.org/settings: Open the page: password, account deletion and shop linking live here. |
| `achievements` | `nemlai --json achievements` | https://nemlai.milops.org/achievements |
| `send_feedback` | `nemlai --json feedback --message "<text>"` | https://nemlai.milops.org/settings: Open the page. Click 'Giv feedback' in the top navigation. Write your message and send it. |
| `household_invite_code` | `nemlai --json household invite-code` | https://nemlai.milops.org/settings: Open the page. Copy 'Invitationskode' in the 'Husstand' section. |
| `whoami` | `nemlai --json whoami` | https://nemlai.milops.org/settings: Open the page (log in if asked). Read the 'Profil' section. |
| `household_info` | `nemlai --json household info` | https://nemlai.milops.org/settings: Open the page (log in if asked). Read the 'Husstand' section: Husstands-ID and Invitationskode. Members are listed on /refresh-orders under 'Andre i husstanden'. |
| `order_fetch_status` | `nemlai --json orders status` | https://nemlai.milops.org/refresh-orders: Open the page; each member's last fetch and any login problem are shown. |
| `refresh_orders` | `nemlai --json orders refresh [--member <user_id>]` | https://nemlai.milops.org/refresh-orders: Open the page (log in if asked). Click 'Opdater' and wait until it says the orders are uploaded. If it asks you to log in to Nemlig again, the user does that on the page (never in chat). |
| `generate_basket` | `nemlai --json basket generate --size <small\|medium\|large> [--budget <kr>]` | https://nemlai.milops.org/kurv/: Open the page (log in if asked): it shows the same basket as MCP and the CLI. If a basket is shown, click 'Ny kurv'. Pick the size and a delivery date and generate the basket; it replaces the saved one. |
| `get_basket` | `nemlai --json basket items` | https://nemlai.milops.org/kurv/: Open the page (log in if asked): it shows the same basket as MCP and the CLI. Read the items and the total ('Anslået beløb'); an empty basket opens on the generate form. |
| `explain_item` | `nemlai --json basket why <sku>` | https://nemlai.milops.org/kurv/: Open the page (log in if asked): it shows the same basket as MCP and the CLI. Tap the (i) next to the item. |
| `add_to_basket` | `nemlai --json basket add --product-number <sku> --quantity <n>` | https://nemlai.milops.org/kurv/: Open the page (log in if asked): it shows the same basket as MCP and the CLI. Type the product in 'Tilføj en vare eller søgning…' and click 'Tilføj'. |
| `set_quantity` | `nemlai --json basket adjust --product-number <sku> --quantity <n>` | https://nemlai.milops.org/kurv/: Open the page (log in if asked): it shows the same basket as MCP and the CLI. Change the item's quantity with − and +; it is saved at once. |
| `remove_from_basket` | `nemlai --json basket remove --product-number <sku>` | https://nemlai.milops.org/kurv/: Open the page (log in if asked): it shows the same basket as MCP and the CLI. Press − at quantity 1, then the bin next to the item. |
| `clear_basket` | `nemlai --json basket clear` | https://nemlai.milops.org/kurv/: Open the page (log in if asked): it shows the same basket as MCP and the CLI. Remove the items (− at quantity 1, then the bin), or click 'Ny kurv' and generate a new basket, which replaces it. |
| `search_products` | `nemlai --json search <query> [--limit <n>]` | https://nemlai.milops.org/kurv/: Open the page. Use the search field. |
| `product_info` | `nemlai --json search info <sku>` | https://nemlai.milops.org/kurv/: Open the page. Search for the product and open it. |
| `list_categories` | `nemlai --json categories` | none |
| `find_dishes` | `nemlai --json dish <query>` | https://nemlai.milops.org/kurv/: Open the page. Pick a dish suggestion (e.g. '+ Fisk en hverdag'). |
| `dish_info` | `nemlai --json dish info <#>` | none |
| `dish_recipe` | `nemlai --json dish recipe` | none |
| `add_dish_to_basket` | `nemlai --json dish add all` | https://nemlai.milops.org/kurv/: Open the page (log in if asked): it shows the same basket as MCP and the CLI. Pick a dish and add its ingredients. |
| `list_shops` | `nemlai --json shops status` | https://nemlai.milops.org/settings/shops: Open the page; linked shops and each member's status are shown. Price shops are the tabs above the basket on /kurv/. |
| `refresh_shop` | `nemlai --json shops refresh <bilka\|foetex>` | https://nemlai.milops.org/settings/shops: Open the page. Click 'Hent kvitteringer' for the shop. |
| `compare_shop_prices` | `nemlai --json basket prices [--shop <shop_id>]` | https://nemlai.milops.org/kurv/: Open the page. Click the shop tabs above the basket (Bilka, Netto, …) to see each shop's total. |
| `list_alternatives` | `nemlai --json basket alternatives <item> --shop <shop_id> [--search <text>]` | https://nemlai.milops.org/kurv/: Open the page. Open the shop's tab. Click 'Vælg alternativ' on the item. |
| `list_offers` | `nemlai --json offers [--all] [--shop <nemlig\|bilkatogo>] [--category <text>]` | none |
| `send_basket` | `nemlai --json basket send --keep\|--replace` | https://nemlai.milops.org/kurv/: Open the page (log in if asked): it shows the same basket as MCP and the CLI. Check the basket and the total with the user. Nemlig: click 'Send til nemlig'. BilkaToGo: open the 'Bilka' tab and send from there. If the shop's cart already holds items, the page asks: 'Behold dem og læg de nye oveni' or 'Tøm kurven og send kun de nye'. Let the user choose. Open the cart link and let the user check out themselves. |
| `get_shop_cart` | `nemlai --json shop cart [nemlig\|bilkatogo]` | none |
| `get_delivery_info` | `nemlai --json delivery <nemlig\|bilkatogo>` | none |
| `list_delivery_slots` | `nemlai --json delivery slots [nemlig\|bilkatogo] [--days <n>]` | none |
| `analytics_overview` | `nemlai --json analytics overview --last 3m` | https://nemlai.milops.org/analytics: Open the page. The Forbrug tab shows the total for the chosen period. Use the Top and Trend tabs for the ranking and the trend. |
| `analytics_spend` | `nemlai --json analytics spend --last 30d` | https://nemlai.milops.org/analytics: Open the page. Stay on the Forbrug tab and set the period and the Produkt / Kategori filters. |
| `analytics_top` | `nemlai --json analytics top products --last 90d` | https://nemlai.milops.org/analytics: Open the page. Open the Top tab, pick Produkter or Kategorier and set the period. |
| `analytics_trend` | `nemlai --json analytics trend --bucket auto --last 12m` | https://nemlai.milops.org/analytics: Open the page. Open the Trend tab and set the period and filters. |
| `analytics_drill` | `nemlai --json analytics drill --product <name> --last 30d` | https://nemlai.milops.org/analytics: Open the page. Open the Detaljer tab and set the filters to see the order lines. |
| `analytics_receipts` | `nemlai --json analytics receipts --last 30d` | https://nemlai.milops.org/analytics: Open the page. Open the Kvitteringer tab and set the period and filters. |
| `analytics_categories_tree` | `nemlai --json analytics categories-tree --last 12m` | https://nemlai.milops.org/analytics: Open the page. Open the Kategori filter dropdown; it lists the category tree. |
| `get_settings` | `nemlai --json settings show` | https://nemlai.milops.org/settings: Open the page. Read 'Forudsigelsesindstillinger'. |
| `update_settings` | `nemlai --json settings update --cutoff <YYYY-MM-DD\|none> --auto-learn <true\|false>` | https://nemlai.milops.org/settings: Open the page. Change the setting. Click 'Gem indstillinger'. |
| `get_preferences` | `nemlai --json preferences show` | https://nemlai.milops.org/settings |
| `update_preferences` | `nemlai --json preferences update --dietary <d> --prefer-eco <true\|false> --notes <n>` | https://nemlai.milops.org/settings: Open the page. Edit the preferences. Save. |
| `preference_definitions` | `nemlai --json preferences definitions` | none |
| `recalculate_preferences` | `nemlai --json preferences recalculate` | none |
| `notifications_status` | `nemlai --json notifications status` | https://nemlai.milops.org/settings: Open the page. Read 'Notifikationer'. |
| `send_test_notification` | `nemlai --json notifications test` | https://nemlai.milops.org/settings: Open the page. Click 'Send test' under 'Notifikationer'. |
| `get_exclusions` | `nemlai --json exclusions list` | https://nemlai.milops.org/ekskluder: Open the page; excluded products and orders are listed. |
| `list_order_items` | `nemlai --json exclusions items <order_id>` | https://nemlai.milops.org/ekskluder: Open the page. Open the order to see its lines. |
| `update_exclusions` | `nemlai --json exclusions exclude\|include <order_id>  /  nemlai --json exclusions hide\|unhide <order_id> <product_number>` | https://nemlai.milops.org/ekskluder: Open the page. Exclude or include the product or order. |
| `job_status` | `nemlai --json jobs status <job_id>` | none |
| `nemlai_guide` | `nemlai --json help` | https://nemlai.milops.org/agents: Open the page: the agent guide, the workflows and every tool's CLI command and web page. |
