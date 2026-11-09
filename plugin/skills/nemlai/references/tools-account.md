# Account, household and feedback

### update_profile
When: "change my name". A changed Nemlig email is not set here: the user changes it in the web app.
Arguments: `display_name` (1-64 characters).
Returns: the new profile and `changed`.
Pitfalls: there is no password argument and there never will be.
Fallback: CLI `nemlai --json profile update --name <name>`; web https://nemlai.milops.org/settings (Open the page. Edit the 'Profil' section. Click 'Gem ændringer'.)

### export_my_data
When: "give me my data" (GDPR).
Arguments: `section` (profile | orders | order_items | preferences | suggestions), `limit` (default 100, max 500), `cursor`. `order_items` are the products in each order; `suggestions` lists when NemlAI suggested a basket.
Returns: without `section`, a summary (counts, first/last order date, bytes per section); with it, that part paged with `next_cursor`.
Next: read sections one by one. The whole file downloads at `/auth/export` in the browser.
Pitfalls: the full export can be megabytes (up to 5000 order items), so don't page through everything unless asked.
Fallback: CLI `nemlai --json account export [--section <profile|orders|order_items|preferences|suggestions>]`; web https://nemlai.milops.org/auth/export (Open the URL while logged in; the browser downloads a JSON file.)

### account_links
When: the user wants something an agent must not do: signup, login, change_password, reset_nemlig_password, link_shop (Bilka/Føtex), reauth_nemlig (expired Nemlig session), delete_account.
Arguments: `topic` (optional; all topics when omitted).
Returns: per topic the `url`, short `steps`, `why_web_only`, and the CLI commands or routes it replaces.
Next: give the user the link and steps.
Pitfalls: never ask the user to paste a password into chat. The list is built from `coverage.py`, so it matches what is excluded.
Fallback: web https://nemlai.milops.org/settings (Open the page: password, account deletion and shop linking live here.)

### achievements
When: "what have I unlocked?".
Arguments: none.
Returns: `unlocked` (id, name, description, `unlocked_at`), `locked`, counts.
Next: none.
Pitfalls: achievements belong to the household, not the member. `total_count` is null if the catalogue was unavailable.
Fallback: CLI `nemlai --json achievements`; web https://nemlai.milops.org/achievements

### send_feedback
When: only when the user asks to send feedback to the NemlAI team.
Arguments: `message` (the user's own words, max 4000 characters).
Returns: `feedback_id`.
Next: tell the user it was sent.
Pitfalls: people on the NemlAI team read it (quoted verbatim, with an opaque reference instead of the user's name or household). 5 per 10 minutes; the same text twice in 5 minutes is `rate_limited`. Don't send on your own initiative.
Fallback: CLI `nemlai --json feedback --message "<text>"`; web https://nemlai.milops.org/settings (Open the page. Click 'Giv feedback' in the top navigation. Write your message and send it.)

### household_invite_code
When: "how do I invite my partner?".
Arguments: none.
Returns: `invite_code`.
Next: the new member enters it during signup (`account_links` topic `signup`).
Pitfalls: anyone with the code can join the household and see its orders; show it only to the user who asked.
Fallback: CLI `nemlai --json household invite-code`; web https://nemlai.milops.org/settings (Open the page. Copy 'Invitationskode' in the 'Husstand' section.)

### whoami
Available; documentation coming.
Fallback: CLI `nemlai --json whoami`; web https://nemlai.milops.org/settings (Open the page (log in if asked). Read the 'Profil' section.)
