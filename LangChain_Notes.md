# LangChain — Complete Detailed Guide for AI Engineer Interview

---

## 1. What is LangChain? (The Big Picture)

**Simple answer:** LangChain is a Python framework that makes it easy to build applications powered by LLMs (Large Language Models like GPT-4, Claude, etc.)

**Why does it exist?** Because using an LLM directly (just calling the API) is limited:
- LLMs don't know your company's data
- LLMs can't access the internet or databases
- LLMs have no memory (each call is independent)
- LLMs can't take actions (send emails, update records)

LangChain solves ALL of these by providing building blocks to:
- Connect LLMs to YOUR data (RAG)
- Give LLMs tools (search, calculator, database queries)
- Add memory (remember past conversations)
- Chain multiple steps together (prompt → LLM → parse → store)

**Real-world analogy:** Think of an LLM as a very smart person sitting in a room with no phone, no internet, no files. LangChain gives them a phone (tools), a filing cabinet (memory), a library (RAG), and a to-do list (chains) so they can actually get work done.

---

## 2. Core Architecture — How LangChain is Organized

LangChain is split into multiple packages (important for imports!):

```
langchain-core       → Foundation: Runnables, prompts, parsers, LCEL
langchain            → High-level: chains, agents, memory
langchain-openai     → OpenAI integration (ChatGPT, embeddings)
langchain-community  → 3rd party integrations (Chroma, BM25, Wikipedia, etc.)
langchain-chroma     → ChromaDB vector store specifically
langgraph            → Stateful agent workflows (separate framework)
```

**Why the split?** So you only install what you need. A RAG app doesn't need the agent package. An agent doesn't need the ChromaDB package.

**Common mistake:**
```python
# WRONG (old import, breaks in new versions)
from langchain.chat_models import ChatOpenAI

# CORRECT
from langchain_openai import ChatOpenAI
```

---

## 3. LLMs & Chat Models — Talking to AI

**The problem:** Each LLM provider (OpenAI, Anthropic, Google, Mistral) has its own SDK, its own function names, its own response format. If you write code against OpenAI's SDK and later want to switch to Claude, you'd have to rewrite everything.

**The solution — a unified interface:** LangChain wraps every provider behind ONE standard interface (`.invoke()`, `.stream()`, etc.). Swapping `ChatOpenAI` for `ChatAnthropic` is a one-line change — the rest of your code stays identical. This is why the chat model is the foundation everything else builds on.

### The simplest thing: just call an LLM

```python
from langchain_openai import ChatOpenAI

# Create the model
llm = ChatOpenAI(model="gpt-4o", temperature=0)

# Call it
response = llm.invoke("What is the capital of France?")
print(response.content)  # "The capital of France is Paris."
```

### What is `temperature`?

Temperature controls randomness:
- `temperature=0` → Always gives the SAME answer. Best for factual Q&A, data extraction, code.
- `temperature=0.7` → Balanced creativity. Good for conversation.
- `temperature=1.0` → Maximum creativity. Best for creative writing, brainstorming.

**Rule:** For anything where accuracy matters (RAG, data extraction, agents) → use temperature=0.

### Message Types

Chat models use messages with roles:

```python
from langchain_core.messages import HumanMessage, SystemMessage, AIMessage

messages = [
    SystemMessage(content="You are a helpful assistant."),      # sets behavior
    HumanMessage(content="What is LangChain?"),                 # user's question
    AIMessage(content="LangChain is a framework for..."),       # AI's past response
    HumanMessage(content="Tell me more about RAG."),            # follow-up
]

response = llm.invoke(messages)
```

- **SystemMessage:** Instructions for the AI (persona, rules, constraints)
- **HumanMessage:** What the user says
- **AIMessage:** What the AI previously said (for context in multi-turn)

---

## 4. Prompt Templates — Reusable Prompts with Variables

**The problem — hardcoded prompts don't scale.** Imagine you're building a translation app. Without templates you'd write the prompt with the text glued directly inside:
```python
# The painful way — a new string every time, glued together by hand
prompt1 = "Translate 'Hello' from English to Japanese"
prompt2 = "Translate 'Goodbye' from English to Japanese"
prompt3 = "Translate 'Thank you' from English to French"
# ...you're rebuilding the whole sentence for every single request
```
This is messy, error-prone (easy to typo the instruction), impossible to reuse, and mixes your DATA (the word to translate) with your INSTRUCTION (how to translate). If you want to tweak the wording of the instruction, you'd have to change it in a hundred places.

**The solution — a template with blanks to fill in.** A prompt template is like a fill-in-the-blank form: you write the instruction ONCE with `{placeholders}`, then just supply the values each time.

```python
from langchain_core.prompts import ChatPromptTemplate

# Write the instruction ONCE, with blanks
prompt = ChatPromptTemplate.from_template(
    "Translate '{text}' from {source} to {target}"
)

# Reuse it with different values — instruction stays fixed
prompt.format(text="Hello", source="English", target="Japanese")
prompt.format(text="Goodbye", source="English", target="French")
```

Now the instruction lives in one place, and only the data changes. That's the whole point.

### Simple template:
```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a {role} who speaks {language}."),
    ("human", "{question}")
])

# Fill in the variables
messages = prompt.format_messages(
    role="chef",
    language="Japanese",
    question="How do I make ramen?"
)

response = llm.invoke(messages)
```

### Why use templates?

1. **Reusable:** Same template, different inputs
2. **Testable:** Can test prompt logic independently from LLM
3. **Version control:** Track prompt changes in git
4. **Separation of concerns:** Data logic separate from prompt logic

### Template with multiple variables and context (RAG pattern):
```python
rag_prompt = ChatPromptTemplate.from_messages([
    ("system", """You are a product assistant. Answer questions using ONLY 
    the provided context. If the answer is not in the context, say 
    "I don't have that information."
    
    Context: {context}"""),
    ("human", "{question}")
])
```

---

## 5. Output Parsers — Getting Structured Data from LLMs

**The problem — LLMs only speak text, but your code needs data.** An LLM always replies with a blob of human text. But your program can't DO anything with a paragraph. Say you ask for product details:
```
LLM replies: "Sure! The Clever Cutter costs 1580 yen and it's in stock."
```
Your code can't easily pull "1580" out of that sentence to, say, save it in a database or compare prices. You'd be stuck writing fragile string-slicing code, and the LLM might phrase it differently every time ("It's priced at ¥1580" vs "1580 yen").

**The solution — parsers convert text into usable structures.** An output parser does two jobs: (1) it tells the LLM what format to reply in, and (2) it converts that reply into a clean Python object (string, list, dict, or a typed object) your code can use directly.

```
LLM text  →  [Output Parser]  →  clean Python data
"...1580 yen..."              →  {"name": "Clever Cutter", "price": 1580, "available": true}
                                  ↑ now your code can do data["price"] < 2000
```

### String Parser (simplest):
```python
from langchain_core.output_parsers import StrOutputParser

parser = StrOutputParser()
# Just extracts the text content from the response
```

