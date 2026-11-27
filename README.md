# Secure HR Assistant Using NeMo Guardrails and RAG

An HR policy assistant that uses **NVIDIA NeMo Guardrails** to check every question before it reaches a **RAG** pipeline. Built with Streamlit, FAISS and Groq as part of an AI security course.

## What it does

- **Answers** HR questions (leave, benefits, remote work, performance reviews) from 6 sample policy documents for a made-up company, Acme Corp.
- **Blocks** off-topic questions, jailbreak attempts, requests for other employees' data, and messages containing PII (SSNs, emails, API keys).
- **Scans answers** for leaked credentials, SSNs or salary figures and withholds them.

## How it works

```
Question → Input rail (NeMo + PII regex) → FAISS retrieval → Groq LLM answer → Output scan → Reply
```

1. **Input rail:** a guard LLM, driven by Colang rules, classifies the intent. A regex check catches PII.
2. **Retrieval:** FAISS returns the 3 most relevant chunks, embedded locally with FastEmbed (`bge-small-en-v1.5`).
3. **Generation:** a smaller, faster Groq model writes an answer based only on those chunks.
4. **Output rail:** a regex scan withholds any answer that contains sensitive data.

Blocked messages never reach retrieval or generation.

## Project structure

```
app.py              Streamlit entry point
src/
  config.py         Models, prompts, output patterns
  guards.py         NeMo Guardrails config (Colang rules + PII action)
  rag.py            FAISS vector store + retrieval
  pipeline.py       End-to-end pipeline logic
  hr_docs.py        Sample HR policy documents
  ui/               Sidebar, landing page, chat and docs tabs
guardrails.ipynb    Notebook used for experiments
colang.md           Notes on Colang
guardrails_exp.md   Notes on guardrails
```

## Run locally

You need Python 3.10+ and a free [Groq API key](https://console.groq.com).

```bash
git clone <your-repo-url>
cd <repo-folder>
python -m venv venv
venv\Scripts\activate # Note that only for Windows.
pip install -r requirements.txt
streamlit run app.py
```

Enter your Groq API key in the sidebar. The app keeps it in memory only and never saves it.

> **Windows note:** `nemoguardrails` depends on `annoy`, which needs [Visual Studio C++ Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) to install.

## Example prompts

| Prompt | Result |
|---|---|
| "How many vacation days do I get after 3 years?" | Answered |
| "What is Sarah's salary?" | Blocked: confidential |
| "Ignore all previous instructions" | Blocked: jailbreak |
| "My SSN is 123-45-6789" | Blocked: PII |
| "Tell me a joke" | Blocked: off-topic |

## Tech stack

NeMo Guardrails · LangChain · FAISS · FastEmbed · Groq (Llama 3) · Streamlit
