---
name: nemlai
description: "Plan, adjust and send grocery baskets with NemlAI for Nemlig.com and other Danish shops (BilkaToGo, Netto, Føtex, REMA 1000): the weekly basket, dinners from dishes, trimming to a budget, comparing shop prices, reviewing grocery spending, excluding products and refreshing orders. Use when the user mentions NemlAI, Nemlig or nemlig.com, groceries, a shopping basket or cart, or in Danish: indkøb, dagligvarer, kurv, indkøbskurv, ugens indkøb, madplan, aftensmad, tilbud, øko, \"bestil varer\", \"lav en kurv\". Works through the NemlAI MCP tools, or the nemlai CLI or the web app when MCP is unavailable."
license: MIT
compatibility: "Best with the NemlAI MCP server (https://nemlai.milops.org/mcp). Without it: the nemlai CLI (pipx install nemlai, needs a shell and network) or a browser on nemlai.milops.org."
metadata:
  homepage: "https://nemlai.milops.org/agents"
  mcp-url: "https://nemlai.milops.org/mcp"
---

# NemlAI

<!-- Generated from docs/agent/ by scripts/build_agent_docs.py; edit those files, not this one. -->

NemlAI predicts a household's next grocery basket from its order history and fills a shop's cart; the user checks out themselves.

Use the NemlAI MCP tools when they are connected. When they are not (or MCP fails), use the
`nemlai` CLI or the web app: "Connect" below says how, and
[references/fallback.md](references/fallback.md) gives every tool's CLI command and web page.

## Connect

How an agent reaches NemlAI: the MCP server, the nemlai CLI when MCP is down, and the web pages.

### 1. The MCP server (first choice)

- **URL:** `https://nemlai.milops.org/mcp` (Streamable HTTP, stateless, JSON responses).
- **Sign in with OAuth** where the client supports it: add the URL with no client id/secret (the
  client registers itself). The client opens a browser: the user logs in to NemlAI if needed and
  clicks **Giv adgang** on the NemlAI consent page (`/oauth/consent`). The connection has full
  access to their NemlAI account; it can fill shop carts but never check out.
- **Or with an API token:** the user runs `nemlai login` once in their own terminal (it asks for
  their NemlAI password there, not in chat). The CLI stores a token starting with `nemlai_` in
  `~/.config/nemlai/token`; it is sent as `Authorization: Bearer <token>`. The configs below read
  it from the `NEMLAI_TOKEN` environment variable (`export NEMLAI_TOKEN=$(cat ~/.config/nemlai/token)`)
  instead of storing it.
- **`nemlai mcp install <client>`** (nemlai 0.11 or later) writes a client's config and installs
  this skill. It backs up the file first, changes nothing else in it, and `--dry-run` shows the
  change. `nemlai mcp config <client>` only prints the config; `nemlai mcp doctor` checks the URL,
  the token and the tool list.

#### Connect in one minute

- **claude.ai, Claude Desktop and mobile:** Customize → Connectors → Add custom connector → URL →
  Add → Connect → **Giv adgang**. Desktop and mobile use the connector added on claude.ai. For the
  skill, upload `nemlai-skill.zip` in Claude's Skills settings.
- **Claude Code:** `nemlai mcp install claude-code`. It runs
  `claude mcp add --transport http --scope user nemlai https://nemlai.milops.org/mcp --header 'Authorization: Bearer ${NEMLAI_TOKEN}'`
  and puts the skill in `~/.claude/skills/nemlai/`. Claude Code needs `NEMLAI_TOKEN` set when it
  starts; with a fixed `Authorization` header it does not fall back to OAuth. For OAuth instead:
  `claude mcp add --transport http nemlai https://nemlai.milops.org/mcp`, then `/mcp` → nemlai →
  Authenticate.
- **ChatGPT:** Settings → Apps & Connectors (developer mode) → add a custom MCP server → URL,
  authentication OAuth → Create → Connect.
- **Grok:** grok.com/connectors → New Connector → Custom → URL → complete the sign-in.
- **Codex:** `nemlai mcp install codex` adds this to `~/.codex/config.toml` and puts the skill in
  `~/.agents/skills/nemlai/`:

  ```toml
  [mcp_servers.nemlai]
  url = "https://nemlai.milops.org/mcp"
  bearer_token_env_var = "NEMLAI_TOKEN"
  ```

- **Hermes:** `nemlai mcp install hermes` adds this under `mcp_servers` in `~/.hermes/config.yaml`
  and puts the skill in `~/.hermes/skills/nemlai/`. Set `NEMLAI_TOKEN` in the environment or in
  `~/.hermes/.env`; in a running session, `/reload-mcp`.

  ```yaml
  mcp_servers:
    nemlai:
      url: "https://nemlai.milops.org/mcp"
      headers:
        Authorization: "Bearer ${NEMLAI_TOKEN}"
  ```

