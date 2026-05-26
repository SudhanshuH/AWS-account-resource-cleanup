# AWS Cost Audit

One prompt. One report. One approval. Done.

An agentic AI tool that audits your AWS account for idle resources and cleans them up — saving thousands per month without dashboards, scripts, or manual console work.

```
You: "Run a full AWS cost audit on my account"

AI: Found 40 idle resources across 6 regions. Potential savings: $14,307/month ($171K/year).
    Top 5 quick wins:
    1. Delete 3 GPU endpoints in us-west-1 → $4,823/month
    2. Delete Canvas apps (2 regions) → $2,800/month
    3. Delete Data Wrangler instances → $2,050/month
    ...

You: "Delete everything flagged"

AI: ✅ Done. 40 resources removed.
```

---

## Why This Exists

AWS accounts used for demos, POCs, workshops, and experimentation accumulate idle resources that silently bleed budget:

- SageMaker GPU endpoints left running after a demo → **$4,000/month each**
- Studio Canvas/Data Wrangler apps that auto-started → **$1,400/month each**
- Kendra indices provisioned for a POC → **$810/month**
- ECS workshop stacks nobody cleaned up → **$570/month**
- HyperPod clusters from training experiments → **$660/month**

No alerts. No traffic. No errors. Just charges — month after month, year after year.

**First real use:** Recovered **$176K/year** from one SA account. Resources had been running 2-4 years with zero usage.

---

## Two Ways to Use

### Option 1: MCP Server (Recommended)

For AI tools that support MCP: Kiro, Amazon Quick, Claude Desktop, Claude Code.

**Setup once → any user just types a prompt.**

[→ Full guide: `aws_cost_audit.md`](aws_cost_audit.md)

### Option 2: Prompt-Only

For tools with terminal access: Kiro, Q Developer CLI, CloudShell.

**Zero setup. Just paste a prompt.**

[→ Full guide: `aws_cost_audit_prompt_only.md`](aws_cost_audit_prompt_only.md)

---

## Quick Start

### Fastest Path (Zero Install)

1. Open [AWS Console](https://console.aws.amazon.com)
2. Click **CloudShell** icon (top-right)
3. Type `q chat`
4. Paste the prompt from [`aws_cost_audit_prompt_only.md`](aws_cost_audit_prompt_only.md)

### MCP Server Setup (5 minutes, one-time)

```bash
# Prerequisites: Python 3.10+, uv, AWS CLI configured
cd aws-cost-audit-mcp
uv sync

# Verify
uv run python -c "from server import mcp; print('Ready')"
```

Add to your AI tool's MCP config:

```json
{
  "mcpServers": {
    "aws-cost-audit": {
      "command": "/path/to/aws-cost-audit-mcp/.venv/bin/python",
      "args": ["/path/to/aws-cost-audit-mcp/server.py"]
    }
  }
}
```

Then just ask: *"Run a full AWS cost audit"*

---

## What Gets Scanned

| Category | Services |
|----------|----------|
| **AI/ML** | SageMaker endpoints, notebooks, Studio apps (Canvas, Data Wrangler, KernelGateways, JupyterLab), HyperPod clusters |
| **Compute** | ECS clusters/services, EKS clusters |
| **Storage** | FSx file systems, EFS |
| **Search & Analytics** | Kendra indices, OpenSearch Serverless collections |
| **Other** | Amazon Q apps, Lookout for Equipment schedulers, Managed Grafana, NAT Gateways |

All regions auto-discovered from Cost Explorer usage type prefixes — no guessing.

---

## How It Works

```
User (natural language)
  → AI Assistant (Kiro / Quick / Claude)
    → MCP Protocol
      → aws-cost-audit server (runs on YOUR machine)
        → boto3 with YOUR credentials
          → AWS APIs (Cost Explorer, SageMaker, ECS, Kendra, etc.)
```

No data leaves your machine. No third-party access. No new IAM roles needed.

### MCP Tools

| Tool | Description |
|------|-------------|
| `get_cost_breakdown` | Cost Explorer data for last N months, auto-discovers active regions |
| `scan_idle_resources` | Scans a region for idle resources (configurable idle threshold) |
| `delete_resource` | Removes a specific resource (handles dependencies) |
| `full_audit` | Complete end-to-end: cost breakdown + scan all regions + flag idle resources |

### Configurable via Natural Language

Users customize by just asking:

- *"Look back 6 months and flag anything idle for 30 days"*
- *"Only scan SageMaker in ap-south-1"*
- *"Show 12 months of cost trends"*
- *"Delete everything except the HyperPod cluster"*

---

## Repository Structure

```
.
├── README.md                      # This file
├── aws_cost_audit.md              # Full guide — MCP server approach
├── aws_cost_audit_prompt_only.md  # Full guide — prompt-only approach (no MCP)
├── aws_cost_audit_blog_draft.md   # LinkedIn blog draft
└── aws-cost-audit-mcp/            # The MCP server
    ├── server.py                  # MCP server (4 tools, ~300 lines)
    ├── pyproject.toml             # Dependencies (mcp, boto3)
    ├── run.sh                     # Shell wrapper for Quick
    └── README.md                  # Server-specific docs
```

---

## Compatibility

| AI Tool | MCP Server | Prompt-Only |
|---------|-----------|-------------|
| **Kiro** | ✅ | ✅ |
| **Claude Code** | ✅ | ✅ |
| **Claude Desktop** | ✅ | — |
| **Amazon Quick** | ✅ (MCP loaded) | — |
| **Q Developer CLI** | — | ✅ |
| **CloudShell** | — | ✅ (zero install) |

---

## Prerequisites

- **AWS CLI configured** (`aws sts get-caller-identity` must work)
- **For MCP server**: Python 3.10+ and [uv](https://docs.astral.sh/uv/getting-started/installation/)
- **For prompt-only**: Nothing beyond AWS CLI

---

## License

MIT — use it, share it, adapt it.
