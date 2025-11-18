# DevOps Copilot Platform (Local LLM + MCP Server Hub)

**DevOps Copilot Platform** is a local-first DevOps assistant that:

- Runs a **local open-source LLM**, optionally fine-tuned for DevOps.
- Exposes a **DevOps MCP server** that works with any compatible MCP host  
  (Claude Desktop, ChatGPT MCP, VS Code MCP extensions, custom hosts).
- Lets you **plug in other MCP servers** (AWS, Kubernetes, Git, observability, etc.)
  just by adding their command + env config.

> ⚠️ This project is designed for **local / workstation use**. You are responsible for
> IAM, kubeconfigs, and any production access. Start with **read-only** permissions.

---

## 1. Features

- ✅ **Open-Source Local LLM** (no proprietary models)
- ✅ **DevOps-centric fine-tuning pipeline** (LoRA/QLoRA)
- ✅ **MCP server hub**:
  - Internal DevOps tools powered by the LLM.
  - External MCP servers (AWS, Kubernetes, Git, etc.) managed via config.
- ✅ **Host-agnostic**:
  - Connect from Claude, ChatGPT, VS Code MCP, or your own host.
- ✅ Built-in **guardrails**:
  - Read-only vs mutation tools.
  - Human confirmation before risky operations.

---

## 2. Architecture

High-level components:

- `llm_runtime/`  
  Local LLM integration (Ollama, vLLM, llama.cpp, etc.).

- `finetuning/`  
  Data prep + PEFT fine-tuning pipeline for DevOps specialization.

- `devops_skills/`  
  Higher-level DevOps orchestration (K8s analysis, IaC review, CI/CD pipelines).

- `mcp_hub/`  
  MCP server hub that:
  - Exposes DevOps tools to MCP hosts.
  - Manages external MCP servers via config.

- `docs/`  
  Integration recipes and design docs.

---

## 3. Prerequisites

### 3.1 System Requirements

- **OS**: Linux / macOS (Windows via WSL recommended)
- **Python**: 3.10+
- **Node.js** (optional, for some host examples): 18+
- **Git**

### 3.2 Local LLM Runtime

Pick **one** runtime (you can swap later):

1. **Ollama**
   - Install from the official site.
   - After install, pull a model, e.g.:

     ```bash
     ollama pull mistral
     ```

2. **vLLM / llama.cpp**
   - Follow their installation docs.
   - Make sure you know your model path and how to run inference.

> The repo assumes **Ollama** by default for simplicity.

---

## 4. Quickstart

### 4.1 Clone the Repo

```bash
git clone https://github.com/your-org/devops-copilot-platform.git
cd devops-copilot-platform
```

