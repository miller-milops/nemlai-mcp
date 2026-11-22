# Account, household and feedback

### update_profile
When: "change my name". The Nemlig email can't be changed anywhere; if it changed at Nemlig, the user tells us with `send_feedback`.
Arguments: `display_name` (1-64 characters).
Returns: the new profile and `changed`.
Pitfalls: there is no password argument and there never will be.
Fallback: CLI `nemlai --json profile update --name <name>`; web https://nemlai.milops.org/settings (Open the page. Edit the 'Profil' section. Click 'Gem ændringer'.)

### export_my_data
When: "give me my data" (GDPR).
Arguments: `section` (profile | orders | order_items | preferences | suggestions | saved_basket), `limit` (default 100, max 500), `cursor`. `order_items` are the products in each order; `suggestions` lists when NemlAI suggested a basket; `saved_basket` is the caller's saved working basket (as on /kurv/).
Returns: without `section`, a summary (counts, first/last order date, bytes per section); with it, that part paged with `next_cursor`.
Next: read sections one by one. The whole file downloads at `/auth/export` in the browser.
Pitfalls: the full export can be megabytes (up to 5000 order items), so don't page through everything unless asked.
Fallback: CLI `nemlai --json account export [--section <profile|orders|order_items|preferences|suggestions|saved_basket>]`; web https://nemlai.milops.org/auth/export (Open the URL while logged in; the browser downloads a JSON file.)

### account_links
When: the user wants something an agent must not do: signup, login, change_password, reset_nemlig_password, link_shop (Bilka/Føtex), reauth_nemlig (expired Nemlig session), delete_account, connected_apps (see or remove connected agents and API tokens).
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
When: the user asks to send feedback to the NemlAI team, or you (the agent) hit a problem or see a better way: a tool that failed or confused you, a wrong result, a missing tool or argument, a flow that took too many steps. Sending your own report is encouraged; it is how NemlAI improves.
Arguments: `message` (max 4000 characters). The user's feedback: their own words. Your own: what you tried (tool and arguments), what happened, what you expected.
Returns: `feedback_id`.
Next: tell the user it was sent (your own report: what it was about, in a line).
Pitfalls: people on the NemlAI team read it (quoted verbatim, with an opaque reference instead of the user's name or household), so leave out personal details the report doesn't need. One message per problem, not one per failed call. 5 per 10 minutes; the same text twice in 5 minutes is `rate_limited`.
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
