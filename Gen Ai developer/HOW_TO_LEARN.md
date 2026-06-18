# How to Learn This Program

This file is your guide to getting the most out of this roadmap.  
Read this before you start anything else.

---

## What This Program Is

This is a **self-paced, project-driven curriculum** to become a production-ready Generative AI developer. It is structured like a university course but built entirely around building real things.

There are 9 phases. Each phase has:
- **Topics to study** (concepts, tools, papers)
- **Assignments (A)** — focused practice on one skill
- **Projects (P)** — larger builds that combine multiple skills

You learn by doing. The assignments make sure you understand the fundamentals. The projects make sure you can ship something real.

---

## Who This Is For

This program assumes you already know:
- Python (functions, classes, loops, file I/O)
- Basic terminal usage (cd, ls, pip, git)
- A little bit of math (you do not need to be an expert — you will learn as you go)

If you are not comfortable with Python yet, spend 2-3 weeks on Python basics first, then come back.

---

## How Long Will This Take

| Phase | Estimated Time |
|-------|---------------|
| Phase 1 — Foundations | 2–3 weeks |
| Phase 2 — ML Core | 3–4 weeks |
| Phase 3 — NLP | 3–4 weeks |
| Phase 4 — LLMs | 3–4 weeks |
| Phase 5 — RAG | 3–4 weeks |
| Phase 6 — Agents | 4–5 weeks |
| Phase 7 — Multimodal | 3–4 weeks |
| Phase 8 — Production | 3–4 weeks |
| Phase 9 — Advanced | 3–4 weeks |
| **Total** | **~7–9 months** |

If you study 2–3 hours every day, you will finish in 7–8 months.  
If you study on weekends only (8–10 hours/week), expect 12–14 months.

Do not rush. Understanding matters more than speed.

---

## The Right Order to Do Things

For each phase, follow this sequence every time:

```
Step 1 — Study the concepts (2–4 days)
Step 2 — Do the assignments in order (1–2 days each)
Step 3 — Build the projects (3–7 days each)
Step 4 — Review and document what you learned
Step 5 — Move to the next phase
```

### Why this order matters
- If you skip Step 1 and go straight to projects, you will get stuck and copy-paste without understanding.
- If you only do Step 1 and skip projects, you will forget everything within a week.
- Assignments bridge the two: they are small enough to finish in a day but force you to implement the concept yourself.

---

## How to Approach Each Assignment

1. **Read the entire file first** before writing any code. Understand what is being asked.
2. **Do not copy-paste code from the internet.** Type it yourself. This is the only way to build muscle memory.
3. **If you are stuck for more than 30 minutes**, look up the specific concept (not the full solution). Ask ChatGPT or Claude to explain the concept, not to write the code for you.
4. **Run your code often.** Do not write 100 lines and then run. Write 10 lines, run, check the output, then continue.
5. **Check your deliverables checklist** at the end. Do not mark an assignment done until all checkboxes are complete.

---

## How to Approach Each Project

Projects are larger and will take 3–7 days each. Here is how to handle them:

### Day 1 — Plan Before You Code
- Read the entire project file
- Draw the architecture on paper (boxes and arrows — what talks to what)
- Break it into 5–10 small tasks
- Set up your project folder, virtual environment, and git repo

### Day 2 to N — Build Incrementally
- Build one piece at a time
- Get each piece working before moving to the next
- Commit to git at the end of every working session (even if incomplete)
- Do not try to build everything at once

### Final Day — Polish and Document
- Make sure all deliverables are done
- Write a short README explaining what the project does and how to run it
- Push to GitHub — having public projects is important for your career

### When You Are Stuck on a Project
Ask yourself in order:
1. Did I understand the individual components? (if no → go back to the assignment)
2. Is this a bug or a design problem?
3. Have I read the official documentation for the library I am using?
4. Is there a minimal reproducible example I can test in isolation?

Only then ask for help — and ask with context: show what you tried and what error you got.

---

## Tools You Will Need

Set these up before you start Phase 1:

### Accounts (all free tiers work)
- **GitHub** — for version control and showing your work
- **Kaggle** — for datasets and free GPU notebooks
- **Google Colab** — for free GPU when you need it (fine-tuning)
- **Hugging Face** — for models and datasets
- **OpenAI or Anthropic** — API keys ($5-10 credit is enough to start)
- **LangSmith** — for Phase 8 observability (free tier)

### Local Setup
```bash
# Install these once
pip install jupyter notebook ipykernel
pip install numpy pandas matplotlib seaborn scikit-learn
pip install torch torchvision  # from pytorch.org, pick your OS
pip install transformers datasets huggingface_hub
pip install langchain langchain-openai langchain-anthropic
pip install chromadb sentence-transformers
pip install fastapi uvicorn httpx pydantic
pip install streamlit
```

### VS Code Extensions
- Python
- Jupyter
- GitLens
- GitHub Copilot (optional, but do not rely on it for learning — only use after you understand a concept)

---

## Managing API Costs

LLM API calls cost money. Follow these rules to avoid surprises:

