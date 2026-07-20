# Shorthills AI — Interview Prep (AI Product Engineer)

> Company: **Shorthills AI** (Generative AI + Data Engineering company)
> Role: **AI Product Engineer**
> Interview date: Tomorrow
> Format: Based on past campus hiring — 3 rounds (HR/project deep-dive, Technical, Director/Cultural fit)

---

## About Shorthills AI (know this before you walk in)

- **What they do:** A Generative AI and data engineering company that builds custom AI solutions for enterprises — AI agents, chatbots, RAG pipelines, data lakehouses.
- **Founded:** 2018, named after Short Hills, New Jersey.
- **Offices:** India, USA, Australia, Canada.
- **Key services:** Agentic AI (multi-agent orchestration), RAG systems, data engineering (Azure/Databricks), enterprise AI strategy.
- **Industries:** Healthcare, automotive, retail, real estate, professional services.
- **Their products:** JumpIQ (AI-powered data platform), custom AI chatbots, deep research agents.
- **Tech stack they use:** Python, LangChain, vector databases, FAISS, Azure, Databricks, LLMs.

**Why this matters for you:** Your resume is a near-perfect match — you've built multi-agent systems, RAG pipelines, voice bots. They'll see you as someone who can hit the ground running.

---

## Shorthills AI Interview Process (from campus hiring data)

Based on the GeeksforGeeks 2024 on-campus experience:

### Round 1 — HR + Project Discussion
- Background, projects, internships, skill sets
- Questions about APIs
- Testing basic skills and interest in the company
- **~34 → 15 students passed**

### Round 2 — Technical Assessment
- Programming language skills (Python, C++)
- OOP concepts (4 pillars, virtual functions, polymorphism)
- Basic Python knowledge
- **~15 → 7 students passed**

### Round 3 — Director Round (Cultural Fit)
- HR questions, resume deep-dive
- Cultural fit assessment
- **~7 → 1 selected**

**For your AI Product Engineer role**, expect heavier emphasis on:
- Your RAG/agent projects (deep technical questions)
- System design for AI products
- LLM concepts, embeddings, vector search
- Python proficiency
- How you think about building AI products end-to-end

---

## SECTION 1: Questions from YOUR RESUME (with detailed answers)

### DMI Infotech — Voice Bot Project

**Q: Walk me through the multi-agent voice bot architecture you built.**

A: I built a voice bot for Yamada Bee Farm (a Japanese company) with 6 specialized agents:
1. **Customer Support Agent** — handles FAQs, order status, complaints
2. **Sales Agent** — product recommendations, upselling
3. **Subscription Agent** — manages recurring orders
4. **Order Management Agent** — modifications, cancellations, returns
5. **Routing/Triage Agent** — determines caller intent and routes to the right agent
6. **Fallback/Escalation Agent** — handles edge cases, transfers to human

The system handles real-time voice in Japanese and English, automating 80%+ of inbound calls.

**Q: What is FSM-driven workflow? Why did you use it?**

