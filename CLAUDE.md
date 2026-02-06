# WheelsFeels Chat Widget Project

## Workflow Info

- **Workflow Name**: Website Chat Widget - AI Assistant
- **Workflow ID**: `DKfVfqHubRSwIGYT`
- **n8n Instance**: wheelsfeels.app.n8n.cloud

## MCP Tools

Use the `n8n-wheelsfeels` MCP server to manage the workflow:

```bash
# Get workflow
mcp__n8n-wheelsfeels__n8n_get_workflow(id="DKfVfqHubRSwIGYT")

# Update workflow (partial - preferred)
mcp__n8n-wheelsfeels__n8n_update_partial_workflow(id="DKfVfqHubRSwIGYT", operations=[...])

# List all workflows
mcp__n8n-wheelsfeels__n8n_list_workflows()
```

## Current State (v14)

### Nodes (13 total)

| Node | Type | Purpose |
|------|------|---------|
| Chat Trigger | chatTrigger | Receive chat messages (production) |
| AI Agent | agent | Main conversation handler (Anton) |
| OpenAI Chat Model | lmChatOpenAi | GPT-4.1, temp 0.7 |
| Window Buffer Memory | memoryBufferWindow | Session memory (10 messages) |
| Product Lookup Tool | agentTool | Nested agent for vehicle/product matching |
| OpenAI Chat Model 2 | lmChatOpenAi | GPT-4.1, temp 0 (for tool) |
| Find Vehicle Generation | supabaseTool | Query vehicle_generations table |
| Search Products by Generation | supabaseTool | Query products table |
| Evaluation Trigger | evaluationTrigger | Run test cases from Google Sheets |
| Check If Evaluating | evaluation | Route to eval vs production branch |
| Set Metrics | evaluation | AI-based Correctness scoring (1-5) |
| OpenAI Chat Model (Metrics) | lmChatOpenAi | GPT-4.1, temp 0 (for evaluation) |
| Set Outputs | evaluation | Write results back to sheet |

### Workflow Flow

```
Evaluation Trigger ──┐
                     ├──→ AI Agent → Check If Evaluating → Set Metrics → Set Outputs
Chat Trigger ────────┘         ↓                               ↓
                          (production)              OpenAI Chat Model (Metrics)
```

### Key Features

- **Markdown Link Formatting**: All product links use `[Link Text](url)` format for clean display
- **Always Ask for Year**: When customer mentions vehicle without year, Anton asks for it first
- **Generation Handling**: Recognizes generation names (5th Gen, 6th Gen, JL/JLU) as year-known, proceeds directly to product lookup
- **Generation-to-Year Mapping**: RAV4 5th Gen=2022, 4Runner 5th Gen=2020, Outback 6th Gen=2022, Wrangler JL/JLU=2020
- **Model Name Normalization**: Enforces correct case (RAV4 not Rav4, 4Runner exact case) for database queries
- **Product Lookup**: Nested AI agent queries Supabase for vehicle generations and products
- **All Products Returned**: Product Lookup Tool returns ALL matching products as an array (not just one)
- **Model Stripping**: Automatically strips trims (Wilderness, TRD Pro, etc.) and engine displacement numbers (GX460→GX, 530i→5-series)
- **Product Data Extraction**: Extracts dimensions, installation, price, delivery costs, accessories, warranty, and returns from product records
- **Warranty & Returns**: Product Lookup Tool extracts warranty (1-year) and returns (7 days) from accordion_items
- **AI Metrics Scoring**: Built-in Correctness metric scores responses 1-5 semantically
- **Dynamic Session Keys**: Isolates memory between evaluation runs vs production
- **Condensed Prompt**: AI Agent system message heavily condensed (~50% shorter) for efficiency
- **Direct Email Asks**: Uses "Drop your email" style instead of permission-based asks
- **Never Say No Match**: Never reveals "no products found" - always offers to send details via email
- **Always Use Tool**: Must call Product Lookup Tool for ANY product question (dimensions, installation, specs, warranty, returns, etc.)

### Credentials Required

| Credential | Purpose |
|------------|---------|
| OpenAi WheelsFeels | GPT-4.1 API access |
| WheelsFeels Supabase | Product/vehicle database |
| Google Sheets account | Evaluation dataset |

## Evaluation System

### Dataset

- **Spreadsheet ID**: `1JClbNIx5HzN1Kt4L3voHF229eSYUETTO2zkGdWoUOLc`
- **Sheet Name**: `Chat Evaluation`
- **Test Cases**: 21

### Test Categories (3 total)

| Category | Description | Test Count |
|----------|-------------|------------|
| `product` | Vehicle lookups, unknown vehicles, missing year, Lexus/BMW model handling, dimensions | 8 |
| `support` | Lead time, discounts, returns, orders, videos, technical, collaboration | 9 |
| `conversation` | Greeting, email collection, multilingual | 4 |

### Google Sheet Columns

| Column | Purpose |
|--------|---------|
| `sessionId` | Test case ID (eval-tc001 through eval-tc019) |
| `category` | Test type: product, support, conversation |
| `notes` | Brief description of what the test validates |
| `chatInput` | User message to test |
| `expected_response` | Criteria for correct response (semantic) |
| `actual_response` | AI's actual output (auto-filled) |
| `response_quality` | Correctness score 1-5 (auto-filled) |
| `expected_tools` | Tools that should be called |
| `actual_tools_used` | Tools actually called (auto-filled) |
| `tools_correct` | YES/NO/PARTIAL for tool accuracy |

### How Correctness Metric Works

The built-in Correctness metric uses **semantic matching**, not exact text:
- Compares `actual_response` vs `expected_response` for MEANING
- Scores 1-5 (5 = all key points match, 1 = completely wrong)
- Tracks over time in n8n Evaluations tab with graphs

**Example `expected_response` formats (all work):**
```
"Include product URL for Outback 6th gen, ask for name and email"
"Ask for year since not provided. Ask for name and email."
"Direct to help@wheelsfeels.com for refunds, offer to follow up with contact info."
```

### Running Evaluations

1. Open workflow in n8n
2. Go to **Evaluations** tab
3. Click **Run Evaluation**
4. Check results:
   - Google Sheet: `response_quality` has 1-5 scores
   - Evaluations tab: Shows Correctness metric with historical graphs

### Session Key Logic

```javascript
// Fresh memory for each eval run, persistent for production
$json.sessionId.startsWith('eval-')
  ? $json.sessionId + '-' + $execution.id  // Evaluation
  : $json.sessionId                         // Production
```

## Planned Enhancements

1. **Slack Monitoring**: Send chat logs to Slack with threading by session
2. **Follow-up Emails**: Trigger email follow-ups when chat sessions end
3. **Tools Used Metric**: Add built-in Tools Used metric to track tool accuracy

## Related Files

- `workflow.json` - Exported n8n workflow (safe to share, contains no API keys)

## Security

This is a PUBLIC repository. Do not commit:
- `.mcp.json` (contains API keys)
- `.env` files
- Any credential files
- Google Sheet URLs with sensitive data
- `chat-widget-implementation-plan.md` (internal planning)
- `tawk.to chat examples/` (contains customer data)
