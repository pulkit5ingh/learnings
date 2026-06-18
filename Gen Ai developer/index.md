# Generative AI Developer Roadmap

---

## Phase 1: Foundations

### Mathematics & Statistics
- Linear Algebra (vectors, matrices, dot products, eigenvalues)
- Calculus (derivatives, gradients, chain rule)
- Probability & Statistics (distributions, Bayes theorem, entropy)
- Optimization (gradient descent, Adam, momentum)

### Python for AI
- NumPy, Pandas, Matplotlib
- Object-oriented programming
- Virtual environments, dependency management

### Assignments
- **A1.1** — Implement gradient descent from scratch to minimize a quadratic function. Plot the loss curve over iterations.
- **A1.2** — Build a numpy-only logistic regression. Train it on the Iris dataset, compute accuracy, precision, recall manually without sklearn.
- **A1.3** — Visualize how dot product similarity works: embed 10 words as random vectors, build a cosine similarity matrix, and display a heatmap.

### Projects
- **P1.1 — NumPy Neural Net** — Build a 2-layer neural network using only NumPy (no frameworks). Train it on MNIST digits. Implement forward pass, backward pass, weight updates manually.
- **P1.2 — Data EDA Dashboard** — Pick any real-world dataset (e.g., Kaggle housing prices). Build a full exploratory analysis with Pandas + Matplotlib/Seaborn, including missing value handling, feature distributions, and correlation analysis.

---

## Phase 2: Machine Learning Core

### Classical ML
- Supervised learning (regression, classification)
- Unsupervised learning (clustering, dimensionality reduction)
- Model evaluation, cross-validation, regularization

### Deep Learning
- Neural networks (feedforward, backpropagation)
- CNNs, RNNs, LSTMs
- Frameworks: **PyTorch** (primary), TensorFlow

### Assignments
- **A2.1** — Train a Random Forest and XGBoost on a tabular dataset. Compare feature importances. Tune hyperparameters with Optuna.
- **A2.2** — Build a CNN in PyTorch for CIFAR-10 classification. Implement data augmentation (random crop, flip). Log training/validation loss per epoch.
- **A2.3** — Train an LSTM to predict the next character in a text corpus (e.g., Shakespeare). Generate sample text after training.
- **A2.4** — Implement dropout and batch normalization manually inside a custom PyTorch `nn.Module`. Measure the effect on overfitting.

### Projects
- **P2.1 — Kaggle Tabular Competition** — Compete on any active or past Kaggle tabular competition. Build a full pipeline: EDA → feature engineering → stacking ensemble → submission. Hit top 30% of leaderboard.
- **P2.2 — Image Classifier API** — Train a CNN (ResNet-18 fine-tuned via transfer learning) on a custom image dataset (e.g., your own 5-class dataset). Wrap it in a FastAPI endpoint that accepts an image and returns predictions with confidence scores.
- **P2.3 — Time Series Forecaster** — Use an LSTM to forecast stock or weather data. Compare against a classical ARIMA baseline. Visualize predictions vs actuals.

---

## Phase 3: Natural Language Processing

### Core NLP
- Tokenization, embeddings (Word2Vec, GloVe)
- Attention mechanism
- Transformers architecture (encoder, decoder, encoder-decoder)

### Pretrained Models
- BERT, RoBERTa (understanding tasks)
- GPT-2 family (generation tasks)
- T5, BART (seq2seq tasks)
- Hugging Face `transformers` library

### Assignments
- **A3.1** — Train Word2Vec embeddings on a custom corpus using `gensim`. Visualize word clusters with t-SNE. Find nearest neighbors for 10 query words.
- **A3.2** — Fine-tune BERT on a binary sentiment classification task (SST-2 or IMDb). Report F1, confusion matrix. Use Hugging Face Trainer.
- **A3.3** — Implement scaled dot-product attention from scratch in PyTorch. Test it on a toy sequence and verify output shapes.
- **A3.4** — Fine-tune T5 for text summarization on CNN/DailyMail. Evaluate with ROUGE scores.

### Projects
- **P3.1 — Named Entity Recognition (NER) API** — Fine-tune a BERT-based NER model on CoNLL-2003. Build a REST API that accepts text and returns annotated entities (person, org, location). Used heavily in legal, finance, and healthcare AI.
- **P3.2 — Semantic Search Engine** — Build a semantic search system over Wikipedia articles using Sentence Transformers. Support natural language queries and rank results by embedding similarity. Add a simple web UI with Streamlit.
- **P3.3 — Text Summarizer** — Fine-tune BART or T5 on news articles. Deploy as a web app where users paste any article and get a 3-sentence summary.

