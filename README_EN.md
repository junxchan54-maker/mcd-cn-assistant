# mcd-cn-assistant

> An out-of-the-box AI Agent skill built on the **official McDonald's China MCP service** — never miss a campaign, never miss a coupon, never overpay for a meal.

[![MCP](https://img.shields.io/badge/MCP-Streamable%20HTTP-0F6E56)](https://open.mcd.cn/mcp)
[![License](https://img.shields.io/badge/License-MIT-3B6D11)](./LICENSE)

## What it is

The official McDonald's China MCP server (`https://mcp.mcd.cn`) exposes 35 atomic tools — stores, menus, coupons, pricing, ordering, points, mall, lottery, events. Atomic tools are not an assistant: the user still has to know which tool to call, in what order, and how to combine coupons to get the cheapest basket.

This project orchestrates those tools into **three real user journeys** and hard-codes the safety boundaries.

## The three capabilities

| Capability | Trigger | Core orchestration |
|---|---|---|
| **① Latest campaign push** | "What's on at McDonald's this month?" | `now-time-info` → `campaign-calendar` |
| **② Keyword campaign watch** | "Watch the collab editions for me" | `config/keywords.json` → fetch campaigns → keyword + synonym match → push only hits |
| **③ Best-value ordering** | "How do I order this cheapest?" | Locate store → gather coupons → menu → enumerate combos → **price each via `calculate-price`** → compare → order after confirmation |

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
├── config/keywords.json       # Watch keywords for capability ②
├── references/mcd-tools.md    # All 35 MCP tools + call order + error codes
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
2. **Never do the math yourself.** All amounts come from `calculate-price` — coupon face value does not equal real discount.
3. **Never trust the model's internal clock.** Always call `now-time-info` first.
4. **Be rate-limit friendly.** 600 requests/minute per token; call serially and reuse results.

## Compatibility

Framework-agnostic. The skill describes flows using official tool names only, so it works with any MCP-capable agent: Claude Code, Cursor, Cline, Cherry Studio, Trae, Kiro, VSCode Copilot, WorkBuddy, and others.

## Disclaimer

This is an **independent community project**, not affiliated with, endorsed by, or sponsored by McDonald's China or its affiliates. It does not bundle or redistribute any McDonald's source code, token, or private data. Using the MCP service requires your own token and is subject to McDonald's China's Terms of Use and MCP Service Rules. Non-commercial use only. Provided "as is", without warranty of any kind.

## License

[MIT](./LICENSE)