A: FSM = Finite State Machine. Each agent has defined states (greeting, listening, acting, confirming, closing) and transitions between them. I designed 30+ state graphs because:
- Voice conversations are inherently stateful (you can't process utterances out of order)
- FSM gives deterministic control over conversation flow — no hallucinated transitions
- Each state has **tool visibility rules** (an agent in "confirming" state can't trigger a new order)
- Transitions are triggered by intent classification or explicit user commands

**Q: Explain the hybrid RAG pipeline you integrated.**

A: I used FAISS (semantic search via embeddings) + BM25 (keyword matching) across 500+ products:
- **FAISS component (weight ~0.7-0.8):** Products embedded using a sentence-transformer model, stored in a FAISS index. At query time, the user's question is embedded and we find top-K nearest neighbors.
- **BM25 component (weight ~0.2-0.3):** Classical TF-IDF-based keyword search — catches exact product names, codes, numbers that embeddings might miss.
- **Fusion:** Reciprocal Rank Fusion (RRF) or weighted score combination.
- **Result:** ~30% improvement in retrieval accuracy because semantic search alone misses exact keywords, and keyword search alone misses meaning.

**Q: What is LiveKit Agents SDK? How did you achieve sub-500ms latency?**

A: LiveKit is a real-time communication framework. I used their Agents SDK to build the STT → LLM → TTS pipeline:
- **STT (Speech-to-Text):** Streams audio in real-time, partial transcripts available as the user speaks
- **LLM:** Processes the transcript, generates a response
- **TTS (Text-to-Speech):** Converts response back to speech, streams first audio chunk before full response is ready

For sub-500ms:
- **Streaming everywhere** — don't wait for full transcription; start LLM inference on partial input
- **Buffer speech** — while agent-to-agent handoff happens, play a natural filler ("Let me check that for you...") to mask the transition latency
- **Agent-to-agent handoff:** The new agent receives the conversation context immediately; no re-processing

**Q: How do you handle agent-to-agent handoffs?**

A: When the routing logic detects the conversation needs a different specialist:
1. Current agent saves conversation state (context, user info, pending items)
2. Buffer speech plays to the caller
3. New agent initializes with the saved context
4. Handoff is seamless — caller doesn't notice a "transfer"

---

### Hero MotoCorp — Analytics Internship

**Q: What did you do at Hero MotoCorp?**

A: I worked in the Digital & IT department using Adobe Analytics to track customer journeys across VIDA (electric) and ICE (petrol) websites. Key contributions:
1. Tracked KPIs and lead conversion funnels → identified changes that increased conversion by 15%
2. Analyzed 50,000+ records in Python → found 100CC Deluxe captured 40% of first-time sales (age 20-40) and 77.7% of repeat purchases in 24-36 months
3. Discovered low finance adoption (16%) with regional hotspots (2-3x higher in Uttarakhand, Puducherry, Assam) → enabled targeted campaigns boosting adoption by 10%

**Q: What ETL process did you execute?**

A: Extracted raw clickstream and transaction data from Adobe Analytics APIs → cleaned/transformed with Pandas (handling nulls, deduplication, type casting, merging datasets) → loaded into structured formats for visualization in Tableau. Reduced data processing time by 35%.

---

### DocMind — Local RAG Pipeline

**Q: Explain the DocMind architecture end-to-end.**

A:
```
PDF Upload → Text Extraction → Recursive Chunking (sentence-aware)
→ Embedding (Sentence Transformers) → FAISS Index + BM25 Index
→ User Query → Hybrid Retrieval (70% FAISS + 30% BM25)
→ Cross-Encoder Reranking → Top-K chunks → Mistral 7B (local LLM)
→ Answer with Source Citations
```

Key design decisions:
- **Privacy-first:** Mistral 7B runs locally, zero API calls
- **Sentence-aware recursive chunking:** Split by paragraphs first, then sentences, respecting semantic boundaries (not mid-sentence cuts)
- **Content-hash embedding cache:** If a PDF chunk hasn't changed, skip re-embedding → 40-60% faster repeat ingestion
- **Cross-encoder reranking:** After initial retrieval of top-20, a cross-encoder re-scores them for relevance → ~25% better answer quality
- **Architecture:** FastAPI backend (stateless REST API) + Streamlit frontend (chat UI with citations)

**Q: Why hybrid search (FAISS + BM25) instead of just FAISS?**

A: Pure semantic search (FAISS) is great for meaning-based queries but:
- Misses exact matches (document ID "XYZ-123", specific names, acronyms)
- BM25 catches these because it's keyword-based (TF-IDF)
- Together they complement: FAISS finds conceptually relevant chunks, BM25 ensures exact-term chunks aren't missed
- The 70/30 weighting was empirically tuned on my test queries

**Q: What is cross-encoder reranking and why is it needed?**

A: 
- **Bi-encoder (embedding search):** Encodes query and documents independently → fast but approximate
- **Cross-encoder:** Takes (query, document) PAIR as input → much more accurate but slow (O(n) forward passes)
- **Solution:** Use bi-encoder for fast recall (top-20), then cross-encoder to rerank just those 20 → best of both worlds
- Improved answer relevance by ~25%

**Q: How does the embedding cache work?**

A: 
- When a PDF is ingested, each chunk gets a content hash (MD5/SHA256 of the text)
- Embeddings are stored mapped to their content hash
- On re-ingestion, if the hash matches, we skip the embedding computation
- Result: 40-60% time saved when updating a document that's mostly unchanged

---

### Aadhaar KYC Fraud Detection

**Q: Explain the fraud detection pipeline.**

A:
```
Input: Live selfie via WebRTC + Aadhaar card image
    ↓
1. Face Extraction (MTCNN)
   → detect & crop face regions from BOTH selfie and Aadhaar
   → foundation step — every downstream model needs the face crop, not the full image
    ↓
2. Liveness Detection (Mediapipe Face Mesh + EAR blink tracking)
   → runs on live WebRTC selfie stream (in parallel)
   → require ≥7 natural blinks to confirm a real, live person
    ↓
3. Deepfake Detection (Fine-tuned EfficientNet-B0)
   → runs on the extracted selfie face crop
   → trained on 10K StyleGAN3 images → 97.5% accuracy
   → outputs deepfake probability + Grad-CAM heatmap for interpretability
    ↓
4. Face Matching (FaceNet)
   → embed both extracted faces (selfie & Aadhaar)
   → cosine similarity → face-dissimilarity score
    ↓
5. Multi-Signal Decision Engine
   → 40% deepfake probability + 35% face dissimilarity + 25% liveness penalty
   → 0-100% risk score → Verified / Suspicious / Fraudulent
    ↓
6. Privacy layer: Aadhaar number masked via Gaussian blur
```

**Why face extraction comes first:** Deepfake and face-matching models expect tightly cropped face images, not the full scene. Feeding raw selfies (with backgrounds, body, lighting noise) would degrade accuracy. MTCNN normalizes the input first, then the specialized models run on clean face crops.

**Q: What is EAR (Eye Aspect Ratio) for liveness?**

A: EAR = ratio of the vertical eye distance to horizontal eye distance (from facial landmarks). When someone blinks, EAR drops sharply. I track blinks in real-time via WebRTC:
- If user blinks ≥7 times naturally → they're live (not a photo/video)
- If no blinks detected → likely a printed photo or screen replay → flag as suspicious

**Q: Why EfficientNet-B0 specifically?**

A: It's the best accuracy-to-compute tradeoff for binary classification (real vs fake). B0 is the smallest variant — fast inference for real-time use, yet still achieved 97.5% accuracy. Larger models (B3, B7) gave marginal gains not worth the latency.

**Q: What is Grad-CAM and why did you use it?**

A: Gradient-weighted Class Activation Mapping — it highlights which regions of the image the model "looked at" to make its decision. I used it for **interpretability**: when we classify something as "fake," Grad-CAM shows whether the model focused on face artifacts (correct reasoning) or background noise (wrong reasoning). This builds trust in the system.

---

## SECTION 2: Technical Concepts They'll Ask

### RAG (Retrieval-Augmented Generation)

**Q: What is RAG and why is it needed?**

A: RAG augments an LLM with external knowledge retrieved at query time. Needed because:
- LLMs have knowledge cutoffs (don't know recent data)
- LLMs hallucinate when asked about specific/private documents
- RAG grounds the model in real data → more accurate, citable answers

**Q: Chunking strategies — what are the options?**

A:
- **Fixed-size:** 500 tokens per chunk (simple but cuts mid-sentence)
- **Recursive/sentence-aware:** Split by paragraphs → sentences → words, respecting boundaries
- **Semantic chunking:** Use embeddings to detect topic shifts, split at those boundaries
- **Overlapping:** Add 50-100 token overlap between chunks so context isn't lost at boundaries

**Q: What are embeddings?**

A: Dense vector representations (e.g., 768 or 1536 dimensions) that capture semantic meaning. Similar texts → similar vectors (close in vector space). Generated by models like Sentence-BERT, OpenAI text-embedding-3-large.

**Q: FAISS vs other vector databases?**

A:
- **FAISS:** Facebook's library, runs in-memory, extremely fast for brute-force or IVF search. Best for local/offline use.
- **Pinecone:** Cloud-managed, auto-scaling, easy to use. Best for production SaaS.
- **Weaviate/Qdrant/Chroma:** Self-hosted alternatives with metadata filtering.
- I chose FAISS because DocMind is local/privacy-first (no cloud).

---

### LLMs & Agents

**Q: What is a multi-agent system?**

A: Multiple AI agents, each specialized for a task, working together. A routing/orchestrator agent decides which specialist to invoke based on user intent. Benefits:
- Each agent is focused → better at its specific task
- Modular → easy to add/remove agents
- Tool isolation → agents only see tools relevant to their role

**Q: What is LangChain vs LangGraph?**

A:
- **LangChain:** Library for building LLM applications — chains, prompts, retrievers, tools, memory.
- **LangGraph:** Extension for building **stateful, multi-actor** applications as graphs. Nodes = agents/functions, edges = transitions. Built for complex workflows where LangChain's linear chains aren't enough.

**Q: What is prompt engineering?**

A: Designing the input text to an LLM to get the desired output. Techniques:
- Few-shot examples
- Chain-of-thought ("think step by step")
- System prompts (role, constraints, output format)
- Structured output (JSON mode)

**Q: How do you evaluate RAG systems?**

A: Metrics:
- **Retrieval:** Recall@K, MRR, NDCG (are the right chunks being retrieved?)
- **Generation:** Faithfulness (does the answer match retrieved context?), relevance, completeness
- **End-to-end:** Human eval, LLM-as-judge (what we did in the SMS classifier)
- Tools: RAGAS framework, custom benchmarks

---

### Python & OOP (they WILL ask this)

**Q: 4 pillars of OOP?**

A:
1. **Encapsulation:** Bundling data + methods; hiding internal state (_private)
2. **Inheritance:** Child class inherits parent's attributes/methods
3. **Polymorphism:** Same interface, different behavior (method overriding)
4. **Abstraction:** Hide complexity, expose only what's needed

**Q: What are decorators in Python?**

A: Functions that wrap other functions to extend behavior without modifying them. Example: `@timer` to measure execution time, `@lru_cache` for memoization.

**Q: Difference between list and tuple?**

A: List = mutable (can change), Tuple = immutable (can't change). Tuples are hashable (can be dict keys), slightly faster, and signal "this data shouldn't be modified."

**Q: What is GIL?**

A: Global Interpreter Lock — CPython allows only one thread to execute Python bytecode at a time. For CPU-bound parallelism, use multiprocessing. For I/O-bound concurrency, use threading or asyncio.

**Q: What are generators?**

A: Lazy iterators using `yield`. They produce values one at a time, not all in memory. Use for large datasets or infinite sequences.

---

### APIs

**Q: What is a REST API?**

A: Representational State Transfer — architectural style for web services. Key principles:
- Stateless (each request contains all info needed)
- Resources identified by URLs
- HTTP methods: GET (read), POST (create), PUT (update), DELETE (remove)
- Returns JSON typically

**Q: What is FastAPI and why did you use it?**

A: Modern Python web framework for building APIs. Advantages:
- Auto-generates OpenAPI docs
- Type hints → automatic validation
- Async support (high concurrency)
- Very fast (built on Starlette + Pydantic)

---

## SECTION 3: System Design Questions (AI Product Engineer specific)

**Q: Design a customer support chatbot for an enterprise.**

A: 
1. **Data ingestion:** Crawl company docs/FAQs → chunk → embed → vector store
2. **Query pipeline:** User question → embed → retrieve top-K → rerank → feed to LLM with context
3. **Conversation memory:** Store last N turns for multi-turn context
4. **Fallback:** If confidence is low, escalate to human agent
5. **Monitoring:** Log queries, track resolution rate, detect drift
6. **Guardrails:** Input/output filtering, don't hallucinate, stay on-topic

**Q: How would you handle hallucination in a RAG system?**

A:
- Better retrieval (more relevant chunks → less room to hallucinate)
- Prompt engineering ("Only answer based on the provided context. If unsure, say you don't know.")
- Citation enforcement (model must cite chunk IDs)
- Post-generation verification (check if answer is grounded in retrieved text)
- Temperature = 0 for factual tasks

**Q: How would you deploy an AI voice bot to production?**

A:
- Containerize with Docker
- Use WebSocket/WebRTC for real-time audio streaming
- Auto-scaling based on concurrent calls
- Latency monitoring (STT → LLM → TTS pipeline should be <500ms)
- Fallback to human if bot confidence drops
- A/B testing different prompts/models
- Logging all conversations for quality review

---

## SECTION 4: Behavioral / HR Questions

**Q: Tell me about yourself.**

A: "I'm Ronit Sharma, final year B.Tech from DTU. I'm currently working as a Data Science Intern at DMI Infotech where I built a multi-agent voice bot handling real-time voice interactions in Japanese and English for a Japanese client. Before that, I interned at Hero MotoCorp doing customer analytics. My core interest is building AI products — I've worked on RAG pipelines, multi-agent systems, fraud detection, and voice AI. I'm passionate about taking AI from prototype to production."

**Q: You're from Mechanical Engineering — why did you shift to AI/Data Science?**

A: "I entered DTU in Mechanical, but during my second year I started learning Python for automating engineering calculations and got hooked on programming. I took Andrew Ng's ML course on Coursera, did Kaggle competitions, and realized I was spending all my free time on ML — not thermodynamics. The shift wasn't overnight — I systematically built skills: first Python and stats, then ML, then deep learning, then NLP and LLMs. By the time I started applying for internships, I had real projects (DocMind, Aadhaar fraud detection) to prove I wasn't just a hobbyist. My mechanical background actually helps — I think in systems: inputs, processes, outputs, feedback loops. That's exactly how you design AI pipelines."

**Q: Don't you think you're at a disadvantage compared to CS students?**

A: "Honestly, no — not at this stage. In the first year, maybe. But I spent 2+ years filling any gap: DSA, OOP, system design, databases, APIs — all self-taught. And I have something many CS students don't — the hunger of someone who chose this path deliberately, not by default. My internship at DMI building a production voice bot proves I can deliver at the same level. The domain knowledge (mechanical) is a bonus when working with manufacturing or automotive clients — like I did at Hero MotoCorp."

**Q: Why Shorthills AI?**

A: "Shorthills AI works at the intersection of Generative AI and data engineering — building custom AI agents and RAG systems for enterprises. That's exactly what I've been doing at DMI Infotech and in my projects. I want to work somewhere where I can build production AI systems end-to-end, and Shorthills' focus on agentic AI and enterprise solutions is the right fit for my skills and interests. Also, as a growing company, I'll get more ownership and exposure to diverse problems compared to a large MNC."

**Q: Why AI Product Engineer and not pure Data Scientist?**

A: "I enjoy the full product lifecycle — not just building models, but designing the system around them: APIs, pipelines, deployment, user experience. At DMI, I didn't just train a model — I built the entire voice bot system from architecture to deployment. That's what excites me. A data scientist might build the retrieval model; I want to build the whole product around it."

**Q: Your biggest technical challenge?**

A: "Achieving sub-500ms voice latency in the multi-agent bot. The naive approach (wait for full STT → process → full TTS) gave 2-3 second delays. I solved it by streaming at every stage — partial transcripts to the LLM, streaming LLM output to TTS, and using buffer speech during agent handoffs. Getting all these async pipelines to work reliably in production was the hardest part."

**Q: Tell me about a time you failed or something didn't work.**

A: "In DocMind, my first approach to chunking was fixed-size (500 tokens). The answers were mediocre because chunks cut mid-sentence and lost context. I spent two days debugging the LLM prompt thinking it was a generation issue, before realizing the problem was upstream — bad chunks = bad retrieval = bad answers. I rebuilt the chunker with sentence-aware recursive splitting and overlap, and answer quality jumped immediately. Lesson: in RAG, retrieval quality matters more than the LLM itself."

**Q: Where do you see yourself in 3-5 years?**

A: "I want to become a senior AI engineer who can architect end-to-end AI systems — from data pipelines to production deployment. In 3 years, I see myself leading a small team building AI agents or products. Long-term, I'm interested in the product side — understanding what to build, not just how to build it. A role at Shorthills gives me that foundation because you work directly with clients and ship real products."

**Q: What are your strengths and weaknesses?**

A: 
- **Strengths:** I learn fast and go deep — I went from zero ML knowledge to building production AI systems in 2 years. I'm also good at connecting the dots between different technologies (RAG + voice + agents + real-time systems).
- **Weakness:** I sometimes over-engineer solutions. In the voice bot project, I initially designed a complex routing system with 10+ intent categories when the client only needed 4. My manager helped me learn to start simple and iterate. I now always ask "what's the simplest thing that could work?" first.

**Q: Why should we hire you?**

A: "Three reasons: First, I've already built exactly what Shorthills delivers — multi-agent systems, RAG pipelines, production AI products. I won't need months of ramp-up. Second, I've worked in a client-facing environment (DMI Infotech, Japanese client) so I understand delivering to real requirements, not just building demos. Third, I'm hungry — I chose this field deliberately, taught myself everything, and I bring the energy of someone who genuinely loves this work."

**Q: Are you comfortable working on things outside your comfort zone?**

A: "That's literally been my journey — mechanical engineer building Japanese voice bots. At DMI, I had to learn LiveKit, real-time audio processing, Japanese language handling — all new to me. I'm at my best when I'm slightly uncomfortable because it means I'm growing."

**Q: Do you have any questions for me?**

A: Always say YES. (See Section 5 below.)

**Q: What's your expected CTC / salary expectation?**

A: "I'm flexible on this — I'm more focused on the learning opportunity and the kind of work I'll get to do. I trust Shorthills offers fairly for the role. That said, I'd appreciate it if you could share the range for this position so we can align."

(Tip: don't throw a number first if you can avoid it. If pressed, research the range — for a fresher AI role at Shorthills, expect 6-10 LPA based on similar companies.)

**Q: Are you open to relocation?**

A: "Yes, absolutely. I'm currently in Delhi and open to relocating wherever the team is based."

**Q: Do you have other offers / are you interviewing elsewhere?**

A: Be honest but brief: "I'm in the process with a couple of other companies, but Shorthills is my priority because of the alignment with what I want to build." (Shows you're in demand without being arrogant.)

---

## SECTION 5: Questions YOU should ask them

1. "What does the AI product engineering team work on day-to-day? Is it more R&D or client delivery?"
2. "What LLM providers and frameworks do you primarily use? (OpenAI, Azure, open-source?)"
3. "What's the typical project lifecycle — from client requirements to production deployment?"
4. "How much autonomy do engineers get in choosing technical approaches?"
5. "What's the team structure — how many people would I collaborate with?"

---

## Quick Revision Checklist (read this 30 min before)

- [ ] Know your DMI voice bot architecture cold (6 agents, FSM, LiveKit, hybrid RAG)
- [ ] Know DocMind architecture (chunking → embed → FAISS+BM25 → rerank → Mistral → answer)
- [ ] Know Aadhaar fraud pipeline (EfficientNet → FaceNet → EAR liveness → risk score)
- [ ] OOP 4 pillars + Python generators + decorators + GIL
- [ ] RAG = retrieve relevant chunks → feed to LLM with context
- [ ] FAISS = vector similarity search; BM25 = keyword search; together = hybrid
- [ ] Cross-encoder reranking = more accurate but slower, used on top-K results only
- [ ] Shorthills AI = GenAI + data engineering, builds AI agents for enterprises
- [ ] Your pitch: "I build production AI systems end-to-end — agents, RAG, voice"

---

Good luck! Your resume is genuinely strong for this role. Be confident, explain your systems clearly, and show you understand the *why* behind your technical choices.
