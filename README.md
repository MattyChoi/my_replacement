# Building my replacement

## Plan

### Model serving Layer

[Ollama](https://ollama.com/download?source=post_page-----1f3fb8f984bb------------------------------------------) — A popular entry point. One command downloads and runs models. Exposes an OpenAI-compatible API. Bundles llama.cpp under the hood and handles quantization automatically.

Strengths: Easy setup, broadest model library, hot-swap models without restart
Limitations: Lower throughput under concurrent load (41 TPS peak vs. vLLM's 793 TPS on Llama 3.1 8B, A100 GPU), not optimized for high-concurrency production
Best for: Teams under 20 users, development/testing, environments where model flexibility matters more than throughput

```
curl -fsSL https://ollama.com/install.sh | sh
```

[vLLM](https://vllm.ai/) — Built for production inference. Uses PagedAttention to virtually eliminate KV-cache memory waste (from 60-80% in traditional systems to under 4%) and increase throughput by 2-4x compared to prior serving systems. Benchmarks show 793 TPS peak vs. Ollama's 41 TPS on the same hardware (Llama 3.1 8B, A100 GPU).

Strengths: Highest throughput, lowest latency under load, continuous batching, broad GPU support (NVIDIA, AMD ROCm, Intel, TPU)
Limitations: Steeper setup, requires restart to switch models
Best for: Teams of 20+ users with concurrent access, production deployments where performance matters

```
uv pip install vllm --torch-backend auto
```

### Chat Interface Layer

A model API, alone, isn't user-friendly. This layer provides the web-based chat experience your team interacts with.

A good chat interface gives each user their own conversation history, lets them upload documents for ad-hoc context, and connects to multiple backend models.

A great one goes further: shared conversations so teams can collaborate on research, prompt templates and reusable agents that non-technical users can build and share, web search, deep research, and tool use built in, and admin controls for managing users and models from a single dashboard.


### Knowledge Retrieval Layer

The RAG pipeline: To ground AI responses in your data, you need:

- Data connectors to pull content from your tools (Slack, Confluence, Jira, Google Drive, etc.)
- A document processing pipeline to chunk, embed, and index that content
- A vector database to store embeddings (PGVector, Qdrant, Milvus, ChromaDB, etc.)
- A retrieval system to find relevant content when users ask questions
- A synthesis layer to combine retrieved context with LLM generation

### Enterprise Layer

- Authentication: SSO (OIDC/SAML), LDAP, OAuth, so users sign in with their existing corporate credentials. Even better, SCIM to inherit custom configs from your IdP
- Authorization (RBAC): Role-based access controls so admins can manage who can access what features and data
- Permission inheritance: The AI should respect source system permissions. If someone can't access a Confluence space, they shouldn't see its content in AI answers
- Analytics: Usage dashboards, query history, and admin visibility for governance and optimization
- Audit trail: Logging of AI interactions for compliance and oversight
- White-labeling: Custom branding for internal rollout

## Resources

- https://onyx.app/insights/self-hosted-llm-teams
- https://huggingface.co/models
- 