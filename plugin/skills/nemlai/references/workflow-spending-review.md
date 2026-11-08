# Spending review

Answer questions about what the household spends on groceries: totals, trends, top products, single orders and receipts.

**User says:** "hvad bruger vi på mad?", "how much did we spend on milk this year?", "what went up?".

1. Start broad: `analytics_overview()` for the period the user names (this month, last 3 months,
   12 months).
2. Pick the report that answers the question:
   - "how much on X / in category Y": `analytics_spend()` (a product or category, optionally compared
     with the previous period);
   - "what do we buy most / spend most on": `analytics_top()`;
   - "is it going up": `analytics_trend()` by week or month;
   - "what was in order 42", "why was that order so big": `analytics_drill()`;
   - "show the receipts for jordbær": `analytics_receipts()`;
   - "which categories are there": `analytics_categories_tree()`.
3. Report numbers as the tool returns them, in kr, with the period. Say how many orders the period
   covers; few orders make trends unreliable.
4. Offer a follow-up that acts on it: exclude an unusual order (`exclude-products`), trim the next
   basket (`trim-to-budget`).

**Pitfalls:** product names are Danish ("mælk", "jordbær", "øko"); search in Danish; excluded
orders are left out of analytics; spending includes linked shops' receipts (Bilka, Føtex) when they
are linked; don't do your own sums across reports.