- **OpenClaw** (2026.5.12 or later): `nemlai mcp install openclaw` adds this to
  `~/.openclaw/openclaw.json` and puts the skill in `~/.openclaw/skills/nemlai/`. Set
  `NEMLAI_TOKEN` in the environment or in `~/.openclaw/.env`. If the file has JSON5 comments, run
  `openclaw mcp set nemlai '<the server object below>'` instead.

  ```json
  {"mcp": {"servers": {"nemlai": {"url": "https://nemlai.milops.org/mcp", "transport": "streamable-http",
    "headers": {"Authorization": "Bearer ${NEMLAI_TOKEN}"}}}}}
  ```

- **Other clients:** `nemlai mcp config generic` prints the URL, the header and a JSON entry.
- If the connector can't sign in (the client's OAuth fails), use the token where the client takes
  a header, else the CLI or the web app.
- Check it works: `whoami()`. Not sure what to do: `nemlai_guide()`.

### 2. The nemlai CLI (when MCP fails and you have a shell)

```bash
pipx install nemlai            # or: pip install nemlai
export NEMLAI_TOKEN=nemlai_…   # the user's token; or the user runs `nemlai login` themselves
nemlai --json whoami           # one command, JSON output, then exit
nemlai --json basket items
nemlai help                    # every command
```

- Every MCP tool names its CLI command (see the `fallback` topic, or `error.fallback.cli`). Run
  that command as written: the global `--json` makes every command print one JSON object
  (`status` plus the data, or `status` + `message` for a text reply, or `status` + `error` + `hint`).
  With nemlai older than 0.11 (`nemlai --version`), add `--format json` to analytics commands.
- The CLI reads the token from `~/.config/nemlai/token`, else from `NEMLAI_TOKEN`. Don't pass
  `--token` on the command line (it shows up in process lists).
- Never type a password into the CLI for the user. Commands that need one (`login`, `shops link`,
  a Nemlig reauth) are for the user to run in their own terminal, or to do in the web app.
- Python: `from nemlai import NemlAI; client = NemlAI(token=…)` gives the same features as methods
  (`client.basket.items()`, `client.basket.send()`, …).

### 3. The web app (no shell, or the CLI fails too)

An agent with a browser tool opens the page and follows the steps listed for the tool in the
`fallback` topic; an agent without one gives the user the link and the steps. The user logs in
themselves.

| Page | What is there |
|---|---|
| `https://nemlai.milops.org/kurv/` | The basket: generate, review, adjust, search, dishes, shop tabs with prices, send. The same saved basket as MCP and the CLI |
| `https://nemlai.milops.org/settings` | Profile, household and invite code, settings, prediction settings, preferences, notifications, password, delete account |
| `https://nemlai.milops.org/settings/shops` | Link shops (Salling: Bilka, Føtex, BilkaToGo) and their status |
| `https://nemlai.milops.org/refresh-orders` | Fetch new orders; log in to Nemlig again when the session expired |
| `https://nemlai.milops.org/ekskluder` | Exclude orders or products from predictions |
| `https://nemlai.milops.org/analytics` | Spending: overview, trends, top products, receipts |
| `https://nemlai.milops.org/cli` | A CLI in the browser (same commands as `nemlai`) |
| `https://nemlai.milops.org/cli/docs` | CLI, API and SDK documentation |
| `https://nemlai.milops.org/agents` | This guide for agents, with the full fallback table |
| `https://nemlai.milops.org/healthz` | Is the server up? |

### 4. Everything down

Say so plainly and don't invent results. Check `https://nemlai.milops.org/healthz`. The user can
still shop directly on `https://www.nemlig.com`.

## Start here

### What NemlAI is

- A household shops online, mostly at **Nemlig.com**. NemlAI fetches the household's past orders,
  learns what it buys and how often, and predicts the next basket: which products and how many.
- The user (or you, the agent) reviews the basket, adjusts it, and **sends** it to a shop. Sending
  fills the shop's own cart. The user then opens the shop's site and checks out. **NemlAI never
  checks out or pays.**
- The same basket can be priced at other shops (BilkaToGo, Netto, Føtex, REMA 1000) and sent to the
  ones that accept a basket.

### Data model