### JSON Parser (structured output):
```python
from langchain_core.output_parsers import JsonOutputParser
from pydantic import BaseModel, Field

# Define the structure you want
class ProductInfo(BaseModel):
    name: str = Field(description="Product name")
    price: float = Field(description="Price in yen")
    available: bool = Field(description="Is it in stock?")

parser = JsonOutputParser(pydantic_object=ProductInfo)

# The parser adds formatting instructions to the prompt automatically
# LLM will return: {"name": "Clever Cutter", "price": 1580, "available": true}
```

### With Structured Output (newer, better approach):
```python
# Forces LLM to return valid JSON matching the schema
structured_llm = llm.with_structured_output(ProductInfo)
result = structured_llm.invoke("Tell me about Clever Cutter")
# result is a ProductInfo object directly!
```

---

## 6. LCEL (LangChain Expression Language) — The Pipe Operator

**The problem — connecting components by hand is ugly and limited.** A real LLM app is a sequence of steps: fill a prompt → call the LLM → parse the output. Without LCEL you'd nest these as function calls:
```python
# The painful way — nested, hard to read, runs strictly one-at-a-time
result = parser.parse(llm.invoke(prompt.format(topic="dogs")))
```
And if you later wanted streaming (show tokens as they arrive), batching (process 100 inputs at once), async, or automatic retries — you'd have to write ALL of that plumbing yourself, for every chain.

**The solution — connect steps with a pipe `|`, like a factory line.** LCEL lets you describe the flow left-to-right, exactly as it runs. The output of each step automatically becomes the input of the next.

```python
chain = prompt | llm | parser
#        step1    step2  step3   → reads like a sentence
```

The magic bonus: because every piece follows the same standard (a "Runnable"), the WHOLE chain instantly gets streaming, batching, async, and retries for free — you write zero plumbing. That's why LCEL is the heart of modern LangChain.

```python
chain = prompt | llm | parser
```

**How to read this:** "Take prompt output → feed to LLM → feed to parser"

### Full example:
```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template("Tell me a joke about {topic}")
llm = ChatOpenAI(model="gpt-4o", temperature=0.7)
parser = StrOutputParser()

# Chain them
chain = prompt | llm | parser

# Run it
result = chain.invoke({"topic": "programming"})
print(result)  # "Why do programmers prefer dark mode?..."
```

### What makes LCEL powerful:

Every component is a **Runnable** with these methods:
- `.invoke(input)` → process one input, return one output
- `.stream(input)` → yield output tokens one by one (real-time)
- `.batch([inputs])` → process multiple inputs in parallel
- `.ainvoke(input)` → async version of invoke

So ANY chain automatically gets streaming, batching, and async for free!

### More complex LCEL — RAG chain:
```python
from langchain_core.runnables import RunnablePassthrough

def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

# What happens:
# 1. User question goes to retriever AND passes through as "question"
# 2. Retriever finds relevant docs, format_docs makes them text
# 3. Both go into prompt template (fills {context} and {question})
# 4. Filled prompt goes to LLM
# 5. LLM response gets parsed to string
```

### RunnablePassthrough — what is it?

It just passes input through unchanged. Used when you need the original input alongside transformed data:

```python
# "question" needs to be the raw user input (no transformation)
# "context" needs to be retrieved docs (transformed)
{
    "context": retriever | format_docs,   # transformed
    "question": RunnablePassthrough()      # unchanged, just pass through
}
```

### RunnableParallel — run things simultaneously:
```python
from langchain_core.runnables import RunnableParallel

parallel = RunnableParallel(
    summary=summary_chain,
    translation=translation_chain,
    keywords=keyword_chain,
)

# All three run at the same time!
result = parallel.invoke({"text": "Some long document..."})
# result = {"summary": "...", "translation": "...", "keywords": [...]}
```

---

## 7. Document Loaders — Getting Data In

**The problem — your data lives in a dozen different formats.** To build RAG over your own data, you first have to GET that data into the program. But it's scattered everywhere: PDFs, Word docs, CSVs, web pages, Notion, databases — each with a totally different way of being read. Reading a PDF needs `pdfplumber`; a web page needs an HTTP request + HTML parsing; a CSV needs the csv module. Writing and maintaining all that loading code is tedious and repetitive.

**The solution — loaders give every source the same output shape.** A document loader knows how to read one source type and always hands you back the SAME thing: a list of `Document` objects with `page_content` (the text) + `metadata` (where it came from). So the rest of your pipeline doesn't care whether the data started as a PDF or a webpage — it's all just `Document`s now.

```python
from langchain_community.document_loaders import (
    PyPDFLoader,         # PDF files
    CSVLoader,           # CSV files  
    JSONLoader,          # JSON files
    WebBaseLoader,       # Web pages
    TextLoader,          # Plain text files
    DirectoryLoader,     # Entire folders
)

# Load a PDF
loader = PyPDFLoader("contract.pdf")
documents = loader.load()
# Returns: [Document(page_content="...", metadata={"source": "contract.pdf", "page": 0}), ...]
```

Every loader returns `Document` objects with:
- `page_content` — the actual text
- `metadata` — source info (filename, page number, URL, etc.)

**Why metadata matters:** When your RAG answers a question, you can show the user WHERE the answer came from (which document, which page).

---

## 8. Text Splitters — Chunking Documents

**The problem — whole documents are too big to use.** Say you loaded a 50-page PDF. You can't feed the entire thing to the retrieval system because: (1) embeddings work best on small focused passages — embedding a whole book into one vector blurs all meaning into mush; (2) the LLM has a limited context window; (3) if you retrieve a giant document, 99% of it is irrelevant noise for any specific question. You need to find the ONE relevant paragraph, not the whole book.

**The solution — split documents into bite-sized chunks.** A text splitter breaks big documents into smaller pieces (e.g., 1000 characters each). Now each chunk is a focused passage that embeds cleanly and retrieves precisely — when someone asks a question, you fetch just the 3-4 relevant chunks instead of the entire document.

```
One 50-page PDF  →  [splitter]  →  ~200 small chunks
                                    each ≈ 1 paragraph, embeds & retrieves cleanly
```

### RecursiveCharacterTextSplitter (most common):

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,       # max characters per chunk
    chunk_overlap=200,     # overlap between consecutive chunks
    separators=["\n\n", "\n", ". ", " ", ""]  # try these in order
)

chunks = splitter.split_documents(documents)
```

**Why "recursive"?** It tries the first separator (\n\n = paragraphs). If a resulting chunk is still too big, it tries the next separator (\n = lines). Then sentences (". "). Then words (" "). Recursively uses finer separators until everything fits.

**Why overlap?** Without overlap, a sentence might be split across two chunks. With 200 char overlap, the end of chunk N is also the beginning of chunk N+1 — so context isn't lost at boundaries.

### Token-based splitting (for LLM context limits):
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    encoding_name="cl100k_base",  # GPT-4's tokenizer
    chunk_size=500,               # tokens, not characters
    chunk_overlap=50,
)
```