### 4.2 Create a Virtual Environment

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install --upgrade pip
```

### 4.3 Install Python Dependencies

```bash
pip install -r requirements.txt
```

Typical dependencies (simplified):

- fastapi or uvicorn (if we expose HTTP helper endpoints)
- pydantic
- transformers, peft, accelerate (for fine-tuning)
- mcp / jsonrpc client/server libs
- typer or click (for CLI)

### 4.4 Configure Local LLM

Create `config/llm.config.json`:

```json
{
  "backend": "ollama",
  "model": "mistral",
  "temperature": 0.2,
  "max_tokens": 2048,
  "context_length": 8192
}
```

If using a different backend, adjust accordingly.

---

## 5. Running the DevOps MCP Server Hub

The MCP server hub exposes all DevOps tools and connected external MCP servers to any MCP host.

### 5.1 MCP Hub Config

Create `config/mcp.config.json`:

```json
{
  "servers": {
    "devops-llm": {
      "type": "internal",
      "tools": [
        "devops_analyze_k8s_state",
        "devops_review_terraform",
        "devops_ci_pipeline_assistant",
        "devops_incident_summary"
      ]
    },

    "aws-api-mcp": {
      "type": "external",
      "command": "python",
      "args": ["-m", "awslabs.aws_api_mcp_server.server"],
      "env": {
        "AWS_REGION": "ap-south-1",
        "READ_OPERATIONS_ONLY": "true"
      }
    }

    // You can add other MCP servers here, e.g. Kubernetes, Git, etc.
  }
}
```

### 5.2 Start the MCP Hub

```bash
python -m mcp_hub.server
```

This process:

1. Loads `llm.config.json` and initializes the local DevOps model.
2. Reads `mcp.config.json` to:
   - Register internal DevOps tools.
   - Prepare commands for external MCP servers.
3. Listens on stdio for MCP JSON-RPC (per MCP spec).

---

## 6. Using the DevOps MCP Server from Different Hosts

This project doesn’t ship its own rich UI (on purpose). Instead, you connect to it from your preferred MCP host.

### 6.1 VS Code MCP (GitHub Copilot / MCP-compatible extension)

Create `.vscode/mcp.json` inside your project:

```json
{
  "servers": {
    "devops-copilot-platform": {
      "command": "python",
      "args": [
        "-m",
        "mcp_hub.server"
      ],
      "env": {
        "DEVOPS_COPILOT_CONFIG": "config/mcp.config.json",
        "LLM_CONFIG": "config/llm.config.json"
      }
    }
  }
}
```

Then:

1. Open VS Code in this folder.
2. Ensure your MCP-aware extension is installed.
3. The extension should detect `devops-copilot-platform` as a server and allow you to enable it.

In Copilot chat, you can now say, for example:

> “Use the `devops_review_terraform` tool from the DevOps MCP server to review the Terraform in this repo and highlight potential cost and reliability issues.”

### 6.2 Claude Desktop / ChatGPT (MCP Hosts)

Use the UI to add a new MCP server with:

- **Command**: `python`
- **Args**: `["-m", "mcp_hub.server"]`
- **Env**:
  - `DEVOPS_COPILOT_CONFIG=config/mcp.config.json`
  - `LLM_CONFIG=config/llm.config.json`

Then ask something like:

> “Connect to the DevOps MCP server and:
>
> - Use the AWS MCP server to list EC2 instances in ap-south-1.
> - Use your DevOps LLM tools to classify which ones look underutilized and propose optimization actions.”

---

## 7. Fine-Tuning the DevOps Model

The fine-tuning pipeline lives in `finetuning/`.

### 7.1 Data Layout

Place your training data here:

```
finetuning/data/
  ├── public/
  │   ├── k8s_scenarios.jsonl
  │   ├── terraform_qa.jsonl
  │   └── ci_cd_patterns.jsonl
  └── private/
      ├── runbooks.jsonl
      └── incidents.jsonl
```

Each `.jsonl` entry should contain a structured record, e.g.:

```json
{
  "task_type": "k8s_debug",
  "input": "kubectl describe pod ...",
  "output": "Likely cause is OOMKilled due to ..."
}
```

### 7.2 Run a Basic LoRA Fine-Tuning

From the project root:

```bash
cd finetuning
python train_lora.py \
  --base_model mistral \
  --output_dir ./artifacts/devops-lora \
  --train_file data/public/k8s_scenarios.jsonl \
  --max_steps 2000
```

The script should:

1. Load the base open-source model.
2. Apply LoRA adapters.
3. Save the resulting adapter weights.

### 7.3 Export for Serving

Depending on your runtime (Ollama, vLLM, llama.cpp), use provided scripts to:

- Merge LoRA weights into a full model.
- Or configure the runtime to load base + adapters.

Update `config/llm.config.json` to point to the new model.

---

## 8. Security & Permissions

- **Cloud access**: Use AWS profiles (`~/.aws/credentials`) or env vars for the AWS MCP server.
- **Start with read-only IAM policies** (e.g., ReadOnlyAccess for AWS, view roles for K8s).
- **Kubernetes**: Use kubeconfigs with limited RBAC scopes.
- **Tool policies**: In future releases, `mcp_hub` will support a `policy.config.json` for allow/deny rules per tool.

---

## 9. Roadmap

- Add K8s MCP server integration recipe.
- Add Git MCP server for branch status and PR analysis.
- Implement execution plan previews for mutation tools.
- Add a minimal web UI as an optional host for local use.
- Publish pre-tuned DevOps model configs for popular runtimes.

---

## 10. Contributing

1. Fork the repo.
2. Create a feature branch.
3. Write tests where applicable.
4. Submit a PR with a clear description of:
   - New DevOps skill
   - New MCP integration
   - Improvements to LLM tuning / evaluation.

---

## 11. Disclaimer

This project is provided as-is. You are responsible for:

- Protecting credentials and secrets.
- IAM and RBAC policy design.
- Reviewing and validating any changes before applying them to production.