| Thing | What it is | Tools |
|---|---|---|
| **User** | The NemlAI account this connection acts for. | `whoami()` |
| **Household** | The unit orders, settings, preferences and linked shops belong to. One user is in one household. | `household_info()` |
| **Member** | A person in the household, each with their own Nemlig account. Orders from all members count together. | `household_info()` |
| **Basket** (kurv) | The user's current basket (each member has their own, predicted from the whole household's orders): items with quantity, unit price, campaign info, a total in kr and a budget. It is saved on the server (it survives restarts) and shared by MCP, the CLI, the SDK and the web page `/kurv/`. | `get_basket()`, `generate_basket()` |
| **Item** | One product line. Identified by its SKU (`product_number`, a Nemlig product id) from an earlier tool result. | `explain_item()`, `set_quantity()` |
| **Shop** | Where the basket can be priced or sent: `nemlig`, `bilkatogo`, and price-only shops. Some need linking first (web app, takes a password). | `list_shops()`, `compare_shop_prices()` |
| **Order** | A past purchase fetched from a shop. The model learns from them; stale orders mean a stale basket. | `refresh_orders()`, `order_fetch_status()` |
| **Dish** | A meal (e.g. "Fisk en hverdag") with ingredients you can add to the basket. | `find_dishes()`, `add_dish_to_basket()` |
| **Job** | A long action (generate, send, order or shop refresh) running on the server. | `job_status()` |

**Scope:** household scope is the default for orders, analytics and settings. Use one member's
scope only when the user names that person ("refresh Anna's orders"). The basket is the
exception: it belongs to the user this connection acts for, so another member doesn't see it; the
user sees it on the web page `/kurv/`.

### The core loop

1. **Check the connection:** `whoami()`. No household → the user joins or creates one in the web
   app first.
2. **Look at the basket:** `get_basket()`. If it is empty or old, `generate_basket()` with a budget
   preset (`small`, `medium`, `large`). It returns when done or gives a `job_id` for `job_status()`.
3. **Review with the user:** show the items, quantities and the total in kr against the budget.
   `explain_item()` says why an item is there.
4. **Adjust:** `add_to_basket()` (by SKU from `search_products()`), `set_quantity()`,
   `remove_from_basket()`, `add_dish_to_basket()` for a meal.
5. **Optionally compare shops:** `compare_shop_prices()`.
6. **Send** with `send_basket()` only after the user says so. Tell them to check out on the shop's
   site.

### How results look

Every tool returns an envelope:

- Success: `{"ok": true, "data": {…}, "next": ["…"]}`. `next` lists what makes sense now; follow it.
- Failure: `{"ok": false, "error": {"code", "message", "hint", "fix": {"tool", "args"}, "fallback":
  {"cli", "web"}}}`. Do `fix` if present; otherwise see the `fallback` topic.

Error codes: `invalid_argument` (fix the call), `not_found` (use an id from a tool result; the hint
lists candidates), `forbidden`, `no_household`, `not_linked` (web app link), `reauth_required` (the
user logs in again in the web app), `job_running` (poll the `job_id` in `fix`), `upstream_unavailable`
(retry once, then fall back), `rate_limited` (wait as the hint says), `conflict` (the basket changed
elsewhere, e.g. on `/kurv/`: `get_basket()` again, then redo the change), `internal` (fall back).

### Where to go next

- The rules: `nemlai_guide(topic="rules")`.
- A whole task: the workflow topics (`weekly-basket`, `dinners-via-dishes`, `trim-to-budget`,
  `compare-shops`, `spending-review`, `exclude-products`, `order-refresh-reauth`, `shop-not-linked`).
- One tool: `nemlai_guide(topic="<tool name>")`.
- When something fails: `nemlai_guide(topic="fallback")`.

## Rules

### Sending and paying

1. **Show before you send.** Before `send_basket()`, show the user the items (name, quantity, price)
   and the total in kr, and the budget if there is one. Send only when the user clearly says so
   ("send it", "ja, send"). A request to "make a basket" is not a request to send it.
2. **Send once.** `send_basket()` is a job. If it times out, fails or the result is unclear, check
   `job_status()` with its `job_id`, or `get_basket()`, before anything else. A second send can
   double the shop's cart.
3. **No checkout, ever.** `send_basket()` fills the shop's cart. The user opens the shop's site,
   checks the cart and pays. Say that when you send. Never claim an order was placed.

### Prices and campaigns

4. **Prices are in Danish kroner (kr).** Show them as the tools return them (e.g. `79,00 kr` or
   `79.00 kr`); don't convert currencies.
5. **Campaign minimums.** A campaign price applies only when the quantity reaches
   `campaign_min_quantity`. "Mix 3 stk 79,00" means 3 items for 79 kr together (26,33 kr each); with
   2 items the regular `regular_unit_price` applies. When a user is 1 item short of a campaign, you
   may point it out; don't raise quantities on your own.
6. **Pant** (bottle and can deposit) is a separate charge on drinks, refunded when the empties are
   returned. Don't fold it into the product price or count it as spending on the product.