---

## 9. Vector Stores & Retrieval — RAG Core

**The problem — how do you find the RIGHT chunks for a question?** You've split your data into 200 chunks. A user asks "What's the return policy?" How do you find which chunks are relevant? Keyword matching fails badly here — the relevant chunk might say "refunds are accepted within 30 days" and never use the word "return" at all. You need to search by MEANING, not exact words.

**The solution — store chunks as vectors and search by similarity.** This is the core of RAG:
1. Convert each chunk into an embedding (a vector capturing its meaning).
2. Store all those vectors in a vector store (a database built for fast similarity search).
3. At query time, embed the question too, and find the chunks whose vectors are CLOSEST in meaning.

So "What's the return policy?" lands near "refunds accepted within 30 days" because they MEAN the same thing — even with zero shared words. The vector store + retriever is what makes this fast.

```
Question → embed → 🔍 find nearest vectors → return the most relevant chunks
```

### The RAG pipeline:

```python
from langchain_openai import OpenAIEmbeddings
from langchain_chroma import Chroma

# 1. Create embeddings
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

# 2. Store in vector DB
vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    persist_directory="./chroma_db"
)

# 3. Create retriever
retriever = vectorstore.as_retriever(
    search_type="similarity",  # or "mmr" for diversity
    search_kwargs={"k": 5}     # return top 5 results
)

# 4. Use it
docs = retriever.invoke("What is the return policy?")
```

### Search types:

- **similarity:** Return K most similar chunks (cosine similarity)
- **mmr (Maximal Marginal Relevance):** Return diverse results — avoids returning 5 nearly identical chunks. Balances relevance AND diversity.
- **similarity_score_threshold:** Only return chunks above a minimum similarity score

### Ensemble Retriever (hybrid search):

```python
from langchain.retrievers import EnsembleRetriever
from langchain_community.retrievers import BM25Retriever

# BM25 for keyword matching
bm25 = BM25Retriever.from_documents(chunks)
bm25.k = 5

# Vector for semantic matching
vector_retriever = vectorstore.as_retriever(search_kwargs={"k": 5})

# Combine both
ensemble = EnsembleRetriever(
    retrievers=[bm25, vector_retriever],
    weights=[0.3, 0.7]  # 30% keyword, 70% semantic
)
```

---

## 10. Memory — Remembering Conversations

**The problem — the LLM forgets everything between messages.** Each API call is completely independent. So in a chat:
```
You: "My name is Ronit."        AI: "Nice to meet you, Ronit!"
You: "What's my name?"          AI: "I don't know your name." ← it already forgot!
```
The second call only sent "What's my name?" — the model has no idea what came before. For a chatbot, this is useless: it can't follow up, can't remember context, can't hold a real conversation.

