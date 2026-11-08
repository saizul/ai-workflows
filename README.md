# ai-workflows

Importable [n8n](https://n8n.io) workflows for AI agents and prompt systems, with the description and a diagram of each one.

[![Validate workflows](https://github.com/saizul/ai-workflows/actions/workflows/validate.yml/badge.svg)](https://github.com/saizul/ai-workflows/actions/workflows/validate.yml)

| Workflow | File | What it does |
|----------|------|--------------|
| Multi AI Agent (Telegram AI Bot) | [`Multi Ai Agent.json`](Multi%20Ai%20Agent.json) | One Telegram bot that routes each message by keyword to a different AI persona and model, or to all three at once for a combined answer |

## Multi AI Agent (Telegram AI Bot)

![Multi AI Agent workflow](Multi%20Ai%20Agent%20Workflow.png)

**What it does.** A Telegram Trigger receives every message. A Switch node lowercases the text and checks, in order, whether it contains `force`, `nexus`, `victor` or `team`. The first match picks the route; anything else goes to the fallback output (named `Sizu`), which is handled by the default agent.

| Route | Trigger word | Agent(s) | Model |
|-------|--------------|----------|-------|
| Force | `force` | Ai Force: creative, bold, direct | Google Gemini node, `models/gemma-4-26b-a4b-it` |
| Nexus | `nexus` | Ai Nexus: analytical, structured | OpenRouter, `inclusionai/ring-2.6-1t:free` |
| Victor | `victor` | Ai Victor: fast and concise | Groq, `llama-3.3-70b-versatile` |
| Team | `team` | Team Force, Team Nexus and Ai Victor1 run in parallel as a three-member "council" (2-4 line answers, each prefixed with its name); a Merge node (3 inputs) and a Code node combine them into one message | Gemini `models/gemini-2.5-flash-lite`, OpenRouter `inclusionai/ring-2.6-1t:free`, Groq `llama-3.3-70b-versatile` |
| Fallback (`Sizu`) | anything else | Ai Default (persona "Sizu"), which can search and read Gmail through a tool | Groq, `llama-3.3-70b-versatile` |

All agents replying to the same Telegram chat share one `Simple Memory` node (window of 15 messages, keyed by the Telegram chat id). Every route ends in a single `Send a text message` Telegram node that replies to the chat the message came from. The agents answer in the language of the user (Bengali, English or mixed).

**Gmail tool.** `Get many messages in Gmail` is attached to Ai Default as a tool. The agent fills in `limit` and a Gmail search query itself (for example `is:unread` or `newer_than:1d`), so "check my email" or "summarise my last 5 emails" works from Telegram.

**Nodes used.** Telegram Trigger, Switch, AI Agent (x7), Google Gemini Chat Model (x2), OpenRouter Chat Model (x2), Groq Chat Model (x3), Simple Memory, Gmail Tool, Merge, Code, Telegram.

**Credentials needed** (create them in n8n, then select them on each node after import):

| Credential type | Used by |
|-----------------|---------|
| Telegram API (bot token from [@BotFather](https://t.me/BotFather)) | Telegram Trigger, Send a text message |
| Google Gemini (PaLM) API key | Both Google Gemini Chat Model nodes |
| OpenRouter API key | Both OpenRouter Chat Model nodes |
| Groq API key | All three Groq Chat Model nodes |
| Gmail OAuth2 | Get many messages in Gmail |

The credential names stored in the file (for example "Groq account") belong to the original instance; n8n asks you to pick your own when you open each node. No keys or tokens are stored in the JSON.

**Setup.**

1. In n8n choose **Workflows -> Import from file** and select `Multi Ai Agent.json`.
2. Create the five credentials above and attach them to the nodes that show a warning.
3. Check the model names. The free OpenRouter and Gemini/Gemma model ids in the file can be retired by the providers, so swap them for any model you have access to.
4. Edit the trigger words in the **Switch** node if you want different routes.
5. Activate the workflow and message your bot, for example `force give me three headline ideas`, `team what should I post this week`, or any plain message for the default agent.

Longer notes on the design are in [`Multi Ai Agent Descriptions`](Multi%20Ai%20Agent%20Descriptions) and [`Multi Ai Agent Feature`](Multi%20Ai%20Agent%20Feature).

## Validation

`scripts/validate-workflows.js` checks every workflow JSON in this repo (valid JSON, well-formed nodes, connections to existing nodes, no orphaned nodes, no embedded secrets). It runs in GitHub Actions on every push and pull request and locally with:

```bash
node scripts/validate-workflows.js
```

## Author

[Saizul Amin](https://saizul.com): web developer, SEO and automation specialist.