---

## Phase 4: Large Language Models (LLMs)

### Understanding LLMs
- Pretraining vs fine-tuning
- Instruction tuning & RLHF
- Scaling laws, emergent behavior
- Context windows, tokenization

### Working with LLMs
- OpenAI API, Anthropic API (Claude)
- Prompt engineering (zero-shot, few-shot, chain-of-thought)
- System prompts, temperature, top-p sampling

### Fine-tuning
- LoRA / QLoRA (parameter-efficient fine-tuning)
- PEFT library
- Datasets: preparation, formatting for instruction tuning
- Hugging Face Trainer, TRL (SFT, DPO)

### Assignments
- **A4.1** — Write 5 different system prompts for the same task (e.g., code reviewer). Measure output quality differences. Document which prompt strategies work best and why.
- **A4.2** — Implement chain-of-thought prompting vs direct prompting on 10 math word problems. Compare accuracy. Show how CoT improves step-by-step reasoning.
- **A4.3** — Fine-tune Llama-3.1-8B using QLoRA on a custom instruction dataset (min 500 examples). Evaluate before/after on 20 held-out prompts. Use Google Colab (free GPU).
- **A4.4** — Build a prompt injection attack on a simple LLM app, then patch it. Document the vulnerability and mitigation.

### Projects
- **P4.1 — AI Code Reviewer** — Build a CLI tool that takes a GitHub PR diff as input and uses Claude/GPT-4 to generate structured code review comments (bugs, style, performance). Output in markdown. This is directly used in real dev tooling at companies.
- **P4.2 — Domain-Specific Chatbot (Fine-tuned)** — Fine-tune Mistral-7B or Llama-3 on a domain dataset (e.g., medical Q&A, legal FAQs, customer support transcripts). Deploy with Ollama locally. Benchmark against base model.
- **P4.3 — Prompt Playground Dashboard** — Build a Streamlit app that lets you A/B test prompts side by side, adjust temperature/top-p, track token usage and cost, and save prompt versions. Useful for any LLM product team.

---

## Phase 5: Retrieval-Augmented Generation (RAG)

### Vector Databases
- Embeddings for retrieval (OpenAI, Sentence Transformers)
- Vector stores: Pinecone, Weaviate, Chroma, Qdrant, pgvector

### RAG Pipeline
- Document loaders, chunking strategies
- Hybrid search (dense + sparse)
- Reranking (Cohere, cross-encoders)
- Evaluation: RAGAS, faithfulness, answer relevancy

### Frameworks
- LangChain
- LlamaIndex

### Assignments
- **A5.1** — Build a naive RAG pipeline from scratch (no LangChain): chunk a PDF, embed with Sentence Transformers, store in Chroma, retrieve top-5, pass to GPT-4. No framework abstractions.
- **A5.2** — Compare 3 chunking strategies (fixed-size, sentence-level, semantic) on the same document. Measure retrieval quality using a set of 10 test questions.
- **A5.3** — Add a reranking step (Cohere Rerank or a cross-encoder) to your RAG pipeline. Measure MRR (Mean Reciprocal Rank) before and after.
- **A5.4** — Evaluate your RAG pipeline using RAGAS. Report faithfulness, answer relevancy, and context precision scores.

### Projects
- **P5.1 — Personal Knowledge Base Chat** — Build a RAG app over your own notes, PDFs, or Notion exports. Support multi-document retrieval, source citation in answers, and a chat history. Use LangChain + Chroma + Claude/GPT-4. Deploy on Vercel or Railway.
- **P5.2 — Company Docs Q&A Bot** — Ingest a company's public documentation (e.g., Stripe docs, AWS docs). Build a production-grade RAG pipeline with hybrid search (BM25 + dense), reranking, and streaming responses. Add RAGAS-based eval suite.
- **P5.3 — Research Paper Assistant** — Ingest 50+ ArXiv papers in a domain. Build a chat interface where users can ask questions across all papers. Include "which paper talks about X?" and "summarize the key findings on Y" queries.

---

## Phase 6: AI Agents & Agentic Systems

### Agent Fundamentals
- ReAct pattern (reasoning + acting)
- Tool use / function calling
- Memory: short-term (context), long-term (vector store)

### Multi-Agent Systems
- Agent orchestration
- CrewAI, AutoGen, LangGraph
- Human-in-the-loop patterns

### Agent Tools
- Web search, code execution
- API integrations, browser automation
- File / database access

