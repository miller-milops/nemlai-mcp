# Order refresh needs reauth

Fetch new orders from Nemlig; when the stored Nemlig session has expired, hand the login to the user in the web app.

**User says:** "hent mine nye ordrer", "the basket doesn't know about last week's order", or a tool
returned `reauth_required`.

1. `order_fetch_status()`: when each member's orders were last fetched.
2. `refresh_orders()` for the household (or one member, only if the user names them). It uses the
   stored Nemlig session only and never takes a password. It is a job: wait, or poll `job_status()`.
3. Done → tell the user how many new orders arrived; offer `generate_basket()` for a fresh basket.
4. **`reauth_required`** (the Nemlig session expired):
   - Don't ask for the Nemlig password, and don't accept it if the user pastes it.
   - Give the user the link from the error (or `account_links()`):
     `https://nemlai.milops.org/refresh-orders`. There they enter their Nemlig.com password
     ("Log ind på Nemlig.com"; "Andre i husstanden" for another member) and the refresh runs.
   - A user with a terminal can start the interactive CLI (`nemlai`) themselves and type
     `orders refresh`; it asks for the Nemlig password without echoing it. (`nemlai --json orders
     refresh`, the agent's CLI fallback, uses the stored session only and never asks.)
   - When they say it's done, check with `order_fetch_status()` and carry on.
5. `job_running`: a refresh is already running; poll the `job_id` in `fix` with `job_status()`.
   Never start a second one.

**Pitfalls:** never retry `refresh_orders()` in a loop on `reauth_required`; it won't fix itself.
