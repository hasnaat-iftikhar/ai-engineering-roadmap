# RAG (Retrieval-Augmented Generation)

> **Track:** LLM Engineering
> **Level:** Beginner → Intermediate
> **Last updated:** 2026-04-02
> **Status:** In progress

---

## 1. What is RAG?

RAG stands for **Retrieval-Augmented Generation**. Let's break that down:


| Word           | Meaning                                                                |
| -------------- | ---------------------------------------------------------------------- |
| **Retrieval**  | Fetching relevant information from a knowledge source                  |
| **Augmented**  | Adding that information to the AI's input                              |
| **Generation** | AI generates a response using both its training AND the retrieved info |


**In simple words:** before you ask the AI a question, you first grab the right information from your own data and paste it into the prompt. Now the AI has extra knowledge it did not have before.

Think of it like an **open-book exam:**

- **Without RAG:** the student answers from memory only. They might forget things or make stuff up.
- **With RAG:** the student can look at their notes before answering, so the answer is more accurate.

---

## 2. The Problem RAG Solves

LLMs (like GPT, Claude, Gemini) have a big limitation: **they only know what they were trained on.**


| Problem                    | Example                                                                              | How RAG fixes it                                       |
| -------------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| **Outdated knowledge**     | "What is the latest version of React?" The model might say React 17 if trained in 2022 | Retrieve current docs and inject them                  |
| **No access to your data** | "Summarize our company's Q4 report." The model has never seen your report              | Retrieve the report and inject it                      |
| **Hallucination**          | Model confidently makes up facts that sound real                                       | Retrieved facts ground the response in reality         |
| **Generic answers**        | "How to write good prompts?" Gives a textbook answer                                   | Inject YOUR best practices for domain-specific answers |


### The key insight

Instead of retraining the whole model (which is expensive and slow), you just **give it the right information at the right time**.

```
Without RAG:  User Question ──────────────────► LLM ──► Generic Answer

With RAG:     User Question ──► Retrieve Data ──► LLM ──► Specific, Accurate Answer
                                     │
                              Your Knowledge Base
                              (docs, data, facts)
```

---

## 3. How RAG Works (The Big Picture)

Every RAG system follows the same three-step flow:

```
┌─────────────────────────────────────────────────┐
│                  RAG PIPELINE                    │
│                                                  │
│  Step 1: RETRIEVE                                │
│  ┌──────────────┐      ┌───────────────────┐    │
│  │ User Query   │ ───► │ Knowledge Base    │    │
│  └──────────────┘      │ (docs, data, etc) │    │
│                        └───────┬───────────┘    │
│                                │                 │
│                    relevant chunks               │
│                                │                 │
│  Step 2: AUGMENT               ▼                 │
│  ┌──────────────────────────────────────────┐   │
│  │ System Prompt + Retrieved Context +      │   │
│  │ User Query = Enhanced Prompt             │   │
│  └──────────────────────┬───────────────────┘   │
│                         │                        │
│  Step 3: GENERATE       ▼                        │
│  ┌──────────────────────────────────────────┐   │
│  │ LLM generates response using the         │   │
│  │ enhanced prompt (now with extra context)  │   │
│  └──────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

---

## 4. The Three Steps of RAG

### Step 1: Retrieve - "Go fetch the right information"

Find the pieces of information that are relevant to what the user asked. This could mean:

- Searching a database
- Looking through documents
- Querying a vector store
- Or simply loading a curated list (like [Prompt Critic & Optimizer](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) does)

At the end of this step you have **raw text snippets**, but you have not sent anything to the AI yet.

### Step 2: Augment - "Build the bigger prompt"

Take those retrieved snippets and **combine** them with the user's question into one enriched prompt. The AI now sees three things in one message:

1. Its system instructions (how it should behave)
2. The retrieved context (your knowledge snippets)
3. The user's actual question

Think of it as: **system prompt + your knowledge + user question = one enriched prompt**.

### Step 3: Generate - "Let the AI answer"

Send that enriched prompt to the LLM. The AI writes a response that is grounded in your retrieved data, not just its general training. Because the knowledge was right there in the prompt, the answer is more accurate and specific.

> **See it in action:** in [Prompt Critic & Optimizer](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer), all three steps happen in `services/pipeline.py`:
>
> - **Retrieve:** `get_knowledge_context()` loads 7 prompt best practices from `rag/knowledge_base.py`
> - **Augment:** the retrieved practices are combined with the system prompt
> - **Generate:** the combined message is sent to OpenAI via `call_llm(messages)`

---

## 5. RAG vs Fine-Tuning vs Prompt Engineering

People often mix these three up. Here is how they are different and when to use each one:


| Feature                    | Prompt Engineering                    | RAG                                 | Fine-Tuning                  |
| -------------------------- | ------------------------------------- | ----------------------------------- | ---------------------------- |
| **What it does**           | Writes better instructions for the AI | Gives AI extra knowledge at runtime | Retrains the AI model itself |
| **Data needed**            | None                                  | Your documents/knowledge            | Thousands of examples        |
| **Cost**                   | Free                                  | Low (API calls + storage)           | High (compute + time)        |
| **Setup time**             | Minutes                               | Hours                               | Days to weeks                |
| **Updates**                | Change the prompt                     | Update the knowledge base           | Retrain the model            |
| **Best for**               | Simple tasks, formatting              | Domain knowledge, current data      | Specialized behavior, tone   |
| **AI learns permanently?** | No                                    | No                                  | Yes                          |


### When to use what

```
"I want AI to respond in a specific format"
    → Prompt Engineering (system prompt)

