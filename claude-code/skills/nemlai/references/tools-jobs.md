# Jobs

Generating a basket, sending it to a shop, refreshing orders and refreshing a shop take from a few
seconds to a few minutes, so they run as background jobs. The tool that starts one waits up to
`wait_seconds` (default 20, max 50) and then returns either the finished job or
`{job_id, status: "running"}`. `job_status` follows it up.

### job_status
When: a start tool (`generate_basket`, `send_basket`, `refresh_orders`, `refresh_shop`, `find_dishes`) returned
`status: "running"`, or the user asks whether a basket send or refresh has finished.
Arguments: `job_id` exactly as the start tool returned it (`"send:…"`, `"generate:…"`,
`"order_refresh:…"`, `"mcp:…"`); a bare id from the web app also works. `wait_seconds` (default 20,
max 50, 0 = answer at once) waits server-side, so poll with the default instead of calling in a
tight loop.
Returns: `status` (`pending` | `running` | `completed` | `failed`), `state` (the job's own status,
e.g. `ready`, `session_expired`), `kind` (the tool that started it), `message` (often Danish, e.g.
"Bygger kurv..."), `started_at`, `elapsed_seconds`, and when completed a `result`: for a send
`cart_url`, `item_count`, `basket_total_kr`; for a generate `item_count`, `total_kr`; for an order
refresh `orders_found`, `orders_new`, `orders_uploaded`, `steps`. A failed job comes back as an
error with the job's code (`reauth_required`, `not_linked`, `upstream_unavailable`, …).
Next: after a completed send, give the user `cart_url` (they check out themselves; a failed send
has no link); after a generate,
`get_basket`; after an order refresh, `generate_basket` if the user wants a new basket; after a
shop refresh, `list_shops`.
Pitfalls: never start the same job again while it runs, and never re-send a basket because a
status call timed out: poll `job_status` first. Jobs live in server memory for about an hour and
are lost on a server restart: `not_found` then lists your household's recent jobs; don't guess an
id. Another household's job is reported as `not_found`. `reauth_required` means the stored Nemlig
login expired: send the user to the web page in the hint, and never ask for a password in chat.
Fallback: CLI `nemlai --json jobs status <job_id>`
