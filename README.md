# Agentic AI for DevOps — Masterclass

A simple **LangChain AI Agent** that uses a **local Large Language Model (LLM)** running through **Ollama** to answer **Kubernetes** and **Docker** questions by executing real **kubectl** and **docker** commands as tools.

This project demonstrates how to build an **Agentic AI** application for DevOps engineers, where the AI can interact with real infrastructure instead of only generating text.

Built as part of the **Masterclass**.

---

# Table of Contents

- Introduction
- Features
- What the Agent Does
- Architecture
- How the Agent Works
- Key Concepts
- Prerequisites
- Installation
- Running the Project
- Example Prompts
- Choosing an LLM
- Why `num_ctx` Matters
- Troubleshooting
- Project Structure
- Future Improvements
- License

---

# Introduction

Traditional Large Language Models can only answer questions based on their training data.

An **AI Agent** goes one step further.

Instead of guessing answers, it can:

- Execute Linux commands
- Query Kubernetes
- Inspect Docker
- Read files
- Access APIs
- Make decisions
- Return live information

This project demonstrates that workflow using **LangChain Agents**.

Instead of answering from memory, the model decides which tool to use, runs the tool, reads its output, and then generates the final response.

---

# Features

- Local LLM using Ollama
- LangChain Agent
- Kubernetes Tool Integration
- Docker Tool Integration
- Real-time Infrastructure Inspection
- Natural Language Interface
- Tool Calling
- Modular Design
- Easy to Extend

---

# What the Agent Does

The agent has two built-in tools.

## 1. get_pods

Runs:

```bash
kubectl get pods -A
```

Returns:

- Namespace
- Pod Name
- Status
- Ready
- Restart Count
- Age

---

## 2. get_docker_containers

Runs:

```bash
docker ps
```

Returns:

- Container ID
- Image
- Name
- Status
- Ports

---

You simply ask questions in plain English.

Example:

> Show me all running pods.

or

> Which Docker containers are currently running?

The LLM decides which tool should be executed.

---

# Architecture

```
                    User
                     │
                     ▼
           Natural Language Question
                     │
                     ▼
             LangChain Agent
                     │
          Reads Tool Descriptions
                     │
                     ▼
          Chooses Appropriate Tool
             │                  │
             │                  │
             ▼                  ▼
 kubectl get pods -A      docker ps
             │                  │
             └──────────┬───────┘
                        ▼
              Live Command Output
                        │
                        ▼
                 Local LLM (Ollama)
                        │
                        ▼
                Final Human Answer
```

---

# How the Agent Works

The execution flow is simple.

```
User Question
      │
      ▼
LLM examines every available tool
      │
      ▼
Chooses the correct tool
      │
      ▼
Python executes kubectl/docker command
      │
      ▼
Returns live output
      │
      ▼
LLM interprets the output
      │
      ▼
Final Answer
```

**Important**

The LLM **never executes shell commands directly**.

Instead:

- Python executes commands
- Python returns output
- LLM only reasons over that output

This makes the design much safer.

---

# Key Concepts (30-Second Glossary)

| Term | Description |
|------|-------------|
| Agent | An LLM combined with tools that can decide what action to take. |
| Tool | A Python function callable by the LLM. |
| Tool Docstring | The description read by the LLM to understand when to use a tool. |
| LLM | The reasoning model (Qwen, Llama, etc.). |
| Ollama | Runs open-source LLMs locally and exposes an API. |
| LangChain | Framework for building AI agents and workflows. |
| System Prompt | Instructions defining the agent's behavior. |
| Temperature | Controls randomness. `0` is deterministic and ideal for tool usage. |
| num_ctx | Context window size in tokens. Larger values allow the model to process bigger command outputs. |

---

# Prerequisites

Before running the project, ensure the following are installed.

## Python

Python 3.10+

Verify:

```bash
python3 --version
```

---

## Ollama

Install Ollama from:

https://ollama.com

Verify:

```bash
ollama --version
```

---

## Kubernetes CLI

Install kubectl.

Verify:

```bash
kubectl version --client
```

