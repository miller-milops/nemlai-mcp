# Settings, preferences and notifications

### get_settings
When: the user asks how predictions are configured, or before changing a setting.
Arguments: none.
Returns: `order_cutoff_date` (YYYY-MM-DD or null), `auto_learn_from_purchases` (true/false/null), `definitions` (what each accepts), `configured` (false until the household has saved any preference).
Next: `update_settings`; `get_preferences` for taste and diet; `notifications_status`.
Pitfalls: these are the only two prediction settings. There is no separate "prediction settings" tool. Diet, eco and brand taste live in `get_preferences`.
Fallback: CLI `nemlai --json settings show`; web https://nemlai.milops.org/settings (Open the page. Read 'Forudsigelsesindstillinger'.)

### update_settings
When: "ignore orders before March", "turn off auto-learn".
Arguments: `order_cutoff_date` as `YYYY-MM-DD`, or the string `none` to clear; `auto_learn_from_purchases` true/false. Pass only what changes: the other setting keeps its value.
Returns: the full new settings and `changed`.
Next: `generate_basket` to see the effect.
Pitfalls: a cutoff silently drops older orders from predictions, so confirm the date with the user. An unknown argument or a bad date returns `invalid_argument`.
Fallback: CLI `nemlai --json settings update --cutoff <YYYY-MM-DD|none> --auto-learn <true|false>`; web https://nemlai.milops.org/settings (Open the page. Change the setting. Click 'Gem indstillinger'.)

### get_preferences
When: "what does NemlAI think I like?", before `update_preferences`.
Arguments: none.
Returns: `calculated` (eco_ratio 0-1, avg_basket_dkk in kr, order_frequency_days, totals, top_categories, brand_affinity; derived from orders), `explicit` (dietary, prefer_eco, notes, exclude_categories, exclude_products, brand_preferences, custom), `settings` (cutoff, auto-learn).
Next: `update_preferences`; `recalculate_preferences` if the calculated numbers look stale.
Pitfalls: calculated values can't be edited, only recalculated. Excluded orders and hidden lines are not here: use `get_exclusions`.
Fallback: CLI `nemlai --json preferences show`; web https://nemlai.milops.org/settings

### update_preferences
When: "I'm vegetarian now", "never suggest liver paste", "I prefer Arla".
Arguments: scalars `dietary`, `prefer_eco`, `notes` replace; list the key in `clear` to remove it. Lists `exclude_categories` / `exclude_products` take `{add: [...], remove: [...]}` (remove is case-insensitive). `brand_preferences` and `custom` merge key by key; a null value deletes the key. Only pass what changes.
Returns: the full new preferences and `changed`.
Next: `generate_basket` to apply.
Pitfalls: never send a full list to "replace" it; use add and remove. Product names are Danish; keep æ, ø, å. To leave a past order out of predictions use `update_exclusions`; for the cutoff use `update_settings`.
Fallback: CLI `nemlai --json preferences update --dietary <d> --prefer-eco <true|false> --notes <n>`; web https://nemlai.milops.org/settings (Open the page. Edit the preferences. Save.)

### preference_definitions
When: the user asks what can be tuned, or you are unsure of a preference key.
Arguments: none.
Returns: `definitions` (id, name, description, category calculated|explicit, data_type, unit) and `editable_keys` (the keys `update_preferences` accepts, with their kind).
Next: `update_preferences`.
Pitfalls: not every definition is editable: calculated ones come from order history.
Fallback: CLI `nemlai --json preferences definitions`

### recalculate_preferences
When: after new orders were imported, or `get_preferences` shows `total_orders` 0 / stale numbers.
Arguments: none.
Returns: `affinity_products_updated`, the new `calculated` values, `elapsed_seconds`.
Next: `get_preferences`, `generate_basket`.
Pitfalls: takes a few seconds; limited to 5 calls a minute (`rate_limited`). It does not fetch new orders: use `refresh_orders` first.
Fallback: CLI `nemlai --json preferences recalculate`

### notifications_status
When: "do I get push notifications?", before `send_test_notification`.
Arguments: none.
Returns: `configured` (server can send), `subscription_count` (devices of the household), `enabled`.
Next: `send_test_notification` (only if asked). With zero devices the user subscribes in the web app (Indstillinger → Notifikationer).
Pitfalls: an agent cannot subscribe devices; that needs the browser.
Fallback: CLI `nemlai --json notifications status`; web https://nemlai.milops.org/settings (Open the page. Read 'Notifikationer'.)

### send_test_notification
When: only when the user asks to test notifications.
Arguments: none.
Returns: `sent` and `failed` device counts.
Next: ask whether it arrived; if `sent` is 0 check `notifications_status`.
Pitfalls: it sends a real push to every device in the household, not just the caller's. Limited to 5 a minute.
Fallback: CLI `nemlai --json notifications test`; web https://nemlai.milops.org/settings (Open the page. Click 'Send test' under 'Notifikationer'.)
