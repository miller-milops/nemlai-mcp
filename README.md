# NemlAI: nemlig MCP-server til Claude, ChatGPT og andre AI-agenter

**Jeres næste indkøb, forudset.** NemlAI er AI-indkøb til nemlig.com: den lærer af jeres egen
ordrehistorik, forudser hvad der skal i næste kurv, og lægger den klar med mængder. Tænk på det som
en automatisk indkøbsliste, der også fylder kurven. De fleste nemlig-MCP'er kan søge og lægge varer i
kurven, når du beder om det; NemlAI ved selv, hvad I plejer at købe. Med NemlAI's MCP-server kan din
AI-assistent (Claude, ChatGPT, Codex, Grok, Hermes, OpenClaw) lave, justere og sende kurven for dig.
Du godkender og betaler selv på nemlig.com.

*English below.* · [nemlai.milops.org](https://nemlai.milops.org) · [Guide til agenter](https://nemlai.milops.org/agents)
· [Dokumentation](https://nemlai.milops.org/dokumentation) · [Sådan virker forudsigelsen](https://nemlai.milops.org/saadan-virker-det)

Dette repo indeholder pakkerne, der forbinder AI-agenter med NemlAI: **MCP-serveren**
(`https://nemlai.milops.org/mcp`, hostet, Streamable HTTP; intet at installere eller køre lokalt) og **`nemlai`-skillen**,
der lærer agenten arbejdsgangene. Selve NemlAI kører hos os; her er ingen serverkode.

## Hvad kan agenten?

- **Ugens kurv, uden at bygge den selv:** "Lav min kurv til 1.500 kr." Kurven bygges ud fra det, I plejer at
  købe, og hvornår I plejer at købe det. Bagefter kan agenten forklare, hvorfor en vare er med.
- **Justere:** tilføje, fjerne og ændre antal; søge i varerne; holde kurven under et budget; tilbud
  regnes med fra det antal, de kræver ("3 for 79").
- **Aftensmad:** find retter og læg ingredienserne i kurven.
- **Sammenlign butikker:** se kurvens pris hos nemlig.com, Bilka, føtex, Netto og REMA 1000, og send
  den til nemlig.com eller BilkaToGo. Kvitteringer fra Bilka og føtex kan forbindes.
- **Forbrug:** hvad I bruger på dagligvarer, top-varer, udvikling over tid, kvitteringer.
- **Indstillinger:** udeluk varer, præferencer, hent nye ordrer.

Kurven er den samme på web, i CLI'en og via MCP, så du kan altid åbne den på
[nemlai.milops.org/kurv/](https://nemlai.milops.org/kurv/).

## Leder du efter en nemlig API?

nemlig.com har ingen offentlig API. NemlAI giver dig én oven på din egen nemlig.com-konto:
MCP-serveren her, CLI'en og Python-SDK'et (`pipx install nemlai`) og et dokumenteret HTTP-API. Se
[dokumentationen](https://nemlai.milops.org/dokumentation). NemlAI taler med nemlig.com på vegne af din
konto; det er ikke en officiel API, og NemlAI er ikke tilknyttet nemlig.com eller Salling Group.

## Kom i gang på ét minut

Du skal have en NemlAI-konto (gratis, lukket beta: du skal bruge en invitationskode,
[sådan får du en](https://nemlai.milops.org/hjaelp#invitation)).

| Klient | Sådan |
|---|---|
| **claude.ai, Claude Desktop og mobil** | Customize → Connectors → *Add custom connector* → URL `https://nemlai.milops.org/mcp` → *Connect* → log ind → **Giv adgang**. Skill: upload `nemlai-skill.zip` fra [Releases](https://github.com/miller-milops/nemlai-mcp/releases). |
| **ChatGPT** | Indstillinger → Apps & Connectors (udviklertilstand) → tilføj en MCP-server med URL'en, godkendelse OAuth → *Connect* → **Giv adgang**. |
| **Grok** | grok.com/connectors → *New Connector* → *Custom* → URL'en → log ind. |
| **Claude Code** | `/plugin marketplace add miller-milops/nemlai-mcp` og `/plugin install nemlai@nemlai` (skill + server). |
| **Codex, Hermes, OpenClaw** | `pipx install nemlai`, `nemlai login`, derefter `nemlai mcp install codex` (eller `hermes`, `openclaw`). |
| **Andre MCP-klienter** | URL'en og OAuth, eller headeren `Authorization: Bearer <token>`. `nemlai mcp config generic` viser opsætningen. |

Med token i stedet for OAuth: `nemlai login` gemmer en token i `~/.config/nemlai/token`, og
konfigurationerne læser den fra `NEMLAI_TOKEN`:

```bash
pipx install nemlai
nemlai login                                         # spørger om din adgangskode i din egen terminal
export NEMLAI_TOKEN=$(cat ~/.config/nemlai/token)
nemlai mcp install claude-code                       # --dry-run viser ændringen først
nemlai mcp doctor                                    # tjekker URL, token og værktøjer
```

Spørg derefter agenten: **"Hvem er jeg i NemlAI?"** og **"Lav ugens kurv."**

## Sikkerhed

- **NemlAI bestiller og betaler aldrig.** Agenten fylder kurven; du gennemfører købet selv på
  butikkens side.
- Agenten viser kurven og totalen, før den sender, og sender kun, når du beder om det.
- Skillen forbyder agenten at spørge efter adgangskoder, og MCP-værktøjerne tager ikke imod dem.
  Login, butikskoder og sletning af kontoen sker i webappen.
- Token ligger ikke i konfigurationsfilerne: de henviser til `NEMLAI_TOKEN`. OAuth-forbindelser kan
  fjernes i klienten.
- Privatliv og data: [nemlai.milops.org/privatliv](https://nemlai.milops.org/privatliv).

---

## English

**NemlAI predicts your household's next nemlig.com grocery basket** from your own order history and
puts it together, with quantities. Most nemlig MCP servers and API wrappers can search and fill a cart
when asked; NemlAI already knows what you're about to run out of. This repo has the packages that connect AI agents to it: the hosted
**NemlAI MCP server** (`https://nemlai.milops.org/mcp`, Streamable HTTP, OAuth or bearer token) and the **`nemlai`
skill** (Agent Skills format) with the workflows: the weekly basket, dinners from dishes, trimming
to a budget, comparing shop prices (nemlig.com, Bilka, føtex, Netto, REMA 1000), sending to
nemlig.com or BilkaToGo, and reviewing grocery spending. NemlAI never checks out or pays.

| Client | Connect |
|---|---|
| claude.ai, Claude Desktop, ChatGPT, Grok | Add a custom connector with `https://nemlai.milops.org/mcp` and sign in with OAuth (**Giv adgang** = grant access). |
| Claude Code | `/plugin marketplace add miller-milops/nemlai-mcp`, then `/plugin install nemlai@nemlai`; the plugin sends `Bearer ${NEMLAI_TOKEN}` (see [`claude-code/`](claude-code/README.md)). |
| Codex, Hermes, OpenClaw | `pipx install nemlai && nemlai login`, then `nemlai mcp install <client>` (writes the config, installs the skill, backs up the file first). |
| Agent Plugins 1.0 clients (Cursor, Copilot, …) | Install [`plugin/`](plugin/README.md); the client signs in with OAuth. |

Agent guide with every tool, its CLI fallback and web page: https://nemlai.milops.org/agents ·
[llms.txt](https://nemlai.milops.org/llms.txt)

nemlig.com has no public API: NemlAI works on behalf of your own account, and is not affiliated with
nemlig.com or Salling Group.

## What's in this repo

| Folder | For |
|---|---|
| [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json), [`claude-code/`](claude-code/) | Claude Code plugin marketplace and plugin (skill + MCP server) |
| [`plugin/`](plugin/) | [Agent Plugins 1.0.0](https://agent-plugins.org/specification) plugin (Codex, Cursor, Copilot and other adopters) |
| [`plugin/skills/nemlai/`](plugin/skills/nemlai/) | The skill on its own (Hermes, OpenClaw, Codex, any Agent Skills client) |
| Releases | `nemlai-skill.zip` for upload to claude.ai and Claude Desktop |

The files are generated from the NemlAI repository; changes made here are overwritten. Questions and
bugs: [open an issue](https://github.com/miller-milops/nemlai-mcp/issues). Licence: MIT.

<!-- Generated by scripts/build_agent_packages.py in the NemlAI repo; edit the sources there, not this file. -->