**The solution — memory re-feeds the past into each new call.** Memory stores the conversation history and automatically prepends it to the next prompt. So the model "sees" the earlier exchange every time and appears to remember. (Reminder: the model still isn't truly remembering — memory is just re-sending the old messages.)

### ConversationBufferMemory (stores everything):
```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory(return_messages=True)

# After each exchange, save it:
memory.save_context(
    {"input": "My name is Ronit"},
    {"output": "Nice to meet you, Ronit!"}
)

# Later, the AI can recall:
# Human: "What's my name?"
# AI: "Your name is Ronit." ← remembers because memory is injected into prompt
```

### Types of Memory:

| Type | What it stores | When to use |
|---|---|---|
| ConversationBufferMemory | ALL messages verbatim | Short conversations |
| ConversationBufferWindowMemory | Last K messages only | Long conversations (bounded) |
| ConversationSummaryMemory | LLM-generated summary of old messages | Very long conversations |
| ConversationTokenBufferMemory | Messages within token limit | When context window matters |

**The problem with memory:** As conversation grows, it uses more and more of the LLM's context window. Summary memory solves this by compressing old messages into a short summary.

---

## 11. Tools & Agents — LLMs That Take Actions

**The problem — an LLM alone can only TALK, not DO.** A raw LLM is a brain in a jar. It can't check today's weather, query your database, do precise math, send an email, or look anything up. Its knowledge is frozen at training time and it has no hands. Ask "what's the weather in Tokyo right now?" and it can only guess or admit it doesn't know.

**The solution — give the LLM tools, and let an agent decide when to use them.** A *tool* is just a Python function (search the DB, call a weather API, run a calculator). An *agent* is an LLM that can look at a question and DECIDE which tool(s) to call, in what order, then use the results to answer. This turns the "brain in a jar" into something that can actually act in the world.

```
Brain in a jar (LLM)  +  tools (hands)  +  agent (decision-maker)  =  can DO things
```

### What is a Tool?

A function that the LLM can decide to call:

```python
from langchain.tools import tool

@tool
def search_products(query: str) -> str:
    """Search the product database. Use this when the user asks about product details."""
    results = database.search(query)
    return str(results)

@tool
def get_weather(city: str) -> str:
    """Get current weather for a city. Use when user asks about weather."""
    return weather_api.get(city)
```

**Important:** The docstring is what the LLM reads to decide WHEN to use the tool. Write clear, specific descriptions!

### What is an Agent?

An LLM that decides WHICH tools to use and in WHAT order, based on the user's question:

```python
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_core.prompts import ChatPromptTemplate

tools = [search_products, get_weather]

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant with access to tools."),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}"),  # where tool results go
])

agent = create_tool_calling_agent(llm, tools, prompt)
executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

result = executor.invoke({"input": "What's the weather in Tokyo and find me a massage product"})
```

### Agent execution loop:

```
User: "What's the weather in Tokyo and find me a massage product"
    ↓
Agent thinks: "I need two tools — weather and product search"
    ↓
Agent calls: get_weather("Tokyo") → "25°C, sunny"
    ↓
Agent calls: search_products("massage") → "Nekoro, Life Up Smart, ..."
    ↓
Agent thinks: "I have both answers now"
    ↓
Agent responds: "The weather in Tokyo is 25°C and sunny. For massage products, 
                 I found Nekoro (¥12,800) and Life Up Smart (¥39,800)..."
```

### Difference between Chain and Agent:

| Chain | Agent |
|---|---|
| Fixed steps: A → B → C always | Dynamic: LLM decides next step |
| Predictable | Unpredictable |
| Fast | Slower (multiple LLM calls) |
| Easy to debug | Harder to debug |
| Use for RAG, simple pipelines | Use when decision-making needed |

---

## 12. Callbacks & Streaming

**The problem — a long LLM response feels frozen, and you can't see what's happening inside.** Two pains: (1) When an LLM takes 5 seconds to write a long answer, the user stares at a blank screen wondering if it crashed. (2) When a chain has many steps, and something goes wrong or costs too much, you have no visibility into what happened in the middle.

**The solution — streaming for the user, callbacks for the developer.** *Streaming* sends the answer token-by-token as it's generated, so the user sees text appearing immediately (like ChatGPT typing). *Callbacks* are hooks that fire on every internal event (LLM start, token, tool call, finish), letting you log, track tokens/cost, and debug what's happening inside the chain.

### Streaming (showing output as it's generated):
```python
# Instead of waiting for full response:
for chunk in chain.stream({"question": "Explain RAG"}):
    print(chunk, end="", flush=True)  # prints token by token
```

### Callbacks (monitoring what happens inside):
```python
from langchain.callbacks import StdOutCallbackHandler

# Prints every step to console
result = chain.invoke(
    {"question": "Hello"},
    config={"callbacks": [StdOutCallbackHandler()]}
)
```

Use callbacks for: logging, cost tracking (count tokens), latency monitoring, debugging.

---

## 13. Fallbacks & Retries

**The problem — APIs fail, and one failure kills your whole app.** LLM APIs are external services that hiccup constantly: rate limits (429 errors), timeouts, temporary outages, overloaded servers. If your app makes a call and the API fails, by default your entire request crashes — and your user sees an error.

**The solution — retry the same call, or fall back to a backup.** A *retry* automatically tries the same call again a few times (failures are often momentary). A *fallback* switches to a different model/provider if the first keeps failing (e.g., GPT-4o is down → automatically use GPT-3.5). Together they make your app resilient instead of fragile.

### Automatic retry on failure:
```python
# Retries 3 times with exponential backoff
llm_with_retry = llm.with_retry(stop_after_attempt=3)
```

### Fallback to different model:
```python
primary = ChatOpenAI(model="gpt-4o")
backup = ChatOpenAI(model="gpt-3.5-turbo")

llm_safe = primary.with_fallbacks([backup])
# If gpt-4o fails (rate limit, timeout) → automatically tries gpt-3.5-turbo
```

---

## 14. Caching — Avoid Redundant API Calls

**The problem — you pay (in time and money) for the same answer over and over.** LLM calls cost money per token and take time. If 100 users ask "What is your return policy?", you make 100 identical paid API calls that all produce the same answer. During development, re-running your script 50 times re-asks the same questions 50 times — burning money and waiting on latency for nothing.

**The solution — cache the answer the first time, reuse it after.** A cache stores the result of each prompt. The first time a prompt is seen, it hits the API. Every identical prompt after that returns the stored answer instantly and for free — no API call at all.

```python
from langchain.globals import set_llm_cache
from langchain.cache import InMemoryCache

set_llm_cache(InMemoryCache())

# First call: hits API (~1 second)
result1 = llm.invoke("What is 2+2?")

# Second identical call: returns cached answer (instant, free!)
result2 = llm.invoke("What is 2+2?")
```

Use caching during development (save money) and production (reduce latency for common queries).

---

## 15. When to Use LangChain vs Direct API Calls

**Use LangChain when:**
- Building RAG (vector stores, retrievers, chunking all provided)
- Need tools/agents
- Need memory
- Want streaming + batching automatically
- Rapid prototyping

**Use OpenAI SDK directly when:**
- Simple single LLM call
- Maximum performance (LangChain adds ~10-50ms overhead)
- You want full control over every detail
- The abstraction makes debugging harder

---

## 16. Embeddings — The Heart of RAG (Deep Dive)

### What an Embedding Actually Is

An embedding is a list of numbers (a vector) that represents the MEANING of a piece of text. Similar meanings → similar vectors.

```
"dog"     → [0.21, -0.45, 0.88, ..., 0.12]   (1536 or 3072 numbers)
"puppy"   → [0.23, -0.41, 0.85, ..., 0.15]   ← very close to "dog"
"airplane"→ [-0.67, 0.32, -0.12, ..., 0.91]  ← far from "dog"
```

The model learned during training that "dog" and "puppy" are used in similar contexts, so it places them close together in this high-dimensional space. "Airplane" lives in a totally different region.

### Why This Matters

Computers can't compare meaning directly — they can only compare numbers. Embeddings turn "do these two sentences mean similar things?" into "are these two vectors close together?" — a math problem we can solve fast.

### Measuring Similarity — Cosine Similarity

The most common measure is cosine similarity — the angle between two vectors.

```
cosine = 1.0  → identical direction → same meaning
cosine = 0.0  → perpendicular → unrelated
cosine = -1.0 → opposite direction → opposite meaning
```

It measures DIRECTION, not magnitude — so it doesn't matter if one text is longer than another, only whether they "point the same way" in meaning-space.

```python
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")  # 1536 dims
# or "text-embedding-3-large" (3072 dims, more accurate, more expensive)

vec = embeddings.embed_query("How do I return a product?")    # one query
vecs = embeddings.embed_documents(["chunk1 text", "chunk2 text"])  # many docs
```

### embed_query vs embed_documents

- `embed_query()` — embeds ONE search query (for retrieval time)
- `embed_documents()` — embeds a LIST of chunks (for build time, batched & efficient)

Some models embed queries and documents slightly differently (asymmetric embeddings), so use the right method for the right job.

### Choosing a Model — Dimensions Tradeoff

| Model | Dims | Quality | Cost | Speed |
|---|---|---|---|---|
| text-embedding-3-small | 1536 | Good | Cheap | Fast |
| text-embedding-3-large | 3072 | Best | 6.5× more | Slower |

More dimensions = captures finer nuance but uses more storage and compute. For most apps, `3-small` is the sweet spot. Use `3-large` when accuracy is critical (legal, medical).

### Key Rule: Same Model for Build and Query

You MUST embed your documents AND your queries with the SAME model. Different models produce vectors in different "spaces" that can't be compared. Mixing them = garbage retrieval.

---

## 17. Few-Shot Prompting — Teaching by Example

Sometimes instructions aren't enough — you show the LLM EXAMPLES of what you want.

### Zero-shot vs Few-shot

```
Zero-shot:  "Classify the sentiment: 'This is terrible'"
Few-shot:   "Here are examples:
             'I love it' → positive
             'It broke immediately' → negative
             Now classify: 'This is terrible'"
```

Few-shot gives the model a pattern to follow. Dramatically improves consistency for formatting, classification, and structured extraction.

### FewShotPromptTemplate

```python
from langchain_core.prompts import FewShotChatMessagePromptTemplate, ChatPromptTemplate

examples = [
    {"input": "2+2", "output": "4"},
    {"input": "5*3", "output": "15"},
]

example_prompt = ChatPromptTemplate.from_messages([
    ("human", "{input}"),
    ("ai", "{output}"),
])

few_shot = FewShotChatMessagePromptTemplate(
    example_prompt=example_prompt,
    examples=examples,
)

final_prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a math tutor."),
    few_shot,                          # examples injected here
    ("human", "{question}"),
])
```

### Dynamic Example Selection

Instead of fixed examples, pick the most RELEVANT examples for each query using an example selector (semantic similarity):

```python
from langchain_core.example_selectors import SemanticSimilarityExampleSelector

selector = SemanticSimilarityExampleSelector.from_examples(
    examples, embeddings, vectorstore_cls, k=2  # pick 2 most similar examples
)
```

This keeps the prompt short (only relevant examples) while still teaching the right pattern.

---

## 18. RunnableLambda, RunnableBranch & Custom Logic in Chains

LCEL isn't just prompt | llm | parser. You can inject custom Python anywhere.

### RunnableLambda — Wrap Any Function

```python
from langchain_core.runnables import RunnableLambda

def clean_text(text: str) -> str:
    return text.strip().lower()

# Now it's a Runnable you can pipe!
chain = RunnableLambda(clean_text) | prompt | llm | parser
```

Any function becomes a chain component. Use for: preprocessing input, post-processing output, custom transformations between steps.

### RunnableBranch — If/Else Routing

```python
from langchain_core.runnables import RunnableBranch

def is_question(x): return "?" in x["input"]

branch = RunnableBranch(
    (is_question, question_chain),   # if condition → this chain
    (lambda x: "urgent" in x["input"], urgent_chain),
    default_chain,                    # else → default
)
```

Routes input to different chains based on conditions. Like an if/elif/else for chains. (For complex routing with loops, use LangGraph instead.)

### RunnablePassthrough.assign — Add Fields

```python
from langchain_core.runnables import RunnablePassthrough

chain = RunnablePassthrough.assign(
    word_count=lambda x: len(x["text"].split())
)
# Input {"text": "..."} → Output {"text": "...", "word_count": 42}
# Keeps original AND adds new computed field
```

---

## 19. Modern Memory — RunnableWithMessageHistory

The old `ConversationBufferMemory` classes are being deprecated. The modern way to add memory to an LCEL chain is `RunnableWithMessageHistory`.

```python
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_community.chat_message_histories import ChatMessageHistory

# Store histories per session
store = {}
def get_history(session_id: str):
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]

chain_with_memory = RunnableWithMessageHistory(
    chain,
    get_history,
    input_messages_key="input",
    history_messages_key="history",  # where past messages get injected
)

# Use with a session id
chain_with_memory.invoke(
    {"input": "My name is Ronit"},
    config={"configurable": {"session_id": "user-123"}}
)
# Later, same session_id → remembers
```

**Why this replaces old memory:** It works natively with LCEL, supports multiple concurrent sessions (each with its own history), and plugs into persistent backends (Redis, Postgres) by swapping the history class. For full stateful agents though, LangGraph's checkpointing is the more powerful option.

---

## 20. Advanced Retrievers & Reranking

Basic similarity search is just the start. Production RAG uses smarter retrieval.

### Reranking — The Two-Stage Approach

```
Stage 1 (Retrieval):  Fast but rough. Get top 20-50 candidates with embeddings.
Stage 2 (Reranking):  Slow but precise. A cross-encoder re-scores those 20-50,
                      keeps the best 3-5.
```

**Why two stages?** Embeddings compare query and doc SEPARATELY (bi-encoder) — fast but loses nuance. A cross-encoder reads query AND doc TOGETHER — much more accurate but too slow to run on all documents. So: cast a wide net cheaply, then precisely rank the catch.

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain_cohere import CohereRerank

reranker = CohereRerank(top_n=3)
compression_retriever = ContextualCompressionRetriever(
    base_compressor=reranker,
    base_retriever=base_retriever,  # returns 20 candidates
)
# Returns the 3 best after reranking
```

### Parent Document Retriever

Embed SMALL chunks (precise matching) but return the LARGER parent document (full context).

**The problem it solves:** Small chunks match queries well but lack context. Big chunks have context but match poorly. Solution: search on small chunks, return their big parents.

### Self-Query Retriever

The LLM converts a natural-language query into a metadata filter + search.

```
User: "massage products under ¥10000 with good ratings"
         ↓ LLM parses
Filter: {category: "massage", price: {"<": 10000}, rating: {">": 4.0}}
Search: "massage products"
```

It automatically extracts structured filters from messy human questions.

### MultiQueryRetriever

Generates several rephrasings of the query, retrieves for each, merges results — improves recall (covered in Q21).

---

## 21. The Indexing API — Keeping Vector Stores in Sync

**Problem:** Your source documents change — some added, some updated, some deleted. Re-embedding EVERYTHING every time is wasteful and expensive.

**Solution:** LangChain's Indexing API tracks what's already indexed (via content hashes) and only processes CHANGES.

```python
from langchain.indexes import index, SQLRecordManager

record_manager = SQLRecordManager("my_namespace", db_url="sqlite:///record.db")
record_manager.create_schema()

index(
    docs,
    record_manager,
    vectorstore,
    cleanup="incremental",   # delete old versions of changed docs
    source_id_key="source",
)
```

**Cleanup modes:**
- `None` — just add everything (may create duplicates)
- `"incremental"` — update changed docs, leave others
- `"full"` — also delete docs no longer in the source

This is essential for production RAG where the knowledge base updates regularly (like your HRERA scraper re-running daily). It avoids re-embedding unchanged documents — saving time and API cost.

---

## 22. Chat History vs Memory vs State — Clearing the Confusion

These three terms get mixed up constantly. Here's the clean distinction:

- **Chat history** = the raw list of messages (Human, AI, System). Just the data.
- **Memory** = the mechanism that STORES that history and INJECTS it back into the next prompt so the LLM "remembers." (e.g., `RunnableWithMessageHistory`).
- **State** = a broader LangGraph concept — not just messages, but ANY data flowing through a workflow (counters, flags, tool results). LangChain has "memory"; LangGraph has "state."

```
Chat history:  [Human: "Hi I'm Ronit", AI: "Hello Ronit!"]   ← the data
Memory:        wraps the chain, re-injects that history       ← the mechanism
State:         {messages: [...], step: 3, retries: 1}         ← LangGraph, broader
```

**The core truth about LLM memory:** LLMs are stateless — every API call is independent and the model remembers NOTHING on its own. "Memory" is an illusion created by re-sending past messages in each new prompt. That's why long conversations cost more (more tokens per call) and eventually overflow the context window.

**Concrete example — see the illusion in action:**
```python
# WITHOUT memory — model forgets instantly
llm.invoke("My name is Ronit")          # AI: "Nice to meet you, Ronit!"
llm.invoke("What is my name?")          # AI: "I don't know your name." ← forgot!

# WHY? The second call sent ONLY "What is my name?" — the first message was gone.

# WITH memory — we manually re-send the history
messages = [
    HumanMessage("My name is Ronit"),
    AIMessage("Nice to meet you, Ronit!"),   # ← past turn re-attached
    HumanMessage("What is my name?"),
]
llm.invoke(messages)                     # AI: "Your name is Ronit." ← now it knows
```
The model didn't "remember" — we just pasted the old conversation back in. That's literally all memory is.

---

## 23. Structured Output — Three Ways, Ranked

Getting an LLM to return clean structured data (JSON/objects) instead of free text is a daily need. Three approaches, from least to most reliable:

**1. Prompt + StrOutputParser (weakest)** — just ask nicely.
```python
prompt = ChatPromptTemplate.from_template("Return JSON with name and price for: {q}")
# LLM MIGHT return valid JSON... or might add "Here's your JSON:" and break parsing
```
Fragile — relies on the model behaving.

**2. Output Parsers (better)** — adds format instructions + parses.
```python
from langchain_core.output_parsers import JsonOutputParser
parser = JsonOutputParser(pydantic_object=ProductInfo)
chain = prompt | llm | parser   # parser injects "respond in this format" automatically
```
Better, but still depends on the LLM following text instructions (can fail).

**3. `.with_structured_output()` (best, recommended)** — uses the model's native tool/JSON mode.
```python
from pydantic import BaseModel

class ProductInfo(BaseModel):
    name: str
    price: float
    available: bool

structured_llm = llm.with_structured_output(ProductInfo)
result = structured_llm.invoke("Tell me about Clever Cutter")
# result is a guaranteed-valid ProductInfo object, not text
```
This is enforced at the API level (function calling) — the output is **guaranteed** to match the schema. Use this whenever the model supports it.

**Rule:** For anything structured (extraction, classification, form-filling), prefer `.with_structured_output()`. Fall back to parsers only for models that don't support it.

---

## 24. RAG Failure Modes & How to Fix Them

RAG breaks in predictable ways. Knowing the failure mode tells you the fix:

| Symptom | Likely Cause | Fix |
|---|---|---|
| Right info exists but not retrieved | Bad chunking / embeddings | Adjust chunk size, add hybrid search |
| Exact terms (IDs, names) missed | Pure vector search blurs keywords | Add BM25 / hybrid search |
| Retrieves relevant + irrelevant mix | No reranking | Add a reranker (cross-encoder) |
| Chunk found but answer wrong | Prompt not grounding | "Answer ONLY from context" + low temp |
| Answer cut off / missing context | Chunks too small | Bigger chunks or Parent Document Retriever |
| Hallucinated facts | Model ignoring context | Force citations, lower temperature |
| Slow responses | Retrieving too many chunks | Reduce K, cache, smaller model |

**The #1 debugging rule:** Most RAG failures are **retrieval** failures, not LLM failures. Before blaming the model, print the retrieved chunks and check — did the right information even reach the LLM? If the chunk isn't there, no LLM can answer correctly.

```python
# Always inspect retrieval first when debugging
docs = retriever.invoke("user question")
for d in docs:
    print(d.page_content[:200], "\n---")
```

---

## 25. Production Concerns — Cost, Latency & Safety

Building a demo is easy; running RAG/agents in production needs these:

**Cost control**
- Cache repeated queries (`InMemoryCache`/`SQLiteCache`) — identical prompts return free, instant.
- Use a smaller/cheaper model for easy steps; reserve GPT-4-class for hard ones.
- Trim prompts: retrieve fewer, better chunks instead of dumping everything.
- Cap `max_tokens` on outputs.

**Latency**
- Stream responses (`.stream()`) so users see output immediately.
- Run independent calls in parallel (`RunnableParallel`, `.batch()`).
- Pre-build and load indexes (don't embed at query time).

**Reliability**
- `.with_retry()` for transient API failures (429s, timeouts).
- `.with_fallbacks([backup_model])` so one provider failing doesn't kill the app.

**Safety / Guardrails**
- Validate/sanitize user input (prevent prompt injection).
- Validate output (schema check, block unsafe content).
- For RAG, instruct the model to refuse when the answer isn't in the context.

These aren't optional extras — an LLM app without caching, fallbacks, and guardrails will be expensive, fragile, and unsafe in production.

---


---

# INTERVIEW QUESTIONS WITH ANSWERS (38 Questions)

---

**Q1. What is LangChain and why would you use it?**

LangChain is a Python framework for building LLM-powered applications. It solves the gap between a raw LLM API call and a production application by providing: data connectors (load PDFs, web pages), text splitters (chunking), vector stores (semantic search), memory (conversation history), tools (give LLM abilities), and agents (LLM decides what to do). I'd use it when I need RAG, conversational memory, or tool-using agents — things that require multiple connected components.

---

**Q2. Explain LCEL. What problem does it solve?**

LCEL (LangChain Expression Language) lets you compose components using the pipe `|` operator: `chain = prompt | llm | parser`. It solves the problem of manually connecting components — without LCEL you'd write nested function calls, handle streaming yourself, implement batching, add retry logic, etc. With LCEL, you get streaming, async, batching, retries, and fallbacks automatically on any chain. It also makes chains readable (left-to-right flow).

---

**Q3. What is a Runnable? Why does everything in LangChain implement it?**

A Runnable is the base interface — any component that has `.invoke()`, `.stream()`, `.batch()`, and `.ainvoke()`. Prompts, LLMs, parsers, retrievers, custom functions — all are Runnables. This uniform interface is what enables the `|` pipe chaining. You can swap any component for another (replace GPT-4 with Claude) without changing the rest of the chain. It's like USB — one standard connector for everything.

---

**Q4. Explain how RAG works in LangChain end-to-end.**

**Build time:** (1) Load documents (PDFLoader, JSONLoader). (2) Split into chunks (RecursiveCharacterTextSplitter, 1000 chars, 200 overlap). (3) Embed each chunk into a vector (OpenAIEmbeddings). (4) Store vectors in a vector database (Chroma, FAISS).

**Query time:** (5) User asks a question. (6) Embed the question. (7) Find similar chunks in vector DB (retriever). (8) Put retrieved chunks + question into a prompt template. (9) Send to LLM. (10) LLM generates answer based on the retrieved context.

The key insight: the LLM only "knows" what's in the retrieved chunks. No retrieval → no answer. Bad retrieval → bad answer.

---

**Q5. What is the difference between a Chain and an Agent?**

A Chain follows a fixed path — prompt → LLM → parser, always the same steps in the same order. Like a recipe. An Agent is dynamic — the LLM DECIDES at each step which tool to call, or whether to answer directly. Like a chef who looks at ingredients and decides what to cook. Chains are predictable (easy to test/debug), agents are flexible (handle varied queries) but slower and harder to control.

---

**Q6. Explain RecursiveCharacterTextSplitter in detail. Why "recursive"?**

It splits text by trying separators in order: `["\n\n", "\n", ". ", " ", ""]`. First tries to split on double-newlines (paragraph boundaries). If any resulting chunk is still too big, it splits THAT chunk on single-newlines (line breaks). Still too big? Try sentence boundaries. Then words. Then characters. "Recursive" = it keeps trying finer separators until everything fits within `chunk_size`. This preserves the most natural text boundaries possible.

---

**Q7. What is the difference between similarity search and MMR in a retriever?**

Similarity search returns the K chunks most similar to the query — but they might all say the same thing (redundant). MMR (Maximal Marginal Relevance) balances RELEVANCE and DIVERSITY — it selects chunks that are relevant to the query BUT also different from each other. Use MMR when you want broader coverage of the topic. Use similarity when you want the absolute most relevant chunks regardless of redundancy.

---

**Q8. How does LangChain memory work? What are its limitations?**

Memory stores conversation history and injects it into the prompt before each LLM call. The LLM sees previous messages as if they were part of the current prompt. Limitations: (1) Context window fills up — old messages push out space for new context. (2) Cost increases — more tokens per call. (3) Not true memory — it's just prepending text to the prompt. (4) No long-term storage by default (lost when process ends). Solutions: use WindowMemory (last K messages), SummaryMemory (compress old messages), or external databases.

---

**Q9. What is a Tool? How does the LLM know when to use it?**

A Tool is a Python function with a name, description, and input schema. The LLM reads the description to decide IF and WHEN to use it. Example: `"""Search products. Use when user asks about specific product details."""` The LLM sees this description, matches it against the user's intent, and decides to call it if needed. The description is critical — bad description → LLM uses tool at wrong times.

---

**Q10. Explain function calling / tool calling. How does it work technically?**

When you bind tools to an LLM (`llm.bind_tools([tool1, tool2])`), LangChain sends the tool schemas to the API. The LLM's response can now be either: (1) a regular text message, OR (2) a structured tool call request with function name + arguments. LangChain detects this, executes the function, sends the result back to the LLM as a ToolMessage, and the LLM then responds with the final answer incorporating the tool result.

---

**Q11. What is EnsembleRetriever? Why would you use it?**

EnsembleRetriever combines multiple retrieval strategies (e.g., BM25 keyword search + vector semantic search) and merges their results using weighted Reciprocal Rank Fusion. Use it because: keyword search catches exact matches (part numbers, names) that embeddings miss, while vector search catches meaning (synonyms, paraphrases) that keywords miss. Combining both gives better recall than either alone. In our project we use FAISS 0.8 + BM25 0.2 weights.

---

**Q12. What are the different types of chains for handling multiple documents?**

- **Stuff:** Puts ALL documents into one prompt. Simple but limited by context window.
- **Map-Reduce:** Processes each doc independently (map), then combines results (reduce). Scales to many documents but loses cross-document context.
- **Refine:** Processes docs one by one, refining the answer with each new doc. High quality but slow (sequential).
- **Map-Rerank:** Gets answer + confidence from each doc, picks highest confidence. Good when answer is in one specific document.

---

**Q13. How do you handle streaming in LangChain? Why does it matter?**

Use `.stream()` instead of `.invoke()`. It yields output tokens as they're generated. Matters because: users perceive faster response (they see the first token immediately vs waiting 3-5 seconds for full response). Implementation: `for chunk in chain.stream(input): print(chunk, end="")`. LCEL makes any chain streamable automatically — you don't have to implement streaming logic yourself.

---

**Q14. What is LangSmith? When would you use it?**

LangSmith is LangChain's tracing/debugging/evaluation platform. It records every step: what prompt was sent, what the LLM returned, latency, token count, tool calls. Use it to: (1) Debug why the LLM gave a wrong answer (see exact prompt). (2) Track costs (token usage over time). (3) Evaluate RAG quality (test datasets). (4) Compare different prompt versions. (5) Monitor production deployments. Think of it as "Chrome DevTools for LLM apps."

---

**Q15. How would you implement caching to reduce API costs?**

`set_llm_cache(InMemoryCache())` — same prompt → same response returned from cache instantly (no API call). Use SQLiteCache for persistence across restarts. Reduces costs during development (testing same prompts repeatedly) and production (common queries). Be careful: cache based on exact prompt match. If prompt has timestamps or session IDs, cache won't hit. Design prompts to be cache-friendly.

---

**Q16. Explain RunnablePassthrough and when you'd use it.**

`RunnablePassthrough()` passes input through unchanged. Used in parallel branches where one branch transforms data but another needs the original. Example in RAG: retriever transforms the question into chunks (transformed), but the prompt also needs the original question (unchanged). `{"context": retriever, "question": RunnablePassthrough()}` — context is retrieved, question passes through as-is.

---

**Q17. How do you evaluate a RAG system?**

Two levels: (1) **Retrieval quality** — are the right chunks found? Metrics: hit rate (correct chunk in top K), MRR, precision@K. Test by creating question-answer pairs where you know which chunk has the answer. (2) **Answer quality** — does the LLM answer correctly from chunks? Metrics: faithfulness (no hallucination), correctness (matches expected answer), relevancy (answers the actual question). Use LangSmith or RAGAS framework.

---

**Q18. What are the limitations of LangChain?**

(1) Overhead — adds latency for simple use cases (just calling an API). (2) Abstraction hides details — hard to optimize when you can't see what's happening. (3) Rapid version changes — documentation often outdated, imports break between versions. (4) Over-engineering for simple tasks — single LLM call doesn't need a framework. (5) Debugging — errors deep in chains are hard to trace without LangSmith. Best practice: use LangChain for complex pipelines, direct SDK for simple calls.

---

**Q19. How does `.with_structured_output()` work? When would you use it?**

It forces the LLM to return output matching a Pydantic model schema. Under the hood, it uses the model's function/tool calling to guarantee valid JSON. Use it when you need structured data (not free text) — extracting entities, filling forms, creating database records. Unlike output parsers that rely on the LLM following instructions (can fail), structured output uses the API's native JSON mode (guaranteed valid).

---

**Q20. How would you handle rate limiting in a production LangChain app?**

(1) `.with_retry(stop_after_attempt=3)` — automatic retries with backoff on 429 errors. (2) `.batch(inputs, max_concurrency=5)` — limit parallel calls. (3) Fallbacks to cheaper model when rate limited. (4) Caching to reduce total API calls. (5) Queue system (Celery/Redis) to smooth bursts. (6) Multiple API keys across different accounts for higher total limits.

---

**Q21. How does a multi-query retriever improve RAG quality?**

Standard RAG: one query → one set of results. Multi-query: generates 3-5 different phrasings of the same question, retrieves for each, deduplicates. Improves recall because different phrasings match different chunks. Example: "What's the return policy?" might also generate "How to return an item?", "Refund process?", "Can I send it back?" — each phrase might match different relevant chunks that the original missed.

---

**Q22. What is token counting and why does it matter in LangChain?**

Tokens ≈ words (roughly 1 token = 4 characters in English, varies for Japanese). Matters because: (1) API pricing is per-token. (2) Context window has a token limit (128K for GPT-4o). (3) Chunk size should be in tokens for accuracy. LangChain uses `tiktoken` library to count tokens. Critical for: deciding chunk sizes, checking if prompt fits, estimating costs, implementing token-based memory limits.

---

**Q23. Explain the difference between synchronous and asynchronous execution in LangChain.**

Synchronous (`.invoke()`, `.batch()`): your code WAITS until the operation finishes. Simple but blocks the thread. Asynchronous (`.ainvoke()`, `.abatch()`): your code continues doing other things while waiting for API response. Essential for: web servers handling multiple users simultaneously, running multiple chains in parallel, keeping UI responsive. LCEL gives you async for free — every chain has `.ainvoke()` automatically.

---

**Q24. How would you implement a custom retriever in LangChain?**

Extend `BaseRetriever` and implement `_get_relevant_documents()`:
```python
from langchain_core.retrievers import BaseRetriever
from langchain_core.documents import Document

class MyCustomRetriever(BaseRetriever):
    def _get_relevant_documents(self, query: str) -> list[Document]:
        # Your custom logic here (database, API, hybrid search, etc.)
        results = my_database.search(query)
        return [Document(page_content=r.text, metadata=r.meta) for r in results]
```
Use when built-in retrievers don't match your data source or search strategy.

---

**Q25. What is the `agent_scratchpad` in agent prompts?**

It's a placeholder where LangChain injects the intermediate steps — tool calls and their results — as the agent works. Without it, the agent can't see its own previous tool calls within the same turn. The scratchpad grows with each tool call: "I called search → got X. I called calculator → got Y. Now I can answer." It's automatically managed — you just include `("placeholder", "{agent_scratchpad}")` in your prompt template.

---

**Q26. How do you handle multimodal content (images + text) in LangChain?**

Use vision-capable models (GPT-4o, Claude) and pass images as part of the message content:
```python
messages = [HumanMessage(content=[
    {"type": "text", "text": "Describe this product"},
    {"type": "image_url", "image_url": {"url": "https://..."}}
])]
```
For RAG with images: use multimodal embeddings (CLIP) to embed both images and text into the same vector space, enabling cross-modal search.

---

**Q27. What is the difference between `invoke`, `stream`, and `batch`? When to use each?**

- `invoke(one_input)` → returns one complete output. Use for: single user request, testing.
- `stream(one_input)` → yields tokens incrementally. Use for: chat UIs (users see response appearing in real-time).
- `batch([many_inputs])` → processes all inputs in parallel, returns list of outputs. Use for: bulk processing (embed 1000 documents, answer 50 questions at once). Batch is much faster than calling invoke in a loop because of parallelism.

---

**Q28. How would you test a LangChain application?**

(1) **Unit tests:** Test each component independently — does the prompt template fill correctly? Does the parser handle edge cases? (2) **Integration tests:** Does the full chain produce correct output for known inputs? Use temperature=0 for deterministic responses. (3) **Evaluation:** Create a test dataset (question → expected answer). Run chain on all, compute metrics (exact match, LLM-as-judge, BLEU). (4) **Regression tests:** Store good outputs, flag when new code changes them.

---

**Q29. What is contextual compression in retrieval?**

After retrieving chunks, a contextual compressor summarizes/filters each chunk to only include parts relevant to the query. Example: you retrieve a 500-word chunk about a product, but only 2 sentences answer the question. The compressor extracts those 2 sentences. This reduces noise in the LLM's context → better answers, lower token costs. Implementation: `ContextualCompressionRetriever(base_compressor=LLMCompressor, base_retriever=retriever)`.

---

**Q30. How do you handle document metadata filtering in vector stores?**

Most vector stores support metadata filtering alongside similarity search:
```python
retriever = vectorstore.as_retriever(
    search_kwargs={
        "k": 5,
        "filter": {"category": "massage", "price_range": "under_10000"}
    }
)
```
First filters by metadata (only massage products under ¥10K), THEN does similarity search within that subset. Use for: multi-tenant apps (filter by user_id), category-specific search, date ranges, access control.

---

**Q31. What is an embedding and how does similarity search use it?**

An embedding is a vector (list of numbers) representing the MEANING of text — similar meanings produce similar vectors. Similarity search embeds the query, then finds the stored chunk vectors closest to it (usually by cosine similarity, which measures the angle between vectors). Close vectors = similar meaning. Critical rule: documents and queries MUST be embedded with the SAME model, or the vectors live in incompatible spaces and retrieval breaks.

---

**Q32. What's the difference between a bi-encoder and a cross-encoder? Why use both?**

A bi-encoder embeds query and document SEPARATELY into vectors, then compares them — fast (can pre-compute all doc vectors) but less precise. A cross-encoder reads query and document TOGETHER and outputs a relevance score — far more accurate but too slow to run on a whole corpus. Production RAG uses both: the bi-encoder retrieves ~20-50 candidates cheaply (stage 1), then the cross-encoder reranks them to pick the best 3-5 (stage 2). Best of both — speed plus precision.

---

**Q33. Explain few-shot prompting and when you'd use it over zero-shot.**

Zero-shot = just instructions. Few-shot = instructions PLUS examples of input→output. Few-shot teaches the model a pattern by demonstration, which dramatically improves consistency for classification, specific formatting, and structured extraction. Use few-shot when the task is ambiguous or when output format matters. Use dynamic example selection (semantic similarity) to pick the most relevant examples per query, keeping the prompt short.

---

**Q34. Why are the old Memory classes being deprecated in favor of RunnableWithMessageHistory?**

The old `ConversationBufferMemory` classes were built for the legacy Chain API and don't compose cleanly with LCEL. `RunnableWithMessageHistory` wraps any LCEL chain, manages history per `session_id` (so multiple users don't share memory), and plugs into persistent backends (Redis, Postgres) by swapping the history class. It's the LCEL-native approach. For full stateful agents with branching/loops, LangGraph checkpointing is even more powerful.

---

**Q35. What is the Indexing API and what problem does it solve?**

It keeps a vector store in sync with changing source documents WITHOUT re-embedding everything. A RecordManager tracks content hashes of indexed docs. On re-run, only NEW or CHANGED documents get embedded; unchanged ones are skipped. Cleanup modes: `incremental` updates changed docs, `full` also deletes docs removed from the source. Essential for production RAG with regularly-updated data (like a daily scraper) — saves significant time and embedding cost.

---

**Q36. Explain RunnableLambda and RunnableBranch.**

`RunnableLambda` wraps any Python function so it becomes a chain component you can pipe with `|` — use it for custom preprocessing/postprocessing between steps. `RunnableBranch` is if/else routing for chains: it checks conditions in order and routes input to the matching sub-chain (with a default fallback). Use RunnableBranch for simple conditional routing; for loops or complex state-driven routing, switch to LangGraph.

---

**Q37. What is the Parent Document Retriever and why is it useful?**

It embeds SMALL chunks (for precise query matching) but returns the LARGER parent document (for full context). This solves the chunk-size dilemma: small chunks match queries well but lack surrounding context, while big chunks have context but match poorly. By searching on small pieces and returning their big parents, you get accurate retrieval AND complete context for the LLM.

---

**Q38. What is a Self-Query Retriever?**

It uses an LLM to translate a natural-language question into a structured metadata filter PLUS a semantic search query. Example: "massage products under ¥10000 with good ratings" becomes filter `{category: massage, price: <10000, rating: >4.0}` + search "massage products." It lets users ask messy natural questions while the system applies precise structured filters automatically — combining the power of metadata filtering with semantic search.

---
