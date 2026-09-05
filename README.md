# vb-ai-migration-pipeline
````markdown
# VB → C#/Java AI Code Migration Pipeline

## Overview

This repository implements a **GitOps-driven, AI-assisted CI/CD pipeline** that converts **VB (.vb)** source code into **C# or Java** using an LLM (OpenAI / CodeLlama).  
All generated code is validated through **containerized compilation**, enforced via **Pull Requests**, and promoted only when it compiles successfully.

This is a **production-oriented pipeline**, not a demo:
- AI output is treated as **untrusted**
- Every change is **reviewable**
- Git history is the **audit trail**
- Builds are **reproducible**

---

## What This Pipeline Does

1. Developer pushes VB code to `vb_banking_v1`
2. GitHub Actions triggers conversion workflow
3. `.vb` files are converted **sequentially (one-by-one)** using LLM
4. Generated code is committed to a dedicated branch
5. A Pull Request is opened for review
6. Compilation runs inside Docker on GitHub-hosted runners
7. Only successful builds are merged to the release branch

---

## Branch Strategy

| Branch | Purpose |
|------|--------|
| `vb_banking_v1` | Source VB code |
| `cs_generated_v1` | AI-generated C# |
| `java_generated_v1` | AI-generated Java |
| `compiled_release_v1` | Verified, compilable output (protected) |

**Rules**
- No direct push to generated or release branches
- Pull Request required for all generated code
- `compiled_release_v1` is protected

---

## Repository Structure

```bash
.
├── .github/workflows/
│   ├── vb-convert.yml        # LLM conversion pipeline
│   └── compile.yml           # Docker-based compilation
├── config/
│   └── conversion.yaml       # Pipeline configuration
├── prompts/
│   ├── vb-to-csharp-v1.md    # C# prompt template
│   └── vb-to-java-v1.md      # Java prompt template
├── scripts/
│   ├── convert_file.py       # LLM conversion logic
│   ├── validate_output.py    # Output validation
│   ├── create_pr.sh          # Branch + PR creation
│   └── notify.sh             # Failure notifications
├── src/
│   └── vb/                   # VB source code
├── artifacts/                # Logs & build outputs
└── README.md
````

---

## Prerequisites

* GitHub repository with Actions enabled
* OpenAI or CodeLlama API access
* GitHub-hosted runners (no self-hosted infra required)
* Docker (used inside GitHub runners)
* Python 3.11 (via GitHub Actions)

---

## Secrets Configuration

Configure secrets in:

**GitHub → Settings → Secrets → Actions**

| Secret           | Description               |
| ---------------- | ------------------------- |
| `OPENAI_API_KEY` | LLM API access            |
| `SMTP_USER`      | Email sender (optional)   |
| `SMTP_PASS`      | Email password (optional) |

Secrets must never be committed.

---

## Configuration

### `config/conversion.yaml`

Controls:

* Target language
* LLM behavior
* Retry policy
* Compilation container & command

Example:

```yaml
target_language: csharp

llm:
  provider: openai
  model: gpt-4o
  temperature: 0
  max_tokens: 4000

execution:
  sequential: true
  max_retries: 2

compile:
  csharp:
    docker_image: mcr.microsoft.com/dotnet/sdk:8.0
    command: dotnet build
```

---

## Workflows

### 1. VB Conversion Workflow

**Trigger:** Push to `vb_banking_v1`

**Flow:**

* Discover `.vb` files
* Convert files sequentially via LLM
* Validate generated output
* Create generated branch
* Open Pull Request

Workflow file:

```
.github/workflows/vb-convert.yml
```

---

### 2. Compilation Workflow

**Trigger:** Pull Request to `cs_generated_v1` or `java_generated_v1`

**Flow:**

* Checkout generated code
* Compile inside Docker container
* Pass or fail the PR check

Workflow file:

```
.github/workflows/compile.yml
```

---

## Validation & Safety

* LLM output is validated before commit
* Conversion halts on first failure
* Compilation runs only inside Docker
* Release branch accepts only successful builds
* No auto-merge of AI-generated code

---

## Failure Handling

Failures may occur at:

* LLM conversion
* Output validation
* Compilation

On failure:

* Pipeline stops immediately
* Logs are captured
* Pull Request is annotated
* Developer notification is triggered

---

## Non-Goals

This pipeline does **not**:

* Guarantee semantic correctness
* Replace developer review
* Auto-merge AI output
* Run AI conversions in parallel

---

## Operating Principles

* AI assists, it does not decide
* GitOps is mandatory
* Compilation is the quality gate
* Humans approve final output

---

## Usage Summary

1. Push VB code to `vb_banking_v1`
2. Review the generated Pull Request
3. Fix issues if compilation fails
4. Merge only after successful build
5. `compiled_release_v1` always contains verified code

---

## Final Notes

This repository is intentionally strict.

Relaxing:

* Pull Request enforcement
* Validation rules
* Containerized compilation

will turn this into a **demo**, not a **production system**.

Use responsibly.

```
```



<!-- Security scan triggered at 2026-09-05 07:21:10 -->