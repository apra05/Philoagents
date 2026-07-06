## 🛠️ Technologies Used
- **Core Orchestration**: `langgraph`, `langchain-core` for the state-machine workflow agent architecture.
- **LLM Providers**: `langchain-groq` (Llama 3.3 70B & Llama 3.1 8B) for primary inference; `openai` (gpt-4o-mini) as a judge for evaluations.
- **API and Networking**: `fastapi[standard]` for the REST API and WebSocket communication.
- **Database / Vector Search**: `pymongo`, `langchain-mongodb`, `langgraph-checkpoint-mongodb` utilizing MongoDB Atlas Local as a vector store, document database, and checkpointer.
- **Embedding Model**: `sentence-transformers/all-MiniLM-L6-v2` via HuggingFace for encoding knowledge bases into 384-dimensional dense vectors.
- **Observability**: `opik` for real-time prompt telemetry and tracing.
- **Evaluation**: `evidently` for text quality, sentiment analysis, correctness, faithfulness, and context quality.
- **Development Tooling**: `uv` for python environments, Docker & Docker Compose, Webpack, npm, and GNU Make.
