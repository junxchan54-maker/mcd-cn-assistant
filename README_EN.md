![mcd-cn-assistant · McDonald's China MCP Assistant · campaign push / collab watch / best-value ordering / group ordering](./assets/banner.svg)

# mcd-cn-assistant

> An out-of-the-box AI Agent skill built on the **official McDonald's China MCP service** — never miss a campaign, never miss a coupon, never overpay for a meal, never do group-bill math by hand.

[![GitHub stars](https://img.shields.io/github/stars/junxchan54-maker/mcd-cn-assistant?style=social&label=Star)](https://github.com/junxchan54-maker/mcd-cn-assistant/stargazers)
[![MCP](https://img.shields.io/badge/MCP-Streamable%20HTTP-0F6E56)](https://open.mcd.cn/mcp)
[![License](https://img.shields.io/badge/License-MIT-3B6D11)](./LICENSE)

## What it is

The official McDonald's China MCP server (`https://mcp.mcd.cn`) exposes 35 atomic tools — stores, menus, coupons, pricing, ordering, points, mall, lottery, events. Atomic tools are not an assistant: the user still has to know which tool to call, in what order, and how to combine coupons to get the cheapest basket.

This project orchestrates those tools into **four real user journeys** and hard-codes the safety boundaries.

## The four capabilities

> **One shared ordering entry point (key design).** Both ordering capabilities start at the same gate: for any ordering request, the agent **asks up front — "Just for yourself, or ordering for several people (a group order)?" — before it even looks up a store**. Group ordering therefore **never depends on the user remembering to ask for it**; answering "group order" is enough to enter the full bill-splitting flow. Vague answers get one clarifying follow-up instead of silently defaulting to solo.

| Capability | Trigger | Core orchestration |
|---|---|---|
| **① Latest campaign push** | "What's on at McDonald's this month?" | `now-time-info` → `campaign-calendar` |
| **② Collab-edition watch** | "Watch the collab editions for me" · "Ping me whenever a collab drops" | `config/keywords.json` (default keyword: collab/co-brand + synonyms, incl. **"联动"** — McDonald's official copy for IP tie-ups reads "联动", not "联名") → fetch campaigns → keyword + synonym match → push only hits; runs daily at 09:00 unattended. ⚠️ Collab **merch IS orderable** (corrected 2026-10-16, superseding a wrong earlier note): merch items sit right in `query-meals` (e.g. `521954` retro cap, `521955` pins) and adding the ¥9.90 15-day membership trial card is usually *cheaper* — merch drops ¥5 and delivery + packing fees go to zero (net saving ¥1.10–12.10 measured). Merch varies by store *and* channel, so scan before concluding. What MCP canNOT do: real-time stock queries, continuous restock monitoring, or redeeming merch from the points mall |
| **③ Best-value ordering** | "How do I order this cheapest?" (entry gate answered **solo**) | Locate store → gather coupons → menu → enumerate combos → **price each via `calculate-price`** → compare → order after confirmation. Note: delivery prices *and product codes* differ from in-store (Big Mac is `1100` in-store / `901100` delivery); in-store orders have essentially no discount-threshold tiers (measured `discount:0`) |
| **④ Group ordering** | Entry gate answered **"several people / group order"**, or "Office tea-break group order, help me collect and split the bill" | Open order session (split preferences) → collect items (agent transcribes, duplicates auto-merged) → discount-threshold upsell + coupon attribution + public-fee splitting (delivery + packing) → per-person bill → settle on the organizer's single account |

### Xiaohongshu-scenario skills (added 2026-10-16)

| Capability | Trigger | Can do | Cannot do (never promised) |
|---|---|---|---|
| **⑤ Hidden-combo restoration** | Paste a viral recipe text or send a screenshot | Parse text/screenshot → map to the current store menu → **real `modification` calls** (drop the sauce, no ice, swap to diet cola for +¥0) → price & order; free extras auto-written into the delivery order remark | Free extras (ketchup/sugar/pepper/creamer) are **not products and cannot be added**; `remark` works **on delivery orders only, ≤50 chars**; **Xiaohongshu links cannot be fetched** (login wall) |
| **⑥ Budget-combo solver** | "Under ¥20 with meat, a staple and a diet drink" | Split hard constraints → gather coupons → enumerate 2–5 candidates → **price every candidate via `calculate-price`** → return the lowest payable | In-store orders have essentially no discount tiers; when there is no tier, say so — never invent one |
| **⑦ Collab-merch radar** | "Which nearby store has the collab cap?" | **Scan stores on demand** (one `query-meals` per store) → **lock the order directly** → membership add-on optimization | **No stock API, no continuous monitoring**; points-mall redemption is unavailable |
| **⑧ Parameterized nutrition** | "Under 450 kcal, over 25g protein, no sauce" | `list-nutrition-foods` full table → filter by constraints → sauce-removal / diet-drink swaps (+¥0) → price & order | Nutrition table is **standard-portion** data; modifications don't auto-recompute calories, so results stay approximate |
| **⑨ Share-card generation** | "Make me a card I can post" | Generate a local SVG/HTML card + caption from **real** order and nutrition data | Costs no MCP calls; **numbers must be real** — if not queried, don't write it |
| **⑩ Scene-aware push** | "What should I eat this Friday evening?" | Day-part + weekday logic (breakfast / tea break / night market, Friday treats, weekend specials) | **No weather data source**; weather needs the user to state it — never fabricate temperature or rain |

> Full dictionaries, algorithms and templates: `references/xhs-playbook.md`. Real conversation samples: `docs/examples.md` examples 13–16.

## Quick start

1. Get an MCP Token at <https://open.mcd.cn/mcp> (phone login → console → activate → copy).
2. Add the server to your MCP client:

```json
{
  "mcpServers": {
    "mcd-mcp": {
      "type": "streamablehttp",
      "url": "https://mcp.mcd.cn",
      "headers": { "Authorization": "Bearer ${MCD_MCP_TOKEN}" }
    }
  }
}
```

3. Clone this repo into your skills directory and ask your agent:

> Use mcd-cn-assistant to show me this month's McDonald's campaigns.

> 🔑 **`${MCD_MCP_TOKEN}` is a placeholder** — replace it with **your own** token (or set an env var of the same name). This repo contains **no real credentials**; every placeholder must be filled in by the user.

Per-client setup guides: **[docs/mcp-setup.md](./docs/mcp-setup.md)**

## Repository layout

```
mcd-cn-assistant/
├── README.md / README_EN.md   # Project introduction
├── CONTEST_DECLARATION.md     # Official contest declaration (unmodified)
├── MCP_INTEGRATION.md         # MCP server / tools / call flow / business value
├── mcp-config.example.json    # Sanitized config, env-var placeholder only
├── workbuddy.md               # WorkBuddy development context
├── SKILL.md                   # The skill itself (loaded by the agent)
├── LICENSE
├── assets/banner.svg          # Animated header banner (hand-drawn SVG)
├── assets/cta-star.svg        # Star CTA graphic at the bottom
├── config/keywords.json       # Collab / campaign watch keywords for capability ②
├── references/mcd-tools.md    # All 35 MCP tools + call order + error codes
├── references/group-order.md  # Capability ④: order-session schema + splitting algorithm + bill template
└── docs/
    ├── mcp-setup.md           # MCP integration guide
    └── examples.md            # Real conversation examples
```

## Contest entry

This project is an entry for the **McDonald's Programmers' Creative Development Challenge** (麦当劳程序员创意开发大赛).

- Registration & ranking window: 2026-10-09 10:30 — 2026-10-25 23:59 (UTC+8)
- Ranking is based on **public GitHub Star count**
- Official declaration: [CONTEST_DECLARATION.md](./CONTEST_DECLARATION.md)

## Design principles

1. **Write operations require explicit confirmation.** `create-order`, `auto-bind-coupons`, `cancel-order`, `draw-lottery`, `mall-create-order`, `party-order-create`, `delivery-create-address` must never run silently.
2. **Never do the math yourself.** All amounts come from `calculate-price` — coupon face value does not equal real discount. Delivery orders also carry a packing fee on top of the delivery fee; and delivery menu prices differ from in-store ones (never reuse prices across channels).
3. **Never trust the model's internal clock.** Always call `now-time-info` first.
4. **Ask solo vs. group order at the ordering entry.** Any ordering request triggers a proactive "just for yourself, or a group order?" before store lookup — never wait for the user to bring up group ordering, and never default to solo when the answer is unclear.
5. **Be rate-limit friendly.** 600 requests/minute per token; call serially and reuse results.

## Compatibility

Framework-agnostic. The skill describes flows using official tool names only, so it works with any MCP-capable agent: Claude Code, Cursor, Cline, Cherry Studio, Trae, Kiro, VSCode Copilot, WorkBuddy, and others.

## Disclaimer

This is an **independent community project**, not affiliated with, endorsed by, or sponsored by McDonald's China or its affiliates. It does not bundle or redistribute any McDonald's source code, token, or private data. Using the MCP service requires your own token and is subject to McDonald's China's Terms of Use and MCP Service Rules. Non-commercial use only. Provided "as is", without warranty of any kind.

## License

[MIT](./LICENSE)

---

## If this skill helps you, please give it a star

![If this skill helps you, please give it a star so more people can see it](./assets/cta-star.svg)

**Click the Star button at the top right of this repo — every star helps someone else find it.**

- **One star = one free "well done"** — and the fuel that keeps this project maintained;
- The star count is the **only ranking criterion** for this entry in the **McDonald's China Developer Innovation Challenge**, so your support directly decides how many people see it;
- A **fork**, or sharing it with the colleague currently drowning in group-order math, helps just as much.

<p align="center">
  <a href="https://github.com/junxchan54-maker/mcd-cn-assistant/stargazers"><img src="https://img.shields.io/github/stars/junxchan54-maker/mcd-cn-assistant?style=for-the-badge&logo=github&label=Star%20this%20repo" alt="Star this repo"></a>
</p>
