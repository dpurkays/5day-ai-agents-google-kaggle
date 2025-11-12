# Day 2 - Agent Tools & Interoperability with Model Context Protocol (MCP)

## Codelabs

These notebooks were originally completed on **Kaggle** as part of the 5-Day AI Agents Intensive Course, then exported here for documentation and version control.

1. Explore new ways to add tools to extend what your agents can do
   📓 [Open Notebook → day-2a-agent-tools.ipynb](./day-2a-agent-tools.ipynb)
2. Explore best practices for tools, including using MCP and long-running operations.
   📓 [Open Notebook → day-2b-agent-tools-best-practices.ipynb](./day-2b-agent-tools-best-practices.ipynb)

## Notes

**Tool**: function or a program an LLM-based app can use to accomplish a task outside the model's capabilities.

- Tool Definition uses JSON schema -> a contract
- returns JSON or Unstructured content like raw text

### Types of tools

| Type of Tools      | Details                                                                                                                                                                  |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Function tools** | - allow the dev to define external functions that the model can call as needed <br>- often use detailed docstring to define contract, inputs and outputs of the function |
| **Built-in Tools** | By the model tool dev, blackbox for dev                                                                                                                                  |
| **Agent Tools**    | - invoking another agent as a tool <br>- doesn't hand out, more like a delegating                                                                                        |

#### Taxonomy of Agent Tools

- **Information Retrieval:** Allow agents to fetch data from various sources, such as web searches, databases, or unstructured documents.
- **Action / Execution:** Allow agents to perform real-world operations: sending emails,posting messages, initiating code execution, or controlling physical devices.
- **System / API Integration:** Allow agents to connect with existing software systems and APIs, integrate into enterprise workflows, or interact with third-party services.
- **Human-in-the-Loop:** Facilitate collaboration with human users: ask for clarification, seek approval for critical actions, or hand off tasks for human judgment.
  - Called **Long-Running Operation** (LRO): tool needs to pause, wait for external input, then resume.
    - When to use:
      - Financial transactions requiring approval (transfers, purchases)
      - Bulk operations (delete 1000 records - confirm first!)
      - Compliance checkpoints (regulatory approval needed)
      - High-cost actions (spin up 50 servers - are you sure?)
      - Irreversible operations (permanently delete account)

### Best Practices

1. Documentation
   - use clear name
   - describe all input and output parameters
2. Describe actions, not implementations
   - describe what not how
3. Publish tasks, not API calls
4. Design for concise output
   - dont return the raw data, like a uri or summerized data
5. Provide descriptive error message
   - tells LLM what went wrong

### Model Context Protocol (MCP)

Aiming to solve the "N X M" integration problem and foster a reusable ecosystem by replacing the fragmented landscape with unified, plug-and-play protocol and decoupling the AI agent.

1. MCP host (traffic cop)
   - creating and manageing clients
   - managing overall user experience
   - orchestrating the use of tools
   - enforcing security policies & guardrails
2. MCP Client (communicator)
   - embedded in the host
   - maintain connection to server
   - manage lifecycle
3. MCP Server
   - program providing the tools
   - listen to commands from client
   - execute the commands
   - sends it back to client

- Seperation is key because it allows dev building the agent logic in host to focus on logic, while another team can focus on the specialized tools in the server
  -> promotes modularity

- Uses JSON-RPC 2.0
  - Locally -> stdio
  - otherwise (more commonly) -> Streamable HTTP

#### Advantages of MCP

- accelerating development & fostering a reusable ecosystem
  - simplified integration
  - plug-and-play ecosystem
- dynamically enhancing agent capabilities & autonomy
  - dynamic tool discovery
  - standardized tool descriptions
  - expanded llm capabilities
- architectural flexibility & future proofing
  - modular design
- foundations for governance and control
  - single enforcement point
  - human-in-the-loop: explicit user approcal before invoking any tools or sharing private data

#### Security Risk

**Confused Deputy**: when ai model is tricked by another entity w/fewer privileges into misusing its authority and performing an action for the attacker

- MUST wrap the raw MCP protocol in layers of external centerlized governance
- have a gateway to check user, rate limiting, etc.
