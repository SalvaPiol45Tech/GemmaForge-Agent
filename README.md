# Gemma 4 SWE Agent

> **Autonomous Repository-Level Software Engineering Agent**

An experimental AI software engineer built around **Gemma 4**, designed to solve real repository-level software engineering tasks by understanding an issue, navigating a codebase, reasoning about the relevant implementation, applying a minimal patch, running tests, analyzing failures, and submitting a validated solution.

The project combines **LLM reasoning, tool use, repository navigation, graph-based code exploration, LoRA adapters, supervised fine-tuning, and test-driven validation** into a single software-engineering agent.

---

## Overview

Most coding assistants focus on generating code from a prompt.

This project targets a harder problem:

> **Can an AI agent autonomously modify an existing software repository to solve a real engineering issue?**

The agent is designed to operate directly on repositories rather than isolated code snippets.

Given an issue, the agent follows a structured loop:

```text
Software Issue
      │
      ▼
Understand Problem
      │
      ▼
Inspect Repository
      │
      ▼
Locate Relevant Code
      │
      ▼
Analyze Dependencies
      │
      ▼
Form Root-Cause Hypothesis
      │
      ▼
Plan Minimal Fix
      │
      ▼
Edit Code
      │
      ▼
Run Targeted Tests
      │
      ▼
Analyze Failures
      │
      ├──── failure ────► revise hypothesis
      │                         │
      └─────────────────────────┘
      │
      ▼
Regression Validation
      │
      ▼
Submit Patch
```

The core principle is simple:

> **Do not guess. Inspect the repository, make the smallest justified change, and validate it with tests.**

---

# Key Features

* 🤖 **Gemma 4-based SWE agent**
* 🧠 Repository-level reasoning
* 🔎 Code search and symbol localization
* 🕸️ Code-neighbor and code-subgraph analysis
* 🛠️ Tool-using autonomous agent
* 🧩 Specialized repository-navigation skill
* 🎯 Minimal-patch strategy
* 🧪 Test-driven validation
* 🔄 Failure analysis and iterative correction
* 🎓 SWE-Bench-based training data
* 🏋️ Supervised fine-tuning pipeline
* ⚡ LoRA / PEFT adapters
* 👥 Specialized sub-agent architecture
* 📦 Competition-ready submission structure

---

# Architecture

## Agent Architecture

```text
                    ┌─────────────────────┐
                    │   Software Issue    │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  Problem Analysis   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Repository Status   │
                    │     & Context       │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    Code Search      │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Source Inspection   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Dependency / Graph  │
                    │      Analysis       │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Root-Cause Reasoning│
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Minimal Patch     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Test Execution    │
                    └──────────┬──────────┘
                               │
                       ┌───────┴───────┐
                       │               │
                    PASS            FAILURE
                       │               │
                       │               ▼
                       │      Failure Analysis
                       │               │
                       │               └──► Iterate
                       │
                       ▼
                Regression Checks
                       │
                       ▼
                 Submit Patch
```

---

# Repository Navigation

The agent does not blindly search the entire repository.

It progressively narrows the problem:

```text
Issue
  ↓
Named symbol / error / behavior
  ↓
search_similar_code()
  ↓
read_file()
  ↓
get_code_neighbors()
  ↓
get_code_subgraph()
  ↓
Root-cause hypothesis
  ↓
Minimal implementation
```

The repository-navigation skill explicitly treats graph information as **localization evidence**, while requiring the actual source code to be inspected before editing.

This helps reduce unnecessary repository exploration and preserves context for the main reasoning agent.

---

# Tooling

The agent is designed around a software-engineering toolset including:

| Tool                  | Purpose                            |
| --------------------- | ---------------------------------- |
| `get_status`          | Inspect repository state           |
| `search_similar_code` | Locate relevant implementation     |
| `read_file`           | Inspect source code                |
| `get_code_neighbors`  | Explore related symbols            |
| `get_code_subgraph`   | Analyze local dependency structure |
| `edit_file`           | Apply source changes               |
| `write_file`          | Create or update files             |
| `run_command`         | Execute tests and commands         |
| `submit_patch`        | Submit the validated patch         |

The agent is instructed to use these tools deliberately rather than exploring indefinitely.

---

# Training

The training pipeline uses real software-engineering tasks rather than synthetic programming questions.

## SWE-Bench Data

The development pipeline loads **SWE-bench Lite** and converts repository issues into structured agent trajectories.

The dataset contains:

* Repository
* Base commit
* Problem statement
* Expected patch
* Tests that should transition from failing to passing
* Regression tests that should remain passing
* Structured agent trajectory

The pipeline successfully generated:

