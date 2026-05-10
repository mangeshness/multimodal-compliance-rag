# Agentic Multimodal Compliance Engine

An industry-grade LLMOps project featuring a stateful Agentic RAG workflow built with LangGraph, Azure OpenAI, and Azure AI Search. This system automates the compliance auditing of multimodal video advertisements by comparing them against regulatory guidelines (FTC & YouTube Ad Specs).

## Key Features

Agentic Orchestration: Uses LangGraph to manage a multi-node workflow (Indexer -> Auditor).

Multimodal Extraction: Leverages Azure Video Indexer to bridge video content (OCR + Transcripts) to LLM-readable data.

Hybrid RAG: High-precision retrieval of regulatory clauses using Azure AI Search.

LLMOps Observability: Full execution tracing and prompt debugging via LangSmith and Azure Application Insights.

Production API: Backend built with FastAPI featuring strict schema validation.

## Architecture

```mermaid
flowchart TD
    classDef azure fill:#0078D4,stroke:#fff,stroke-width:2px,color:#fff;
    classDef agent fill:#F37321,stroke:#fff,stroke-width:2px,color:#fff;
    classDef input fill:#10B981,stroke:#fff,stroke-width:2px,color:#fff;
    classDef monitor fill:#8B5CF6,stroke:#fff,stroke-width:2px,color:#fff;

    User((User)) -->|Submits YouTube URL| API[FastAPI Server]:::input

    subgraph Orchestration [LangGraph Agentic Orchestration]
        direction TB
        Node1[Indexer Node]:::agent
        Node2[Auditor Node]:::agent
        Node1 -->|Passes State:\nTranscript + OCR + Metadata| Node2
    end
    
    API -->|Triggers Workflow| Orchestration

    subgraph VideoProcessing [Multimodal Extraction]
        direction TB
        Blob[(Azure Blob Storage)]:::azure
        VI[Azure Video Indexer]:::azure
        Node1 -->|1. Downloads & Uploads .mp4| Blob
        Blob -->|2. Analyzes Video| VI
        VI -->|3. Returns JSON| Node1
    end

    subgraph KnowledgeBase [RAG Knowledge Base - eastus]
        direction TB
        Embed[Text-Embedding-ada-002]:::azure
        Search[(Azure AI Search)]:::azure
        Embed -->|Vectorizes FTC Guidelines| Search
    end

    subgraph Reasoning [LLM Reasoning Engine - eastus2]
        direction TB
        GPT[GPT-4o]:::azure
        Node2 -->|4. Queries Vector DB| Search
        Search -->|5. Retrieves Compliance Rules| Node2
        Node2 -->|6. Sends Context + Rules| GPT
        GPT -->|7. Returns JSON Audit Report| Node2
    end

    subgraph Observability [LLMOps Observability]
        direction LR
        LS[LangSmith]:::monitor
        AppI[Azure Application Insights]:::monitor
    end

    Orchestration -.- |Traces Chain of Thought| LS
    API -.- |Traces Latency & API Health| AppI
    Orchestration -.- |Logs Node Errors| AppI

    Node2 -->|Final Output| Report[/Compliance Audit Report/]:::input
```

Entry Point: User submits a YouTube URL via FastAPI.

Video Indexing Node: Azure Video Indexer processes the video to extract transcript and onscreen text (OCR).

Knowledge Base: Regulatory PDFs are chunked and indexed in Azure AI Search.

Auditor Node: GPT-4o performs reasoning by comparing the extracted evidence against retrieved rules.

Output: A structured JSON compliance report (Pass/Fail) with severity ratings.

## Setup

Clone the repository.

Install dependencies using uv:

uv sync


Copy .env.example to .env and fill in your Azure & LangSmith credentials.

Run the indexing script to populate your Knowledge Base:

python backend/scripts/index_documents.py


Start the API server:

uv run uvicorn backend.src.api.server:app --reload