"I want AI to know about MY data/docs"
    → RAG

"I want AI to behave fundamentally differently"
    → Fine-Tuning
```

### You can combine them

[Prompt Critic & Optimizer](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) uses **all three ideas together**:

- **Prompt Engineering:** system prompt defines behavior
- **Few-Shot Prompting:** examples teach output format
- **RAG:** best practices are injected as context

This is common in real-world AI systems, and they are not either/or.

---

## 6. Types of RAG (Simple to Advanced)

There are four levels of RAG. Think of them as floors in a building: you start on the ground floor and go higher only when you need to.


| Level | Name             | One-line idea                       | Real-world analogy                                                                             |
| ----- | ---------------- | ----------------------------------- | ---------------------------------------------------------------------------------------------- |
| 1     | Static RAG       | Load all your knowledge every time  | Reading the **entire** cheat sheet before every exam                                           |
| 2     | Search-Based RAG | Find matching items by keywords     | Using **Ctrl+F** to search a document                                                          |
| 3     | Vector RAG       | Find matching items by meaning      | Asking a librarian: "I need books **about** this topic"                                        |
| 4     | Agentic RAG      | AI decides what to search and where | Hiring a **research assistant** who picks the right library, database, or website on their own |


---

### Level 1: Static RAG

**What it means:** You store your knowledge as a fixed list right inside your code. Every time a user asks something, you load **all** of it into the prompt. No searching, no filtering, the AI sees everything.

**Real-world analogy:** You print out a short cheat sheet and hand the **whole thing** to the AI every single time, no matter what the question is.

**When to use it:** Your knowledge base is small (under ~50 items) and rarely changes.

The code below creates a small knowledge list and a function that turns it into one big string. That string gets pasted into the prompt later.

```python
KNOWLEDGE = [
    {"title": "Be Specific", "description": "Replace vague words with details..."},
    {"title": "Add Context", "description": "Give background information..."},
]

def get_context():
    return "\n".join([f"{k['title']}: {k['description']}" for k in KNOWLEDGE])
```

> **See it in action:** this is exactly what [Prompt Critic & Optimizer](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) uses (see `rag/knowledge_base.py`).

---

### Level 2: Search-Based RAG

**What it means:** Instead of loading everything, you **search** your knowledge base using keywords. Only matching items go into the prompt.

**Real-world analogy:** You have a big document, and you press **Ctrl+F** to jump to the part that mentions the words you are looking for.

**When to use it:** Medium-sized knowledge bases where loading everything every time would waste tokens.

The code below takes the user's question, splits it into words, and checks which items in the knowledge base contain any of those words. It returns the top 5 matches.

```python
def retrieve(query, knowledge_base):
    results = []
    for item in knowledge_base:
        if any(word in item["text"].lower() for word in query.lower().split()):
            results.append(item)
    return results[:5]