```text
Examples: 300
Valid JSON records: 300
Invalid records: 0
```

The resulting SFT dataset contains:

```text
Records: 300
SFT records: 300
```

---

# Training Trajectory

Each task is represented as a structured software-engineering trajectory:

```text
1. Understand Issue
2. Inspect Repository
3. Search Code
4. Read Implementation
5. Reason About Root Cause
6. Edit Code
7. Run Failing Tests
8. Run Regression Tests
9. Submit Patch
```

Example:

```json
{
  "step": 1,
  "type": "understand_issue"
}
```

followed by repository inspection, code search, source reading, editing, validation, and submission.

This gives the model a structured representation of **how an engineer solves a repository-level problem**, rather than simply showing the final patch.

---

# Supervised Fine-Tuning

The SWE-Bench trajectories are converted into conversational SFT records containing:

```text
SYSTEM
   ↓
USER: Repository + Issue
   ↓
ASSISTANT: Repository inspection
   ↓
TOOL: get_status
   ↓
ASSISTANT: Code search
   ↓
TOOL: search_similar_code
   ↓
ASSISTANT: Source inspection
   ↓
TOOL: read_file
   ↓
ASSISTANT: Minimal fix
   ↓
TOOL: edit_file
   ↓
TOOL: run_command
   ↓
Regression validation
   ↓
TOOL: submit_patch
```

## The generated records contain the repository metadata, conversational trajectory, gold patch, and validation tests.

# Dataset Quality Control

Before training, the dataset goes through quality control:

### Validation

Required fields include:

* `instance_id`
* `repo`
* `base_commit`
* `messages`
* `gold_patch`
* `validation`

### Deduplication

Instances are deduplicated using their `instance_id`.

### Repository-Level Split

The dataset uses a repository-level train/validation split when multiple repositories are available.

This prevents the same repository from appearing in both training and validation sets.

Final development split:

```text
Total records:       300
Training records:    288
Validation records:   12
Repositories:         12

Split mode: repository_level
Seed: 42
Validation ratio: 20%
```

---

# LoRA / PEFT

The agent architecture supports parameter-efficient fine-tuning through LoRA adapters.

The final submission uses separate adapters for different agent roles:

```text
adapters/
├── main_lora/
│   ├── adapter_config.json
│   └── adapter_model.safetensors
│
└── tool_lora/
    ├── adapter_config.json
    └── adapter_model.safetensors
```

The main adapter can specialize the primary coding agent while the tool adapter can specialize a read-only analysis agent.

This enables multiple specialized behaviors while retaining a single shared base model.

---

# Multi-Agent Architecture

The system supports a specialized analyzer sub-agent:

```text
                 ┌─────────────────────┐
                 │    Main SWE Agent    │
                 │                     │
                 │ Reason → Edit → Test│
                 └──────────┬──────────┘
                            │
                            │ delegates
                            ▼
                 ┌─────────────────────┐
                 │   Code Analyzer      │
                 │                     │
                 │ Search / Navigation │
                 │ Dependency Analysis │
                 └─────────────────────┘
```

The analyzer is intentionally focused on repository exploration so that large intermediate search results do not unnecessarily consume the main agent's context.

---

# Base Model

The declared deployment model is:

```text
gemma-4-31b-it-qat-w4a16-ct
```

The competition environment uses the Gemma 4 model through the provided inference infrastructure rather than requiring the submission to bundle the base model itself.

The project architecture therefore separates:

```text
Base Model
     +
LoRA Adapter
     +
Agent Instructions
     +
Tools
     +
Skills
     =
SWE Agent
```

---

# Agent Configuration

The primary agent is configured with:

```yaml
name: gemma4_swe_agent

model:
  name: gemma-4-31b-it-qat-w4a16-ct

adapter: main_lora
```

and the repository engineering tools:

```text
run_command
submit_patch
get_status
read_file
edit_file
write_file
get_code_neighbors
search_similar_code
get_code_subgraph
```

The architecture also supports specialized sub-agents and configurable generation parameters.

---

# Engineering Philosophy

The agent follows several core principles.

### 1. Repository Evidence Over Guessing

The model must inspect the actual implementation before changing code.

### 2. Minimal Patches

Only the smallest necessary change should be introduced.

### 3. Preserve Existing APIs

Existing interfaces should not be changed unless the issue requires it.

### 4. Test Before Submission

A patch is not considered complete until the relevant tests have been executed.

### 5. Failure-Driven Iteration

When tests fail:

```text
Traceback
   ↓
Locate responsible code
   ↓
Update hypothesis
   ↓
Apply correction
   ↓
Run test again
```

### 6. Submit Last

`submit_patch()` is the final action after implementation and validation.

