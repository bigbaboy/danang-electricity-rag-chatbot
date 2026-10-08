# Electricity Customer Advisory Chatbot

A Vietnamese-language academic prototype that combines **document-grounded question answering** with **deterministic electricity calculations**.

**Author:** Le Minh Dat · **Context:** Individual graduation project, University of Economics – The University of Danang. Proposed and developed during an internship in the Business Department at Da Nang Power Company.

> This is a graduation-project prototype, not an official EVN customer service system. It is not connected to EVN customer accounts or production APIs.

## Problem and scope

Electricity customers need to navigate lengthy documents and understand billing or consumption estimates. This project explores a single interface for document lookup, electricity bill calculation, appliance consumption analysis and solar payback estimation.

The application is organized into five tabs:

| Feature | Implemented behavior | Main files |
| --- | --- | --- |
| Document assistant | Retrieve document passages and generate answers with source references. | `tabs/chat.py`, `rag_pipeline.py` |
| Electricity billing | Apply the configured tiered tariff and compare household-splitting scenarios. | `tabs/tien_dien.py`, `utils.py` |
| Consumption analysis | Estimate electricity use from appliance inputs and display charts. | `tabs/tieu_thu.py` |
| Solar estimates | Estimate output, self-consumption and payback using configured assumptions. | `tabs/solar.py`, `utils.py` |
| Document management | Read uploaded PDFs and build or extend the FAISS index. | `tabs/docs.py`, `doc_pdf_smart.py`, `db_manager.py` |

## My work

I proposed the topic, defined the scope and intended users, documented functions and processing flows, and implemented the web prototype. The repository includes PDF ingestion, vector retrieval, LLM integration, calculation utilities and automated tests. The project connects a customer-service problem with software functionality and traceable source material.

## How it works

1. Read text PDFs with PyMuPDF; optionally use LlamaParse for scanned documents.
2. Split documents into overlapping chunks and generate multilingual embeddings.
3. Store vectors and metadata in FAISS.
4. Route recognized bill-calculation questions to Python calculation functions.
5. For other questions, retrieve and rank passages, assemble context and call the configured Groq LLM.
6. Return the answer with source metadata through Streamlit.

The retrieval score shown by the application is a distance-derived heuristic, not a calibrated probability that an answer is correct. Ranking is based on retrieval scores; the current implementation does not use a separate cross-encoder reranker.

See [Architecture and flows](docs/architecture.md) and [Validation scope](docs/validation.md).

## Technology

Python · Streamlit · LangChain · FAISS · sentence-transformers · Groq API · PyMuPDF · LlamaParse · pandas · Plotly · pytest.

Model names, tariff values and retrieval settings are centralized in `config.py`. The code integrates a pretrained LLM; it does not train or fine-tune a foundation model.

## Run locally

Use a separate Python environment and run commands from the repository root. The existing requirements use broad version ranges; a fresh dependency installation has not been validated as part of this documentation update.

```bash
git clone https://github.com/bigbaboy/chatbot-dienluc-danang.git
cd chatbot-dienluc-danang
python -m venv .venv
```

Activate the environment:

```powershell
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
```

```bash
# macOS / Linux
source .venv/bin/activate
```

```bash
python -m pip install -r requirements.txt
```

Copy `.env.example` to `.env` and provide your own `GROQ_API_KEY`. `LLAMA_CLOUD_API_KEY` is optional for OCR. Keep API keys out of version control.

To build an index from PDFs in `data/`:

```bash
python tao_vector_db.py
python -m streamlit run app.py
```

The first embedding-model load may require a download. Only load FAISS metadata from trusted sources: the current loader enables pickle deserialization. Rebuilding from the supplied PDFs is preferable to loading an unfamiliar index.

The existing project lists a [Streamlit demo](https://chatbotdienlucdanang.streamlit.app). Availability is not verified by this documentation update.

## Tests and evidence

```bash
python -m pip install pytest
python -m pytest test_tinh_toan.py -v
```

Test code covers billing, VAT, household splitting, consumption, solar calculations, question parsing and validation helpers. Tests have not been executed for this documentation update, so no pass count, latency or accuracy figure is claimed here. LLM answer quality requires separate evaluation with reference answers and document versions.

## Current limitations

- Answers depend on the supplied documents and retrieval quality; generated answers can still be incorrect.
- Tariffs and solar assumptions are configuration snapshots, not a live feed of current rules.
- No production EVN integration, authenticated customer-account lookup or measured customer-service impact.
- API availability, rate limits and model changes can affect behavior.
- Dependency locking, repeatable evaluation and deployment hardening remain future work.

## Next steps

- [ ] Save a versioned question/reference-answer evaluation set.
- [ ] Add reproducible test output and dependency versions.
- [ ] Improve retrieval coverage and document-update tracking.
- [ ] Add user feedback and review workflows.

**Academic attribution:** Le Minh Dat, class 48K29.1; supervisor: Dr. Nguyen Phong Son, as recorded in the original project documentation.