### Assignments
- **A6.1** — Build a ReAct agent from scratch using only the OpenAI/Claude function calling API (no LangChain). Give it 3 tools: calculator, web search, Wikipedia. Test with 10 complex queries.
- **A6.2** — Build a stateful agent with long-term memory using a vector store. The agent should remember facts from past conversations and reference them in future sessions.
- **A6.3** — Build a LangGraph workflow for a multi-step task (e.g., research → outline → draft → review). Each node is a separate LLM call. Add conditional branching based on output quality.
- **A6.4** — Build a human-in-the-loop agent: the agent proposes an action, pauses for human approval, then executes. Implement approve/reject/modify flow.

### Projects
- **P6.1 — Autonomous Research Agent** — Agent that takes a topic, searches the web, reads multiple pages, synthesizes findings into a structured report with citations. Built with LangGraph + Tavily search. Used in real consulting and research workflows.
- **P6.2 — AI Data Analyst Agent** — Agent connected to a SQLite/PostgreSQL database. User asks natural language questions, agent writes SQL, executes it, interprets results, and generates charts. Built with Claude function calling + pandas + matplotlib.
- **P6.3 — Multi-Agent Content Pipeline** — CrewAI or AutoGen pipeline with 4 agents: Researcher → Writer → Editor → SEO Optimizer. Each agent has a distinct role and passes structured output to the next. End result: a publication-ready blog post from a single topic input.
- **P6.4 — DevOps Automation Agent** — Agent with tools: run shell commands, read logs, call GitHub API, send Slack alerts. Diagnoses a failing CI build and suggests or applies fixes. Human-in-the-loop before any destructive action.

---

## Phase 7: Multimodal AI

### Vision + Language
- CLIP, BLIP, LLaVA
- GPT-4V, Claude Vision, Gemini Vision
- Image captioning, visual QA

### Audio
- Whisper (speech-to-text)
- TTS (ElevenLabs, OpenAI TTS)
- Audio embeddings

### Image & Video Generation
- Diffusion models (Stable Diffusion, DALL-E, Midjourney API)
- ControlNet, img2img
- Video generation (Sora, Runway, Kling)

### Assignments
- **A7.1** — Use CLIP to build an image search engine: embed 1000 images, embed a text query, retrieve the top-5 most similar images. Test with 20 queries.
- **A7.2** — Transcribe a 10-minute podcast episode with Whisper. Then use an LLM to generate chapter timestamps, key takeaways, and action items.
- **A7.3** — Use GPT-4V or Claude Vision to analyze a chart or diagram. Build a pipeline: image input → structured JSON output (title, axes, key insight). Test on 10 different chart types.
- **A7.4** — Generate 20 product images using Stable Diffusion with ControlNet (pose or edge conditioning). Write a prompt engineering guide based on your findings.

### Projects
- **P7.1 — AI Meeting Assistant** — Record a meeting → transcribe with Whisper → diarize speakers → summarize with action items → generate a formatted PDF report. Full end-to-end pipeline. High demand in enterprise productivity tools.
- **P7.2 — Visual RAG App** — Build a RAG system that handles both text and images. Users can upload PDFs with charts/diagrams; the system extracts text and image content, embeds both, and answers questions that require understanding visuals.
- **P7.3 — AI Product Photo Generator** — User uploads a product image → remove background → generate lifestyle context images using Stable Diffusion img2img + ControlNet → output 5 variations. Used in e-commerce.

---

## Phase 8: Production & MLOps

### Serving LLMs
- vLLM, Ollama (local inference)
- Triton Inference Server
- Quantization: GPTQ, AWQ, GGUF

### API & Backend
- FastAPI for model serving
- Streaming responses (SSE, WebSockets)
- Rate limiting, auth, cost tracking

### Evaluation & Observability
- LLM evals: LangSmith, Weights & Biases, Arize
- Hallucination detection
- Prompt versioning and A/B testing

### Infrastructure
- Docker, Kubernetes for AI workloads
- GPU cloud: AWS SageMaker, GCP Vertex AI, Modal, Replicate
- CI/CD for ML pipelines

### Assignments
- **A8.1** — Serve a quantized LLM (GGUF via Ollama or GPTQ via vLLM) on your local machine. Benchmark throughput (tokens/sec) at different quantization levels (fp16, 8-bit, 4-bit).
- **A8.2** — Build a FastAPI LLM server with streaming (SSE), API key auth, per-user rate limiting (Redis), and request/response logging to a database.
- **A8.3** — Set up LangSmith tracing on an existing RAG or agent app. Identify the 3 slowest steps and the 2 most common failure modes from the trace data.
- **A8.4** — Dockerize an LLM app. Write a GitHub Actions CI pipeline that runs evals on every PR and blocks merge if quality drops below a threshold.