1. **Set a hard budget limit** on OpenAI and Anthropic dashboards — $10/month to start
2. **Use cheaper models for development**: `gpt-4o-mini` or `claude-haiku-4-5` while building, then test with the best model only at the end
3. **Cache responses during development**: save API responses to a JSON file and replay them while debugging, so you do not call the API 100 times for the same test
4. **Use open-source models locally** for heavy experimentation: Ollama + Llama3 or Mistral is free and runs on your laptop (8GB RAM minimum)

---

## Git Workflow for Every Project

```bash
# Start of every project
git init
git add .
git commit -m "initial setup"
gh repo create project-name --public  # creates GitHub repo

# During work
git add .
git commit -m "add retrieval module"

# At the end of each day
git push origin main
```

Use clear commit messages. Future you will thank present you.

---

## What to Do When You Feel Lost

Every learner hits a wall. Here is how to get through it:

**If a concept is not clicking:**
- Watch Andrej Karpathy's video on that topic (he explains from first principles)
- Read the original paper (even just the abstract and conclusion)
- Implement a smaller version: if Transformers are hard, implement just the attention layer first

**If a project feels overwhelming:**
- Break it into smaller pieces on paper
- Build the simplest possible version that runs (even with fake data)
- Add complexity one feature at a time

**If you are debugging for too long:**
- Add print statements everywhere to trace what is actually happening
- Test each component in isolation (a Jupyter cell per function)
- Read the error message carefully — most errors tell you exactly what went wrong

**If you lose motivation:**
- Ship something — even if small. Seeing something work is the best motivator.
- Re-read the project list and pick one that excites you
- Remember why you started

---

## How to Learn Concepts (Before Coding)

For each phase, study the concepts this way:

| Resource | When to Use |
|----------|------------|
| Official Hugging Face course | Best for NLP and model loading (Phase 3-4) |
| fast.ai Practical Deep Learning | Best for ML and deep learning intuition (Phase 2) |
| Andrej Karpathy YouTube | Best for understanding from first principles (Phase 2-3) |
| DeepLearning.AI short courses | Best for LLMs, RAG, Agents (Phase 4-6) |
| Official library docs | Always — for FastAPI, LangChain, etc. |
| Papers | Read the abstract, intro, method, and conclusion — skip math on first pass |

Do not try to watch every video or read every article. Pick one resource per topic, go deep, then build.

---

## Building Your Portfolio

Every project you complete should go on GitHub. By the end of this program you will have:
- 24 public GitHub repositories (one per project)
- Real working demos (Streamlit apps, deployed APIs)
- A README for each that explains what it does and how you built it

This is your portfolio. Companies do not care about certificates — they care about what you can build.

### What makes a good project README
1. What does it do (2 sentences)
2. Demo screenshot or GIF
3. How to run it locally (exact commands)
4. What you learned / what was hard
5. Architecture diagram if it is complex

---

## Milestones — How to Know You Are on Track

After completing each phase, you should be able to do this without help:

| After Phase | You Can Do |
|-------------|-----------|
| Phase 1 | Implement gradient descent, train a logistic regression, explain a confusion matrix |
| Phase 2 | Train a CNN in PyTorch, build a training loop from scratch, debug overfitting |
| Phase 3 | Fine-tune BERT, build a semantic search, evaluate with ROUGE/F1 |
| Phase 4 | Write effective prompts, fine-tune an LLM with QLoRA, build a chatbot with memory |
| Phase 5 | Build a RAG pipeline end-to-end, evaluate with RAGAS, implement hybrid search |
| Phase 6 | Build a multi-step agent with tools, implement LangGraph workflows, add human-in-the-loop |
| Phase 7 | Transcribe audio with Whisper, use vision LLMs for image understanding, generate images |
| Phase 8 | Dockerize an LLM app, set up CI evals, monitor latency and cost in production |
| Phase 9 | Red-team an LLM app, implement mixed precision training, use reasoning models effectively |

If you cannot do the thing in the right column, revisit that phase's assignments before moving forward.

---

## The Most Important Rule

**Do not skip the assignments to get to the projects.**

Assignments feel slow and boring. Projects feel exciting. But if you skip assignments, you will struggle with projects and end up copying code without understanding it. That defeats the purpose.

The assignments in this program are short by design — most take 1-2 days. They are worth doing properly.

---

## Quick Reference — File Naming

| File | What it is |
|------|-----------|
| `index.md` | Full roadmap overview |
| `HOW_TO_LEARN.md` | This file — how to study the program |
| `A1.1.md` | Assignment 1 from Phase 1 |
| `A3.2.md` | Assignment 2 from Phase 3 |
| `P2.1.md` | Project 1 from Phase 2 |
| `P6.3.md` | Project 3 from Phase 6 |

Start with `A` files to build understanding, then do `P` files to build something real.

---

## Final Thought

Most people who start a learning program like this do not finish. Not because the content is too hard — but because they start too many things, go too fast, and never really build anything.

The antidote is simple: **one assignment or project at a time, finished completely, before starting the next.**

Every project you finish — no matter how small — is proof that you can build. Collect that proof consistently for 8 months and you will be a strong Gen AI developer.

Start with `A1.1.md`. Good luck.
