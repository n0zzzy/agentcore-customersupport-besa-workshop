# 🏅 Customer Support Agent on Amazon Bedrock AgentCore

**A production-grade customer support agent system built during the BeSA Cloud Academy program *Agentic AI: PoC to Production on AWS*.**

It covers the full path from a prototype agent to a **secured, observable, continuously evaluated** deployment: JWT auth, a governed tool gateway, long-term memory, online evaluations, and an A/B runtime.

![AgentCore](https://img.shields.io/badge/Bedrock_AgentCore-0d1117?style=for-the-badge&logoColor=dc2626)
![Strands](https://img.shields.io/badge/Strands_Agents_SDK-0d1117?style=for-the-badge&logo=python&logoColor=dc2626)
![CDK](https://img.shields.io/badge/AWS_CDK-0d1117?style=for-the-badge&logo=typescript&logoColor=dc2626)
![Cognito](https://img.shields.io/badge/Cognito_JWT-0d1117?style=for-the-badge&logoColor=dc2626)

---

## 🏗️ Architecture

```
  Frontend (Flask UI)
        │  JWT (Amazon Cognito)
        ▼
┌───────────────────────────────┐        ┌─────────────────────────────┐
│ AgentCore Runtime             │        │ AgentCore Memory            │
│ "CustomerSupport" (Strands)   │◄──────►│ SharedMemory                │
│  • get_return_policy          │        │  • SEMANTIC  /users/{id}/facts
│  • get_product_info           │        │  • SUMMARIZATION per session│
│  • dynamic model config hook  │        └─────────────────────────────┘
└──────────────┬────────────────┘
               │ MCP (OAuth2 M2M egress)
               ▼
┌───────────────────────────────┐
│ AgentCore Gateway (secured)   │  semantic tool search + policy engine
│  ├─ WarrantyCheck  → Lambda   │
│  ├─ ProcessRefund  → Lambda   │
│  └─ customer-support-ab → HTTP runtime "CustomerSupportAB"
└───────────────────────────────┘
               │
               ▼
   Online Evaluations: GoalSuccessRate · Correctness · ToolSelectionAccuracy
```

## ✨ What's inside

| Capability | Implementation |
|---|---|
| **🤖 Agent runtime** | Strands agent on AgentCore Runtime (Python 3.13) with local tools plus a `BeforeModelCallEvent` hook for dynamic model config |
| **🔐 Inbound auth** | `CUSTOM_JWT` authorizer backed by Amazon Cognito, with an allow-listed user-ID header |
| **🛠️ Tool gateway** | AgentCore Gateway exposes Lambda tools (`WarrantyCheck`, `ProcessRefund`) over MCP with JSON tool schemas, **semantic tool search**, and a **policy engine** |
| **🔑 Outbound auth** | OAuth2 client-credentials provider so the agent can call the gateway |
| **🧠 Memory** | Shared long-term memory with **semantic facts** per user and **session summaries** |
| **📊 Online evals** | `QualityMonitor` samples 100% of traffic and scores goal success, correctness, and tool-selection accuracy |
| **🧪 A/B testing** | Second runtime `CustomerSupportAB` (Python 3.14) with its own `ABQualityMonitor` |
| **🧩 Harness agents** | `ContainerAgent`, `OrderResearchAgent` (with a human-in-the-loop test), `PersistentReportAgent` |
| **☁️ IaC** | Everything is defined in `agentcore/agentcore.json` and deployed with **AWS CDK** |

## 📁 Structure

```
agentcore/
  agentcore.json        # runtimes, memory, gateway, credentials, evaluators
  cdk/                  # CDK app that deploys the project
app/
  CustomerSupport/      # main agent: main.py, tools, memory, MCP client, frontend
  CustomerSupportAB/    # A/B variant runtime
  ContainerAgent/       # harness agent
  OrderResearchAgent/   # harness agent + HITL test
  PersistentReportAgent/
```

## 🚀 Deploy

```bash
# requires the AgentCore CLI (github.com/aws/agentcore-cli), Python 3.10+ and uv
agentcore dev       # run locally with hot-reload
agentcore deploy    # deploy to AWS via CDK
agentcore status    # deployment status
agentcore invoke    # invoke the agent (local or deployed)
agentcore eval      # run evaluations
```

---

<sub>Built by <a href="https://github.com/n0zzzy">Russel Carlo</a> · BeSA Cloud Academy, <i>Agentic AI: PoC to Production on AWS</i></sub>
