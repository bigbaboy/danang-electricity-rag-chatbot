# Architecture and Processing Flows

These diagrams document the current source structure. They were prepared retrospectively from the implementation; they are not the original Draw.io deliverables.

## Components

```mermaid
flowchart TD
    UI["Streamlit interface"] --> RAG["Question routing and retrieval"]
    UI --> CALC["Python calculation utilities"]
    PDF["PDF ingestion"] --> DB["FAISS index and metadata"]
    RAG --> DB
    RAG --> LLM["Groq LLM API"]
```

## Question routing

```mermaid
flowchart TD
    Q["Customer question"] --> D{"Recognized billing intent and kWh?"}
    D -->|Yes| C["Calculate bill in Python"]
    D -->|No| R["Retrieve and rank document passages"]
    R --> L["Generate answer from context"]
    C --> A["Display answer"]
    L --> A
```

`rag_pipeline.py` contains the routing and source-reference assembly. `utils.py` contains the deterministic calculations. The passage score uses vector distance; it is not an independent factual verification step.

## Document ingestion

```mermaid
flowchart TD
    P["PDF file"] --> T{"Usable text available?"}
    T -->|Yes| M["PyMuPDF extraction"]
    T -->|No| O["Optional LlamaParse OCR"]
    M --> S["Split pages into chunks"]
    O --> S
    S --> E["Embed and store in FAISS"]
```

## Functional boundaries

| Input | Processing | Output |
| --- | --- | --- |
| Question and chat context | Intent detection, retrieval and LLM call | Answer and source references |
| kWh and calculation parameters | Configured tariff rules | Bill breakdown |
| Appliance usage | Consumption formulas | Estimated use and charts |
| Solar assumptions | Output and financial calculations | Indicative payback estimate |
| PDF document | Extraction, chunking and embedding | Searchable document chunks |

The prototype does not retrieve personal customer records or submit requests to EVN systems.
