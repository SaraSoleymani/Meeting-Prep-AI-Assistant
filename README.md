# Sales Meeting Prep Agentic AI Assistant — n8n Workflow

An agentic n8n workflow that reads a list of prospects from Google Sheets, researches each company using live web search, generates a structured meeting prep brief, and writes it back to the sheet — ready before your next call.

Built and shared as part of my 10 Week Building with Agentic AI series.Folllow the serries eitehr on my [LinkedIn](https://www.linkedin.com/in/sarasoleymani/) or [Medium](https://sarasoleymani.medium.com/)

---

## What It Does

For each prospect in your Google Sheet with `status = pending`, the agent:

1. Reads the prospect name, title, and company
2. Uses Tavily to search for recent news, funding rounds, hires, and product signals
3. Generates a structured meeting prep brief using GPT
4. Writes the brief back to the matching row in Google Sheets
5. Updates the row status to `done`

---

## Workflow Diagram

```
Manual Trigger
      ↓
Get row(s) in sheet (Google Sheets — reads all pending rows)
      ↓
AI Agent (GPT-5-mini)
  ├── Customer_Research tool (Tavily search)
  └── Generates structured brief
      ↓
Update row in sheet (writes brief + sets status to "done")
```

---

## The Brief Output Format

The agent returns a structured brief in this exact format:

**Company Snapshot** — 2-3 sentences on what they do, who they serve, and growth stage

**Recent Signals** — 2-3 bullet points of relevant recent news, funding, hires, or product announcements

**Buyer Profile** — Who you're meeting, what they care about, what success looks like for them

**Most Likely Pain Point** — One specific challenge the company is probably facing

**Suggested Opening** — A 1-2 sentence opener referencing something specific, not generic phrases

Total brief stays under 250 words.

---

## Prerequisites

- [n8n](https://n8n.io/) (self-hosted or cloud)
- OpenAI API account (workflow uses `gpt-5-mini`)
- Google Sheets with your prospects list
- Tavily API key ([tavily.com](https://tavily.com))

---

## Setup

### 1. Import the Workflow
- In n8n, go to **Workflows → Import from File**
- Upload `Meeting_Prep_AI_Assistant.json`

### 2. Configure Credentials
Set up the following credentials in n8n (**Settings → Credentials**):

| Credential | Used By |
|---|---|
| OpenAI API | OpenAI Chat Model |
| Google Sheets OAuth2 | Get row(s) in sheet, Update row in sheet |
| Tavily API | Customer_Research tool |

### 3. Set Up Your Google Sheet
Create a sheet with these exact column names:

| Column | Description |
|---|---|
| `prospect_name` | Full name of the prospect |
| `prospect_title` | Job title |
| `company_name` | Company name |
| `status` | Set to `pending` for rows you want processed |
| `brief` | Leave empty — the agent writes here |

The workflow reads **all rows** from the sheet and processes each one. Set `status = pending` for any prospect you want a brief generated for. Once processed, status updates to `done` automatically.

### 4. Connect Your Sheet
The workflow is pre-configured to a specific Google Sheet ID. To point it to your own sheet:

- Open the **Get row(s) in sheet** node → update the Document ID to your sheet
- Open the **Update row in sheet** node → update the Document ID to match

---

## Swapping OpenAI for Claude (Anthropic)

If you have Anthropic API access:

1. Delete the **OpenAI Chat Model** sub-node from the AI Agent
2. Add an **Anthropic Chat Model** node and connect it to the AI Agent's `Chat Model` input
3. Add your Anthropic API credentials under **Settings → Credentials**
4. Recommended model: `claude-haiku-4-5-20251001` — fast and cost-effective for structured brief generation

---

## How to Run

This workflow runs **manually only** — click **Execute Workflow** in n8n whenever you want to generate briefs for all pending rows in your sheet.

To automate it on a schedule, replace the Manual Trigger with a Schedule Trigger node (e.g. every weekday morning at 7am: `0 7 * * 1-5`).

---

## The Four Building Blocks in This Workflow

| Building Block | How It's Implemented |
|---|---|
| **Knowledge & Context** | Prospect name, title, and company injected into the agent prompt per row. System prompt defines the AE persona and brief format. |
| **Tools** | Google Sheets (read + write), Tavily (live web research) |
| **Memory** | Stateless per run — each row is processed independently. Status column acts as a simple processed/unprocessed flag. |
| **Guardrails** | Agent instructed to say so rather than guess if context is missing. Brief capped at 250 words. Structured output format enforced via system prompt. |


---

## License

MIT — free to use, adapt, and build on.
