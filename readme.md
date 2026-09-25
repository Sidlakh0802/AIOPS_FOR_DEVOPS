



## Tech Stack

| Category | Tools |
|----------|-------|
| Language | Python 3.10+ |
| LLM (Day 1) | Ollama + Gemma 4 (local, free) |
| LLM (Day 2) | Anthropic Claude (KubeHealer) |
| Containers | Docker |
| Orchestration | Kubernetes (Kind) |
| Agent Framework | LangChain |
| MCP | FastMCP |

---



# Module 0 — Know Before You Go

Get your machine ready before Day 1. Takes about 20 minutes.

## You Should Already Have

- **Docker** — [install](https://docs.docker.com/get-docker/)
- **kubectl** — [install](https://kubernetes.io/docs/tasks/tools/)
- **Kind** — [install](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)

## Step 1: Python 3.10+

```bash
python3 --version
```

Need 3.10+. If not:
- **macOS:** `brew install python@3.12`
- **Ubuntu:** `sudo apt install python3.12 python3.12-venv`
- **Other:** [python.org/downloads](https://www.python.org/downloads/)

## Step 2: Install Ollama

Ollama runs LLMs locally — no API key, no cost.

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Pull the model we'll use:

```bash
ollama pull gemma4
```

Test it:

```bash
ollama run gemma4 "Say hello in one sentence"
```

## Step 3: Clone the Repo

```bash
git clone https://github.com/trainwithshubham/agentic-ai-for-devops.git
cd agentic-ai-for-devops
```

## Step 4: Create a Virtual Environment

A virtual environment keeps this project's dependencies separate from your system Python. Everyone should use one so we're all on the same setup.

```bash
python3 -m venv .venv
source .venv/bin/activate
```

You should see `(.venv)` in your terminal prompt. If you open a new terminal, run `source .venv/bin/activate` again.

Install the dependencies:

```bash
pip install -r requirements.txt
```

## Step 5: Verify

```bash
python3 module-0/verify_setup.py
```

If everything shows `[PASS]` — you're ready for Day 1.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Ollama not running | `ollama serve` (or open Ollama app on macOS) |
| Docker not running | Start Docker Desktop, or `sudo systemctl start docker` |
| Docker permission denied | `sudo usermod -aG docker $USER` then re-login |
| Kind not found | `brew install kind` (macOS) or [install docs](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) |
| Python too old | Install newer version, use `python3.12` instead of `python3` |

---


# Module 1 — Docker Error Explainer

Paste a Docker error, get a plain-English explanation and fix. Your first LLM-powered tool.
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/124a57e1-a08b-49b1-8563-0103fdb09b9d" />


One file: [`GenAI.py`](GenAI.py)

## What You'll Learn

- **Prompt** — the text you send to an LLM. Here, it's the Docker error itself.
- **System role** — a message that sets the LLM's personality ("You are a Docker expert..."). Change it, and the tone of every answer changes.
- **Temperature** — controls randomness. 0.0 = same answer every time, 1.0 = creative. We use 0.3 for reliable advice.
- **Tokens** — LLMs read tokens, not words (~3/4 of a word each). More tokens = slower response.

## Run It

```bash
python3 module-1/explainer.py
```

Paste an error, press Enter twice. Try any of these:

**Port already in use**
```
docker: Error response from daemon: Bind for 0.0.0.0:3000 failed: port is already allocated.
```
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/6b2f9847-0a43-4625-afb3-1b1dc407697b" />

**Image not found**
```
docker: Error response from daemon: pull access denied for myapp, repository does not exist or may require 'docker login'
```
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/8c867c85-bb76-4f0c-a7bf-c4a70e846f60" />

**Permission denied**
```
docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock
```

## Experiment

- Change `temperature` to `1.0` — notice how the answer varies
- Edit the system prompt — try "explain like I'm 5"
- Paste a real error from your own terminal

---

Next: **[Module 2 — Docker Troubleshooter Agent](../module-2/)**


# Module 2 — Docker Troubleshooter Agent

In Module 1, you built a chatbot — it reads text and responds. Now you build an **agent** — it decides what actions to take, runs them, reads the results, and keeps going until it has an answer.

<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/d1563d47-9355-4945-bf16-2120071b857d" />



## What You'll Learn

- **Chatbot vs Agent** — a chatbot answers questions. An agent takes actions. Our agent runs Docker commands on its own to diagnose problems.
- **Tool calling** — the LLM doesn't just generate text. It picks which Python function to call and with what arguments. We give it 3 Docker tools and it decides which ones to use.
- **ReAct pattern** — Reason ("I should check the logs"), Act (call `get_logs`), Observe (read the output), repeat until done. LangChain handles this loop for us.

## The Code

One file: [`DockerAgent.py`](DockerAgent.py)

Three tools defined as plain Python functions:

```python
@tool
def list_containers() -> str:
    """List all Docker containers (running and stopped)."""
    result = subprocess.run(["docker", "ps", "-a"], capture_output=True, text=True)
    return result.stdout or result.stderr
```

The `@tool` decorator tells LangChain "the LLM can call this". The docstring becomes the tool's description — the LLM reads it to decide when to use it.

The agent is created in two lines:

```python
llm = ChatOllama(model="gemma4", temperature=0)
agent = create_react_agent(llm, [list_containers, get_logs, inspect_container])
```

That's it. LangChain wires up the ReAct loop — the LLM reasons, picks a tool, we run it, feed the result back, repeat.

## Try It

First, create a broken container:

```bash
docker run -d --name broken-app nginx:alpine sh -c "echo 'app starting...' && sleep 2 && exit 1"
```

Then run the agent:

```bash
python3 module-2/agent.py
```

Ask it:
- "Why is broken-app crashing?"
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/63469a68-3c39-4474-9360-3cfa05ba3b47" />

- "What containers are running?"
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/7c6140e1-403c-44aa-941b-efee3e27a93a" />

- "Show me the logs for broken-app"
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/3fef40e3-2c2f-41e6-940f-b2de2330214e" />


Clean up when done:

```bash
docker rm -f broken-app
```

## Experiment

- Add a 4th tool — maybe `stop_container(name)` or `restart_container(name)`
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/ba0cc532-d9c3-4058-885e-e8513d0a4191" />

- Run multiple broken containers and ask "which containers have problems?"
- Look at the agent's reasoning — it prints what it's thinking before each tool call

---

Next: **[Module 3 — Multi-Tool DevOps Agent + MCP](../module-3/)**

# Module 3 — Multi-Tool DevOps Agent + MCP

You built a Docker troubleshooter in Module 2. Now you add Kubernetes tools to the same agent, and learn how MCP lets any AI system use your tools — not just LangChain.

## What You'll Learn

- **Multi-environment agent** — one agent that knows Docker AND Kubernetes. It decides which tools to use based on what you ask.
- **Agents, Tools, Chains** — quick LangChain primer. An agent picks tools to call. A tool is a Python function the LLM can invoke. A chain is a fixed sequence of LLM calls (we use agents, not chains, because agents are adaptive).
- **MCP (Model Context Protocol)** — a standard way to expose tools to any LLM system. Write tools once, use them in Claude Desktop, VS Code, Cursor, or anything that speaks MCP.

## The Code

Two files this time:

1. **[`Agentic.py`](Agentic.py)** — LangChain agent with 6 tools (3 Docker + 3 K8s)
2. **[`mcp_server.py`](mcp_server.py)** — MCP server that exposes the K8s tools to Claude Desktop

### agent.py — The Unified Agent

We added 3 Kubernetes tools to the Module 2 agent:

```python
@tool
def list_pods(namespace: str = "default") -> str:
    """List all pods in a Kubernetes namespace with their status."""
    result = subprocess.run(
        ["kubectl", "get", "pods", "-n", namespace],
        capture_output=True, text=True,
    )
    return result.stdout or result.stderr
```

Same pattern as Docker tools — subprocess call, capture output, return the result. The agent now has 6 tools and decides which ones to use based on your question.

### mcp_server.py — Protocol-Based Tools

Instead of LangChain, we use FastMCP — a library that implements the Model Context Protocol:

```python
from fastmcp import FastMCP

mcp = FastMCP("Kubernetes Tools")

@mcp.tool
def list_pods(namespace: str = "default") -> str:
    """List all pods in a Kubernetes namespace with their status."""
    # same kubectl subprocess call
```

The key difference: **framework vs protocol**.

| | agent.py | mcp_server.py |
|---|---|---|
| Library | LangChain | FastMCP (MCP protocol) |
| Works with | LangChain agents only | Claude Desktop, VS Code, Cursor, any MCP client |
| Runs as | Python script with REPL | Background server (stdio) |

MCP is like a REST API for AI tools — a standard interface that any LLM system can speak.

## Try It — Part 1: The Agent

### Step 1: Create a Kind cluster

```bash
kind create cluster --name devops-demo
```

This should auto-switch your kubectl context. Verify:

```bash
kubectl cluster-info
```

You should see `https://127.0.0.1:XXXXX` (local). If it still points to an EKS or other remote cluster, manually export the Kind kubeconfig:

```bash
kind export kubeconfig --name devops-demo
```

### Step 2: Deploy a broken pod

```bash
kubectl apply -f module-3/broken_pod.yaml
```

Wait 15 seconds, then check:

```bash
kubectl get pods
```

You'll see `broken-pod` in `CrashLoopBackOff` — it starts, runs for 2 seconds, exits with code 1, and Kubernetes keeps restarting it.

### Step 3: Create a broken Docker container too

```bash
docker run -d --name broken-container nginx:alpine sh -c "echo 'container starting...' && sleep 2 && exit 1"
```

Now you have problems in both Docker and Kubernetes.

### Step 4: Run the agent

```bash
python3 module-3/agent.py
```

Try these:

- "What pods are running in my cluster?"
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/2c275659-f129-4f2e-8d2e-8f20c0797a95" />

- "Why is broken-pod crashing?"
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/59a1cbe7-db2a-46de-9e92-52e26b08a353" />
- "Show me the events in the default namespace"
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/7570ce26-4640-4639-8f6d-cd0a2f2b1298" />

- "What Docker containers are running?"
- "Why is broken-container failing?"
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/cc7e8cdf-995e-4e90-bf93-bcdb0f408505" />

- "What's broken across Docker and Kubernetes?"
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/34b0f81d-fecb-473f-b563-ab1abe70a1b2" />


Watch it pick K8s tools when you ask about pods, Docker tools when you ask about containers, and both when you ask about everything.

### Clean up

```bash
kubectl delete -f module-3/broken_pod.yaml
docker rm -f broken-container
```

## Try It — Part 2: The MCP Server

<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/82cd70f2-1ecc-4c2b-8d7c-2560f280fdfe" />
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/88e20951-38d5-4b90-8b87-5e42ec540360" />
<img width="3420" height="2214" alt="image" src="https://github.com/user-attachments/assets/a9b504b4-5c2a-43a8-b8cc-29fe73704d34" />


### Step 1: Install FastMCP

```bash
pip install fastmcp
```

### Step 2: Configure Claude Desktop

Open the Claude Desktop config file:

- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Linux**: `~/.config/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

Add this (replace both paths with your actual absolute paths):

```json
{
  "mcpServers": {
    "k8s-tools": {
      "command": "/absolute/path/to/agentic-ai-for-devops/.venv/bin/python3",
      "args": ["/absolute/path/to/agentic-ai-for-devops/module-3/mcp_server.py"]
    }
  }
}
```

Get your repo path with `pwd` in the repo root, then append `.venv/bin/python3` for the command and `module-3/mcp_server.py` for the arg.

**Important:** The command must point to your **venv Python**, not just `python3`. Claude Desktop uses the system Python by default, which won't have fastmcp installed.

### Step 3: Restart Claude Desktop

Fully quit Claude Desktop (Cmd+Q on macOS, not just close the window) and reopen it. You should see a tools indicator showing 3 available tools.

### Step 4: Test it

Make sure the broken pod is still running (or redeploy it):

```bash
kubectl apply -f module-3/broken_pod.yaml
```

In Claude Desktop, ask:

- "List the pods in my cluster"
- "Describe the broken-pod"
- "Show me the events in the default namespace"

Claude calls your MCP server's tools to get the answers. Same tools, different AI system.

### Clean up

```bash
kubectl delete -f module-3/broken_pod.yaml
kind delete cluster --name devops-demo
```

## LangChain Quick Primer

You've been using LangChain since Module 2. Here's what each piece does:

**Tool** — a Python function the LLM can call. The `@tool` decorator registers it and the docstring becomes the tool's description (the LLM reads this to decide when to use it).

**Agent** — the decision-maker. It reads your question, picks a tool, runs it, reads the result, and repeats until it has an answer. This is the ReAct pattern: Reason, Act, Observe. `create_react_agent` sets this up.

**Chain** — a fixed sequence of LLM calls where the output of one feeds into the next. We don't use chains here because agents are smarter — they decide what to do next instead of following a script.

## Why MCP Matters

You just saw the same K8s tools delivered two ways:

1. **LangChain agent** (agent.py) — framework-specific. Only works inside LangChain.
2. **MCP server** (mcp_server.py) — protocol-based. Works with any MCP-compatible client.

Why this matters:

- **Write once, use everywhere.** Your MCP server works with Claude Desktop today, VS Code tomorrow, and whatever comes next.
- **Separation of concerns.** The LLM lives in one place, the tools live in another. Update tools without touching the LLM setup.
- **Local execution.** The MCP server runs on your machine with your kubeconfig. Credentials stay local.

## Troubleshooting

**MCP server shows "Server disconnected"**
- Make sure the `command` in your config points to the venv Python (`.venv/bin/python3`), not system `python3`.
- Test manually: run `.venv/bin/python3 module-3/mcp_server.py` — if it crashes, you'll see the error.

**kubectl points to wrong cluster**
- Run `kubectl config get-contexts` to see all clusters.
- Switch with `kubectl config use-context kind-devops-demo`.
- If the Kind context is missing: `kind export kubeconfig --name devops-demo`.

**Tools not showing in Claude Desktop**
- Fully quit (Cmd+Q on macOS) and reopen. Closing the window isn't enough.
- Check JSON syntax in the config file — one wrong comma breaks it.
- Check logs: `~/Library/Logs/Claude/mcp*.log` on macOS.

## Experiment

- Add a 4th K8s tool: `get_pod_logs(pod_name, namespace)` to get logs from a pod
- Add Docker tools to the MCP server — it doesn't have to be K8s-only
- Deploy multiple broken pods in different namespaces and ask the agent to find them all
- Ask Claude Desktop to "restart the broken pod" — what happens? (It can't, because we didn't give it a restart tool. That's coming in Module 5.)

---

Next: **[Module 4 — AIOps Demystified](../module-4/)**
