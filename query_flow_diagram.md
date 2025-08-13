# Query Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              FRONTEND                                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     │ POST /api/query
                                     │ {query, session_id?}
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         FASTAPI (app.py)                                     │
│  @app.post("/api/query")                                                     │
│  async def query_documents(request: QueryRequest)                            │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     │ rag_system.query()
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       RAG SYSTEM (rag_system.py)                            │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐                │
│  │ Session Manager │ │   AI Generator  │ │  Tool Manager   │                │
│  │ get_history()   │ │ generate_resp() │ │ get_tools()     │                │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘                │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     │ tools + history + query
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ANTHROPIC CLAUDE (ai_generator.py)                        │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ AI Decision: "Do I need to search course content?"                   │    │
│  │                                                                     │    │
│  │ IF YES → Call CourseSearchTool                                      │    │
│  │ IF NO  → Generate response from context                             │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     │ (if search needed)
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                   COURSE SEARCH TOOL (search_tools.py)                      │
│  CourseSearchTool.search(ai_generated_search_terms)                         │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     │ vector search
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    VECTOR STORE (vector_store.py)                           │
│                                                                              │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐          │
│  │  Search Query   │    │   ChromaDB      │    │ Course Chunks   │          │
│  │ → Embeddings    │ →  │ Similarity      │ →  │ + Metadata      │          │
│  │                 │    │ Search          │    │ + Sources       │          │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘          │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     │ relevant chunks + sources
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BACK TO CLAUDE AI                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Synthesize Response:                                                │    │
│  │ • Original query + conversation history                             │    │
│  │ • Retrieved course content                                          │    │
│  │ • Generate natural language answer                                  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     │ response + sources
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       BACK TO RAG SYSTEM                                    │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐                │
│  │ Session Manager │ │ Tool Manager    │ │ Response Ready  │                │
│  │ save_exchange() │ │ get_sources()   │ │ answer + sources│                │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘                │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     │ (answer, sources)
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         BACK TO FASTAPI                                     │
│  QueryResponse(answer=answer, sources=sources, session_id=session_id)       │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     │ JSON response
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              FRONTEND                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Update UI:                                                          │    │
│  │ • Display answer in chat                                            │    │
│  │ • Show source references                                            │    │
│  │ • Maintain session_id for next query                                │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘

FLOW SUMMARY:
1. User submits query → FastAPI endpoint
2. RAG System coordinates components
3. AI Generator decides if search needed
4. CourseSearchTool queries vector database if needed
5. ChromaDB returns relevant course content
6. AI synthesizes response with context
7. Session updated, sources collected
8. Response sent back to frontend
9. UI updated with answer and sources
```

## Key Decision Points

1. **AI Tool Usage**: Claude dynamically decides when to search course content
2. **Session Management**: Conversation history influences responses
3. **Vector Search**: Semantic similarity, not keyword matching
4. **Multi-turn Support**: Session state maintained across queries

## Data Persistence

- **Conversation History**: Stored in SessionManager
- **Course Content**: Stored in ChromaDB vector database
- **Sources**: Tracked through tool executions for citations