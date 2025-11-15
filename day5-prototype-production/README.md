# Day 5 - Prototype to Production

## Codelabs

These notebooks were originally completed on **Kaggle** as part of the 5-Day AI Agents Intensive Course, then exported here for documentation and version control.

1. Explore how to use A2A Protocol to have agents interact with each other.
📓 [Open Notebook → day-5a-agent2agent-communication.ipynb](./day-5a-agent2agent-communication.ipynb)
<!-- 2. Deploy your agent to Agent Engine on Google Cloud. (Optional)
   📓 [Open Notebook → day-5b-agent-deployment.ipynb](./day-5b-agent-deployment.ipynb) -->

## Notes - Prototype to Production

- 80% of effort is spent on infrastructure, securitt and validation needed to make it reliable and safe
- Problems if you skip these:
  - A customer service agent is tricked into giving products away for free because you forgot to set up the right guardrails.
  - A user discovers they can access a confidential internal database through your agent because authentication was improperly configured.
  - An agent generates a large consumption bill over the weekend, but no one knows why because you didn't set up any monitoring.
  - A critical agent that worked perfectly yesterday suddenly stops, but your team is scrambling because there was no continuous evaluation in place.

### People & Process

- **prompt engineers** : these individuals blend technical skill in crafting prompts with deep domain expertise.
- **AI engineers** : integrating the guardrails, and RAG/tool integration

### Preproduction - Evaluation-Gated Deployment

1. **Rigorous Evaluation Process**

   - as a quality gate
   - 1. **Manual "Pre-PR" Evaluation** : The AI or Prompt Engineer runs the evaluation suite locally, and the performance report (comparing against a production baseline) is a mandatory artifact for human review in the Pull Request (PR)
   - 2. **Automated In-Pipeline Gate** (For Mature Teams): The evaluation harness is integrated directly into the CI/CD pipeline, and a failing evaluation programmatically blocks the deployment if key metrics (e.g., "tool call success rate") fall below a predefined threshold

2. **Automated CI/CD pipeline** : automated script to manage the complexity of the agent by testing changes in stages and catching errors early ("shifting left")
   - **Phase 1: Pre-Merge Integration (CI)**: Acts as a gatekeeper for the main branch, running fast checks (unit tests, code linting) and, crucially, the agent quality evaluation suite to provide rapid feedback before merging
   - **Phase 2: Post-Merge Validation in Staging (CD):** Focuses on operational readiness in a high-fidelity replica of production. This includes resource-intensive checks like load testing, integration tests, and internal user testing ("dogfooding")
   - **Phase 3: Gated Deployment to Production:** Typically requires a final sign-off (e.g., from a Product Owner) before promoting the tested artifact using appropriate safeguards
3. **Safe Rollout Strategies** : To minimize the risk of unforeseen issues in the real world deployment should use gradual rollout strategies
   - **Canary:** Start with 1% of users, monitoring for prompt injections and unexpected tool usage. Scale up gradually or roll back instantly.
   - **Blue-Green**: Run two identical production environments. Route traffic to "blue" while deploying to "green," then switch instantly. If issues emerge, switch back—zero downtime, instant recovery.
   - **A/B Testing**: Compare agent versions on real business metrics for data-driven decisions. This can happen either with internal or external users.
   - **Feature Flags**: Deploy code but control release dynamically, testing new capabilities with select users first.
