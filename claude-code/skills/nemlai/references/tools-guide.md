# Guide tools

The playbook itself, by topic.

### nemlai_guide
When: you are unsure which tool to use, what a Danish word means, how a whole task goes, or what to do when a tool or MCP fails. Call it first in a new conversation if the instructions are not enough.
Arguments: `topic` (default `start`): `start`, `rules`, `glossary`, `connect`, `fallback`, a workflow (`weekly-basket`, `dinners-via-dishes`, `trim-to-budget`, `compare-shops`, `spending-review`, `exclude-products`, `order-refresh-reauth`, `shop-not-linked`), a tool area (`basket`, `shops`, …), `tools` for the list of every tool, or a tool name (`send_basket`). An unknown topic returns the list of topics.
Returns: `topic`, `title`, `text` (Markdown), `related` topics, and for `fallback` or a tool name the CLI command and web page per tool.
Next: the tools the topic names.
Pitfalls: it is read-only and cheap; prefer it over guessing an argument or an id.
Fallback: CLI `nemlai --json help`; web https://nemlai.milops.org/agents (Open the page: the agent guide, the workflows and every tool's CLI command and web page.)