```

**Limitation:** Keyword search only finds **exact word matches**. If the user says "error handling" but your document says "exception management", keyword search will miss it. That is why Level 3 exists.

---

### Level 3: Vector RAG (Production Standard)

**What it means:** You turn every piece of knowledge into **numbers** (called embeddings). When a user asks a question, you turn that question into numbers too, and then find which stored numbers are **closest** in meaning. This is called **semantic search**, which means searching by meaning, not by matching words.

**Real-world analogy:** You walk into a library and ask the librarian: "I need something about handling mistakes in Python." The librarian understands what you **mean** and hands you a book titled "Exception Management with try/except", even though you never used the word "exception."

**When to use it:** Large knowledge bases (thousands of documents) or when users ask questions in different words than your documents use.

The code below turns the user's question into an embedding (list of numbers), searches a vector database for the 5 closest matches by meaning, and joins them into one context string.

```python
query_embedding = embed("How do I write better prompts?")
results = vector_db.search(query_embedding, top_k=5)
context = "\n".join([r.text for r in results])
```

---

### Level 4: Agentic RAG

**What it means:** An **AI agent** is in charge of retrieval. It decides **what** to search for, **which** knowledge source to use, and **whether it needs to search again** for more information. It can pull from multiple places like databases, APIs, documents, and even the web.

**Real-world analogy:** Instead of searching yourself, you hire a research assistant. You tell them: "Find out about our customer retention last quarter." They decide on their own to check the CRM database, read two internal reports, and call the analytics API, and then they give you a combined answer.

**When to use it:** Complex systems where the answer might require information from multiple different sources, and where multi-step reasoning is needed.

**Example flow:**

```
User: "Why did revenue drop in March?"
         │
    Agent thinks: "I need sales data AND marketing spend"
         │
         ├── Tool 1: Query sales database for March numbers
         ├── Tool 2: Query marketing database for March budget
         │
    Agent thinks: "Sales dropped in region B. Let me check why."
         │
         └── Tool 3: Query CRM for region B customer complaints
         │
    Agent: "Revenue dropped because region B had delivery delays,
            leading to 23% more cancellations."
```

---

## 7. Building Basic RAG (Step by Step)

A simple RAG system from scratch. This is the approach [Prompt Critic & Optimizer](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) uses.

### Step 1: Create your knowledge base

Store your domain knowledge as structured data. The code below creates a Python list where each item has a title and a description. This is your "knowledge base", the facts you want the AI to know about.

```python
# rag/knowledge_base.py

KNOWLEDGE = [
    {
        "title": "Be Specific, Not Vague",
        "description": (
            "Replace generic instructions with precise details. "
            "Instead of 'Write something about dogs', say "
            "'Write a 300-word blog post about rescue dogs.'"
        ),
    },
    {
        "title": "Assign a Role",
        "description": (
            "Tell the AI who it should be. Example: "
            "'You are a senior engineer reviewing code.'"
        ),
    },
]
```

### Step 2: Create a retrieval function

Format the knowledge into a single string so it can be pasted into a prompt. The code below loops through every item in the knowledge list, numbers them, and joins them into one readable block of text.

```python
def get_knowledge_context() -> str:
    lines = ["## Best Practices\n"]
    for i, item in enumerate(KNOWLEDGE, 1):
        lines.append(f"### {i}. {item['title']}")
        lines.append(f"{item['description']}\n")
    return "\n".join(lines)
```

### Step 3: Inject into the prompt

Combine the retrieved context with your system prompt. The code below calls the retrieval function from Step 2, then creates a system message that contains **both** your original instructions and the retrieved knowledge glued together.

```python
rag_context = get_knowledge_context()

system_message = {
    "role": "system",
    "content": f"{SYSTEM_PROMPT}\n\n{rag_context}",
}
```

### Step 4: Send to LLM

The AI now has your knowledge as part of its input. The code below puts the system message (with injected knowledge) and the user's question into a list, then sends it to the LLM. The AI will now answer using **your** facts.

```python
messages = [system_message, user_message]
response = call_llm(messages)
```

That's it. Four steps. The AI now answers using YOUR knowledge, not just its general training.

> **See it in action:** this is exactly how [Prompt Critic & Optimizer](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) works:
>
> - Knowledge base: `rag/knowledge_base.py` (7 best practices)
> - Retrieval function: `get_knowledge_context()` formats them
> - Injection: `pipeline.py` combines system prompt + RAG context
> - LLM call: `pipeline.py` sends everything to OpenAI

---

## 8. When to Use RAG (and When Not To)

### Use RAG when

- AI needs access to **YOUR specific data** (company docs, product info)
- Knowledge **changes often** (news, prices, inventory)
- You need **accurate, fact-based** answers (not creative writing)
- You want to **reduce hallucination**
- You **can't afford** to fine-tune a model

### Don't use RAG when

- The task is purely **creative** (writing poetry, brainstorming)
- **General knowledge** is enough (no private data needed)
- **Real-time speed** is critical (retrieval adds latency)
- Your data is **too small** (just put it in the prompt directly)
- Your data is **too sensitive** to process through an API

### Decision flowchart

```
Does the AI need knowledge it wasn't trained on?
    │
    ├── No → Just use prompt engineering
    │
    └── Yes → Is the knowledge base small (< 50 items)?
              │
              ├── Yes → Static RAG (load everything)
              │
              └── No → Is it under 10,000 documents?
                        │
                        ├── Yes → Vector RAG (ChromaDB / FAISS)
                        │
                        └── No → Production Vector RAG
                                 (Pinecone / Weaviate + re-ranking)
