# UseResponse Daily QA

An automated daily Quality Assurance pipeline for customer support chats, built in n8n.

## What it does

Runs every morning at 08:00. Fetches all completed chats from the previous day via the UseResponse API, evaluates each one against a 16-criterion QA rubric using an AI agent, and logs scored results to a Google Sheet.

## How it works

1. **Schedule Trigger** — fires at 08:00 daily
2. **UseResponse API** — fetches up to 1,000 chats (paginated), filters for yesterday's date range
3. **Theme extraction** — flattens nested `custom_fields` into `theme` / `subtheme` strings and stores them in an internal Chat ID DB (n8n Data Table)
4. **DB deduplication** — checks for duplicates; clears stale records if count exceeds threshold, then loops back to paginate
5. **Message fetching** — for each chat, fetches the full message thread via a second UseResponse API call
6. **Conversation formatting** — filters system/bot messages, deduplicates, sorts chronologically, and formats with GMT+4 timestamps
7. **Rate-limited AI evaluation** — passes each formatted conversation to an AI agent (GPT) with a strict rubric; 2-second delay between calls
8. **Score calculation** — a Code node parses the AI JSON output, applies weighted deductions (total 100 points), and produces a Score + Score Breakdown
9. **Google Sheets logging** — appends one row per chat to the `QA2026` sheet

## QA Criteria (16 total)

| Criterion | Weight |
|---|---|
| Verification | 20 |
| Accuracy and Completeness | 15 |
| Professionalism and Tone | 10 |
| Follow-up | 8 |
| Clarity and Communication | 7 |
| Slack/Centrivo Check | 6 |
| KYC | 5 |
| Correct Theme | 5 |
| Responsiveness and Efficiency | 4 |
| Greeting and Introduction | 4 |
| Offering Alternatives | 4 |
| Apologize for Company Error | 3 |
| Closure | 3 |
| Closure Wait Rule Kept? | 3 |
| Thank the Client for Contacting Us | 2 |
| Answered All Questions? | 1 |

Scores: `Yes` / `No` / `N/A` / `Check` (Check reserved for unverifiable bonus-receipt queries)

## AI Agent Tools

The AI agent has access to 4 tools:
- **PB Knowledge Base** (n8n Data Table) — verifies bonus/promo rule accuracy
- **Chat ID DB** (n8n Data Table) — looks up agent-assigned theme/subtheme per chat
- **Slack** — checks whether the agent escalated to the correct channel within 24h
- **Tickets/Followups** (Google Sheets) — verifies follow-up tasks were logged

## Tech stack

- n8n (self-hosted)
- UseResponse API
- OpenAI (via n8n LangChain agent)
- Google Sheets
- Slack API
- n8n Data Tables

## Setup notes

- Requires UseResponse API key and group ID
- Requires Google Sheets OAuth2 credentials in n8n
- Requires Slack OAuth2 credentials in n8n
- Workflow is exported as JSON — import directly into your n8n instance
- Internal Data Table IDs (`kcqvMjjXRfbmFkZ2`, `U9S4ysxLUbmALIF0`) will differ on a fresh n8n instance — update node references accordingly