### Projects
- **P8.1 — Production RAG API** — Take your Phase 5 RAG project and make it production-ready: streaming responses, auth, rate limiting, LangSmith observability, Dockerized, deployed to Railway or Fly.io. Add an automated eval suite that runs on every deploy.
- **P8.2 — LLM Cost & Quality Dashboard** — Build a monitoring dashboard (Grafana or a custom Streamlit app) that tracks: token usage per user, cost per request, response latency, and quality scores from automated evals. Pull data from LangSmith or W&B.
- **P8.3 — Self-Hosted LLM Gateway** — Deploy an open-source LLM (Llama-3 or Mistral) on a cloud GPU (Modal or RunPod). Build a gateway API in front of it that handles routing, caching repeated queries (Redis semantic cache), and fallback to OpenAI if the local model fails.

---

## Phase 9: Advanced Topics

### Efficient Training
- Mixed precision (fp16, bf16)
- Gradient checkpointing
- Distributed training (DDP, FSDP)

### Security & Safety
- Prompt injection, jailbreaks
- Content moderation
- PII detection and redaction
- Red-teaming LLMs

### Emerging Areas
- AI coding agents (Copilot, Cursor, Claude Code SDK)
- Long-context models (1M+ tokens)
- Mixture of Experts (MoE)
- Test-time compute / reasoning models (o1, DeepSeek-R1, Gemini 2.0 Flash Thinking)

### Assignments
- **A9.1** — Fine-tune a model with mixed precision (bf16) and gradient checkpointing. Compare GPU memory usage and training time vs full-precision training. Document the tradeoffs.
- **A9.2** — Red-team an LLM app you built. Document 5 prompt injection attacks, 3 jailbreak attempts, and 2 data extraction attacks. Write mitigations for each.
- **A9.3** — Build a PII detection + redaction pipeline using a fine-tuned NER model or an LLM. Process a dataset of 1000 fake user records. Measure precision/recall.
- **A9.4** — Use a reasoning model (o1 or DeepSeek-R1) on a complex multi-step problem. Compare it to a standard model with chain-of-thought prompting. Analyze the quality difference.

### Projects
- **P9.1 — LLM Red-Teaming Toolkit** — Build an automated red-teaming tool that generates adversarial prompts, tests them against your LLM app, and produces a security report. Include prompt injection, jailbreak, and data exfiltration test categories.
- **P9.2 — Custom AI Coding Assistant** — Build a VS Code extension (or CLI tool) that uses Claude/GPT-4 with retrieval over your own codebase. Features: explain function, suggest refactor, generate tests, find similar code. Use Claude Code SDK or OpenAI's API.
- **P9.3 — Reasoning + RAG Hybrid** — Build a complex Q&A system that uses a reasoning model (o1-mini or DeepSeek-R1) for multi-hop questions over a large document corpus. Compare accuracy vs standard RAG on a benchmark like HotpotQA.

---

## Master Projects (Capstone — Build These to Stand Out)

These combine multiple phases and reflect real industry products.

| # | Project | Skills Used |
|---|---------|------------|
| M1 | **AI SaaS Product** — Build a full-stack AI product (e.g., AI writing assistant, AI tutor, AI customer support bot). Include auth, billing (Stripe), LLM backend, RAG, streaming, and a React frontend. | Phases 4, 5, 8 |
| M2 | **Open Source LLM App** — Publish a well-documented open-source project on GitHub. Include CI/CD, eval suite, Docker setup, and a detailed README. Get real users and GitHub stars. | Phases 4–8 |
| M3 | **End-to-End Fine-tuned Model** — Collect or curate a dataset, fine-tune an open-source LLM, evaluate rigorously, quantize, and deploy. Write a technical blog post about the process. | Phases 4, 8, 9 |
| M4 | **Agentic Workflow Platform** — Build a no-code/low-code platform where users define agent workflows visually (like n8n but AI-native). Agents can call tools, branch on conditions, and loop. | Phase 6, 8 |

---

## Key Resources

- [Attention Is All You Need](https://arxiv.org/abs/1706.03762) — original Transformer paper
- Andrej Karpathy's Neural Networks / GPT from Scratch series (YouTube)
- fast.ai Practical Deep Learning course
- DeepLearning.AI short courses (LangChain, RAG, Agents, Fine-tuning)
- Hugging Face NLP Course
- LangChain & LlamaIndex documentation
- Anthropic Cookbook / OpenAI Cookbook
- Simon Willison's blog (practical LLM engineering)
- Eugene Yan's blog (LLM systems in production)