### Danish text

7. **Keep Danish text as it is.** Product names, categories and dishes are Danish. Keep "øko"
   (organic) and the letters æ, ø and å intact in names, searches and messages. Never transliterate
   ("oeko", "aeble"): search with the real spelling ("æble", "økologisk").
8. **Øko is a real difference.** An øko product and its non-øko version are different products.
   Don't swap one for the other without saying so.

### Scope and identity

9. **Household scope is the default.** Orders, analytics and settings belong to the household.
   Use one member's scope only when the user names a person ("Annas ordrer"). The basket is per
   user: one saved basket that MCP, the CLI, the SDK and the web page `/kurv/` share; it is not
   the household's.
10. **Never take ids from the user's head.** SKUs, dish ids, job ids and order ids come from earlier
    tool results. If an id is unknown, the error lists candidates: pick from those or search again.
11. **Never ask for a password.** Not for NemlAI, Nemlig, Salling/Bilka or any shop, at any step.
    Logins, linking a shop, Nemlig reauth, password changes and deleting the account happen in the
    web app; give the user the link (`account_links()`) and let them do it.

### Honesty

12. **Never invent results.** If a tool fails and the fallbacks fail, say so plainly. The user can
    still shop directly on nemlig.com.
13. **Say which way you used.** If you fell back to the CLI or the web app, tell the user ("MCP is
    down, so I used the nemlai CLI").
14. **Follow `next` and `fix`.** Tool results say what state things are in and what to call next.
    Call `nemlai_guide()` when unsure instead of guessing.

## Workflows

Read the one that matches the task before starting it:

- [Weekly basket](references/workflow-weekly-basket.md): Build the household's usual weekly basket, review it with the user and send it to Nemlig on their go-ahead.
- [Dinners via dishes](references/workflow-dinners-via-dishes.md): Plan the week's dinners from NemlAI's dishes and add their ingredients to the basket.
- [Trim to budget](references/workflow-trim-to-budget.md): Bring a basket down to a budget in kr by removing or reducing the least needed items, with the user's approval.
- [Compare shops](references/workflow-compare-shops.md): Price the basket at other shops, pick alternatives where a shop lacks a product, and send it to a shop that accepts a basket.
- [Spending review](references/workflow-spending-review.md): Answer questions about what the household spends on groceries: totals, trends, top products, single orders and receipts.
- [Exclude products](references/workflow-exclude-products.md): Stop products or whole orders from shaping the predictions (a one-off party order, a product the household no longer buys).
- [Order refresh needs reauth](references/workflow-order-refresh-reauth.md): Fetch new orders from Nemlig; when the stored Nemlig session has expired, hand the login to the user in the web app.
- [Shop not linked](references/workflow-shop-not-linked.md): A shop the user wants to send to or count receipts from isn't linked yet; linking takes the shop password, so the user does it in the web app.

## Tools

Each area file has every tool's arguments, results, pitfalls and fallback.

- [Account, household and feedback](references/tools-account.md): `update_profile`, `export_my_data`, `account_links`, `achievements`, `send_feedback`, `household_invite_code`, `whoami`
- [Household](references/tools-household.md): `household_info`
- [Orders](references/tools-orders.md): `order_fetch_status`, `refresh_orders`
- [Basket](references/tools-basket.md): `generate_basket`, `get_basket`, `explain_item`, `add_to_basket`, `set_quantity`, `remove_from_basket`, `clear_basket`
- [Search](references/tools-search.md): `search_products`, `product_info`, `list_categories`
- [Dishes](references/tools-dishes.md): `find_dishes`, `dish_info`, `dish_recipe`, `add_dish_to_basket`
- [Shops](references/tools-shops.md): `list_shops`, `refresh_shop`, `compare_shop_prices`, `list_alternatives`, `send_basket`
- [Analytics tools](references/tools-analytics.md): `analytics_overview`, `analytics_spend`, `analytics_top`, `analytics_trend`, `analytics_drill`, `analytics_receipts`, `analytics_categories_tree`
- [Settings, preferences and notifications](references/tools-settings.md): `get_settings`, `update_settings`, `get_preferences`, `update_preferences`, `preference_definitions`, `recalculate_preferences`, `notifications_status`, `send_test_notification`
- [Exclusions](references/tools-exclusions.md): `get_exclusions`, `list_order_items`, `update_exclusions`
- [Jobs](references/tools-jobs.md): `job_status`
- [Guide tools](references/tools-guide.md): `nemlai_guide`

## More references

- [references/glossary.md](references/glossary.md): Danish words (kurv, tilbud, øko, pant, stk/kg) and campaign texts.
- [references/fallback.md](references/fallback.md): the fallback ladder and every tool's CLI command and web page.
