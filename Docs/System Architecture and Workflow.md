## 🏗️ System Architecture & Workflow

### System Architecture
The system consists of a 2D game frontend, a backend WebSocket/REST server, an agent brain orchestrated with LangGraph, local/cloud storage, and monitoring tools:

```mermaid
graph TD
    User([User in Web App]) <-->|"WebSockets"| UI["Game UI - Phaser 3<br>Port 8080"]
    UI <-->|"WebSockets / REST"| API["FastAPI Backend API<br>Port 8000"]
    
    subgraph AgenticSystem [LangGraph Agent Workflow]
        LG[LangGraph Orchestrator]
        Guard[Guardrail Node]
        Retrieve[RAG Retriever Node]
        Conv[Conversation Node]
        Groq[Groq LLM Llama-3.3]
        SumNode[Summarize Node]
        
        LG -->|"1. Check Input"| Guard
        LG -->|"2. Search Memory"| Retrieve
        LG <-->|"3. Generate Response"| Conv
        Conv <-->|"Groq API"| Groq
        Conv -->|"4. Summarize History"| SumNode
    end

    subgraph Storage [Local Infrastructure]
        DB[("MongoDB Local<br>Port 27017")]
    end

    subgraph LLMOps [Monitoring & Evaluation]
        Opik[Opik / Comet ML Cloud]
        Eval[Evidently AI Engine]
        HTML["HTML Reports / Workspace"]
        EvidUI["Evidently UI<br>Port 8085"]
        
        Eval -->|"Generates"| HTML
        EvidUI -->|"Read Dashboard"| HTML
    end

    API <-->|"Execute Graph"| LG
    Retrieve -->|"Query"| DB
    LG -->|"Trace Prompts"| Opik
    Eval -->|"Run Offline Evals"| DB
    LG -->|"Save Checkpoint State"| DB
    Groq -->|"Log Generation"| DB
```

The key system components are:
1. **Frontend (Game UI)**: A retro 2D pixel-art game interface built using the **Phaser 3** framework and Webpack. Users walk around and interact with philosophers. It is served on `http://localhost:8080`.
2. **Backend API**: A high-performance **FastAPI** web server running on `http://localhost:8000`. It exposes REST endpoints and active WebSocket connections for real-time, low-latency chat interactions in the game.
3. **Agent Brain**: Orchestrated using **LangGraph** for flexible, state-machine agent interactions. It uses **Groq** for high-speed LLM inference, and **MongoDB** as a document database, vector store, and agent state checkpoint tracker.
4. **Monitoring & Evals**: Integrates with **Opik** (online logging/tracing) and **Evidently AI** (offline evaluations on a test dataset, exposing an HTML report dashboard at `http://localhost:8085`).

### Agent Workflow
The LangGraph agent workflow acts as a state-machine that processes each incoming message through several nodes:

```mermaid
graph TD
    Start([START]) --> Guardrail[Guardrail Node]
    Guardrail -->|"Check Violation"| IsViolated{"Violated?"}
    IsViolated -->|Yes| Refusal[Refusal Node]
    IsViolated -->|No| Conversation[Conversation Node]
    
    Conversation -->|"Requires Context?"| NeedsContext{"Needs context?"}
    NeedsContext -->|"Yes (Tool Call)"| Retriever[Retriever Node]
    Retriever -->|Query DB| DB[(MongoDB Vector Index)]
    DB -->|Documents| SummarizeCtx[Summarize Context Node]
    SummarizeCtx --> Conversation
    
    NeedsContext -->|No| Connector[Connector Node]
    Refusal --> Connector
    
    Connector -->|Check Message Length| ShouldSummarize{"Messages > 30?"}
    ShouldSummarize -->|Yes| SummarizeConv[Summarize Conversation Node]
    ShouldSummarize -->|No| EndNode([END])
    
    SummarizeConv --> EndNode
```

1. **Guardrail Node**: Analyzes the user query using a classification model to see if it violates era-appropriate constraints (e.g. Socrates speaking of modern politics or technologies post-399 BC). If violated, the graph routes to an in-character Refusal Node and skips the conversation node.
2. **Conversation Node**: The central brain. It takes the query, conversation history, and any retrieved contextual memory to craft an in-character response using the Groq API.
3. **Retriever Node**: Executes if the conversation node requests details from the database. It queries the MongoDB vector store for the philosopher's custom context.
4. **Summarize Context Node**: Dynamically summarizes the retrieved document fragments to save context window tokens and filter out irrelevant data.
5. **Connector Node**: Standardizes intermediate state parameters.
6. **Summarize Conversation Node**: Triggered conditionally when the message thread length exceeds 30 messages. It summarizes previous logs, writes them to the `summary` state parameter, and deletes old messages to keep token usage small.

### Detailed Execution Workflow (Sequence Diagram)
Here is the sequence of runtime operations when a user sends a message to a philosopher in town:

```mermaid
sequenceDiagram
    autonumber
    actor User as User (Browser)
    participant UI as Phaser 3 Game UI
    participant API as FastAPI Backend
    participant LG as LangGraph Agent Workflow
    participant LLM as Groq LLM API
    participant DB as MongoDB (Vector & State DB)
    participant Opik as Opik Tracing

    User->>UI: Walk to Philosopher & Submit Chat Message
    UI->>API: Send Message via WebSocket connection
    API->>LG: Invoke Workflow with current state (messages, thread_id)
    Note over LG: START Workflow Graph
    LG->>Opik: Start trace logging
    
    %% Guardrail step
    LG->>LLM: Call Guardrail Chain (Check era-appropriate constraints)
    LLM-->>LG: Return classification (violates / does not violate)
    
    alt Guardrail Violated
        LG->>LLM: Call Refusal Chain (Generate in-character rejection)
        LLM-->>LG: Return refusal message (e.g. Socrates refusing modern tech query)
        Note over LG: Skip conversation & retrieval, go to Connector
    else Guardrail Passed
        %% Conversation & RAG step
        LG->>LLM: Invoke Conversation Model with chat history & context
        Note over LLM: LLM decides to search database (Tool Call)
        LLM-->>LG: Request retriever tool call
        LG->>DB: Query philosopher_long_term_memory (Vector Search)
        DB-->>LG: Return document chunks
        LG->>LLM: Call Context Summarizer (Extract relevant facts)
        LLM-->>LG: Return summarized facts
        LG->>LLM: Re-call Conversation Model with facts & context
        LLM-->>LG: Return final response text
    end
    
    %% Conversation Summarization check
    opt Messages count > 30
        LG->>LLM: Call Conversation Summary Chain (Condense history)
        LLM-->>LG: Return summarized conversation string
        LG->>DB: Save updated summary and remove older messages from checkpoints
    end
    
    Note over LG: END Workflow Graph
    LG->>DB: Save state checkpoint (thread_id)
    LG->>Opik: Close trace logging
    LG-->>API: Return final AI Message
    API-->>UI: Send response back via WebSocket
    UI-->>User: Render character dialogue bubble in 2D game
```

---