---

# Example Reasoning Flow

A repository issue such as:

```text
A function produces incorrect behavior for nested
CompoundModels.
```

should result in:

```text
1. Understand reported behavior
2. Locate separability implementation
3. Inspect surrounding implementation
4. Search callers / related functions
5. Form root-cause hypothesis
6. Modify the smallest relevant section
7. Run failing tests
8. Run regression tests
9. Submit patch
```

The training pipeline explicitly encodes this workflow.

---

# Evaluation

The fundamental evaluation criterion is not simply whether the model produces plausible code.

The real question is:

> **Does the generated repository patch actually solve the issue without breaking existing behavior?**

Therefore evaluation is based around:

```text
Issue
  ↓
Agent Execution
  ↓
Repository Patch
  ↓
Targeted Tests
  ↓
Regression Tests
  ↓
Patch Validation
```

A successful solution should:

* Address the reported issue
* Make the relevant tests pass
* Preserve regression behavior
* Avoid unrelated modifications
* Produce a valid repository patch

---

# Competition Submission

The final submission package contains:

```text
submission.zip
│
├── agent.yaml
│
├── configs/
│   └── sampling.yaml
│
├── prompts/
│   ├── system.md
│   └── analyzer.md
│
├── sub_agents/
│   └── code_analyzer.yaml
│
├── skills/
│   └── repo_navigation/
│       └── SKILL.md
│
└── adapters/
    ├── main_lora/
    │   ├── adapter_config.json
    │   └── adapter_model.safetensors
    │
    └── tool_lora/
        ├── adapter_config.json
        └── adapter_model.safetensors
```

The final packaging stage verified that all required submission files were present.

---

# Project Structure

```text
gemma4_swe_submission/
│
├── agent.yaml
│
├── configs/
│   └── sampling.yaml
│
├── prompts/
│   ├── system.md
│   └── analyzer.md
│
├── sub_agents/
│   └── code_analyzer.yaml
│
├── skills/
│   └── repo_navigation/
│       └── SKILL.md
│
└── adapters/
    ├── main_lora/
    │   ├── adapter_config.json
    │   └── adapter_model.safetensors
    │
    └── tool_lora/
        ├── adapter_config.json
        └── adapter_model.safetensors
```

---

# Research Directions

This project provides a foundation for further research into autonomous software engineering.

Potential extensions include:

* Better repository retrieval
* More reliable code-graph reasoning
* Longer-horizon planning
* Self-debugging
* Automatic test generation
* Patch ranking
* Critic / verifier agents
* Reinforcement learning from test outcomes
* Preference optimization
* Multi-agent SWE systems
* Repository memory
* Historical commit reasoning
* Automated regression discovery
* Patch quality scoring
* Execution-aware code reasoning

---

# Roadmap

```text
[x] Agent architecture
[x] Repository-level workflow
[x] Tool integration
[x] SWE-Bench data ingestion
[x] Trajectory generation
[x] SFT dataset generation
[x] Dataset quality control
[x] Repository-level train/validation split
[x] LoRA adapter integration
[x] Specialized code-analysis agent
[x] Repository-navigation skill
[x] Final submission packaging

[ ] Larger-scale SWE training
[ ] Stronger tool-use supervision
[ ] Test-result reward modeling
[ ] Reinforcement learning
[ ] Self-reflection / verification
[ ] Multi-agent planning
[ ] Larger SWE benchmark evaluation
```

---

# Technical Stack

```text
Python
PyTorch
Hugging Face Datasets
PEFT / LoRA
Gemma 4
Google ADK
vLLM
SWE-Bench
JSONL
YAML
Git
```

---

# What This Project Demonstrates

This project demonstrates the transition from:

```text
LLM
```

to:

```text
LLM
+
Tools
+
Repository Understanding
+
Code Search
+
Planning
+
Execution
+
Testing
+
Failure Recovery
+
Patch Validation
```

The goal is not to build another code-completion model.

The goal is to build an **autonomous software-engineering system capable of interacting with real codebases and producing validated changes**.

---

# Status

**Research / Engineering Prototype**

The system has a complete agent architecture, SWE-Bench data pipeline, SFT preparation pipeline, repository-navigation capabilities, LoRA adapter structure, specialized analysis agent, and competition-ready submission packaging.

The project should be considered an experimental autonomous SWE system rather than a claim of fully autonomous general software engineering.

---

## Author

**Salva Piol**

AI Research Engineer / AI Engineer

Focus areas:

* AI Agents
* Large Language Models
* Deep Learning
* Reasoning & Planning
* Multimodal AI
* Reinforcement Learning
* Autonomous Software Engineering
* Production AI Systems

---