```

---

## 9. Common Mistakes


| Mistake | Why it's bad | What to do instead |
|---|---|---|
| **Chunks too big** | Wastes tokens, adds noise | Keep chunks 300–1000 chars |
| **Chunks too small** | Loses context | Add overlap between chunks |
| **No overlap** | Information split between chunks | Use 10–20% overlap |
| **Retrieving too many chunks** | Floods the prompt, confuses LLM | Start with top 3–5 |
| **Not telling LLM to use the context** | LLM might ignore retrieved data | Add "Answer based on the provided context" to system prompt |
| **No fallback** | App crashes when retrieval returns nothing | Handle empty results gracefully |
| **Using RAG when you don't need it** | Over-engineering | If data fits in the prompt, just put it there |


---

## 10. Quick Reference Cheatsheet

### RAG in 4 lines

The entire RAG idea boils down to these four lines: (1) retrieve matching info, (2) build a prompt with that info, (3) send it to the LLM, (4) return the answer.

```python
context = retrieve_relevant_info(user_query)
prompt = f"{system_prompt}\n{context}\n{user_query}"
response = llm.generate(prompt)
return response
```

### When to use which level


| Knowledge base size | Approach | Tools |
|---|---|---|
| < 50 items | Static RAG (load all) | Python list/dict |
| 50–1,000 items | Keyword search RAG | SQLite, JSON file |
| 1,000–100,000 items | Vector RAG | ChromaDB, FAISS |
| 100,000+ items | Production Vector RAG | Pinecone, Weaviate, Qdrant |


### Key terms


| Term | Simple meaning |
|---|---|
| **RAG** | Give AI extra knowledge before it answers |
| **Embedding** | Turn text into numbers that capture meaning |
| **Vector** | A list of numbers (what an embedding produces) |
| **Vector DB** | Database that searches by meaning, not keywords |
| **Chunk** | A small piece of a larger document |
| **Top-K** | Return the K most relevant results |
| **Cosine similarity** | Math formula to compare two embeddings |
| **Retrieval** | Finding relevant information |
| **Augmentation** | Adding retrieved info to the prompt |
| **Generation** | AI creating the final response |
| **Hallucination** | AI making up facts that sound real |
| **Grounding** | Using real data to keep AI accurate |


### Libraries to know


| Library | What it does |
|---|---|
| `chromadb` | Simple vector database (great for learning) |
| `faiss-cpu` | Fast vector search by Facebook |
| `langchain` | Framework that connects everything (retrieval, chains, agents) |
| `llama-index` | Framework focused on RAG pipelines |
| `openai` | API for embeddings and LLM calls |
| `sentence-transformers` | Free local embedding models |


---

## 11. One-Line Summary

**RAG** gives your AI **extra knowledge at runtime** by retrieving relevant data and injecting it into the prompt, so the model answers based on **your facts**, not just its training.

---

## 12. Roadmap

| # | Topic | Difficulty | Status |
|---|-------|------------|--------|
| 1 | What is RAG? | Beginner | **Done** |
| 2 | The Problem RAG Solves | Beginner | **Done** |
| 3 | How RAG Works (The Big Picture) | Beginner | **Done** |
| 4 | The Three Steps of RAG | Beginner | **Done** |
| 5 | RAG vs Fine-Tuning vs Prompt Engineering | Beginner | **Done** |
| 6 | Types of RAG | Beginner | **Done** |
| 7 | Building Basic RAG | Beginner | **Done** (built in [Prompt Critic & Optimizer](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer)) |
| 8 | When to Use RAG | Beginner | **Done** |
| 9 | Common Mistakes | Beginner | **Done** |
| 10 | Quick Reference Cheatsheet | Beginner | **Done** |
| 11 | Embeddings and Vector Search | Intermediate | **Coming soon** |
| 12 | Chunking Strategies | Intermediate | **Coming soon** |
| 13 | Vector Databases | Intermediate | **Coming soon** |
| 14 | Building Advanced RAG (Vector DB project) | Intermediate | **Coming soon** |
| 15 | RAG Pipeline Architecture | Intermediate–Advanced | **Coming soon** |
| 16 | Common RAG Patterns (Top-K, Hybrid, Re-ranking) | Intermediate–Advanced | **Coming soon** |
| 17 | Agentic RAG | Advanced | **Coming soon** |

> Sections marked **"Coming soon"** will be added once I build hands-on projects for them.

---

*Notes paired with the [Prompt Critic & Optimizer](https://github.com/hasnaat-iftikhar/prompt-critic-and-optimizer) project, which analyzes, critiques, and improves your prompts using static RAG to inject prompt engineering best practices.*
