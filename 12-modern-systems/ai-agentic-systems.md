# AI agents and RAG

## Overview
Many systems now call Large Language Models (LLMs). Two common patterns: RAG, which gives the model relevant private or fresh documents with each question, and agents, which let the model call tools in a loop to finish a task. The engineering work is in retrieval quality, evaluation, latency, cost, and safety.

## Key concepts
*   **Token**: A piece of a word. Context limits, latency, and price are all counted in tokens.
*   **Context window**: The most tokens (input plus output) a model can handle in one call. Limits differ by model and change often.
*   **Prompt template**: A reusable prompt with placeholders, such as `Answer using only these documents: {docs}. Question: {question}`.
*   **Zero-shot vs. few-shot**: Zero-shot gives only instructions. Few-shot adds a few input/output examples to show the format.
*   **Temperature**: Controls randomness. Lower values give more predictable output.
*   **Embedding**: A list of numbers that represents the meaning of a text. Similar texts get nearby vectors.
*   **RAG (Retrieval-Augmented Generation)**: Find relevant chunks of your documents, put them in the prompt, and ask the model to answer from them. It reduces made-up answers (hallucinations) but does not remove them.
*   **Vector database**: Stores embeddings and finds the nearest ones fast with approximate nearest neighbor (ANN) indexes such as HNSW. Examples: Pinecone, Weaviate, Milvus, Qdrant, Chroma, and pgvector in Postgres. FAISS is a library, not a database.
*   **Hybrid search**: Keyword search (BM25) plus vector search, merged and often reranked. It finds both exact terms and similar meanings.
*   **Tool calling**: The model returns a structured request such as `get_order(id=42)`. Your code runs it and sends the result back. The Model Context Protocol (MCP) is a common standard for exposing tools to models.
*   **Agent**: An LLM in a loop: decide the next step, call a tool, read the result, repeat until done. Frameworks: LangGraph, LlamaIndex, CrewAI, the OpenAI Agents SDK, the Claude Agent SDK, and Microsoft Agent Framework (the successor to AutoGen, which is now in maintenance mode).
*   **Memory**: Short-term memory is the conversation in the context window. Long-term memory is facts or summaries stored in a database and retrieved later.
*   **Streaming**: Sending tokens to the user as they are generated, so the first words appear quickly.

## Real-world usage
*   **Support assistant**: RAG over policy documents, with citations so users and reviewers can check the source.
*   **Data analyst agent**: Writes SQL, runs it on a read-only replica, and plots the result.
*   **Coding agent**: Searches the repository with tools, edits files, runs tests, and opens a pull request for review.
*   **Recommendations**: Embeddings of products and user behavior find similar items.

## Tradeoffs
*   **RAG vs. fine-tuning**: RAG suits facts that change or are private, and supports citations. Fine-tuning suits style, format, or narrow tasks. Fine-tuning is a poor way to add facts.
*   **Agents vs. fixed workflows**: Agents handle open-ended tasks but are less predictable and use more tokens. If the steps are known, a fixed workflow with a few LLM calls is easier to test.
*   **Model changes**: A new model version or a prompt edit can change behavior. Pin model versions and rerun evaluations before switching.
*   **Latency**: Responses take seconds. Use streaming, smaller models for simple steps, caching, and parallel tool calls.

## When not to use
*   **Predictable CRUD**: If a SQL query and a form solve it, an LLM adds cost, latency, and errors.
*   **Exact math and strict rules**: Let the model call a calculator, code, or a rules engine instead of doing it itself.

## Interview focus
*   **Grounding**: "How do you make answers come only from the provided context?"
    *   Retrieve well first: hybrid search, a reranker, and good chunking. Most wrong answers start with bad retrieval.
    *   Instruct the model to answer only from the context, to cite chunk IDs, and to say "I don't know" when the context lacks the answer.
    *   Abstain when retrieval scores are low instead of letting the model guess.
    *   Check after generation: confirm cited chunks exist and support the claims, using an entailment (NLI) model or an LLM judge.
    *   Treat retrieved text as untrusted. It can contain prompt injection.
    *   Note: safety classifiers like Llama Guard detect harmful content. They do not detect hallucinations.
*   **Search types**: "Keyword search vs. semantic search?"
    *   Keyword search matches terms through an inverted index (Postgres full-text, Elasticsearch BM25). It is exact and good for IDs, error codes, and names. It misses synonyms. (`LIKE '%term%'` is not real search; it cannot use a normal index.)
    *   Semantic search compares embeddings, so "laptop" can find "notebook computer". It is approximate, can miss exact identifiers, and needs embeddings computed for every document.
    *   Production systems usually combine both.
*   **Cost**: "How do you cut LLM cost for repeated questions?"
    *   Exact cache: same normalized prompt, same answer.
    *   Semantic cache: embed the question and reuse a stored answer when a past question is close enough. Tune the threshold on real examples. "Cancel my order" and "don't cancel my order" can look similar.
    *   Scope the cache by user permissions and give entries a time limit.
    *   Prompt caching by the provider makes a long, repeated prompt prefix cheaper and faster.
    *   Send fewer tokens: retrieve fewer, better chunks, and use smaller models for easy steps.
*   **Production integration**: "How do you put an LLM into a production app?"
    *   Call it from your backend, never the browser, so API keys stay secret.
    *   Set timeouts, retries with backoff, and a fallback model or message.
    *   Validate structured output against a schema.
    *   Log prompts, responses, token counts, latency, and errors. Track quality with an evaluation set.
*   **Latency**: "How do you make a real-time LLM feature feel fast?"
    *   Stream tokens to the user.
    *   For voice, start the LLM call on partial speech-to-text results instead of waiting for the full sentence.
    *   Cache retrieval results and keep the prompt short.

## Common mistakes
*   **Too much context**: Sending whole documents with every call. It costs more and can make answers worse.
*   **Bad chunking**: Chunks too small lose meaning; chunks too large bury the answer. Split on document structure and keep titles with chunks.
*   **No evaluation**: Judging by "it feels good" after five questions. Build a test set and measure retrieval (recall@k) and answers (faithfulness, correctness), with tools such as RAGAS.
*   **Ignoring permissions**: Retrieving documents the user is not allowed to see. Filter by access rights at retrieval time.
