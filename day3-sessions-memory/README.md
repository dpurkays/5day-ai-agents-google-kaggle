# Day 3 - Context Engineering: Sessions & Memory

## Codelabs

These notebooks were originally completed on **Kaggle** as part of the 5-Day AI Agents Intensive Course, then exported here for documentation and version control.

1. Build stateful agents and perform context engineering.
   📓 [Open Notebook → day-3a-agent-sessions.ipynb](./day-3a-agent-sessions.ipynb)
2. Explore how to use memory with your agent
   📓 [Open Notebook → day-3b-agent-memory.ipynb](./day-3b-agent-memory.ipynb)

## Notes

### Context Engineering

LLMs are inherently stateless.

_Context Engineering is **the dynamic assembly and management of information (history, data, instructions) within an LLM's context window**._

#### Context Flow

1. Fetch Context : retrieve necessary info
2. Prepare Context : dynamically contructs the full prompt, this is blocking ("hot-path" process). Agent cannot proceed until the context is ready.
3. Invoke LLM and Tools : calls LLM and tools interatively until response is generated
4. Upload Context : new info gathered during the turn is uploaded in persistent storage

### Sessions

- foundational element of Context Engineering
- includes immediate dialogue history and working memory for a single, continuous convo.
- Every session contains 2 key components:
  - **events:** chronological history
    - building blocks of the convo
    - user inputs
    - agent response
    - tool call
    - tool output
  - **state:** agent's working memory
    - temporary structured data relevant to the current convo

#### Key Considerations

1. **Security & privacy**
   - non negotiable
   - **Strict Isolation**: most critial security principle
     - must enforce strict isolation to ensure one user can never access another user's session data
2. **Data integrity**
   - **Time-to-Live(TTL)** policy: automatically delete inactive sessions to manage storage costs and reducing data management overhead
   - session history are appended in a _deterministic order_ to maintain the correct chronological sequence of events
3. **Performance**
   - reading and writing must be extremely fast for responsive UX
   - agent runtimes are typically stateless so the entire session history is retrieved from a central fatabase at the start of every turn = _network transfer latency_
     - to mitigate this, its crucial to reduce the size of data transferred.
     - the key is to filter or compact the session history before sending it to the agent

#### Managing long context conversations

- The primary challenge for sessions is managing an ever-growing conversation history, which increases cost and latency and can cause "context rot".

**Cons:** context window limits, increase API costs, decrease latency/speed, reduced quality.

**Compaction strategies** shrink long conversation histories to reduce API costs and latency while preserving the most important context. Includes:

- Keep the last N turns (simplest "sliding window" approach).
- Token-Based Truncation (includes messages up to a predefined token limit).
- Recursive Summarization (replaces older parts of the conversation with an AI-generated summary) - done async

### Memory

Longterm persistence

#### Key Capabilities:

- **personalization:** remembering user preferences, facts and past interactions to tailor future responses
- **context window management:** compacting long convo history by creating summaries or extracting facts = preserving context while reducing cost & latency
- **data mining and insights:** extracting insights from stored memories across aggregated user data
- **agent self-improving and adaptation:** creating procedual memories about its own performance; recording which strategies or reasoning paths led to successful outcomes.

| Memory                                                                                                          | Sessions                                                                          |
| --------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| - longterm memory <br> - stores critical, condensed info for recall in future projects <br> - "filling cabinet" | - manages temporary <br> - turn by turn state of single convo, <br> - "workbench" |

#### Retrieval-Augmented Generation - RAG Vs Memory Managers

| RAG                                                                     | Memory Managers                                              |
| ----------------------------------------------------------------------- | ------------------------------------------------------------ |
| - global facts/ expert knowledge of the world <br> - research librarian | - expert understanding of the user <br> - personal assistant |

#### Architecture of Memory

| Types of Information                   | Description                                |
| -------------------------------------- | ------------------------------------------ |
| Declarative Memory <br> "knowing what" | knowledge of facts, figures, and events    |
| Procedual Memory <br> "knowing how"    | agent's knowledge of skills and workflows. |

**Organization and Storage**

Memory managers organize information using distinct patterns:

- **Structured User Profile:** A continuously updated contact card holding core facts about the user (e.g., name, preferences) for quick lookups.
- **Collections:** Multiple self-contained, natural language memories for a single user, allowing for storage and searching through a less structured pool of information.
- **Rolling Summary:** Consolidates all information into a single, continuously evolving memory that summarizes the entire user-agent relationship.

The common storage architectures are:

- **Vector Databases** : _most common_, excels at semantic similarity search for unstructured content
- **Knowledge Graphs**: stores entities and relationships, ideal for structured, relational queries.

A hybrid approach can combine both for relational and semantic searches

**Memory Scope**

- **User-Level Scope:** The most common; tied to a specific user ID and persists across all sessions to build a long-term, personalized understanding.
- **Session-Level Scope:** Persistent record of insights extracted from a single session, used primarily for compacting long conversations.
- **Application-Level Scope (or Global context):** Accessible by all users, often used for procedural memories or shared context

**Memory Generation Pipeline**

Memory generation is an **active, LLM-driven ETL (Extract, Transform, Load) pipeline** that transforms raw conversational data into structured, meaningful insights

- **Extraction (E) :** Identify and capture only the meaningful infor from raw convo, seperating the noise
- **Consolidation (T) :** To merge new info with exsiting memories, resolving conflicts and deduplicating data.
- **Providence (T) :** tracks a memory's origin and history to establish trustworthiness
- **Retreival (L)** : retrieve a plan that guides the agent on how to exevute a complex task.
  - **proactive:** fetch potential memory prior to the task
  - **reactive:** dialogue injection or memory-as-a-tool -> only relevant to the immediate context of the conversation
