# AWS Cost Audit — Prompt-Only (No MCP)

For use in Kiro, Q Developer CLI, or CloudShell. No MCP server needed — the AI uses AWS CLI directly.

---

## Zero-Setup Option (Nothing Installed)

If you have nothing configured on your machine — no CLI, no credentials, no tools — do this:

1. Open [AWS Console](https://console.aws.amazon.com) in your browser
2. Click the **CloudShell icon** (terminal icon, top-right navigation bar)
3. In CloudShell, type: `q chat`
4. Paste the prompt below

That's it. CloudShell already has your AWS credentials. Q CLI is pre-installed. Zero setup.

---

## For Users

Open Kiro and paste this:

```
Perform a comprehensive AWS cost audit on my account.

- Account purpose: [SA demo account / dev sandbox / workshop account — describe yours]

Do the following:

1. Pull Cost Explorer data for the last 3 months grouped by service. Show top spenders in a table with monthly trends. Use usage type prefixes (USE1, USW1, USW2, APS3, EUW1, etc.) to auto-discover which regions have active spend.

2. For each service costing more than $10/month, scan all active regions for idle resources:
   - SageMaker: endpoints, notebook instances, Studio domains/apps (Canvas, Data Wrangler, KernelGateways, JupyterLab), HyperPod clusters
   - Compute: EC2 instances, ECS clusters/services, EKS clusters
   - Storage: FSx file systems, EFS
   - AI/ML: Kendra indices, OpenSearch (managed + serverless), Lookout for Equipment inference schedulers
   - Other: Amazon Q applications, Managed Grafana workspaces, NAT Gateways, Load Balancers with no targets

3. Flag anything that:
   - Has not been modified or accessed in the last 2 months
   - Is a test/workshop/demo leftover (naming patterns: "test-", "demo-", "workshop-", "example-", timestamp suffixes)
   - Has been running 24/7 with no meaningful traffic
   - Was created by CloudFormation stacks with "workshop", "lab", or "example" in the name

4. Auto-discover regions from Cost Explorer usage type prefixes. Scan every region with active spend.

5. Present findings as:
   - Summary table: monthly spend by service (last 3 months)
   - Flagged resources table: resource name, region, type, creation date, last activity, monthly cost estimate, recommendation (delete/stop/resize)
   - Total potential monthly and annual savings
   - Quick wins: top 5 highest-impact deletions

6. Do NOT delete anything — report only. I will confirm deletions separately.
```

---

## After the Report

Review the flagged resources, then tell Kiro:

```
Delete everything flagged except [anything you want to keep]
```

Or be selective:

```
Delete only the Quick Wins
```

```
Delete all the us-west-1 resources
```

---

## Customization

Just say what you need in natural language. Examples:

```
Run a cost audit — look back 6 months and flag anything idle for 30 days
```

```
Only scan SageMaker resources in ap-south-1
```

```
Show me what's costing more than $100/month
```

---

## Prerequisites

- AWS CLI configured on the machine running Kiro (`aws sts get-caller-identity` must work)
- That's it. No MCP server, no Python, no dependencies.