Your kubeconfig should already point to a working cluster.

Test:

```bash
kubectl get pods -A
```

---

## Docker

Install Docker Desktop (Windows/macOS) or Docker Engine (Linux).

Verify:

```bash
docker ps
```

---

# Installation

## Step 1

Clone the repository.

```bash
git clone <repository-url>
```

---

## Step 2

Navigate into the project.

```bash
cd Agentic-AI-DevOps
```

---

## Step 3

Create a virtual environment.

```bash
python3 -m venv venv
```

---

## Step 4

Activate it.

Linux/macOS

```bash
source venv/bin/activate
```

Windows

```powershell
venv\Scripts\activate
```

---

## Step 5

Install dependencies.

```bash
pip install -r requirements.txt
```

---

## Step 6

Start Ollama.

```bash
ollama serve
```

---

## Step 7

Download the model.

```bash
ollama pull qwen3-coder:30b
```

This is a one-time download (~18 GB).

---

## Step 8

Verify installed models.

```bash
ollama list
```

---

# Running the Agent

Launch the application.

```bash
python agent.py
```

You should see something like:

```
Ask your Kubernetes Agent a Question:
>
```

---

# Example Prompts

## Kubernetes

```
Show me all pods.
```

```
Which pods are failing?
```

```
How many namespaces exist?
```

```
List all CrashLoopBackOff pods.
```

```
Show me pods in kube-system.
```

---

## Docker

```
What Docker containers are running?
```

```
List all running containers.
```

```
Which container exposes port 8080?
```

```
How many containers are active?
```

---

# Choosing a Model

The model can be changed by modifying the `model=` parameter inside `agent.py`.

| Model | Best For | Approx. Size |
|--------|----------|--------------|
| qwen3-coder:30b | Best overall tool calling | ~18 GB |
| qwen3:8b | Balanced performance | ~5 GB |
| llama3.2 | Low-memory systems | ~2 GB |

Example:

```python
model="qwen3:8b"
```

Then download:

```bash
ollama pull qwen3:8b
```

---

# Why `num_ctx` Matters

Ollama has a limited context window.

If your Kubernetes cluster contains hundreds of pods, the output of:

```bash
kubectl get pods -A
```

may exceed the default context size.

When that happens:

- the output is truncated,
- the model only sees part of the data,
- answers may be incomplete or inaccurate.

Setting:

```python
num_ctx = 8192
```

(or larger for bigger clusters) allows the model to process the full command output.

---

# Troubleshooting

| Problem | Solution |
|----------|----------|
| Connection refused | Start Ollama using `ollama serve`. |
| Model not found | Run `ollama pull qwen3-coder:30b`. |
| Empty pod list | Verify `kubectl get pods -A` works manually. |
| No Docker containers | Ensure Docker is running and `docker ps` returns data. |
| ImportError: create_agent | Upgrade to LangChain 1.x using `pip install -U -r requirements.txt`. |

---

# Project Structure

```
.
├── agent.py
├── requirements.txt
└── README.md
```

### agent.py

Contains:

- LangChain Agent
- Tool Definitions
- Prompt
- Ollama LLM
- Main Program

---

### requirements.txt

Contains Python dependencies.

Example:

```
langchain
langchain-community
langchain-ollama
ollama
```

---

### README.md

Project documentation.

---

# Future Improvements

Potential enhancements include:

- Kubernetes Logs Tool
- Pod Description Tool
- Node Information Tool
- Helm Integration
- Terraform Tool
- AWS CLI Tool
- Azure CLI Tool
- GCP CLI Tool
- GitHub Tool
- Jenkins Tool
- Prometheus Metrics Tool
- Grafana Integration
- Slack Notifications
- Multi-Agent Architecture
- Memory Support
- RAG (Retrieval-Augmented Generation)
- Autonomous Incident Response

---

# Learning Outcomes

After completing this project, you will understand:

- Agentic AI fundamentals
- LangChain Agents
- Tool Calling
- Ollama integration
- Local LLM deployment
- Kubernetes automation
- Docker automation
- Prompt Engineering
- AI-assisted DevOps workflows

---
