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

# Update workflow
mcp__n8n-wheelsfeels__n8n_update_full_workflow(...)

# List all workflows
mcp__n8n-wheelsfeels__n8n_list_workflows()
```

## Current State (v2)

### Nodes (11 total)

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
| Set Outputs | evaluation | Write results back to sheet |

### Key Features

- **Product Lookup**: Nested AI agent queries Supabase for vehicle generations and products
- **Model Stripping**: Automatically strips trims (Wilderness, TRD Pro, etc.) for accurate matching
- **Evaluation System**: Automated testing via Google Sheets dataset
- **Dynamic Session Keys**: Isolates memory between evaluation runs vs production

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
- **Test Cases**: 10 (vehicle lookup, unknown vehicles, email collection, edge cases)

### Running Evaluations

1. Open workflow in n8n
2. Go to **Evaluations** tab
3. Click **Run Evaluation** or **Evaluate All**
4. Results written to `actual_response` and `actual_tools_used` columns

### Session Key Logic

```javascript
// Fresh memory for each eval run, persistent for production
$json.sessionId.startsWith('eval-')
  ? $json.sessionId + '-' + $execution.id  // Evaluation
  : $json.sessionId                         // Production
```

## Planned Enhancements

1. **Page Context**: Pass current product page URL via `metadata` option
2. **Slack Monitoring**: Send chat logs to Slack with threading by session
3. **Follow-up Emails**: Trigger email follow-ups when chat sessions end
4. **Set Metrics Node**: Add AI-based scoring for response quality

## Related Files

- `workflow.json` - Exported n8n workflow (safe to share, contains no API keys)
- `chat-widget-implementation-plan.md` - Detailed implementation plan (not committed)

## Security

This is a PUBLIC repository. Do not commit:
- `.mcp.json` (contains API keys)
- `.env` files
- Any credential files
- Google Sheet URLs with sensitive data
