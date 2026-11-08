# Shop not linked

A shop the user wants to send to or count receipts from isn't linked yet; linking takes the shop password, so the user does it in the web app.

**User says:** "send it to Bilka", "include my Føtex receipts", or a tool returned `not_linked`.

1. `list_shops()`: which shops are linked, their status, and which can receive a basket.
2. Not linked:
   - Don't ask for the shop's email or password, and don't accept them in chat.
   - Give the user the link (from the error, or `account_links()`):
     `https://nemlai.milops.org/settings/shops`. There they log in with their Salling profile
     ("Salling Profil") to link Bilka, Føtex and BilkaToGo.
   - A user with a terminal can run `nemlai shops link --email <their email> --shops bilka,foetex`
     themselves; it asks for the password without echoing it.
3. When they say it's done: `list_shops()` again. A newly linked shop may need
   `refresh_shop()` (a job; poll `job_status()`) before its receipts or prices show up.
4. Carry on with what the user wanted (`compare-shops`, `spending-review`).

**Pitfalls:** a shop can be linked but in an error state (expired login): `list_shops()` shows it,
and the fix is the same web page; price-only shops (Netto, REMA 1000) can't be linked or sent to.
