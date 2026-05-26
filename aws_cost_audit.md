# AWS Cost Audit (MCP Server)

## Overview

An MCP server that turns any AI assistant into an AWS cost auditor. One prompt finds idle resources bleeding money across your account. One approval cleans them all up.

**Invent and Simplify.** The traditional approach — logging into Cost Explorer, clicking through each service console, cross-referencing CloudWatch metrics across regions — takes hours of manual work that nobody prioritizes until the bill becomes painful. This replaces that entire workflow with a single sentence.

Built as a lightweight MCP (Model Context Protocol) server, it works with Amazon Quick, Kiro, Claude Desktop, or any MCP-compatible tool. Runs locally using your AWS credentials — no data leaves your machine.

**First real-world result:** Identified **$176K/year** in waste from a single AWS account — 40+ idle resources across 6 regions, cleaned up in under 5 minutes.

### Architecture

```
User: "Run a full AWS cost audit" → AI → MCP → aws-cost-audit server (local) → boto3 → AWS APIs
```

### Tools

| Tool | Description |
|------|-------------|
| `get_cost_breakdown` | Cost Explorer data, auto-discovers active regions |
| `scan_idle_resources` | Finds idle resources per region (SageMaker, ECS, Kendra, OpenSearch, FSx, L4E, Q, Grafana) |
| `delete_resource` | Removes a flagged resource with dependency handling |
| `full_audit` | Complete audit across all active regions in one call |

### Compatibility

| Tool | Works? |
|------|--------|
| Kiro | ✅ |
| Claude Code / Claude Desktop | ✅ |
| Amazon Quick (Desktop, via MCP) | ✅ (tools loaded; chat in Preview) |
| Q Developer CLI / CloudShell | Use [prompt-only version](aws_cost_audit_prompt_only.md) |

---

## For Users

Open your AI assistant (Kiro, Amazon Quick, or Q Developer CLI) and type:

```
Run a full AWS cost audit on my account
```

That's it. You'll get a report showing what's costing money, what's idle, and what to delete.

**Want to customize?** Just say it naturally:

```
Run a cost audit — look back 6 months and flag anything idle for more than 30 days
```

```
Scan only us-east-1 for idle SageMaker resources
```

```
Show me cost trends for the last 12 months
```

Once you see the report, approve deletions:

```
Delete everything flagged except the FSx filesystem
```

---

## What You'll See

A report like this appears in ~2 minutes:

| Service | Monthly Cost | Trend |
|---------|-------------|-------|
| Amazon SageMaker | $9,047 | 📉 |
| Amazon Kendra | $622 | ➡️ |
| ECS (Fargate) | $437 | ➡️ |
| Amazon FSx | $401 | ➡️ |
| ... | | |

**Flagged for deletion:**

| Resource | Region | What It Is | Monthly Cost | Action |
|----------|--------|-----------|-------------|--------|
| GPU endpoint (unused since 2023) | us-west-1 | AI model left running | $4,075 | ❌ Delete |
| Data Wrangler (3 instances) | us-east-1 | ML tool never used | $2,050 | ❌ Delete |
| Canvas App | us-east-1 | No-code ML tool | $1,400 | ❌ Delete |
| Kendra search index | us-east-1 | Empty test index | $810 | ❌ Delete |

**Potential savings: $14,307/month ($171K/year)**

You approve. It deletes. Done.

---

## Overview

**Invent and Simplify.** AWS accounts accumulate idle resources that silently burn budget — GPU endpoints from forgotten demos, ML tools that auto-started and never stopped, search indices provisioned for a POC and never decommissioned. The traditional approach — logging into Cost Explorer, clicking through each service console, cross-referencing CloudWatch metrics across regions — takes hours of manual work that nobody prioritizes until the bill becomes painful.

This tool replaces that entire workflow with a single sentence. One prompt. One report. One approval to clean up. It's the simplest possible path from "I think I'm overspending" to "here's exactly what to delete and how much you'll save."

In its first real use, it identified **$176K/year** in waste from one SA account — resources running 2-4 years with zero usage.

## The Problem

AWS accounts used for demos, workshops, POCs, and experimentation develop a long tail of forgotten resources:

- **SageMaker endpoints** left running after a demo (a single ml.g4dn.12xlarge costs ~$4,000/month)
- **Studio apps** (Canvas, Data Wrangler, KernelGateways) that auto-start and never get stopped
- **AI service indices** (Kendra, OpenSearch Serverless) provisioned for a POC and never decommissioned
- **ECS/EKS clusters** from workshop stacks that nobody cleaned up
- **HyperPod clusters + FSx volumes** spun up for training experiments months ago

These resources don't trigger alerts. No traffic, no errors, no notifications. They just quietly charge — month after month, year after year.

---

---

## How It Works (Behind the Scenes)

```
User types prompt → AI calls MCP tools → Tools run boto3 on local machine → AWS APIs respond → AI presents report
```

The `aws-cost-audit-mcp` server runs locally and provides 4 tools:
- **get_cost_breakdown** — Cost Explorer data, auto-discovers active regions
- **scan_idle_resources** — Finds unused resources in a region
- **delete_resource** — Removes a flagged resource
- **full_audit** — Complete scan across all regions in one call

No data leaves the user's machine. The AI just orchestrates.

---

## Admin Setup (One-Time)

Someone sets this up once. After that, users just type the prompt.

### Step 1: Install Prerequisites

```bash
# Install uv (Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Ensure AWS CLI is configured
aws sts get-caller-identity
```

### Step 2: Install the MCP Server

```bash
cd /path/to/aws-cost-audit-mcp
uv sync
```

### Step 3: Connect to the AI Tool

**Kiro** — add to `.kiro/settings/mcp.json`:
```json
{
  "mcpServers": {
    "aws-cost-audit": {
      "command": "uv",
      "args": ["run", "--project", "/path/to/aws-cost-audit-mcp", "python", "server.py"],
      "disabled": false
    }
  }
}
```

**Amazon Quick** — add to `~/.quickwork/profiles/<profile>/mcp_config.json`:
```json
{
  "mcpServers": {
    "aws-cost-audit": {
      "command": "/path/to/aws-cost-audit-mcp/.venv/bin/python",
      "args": ["/path/to/aws-cost-audit-mcp/server.py"],
      "_quick": { "name": "AWS Cost Audit" }
    }
  }
}
```

### Step 4: Done

Users open their AI tool and type: *"Run a full AWS cost audit"*

---

## Recommended Cadence

| Account Type | Frequency |
|-------------|-----------|
| SA / demo / sandbox | Monthly |
| Workshop / training | After each event |
| Dev / staging | Quarterly |
| Production | Semi-annually |

Run it anytime you want — just type the prompt. The cadence above is a suggestion for scheduled recurring runs.

---

## Distributing to Your Team

Share the `aws-cost-audit-mcp/` folder. Admin runs Steps 1-3 on each user's machine (or bakes it into a team onboarding script). After that, users never think about setup — they just ask.
