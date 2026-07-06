## 📖 About This Course

Ever dreamed of building your own AI-powered game? Get ready for an exciting journey where we'll combine the thrill of game development with cutting-edge AI technology!

Welcome to **PhiloAgents** (a team-up between [Decoding ML](https://decodingml.substack.com) and [The Neural Maze](https://theneuralmaze.substack.com)) - where ancient philosophy meets modern AI. In this hands-on course, you'll build an AI agent simulation engine that brings historical philosophers to life in an interactive game environment. Imagine having deep conversations with Plato, debating ethics with Aristotle, or discussing artificial intelligence with Turing himself!

**In 6 comprehensive modules**, you'll learn how to:
- Create AI agents that authentically embody historical philosophers
- Master building agentic applications
- Architect and implement a production-ready RAG, LLM and LLMOps system from scratch

### 🎮 The PhiloAgents Experience. What You'll Do:

Transform static NPCs into dynamic AI personalities that:
- Build a game character simulation engine, powered by AI agents and LLMs, that impersonates philosophers from our history, such as Plato, Aristotle and Turing.
- Design production-ready agentic RAG systems.
- Ship the agent as a RESTful API.
- Apply LLMOps and software engineering best practices.
- Use industry tools: Groq, MongoDB, Opik, LangGraph, LangChain, FastAPI, Websockets, Docker, etc.

After completing this course, you'll have access to your own agentic simulation engine, as seen in the video below:

<video src="https://github.com/user-attachments/assets/aedc041e-00ed-42ce-99f2-24ce74847e7a"/></video>

-------
## 📊 LLM Observability & Evaluation (Evidently AI)
To ensure our agents converse accurately and stay in character, the project includes an offline evaluation suite using **Evidently AI**:
- **Dataset**: Built from predefined golden query-response test cases at [evaluation_dataset.json](philoagents-api/data/evaluation_dataset.json).
- **Core Metrics & Descriptors**:
  - **TextLength**: Measures character length of generated response.
  - **Sentiment**: Computes sentiment polarity (positive, neutral, negative) to check if the tone is aligned.
  - **Semantic Similarity**: Computes similarity between generated response and expected response using sentence embeddings.
  - **LLM-as-a-Judge Metrics** (utilizing `gpt-4o-mini` via OpenAI API):
    - **Correctness**: Checks if the generated answer matches the expected answer factually.
    - **Faithfulness**: Checks if the generated response is fully supported by the retrieved context (detecting hallucinations).
    - **Context Quality**: Measures how relevant the retrieved context from MongoDB was to the user query.
    - **Toxicity**: Evaluates if the response contains any toxic language.
- **Evidently UI**: The results are saved as standalone HTML/JSON files (`data/evidently_report.html`) and logged to a local workspace (`workspace/`) which can be visualized in real-time by running the Evidently UI server (`http://localhost:8085`).

---
