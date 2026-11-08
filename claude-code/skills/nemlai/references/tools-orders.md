# Orders

NemlAI predicts baskets from the household's order history. The history is fetched from Nemlig
with the login NemlAI stores per member (refreshed nightly and on demand). Logging in to Nemlig
again takes the Nemlig password, so that only ever happens in the web app.

### order_fetch_status
When: the user asks whether their orders are up to date, a basket looks stale, or a refresh failed
with `reauth_required`.
Arguments: none.
Returns: `members[]` with `alias` (the member's user id), `display_name`, `is_self`,
`session_valid`, `last_fetch_success_at`, `failed`, `reason_category` (`fetch_error` |
`no_valid_session` | null) and `needs_reauth`; `any_failed`; `reauth_url`.
Next: `refresh_orders` to fetch now; for `needs_reauth`, give the user `reauth_url`.
Pitfalls: a member with no session row is not a failure (automatic fetching just hasn't run for
them). `account_email_masked` is masked on purpose; don't ask the user for the full email.
Fallback: CLI `nemlai --json orders status`; web https://nemlai.milops.org/refresh-orders (Open the page; each member's last fetch and any login problem are shown.)

### refresh_orders
When: the user says they ordered recently ("jeg har lige handlet") and wants the history or the next
basket to include it.
Arguments: `member` only when the user names another household member: a `user_id` from
`household_info` (not a name). `wait_seconds` (default 20, max 50): a refresh usually takes 20 s to
2 minutes.
Returns: a job. Completed: `result.orders_found`, `orders_new`, `orders_uploaded`,
`orders_failed`, `stored_order_total`, `steps`, `member`. Still running: `job_id` for `job_status`.
`already_running: true` when a refresh was already in progress (you get that job, not a new one).
Next: `job_status` while running; `generate_basket` afterwards if the user wants a fresh basket.
Pitfalls: `reauth_required` means the stored Nemlig login expired: send the user to the link in the
hint and never ask for the Nemlig password in chat. At most 5 refreshes a minute per household
(`rate_limited`). It fetches at most 50 new orders per run; run it again if `orders_new` was 50.
Fallback: CLI `nemlai --json orders refresh [--member <user_id>]`; web https://nemlai.milops.org/refresh-orders (Open the page (log in if asked). Click 'Opdater' and wait until it says the orders are uploaded. If it asks you to log in to Nemlig again, the user does that on the page (never in chat).)
