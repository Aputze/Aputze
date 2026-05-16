# Sergei Lerner - ERP & AI Integration Architect

One of the areas I've been focused on lately: building AI-native infrastructure on top of Priority ERP - an orchestration layer that understands ERP business logic, not just its API surface.

---

## What makes this different from "connecting an LLM to an ERP"

Most ERP+AI integrations treat the system as a flat database. Priority is not a flat database - it's a system with deeply embedded business logic. Updating a field is not a write operation; it's understanding which process owns that field, what preconditions apply, and what the ERP expects to happen before and after.

The infrastructure I built reflects that:

```
User intent (natural language)
        │
        ▼
  Knowledge Layer          ← What does Priority know about this domain?
  (ERP ontology, entity    ← What are the lifecycle rules?
   capabilities, routing   ← What can/can't be done via which path?
   intelligence, CLAUDE.md)
        │
        ▼
  Heuristic Layer          ← Pre-flight: which tool, which endpoint, which pattern?
  (OData vs WebSDK,        ← Is this a known failure path? (KNOWN_ISSUES.md)
   path selection,         ← Does a tested pattern exist? (TEST_CASES.md)
   governance rules)
        │
        ▼
  Operational Layer        ← Execution with business-logic awareness
  (MCP Gateway:            ← Not "write field X" but "run the process that updates X"
   OData for CRUD,
   WebSDK for procedures,
   form lifecycle, reports)
        │
        ▼
  Priority ERP (live)
```

The agent that executes doesn't write to Priority. It understands Priority's business logic and acts accordingly.

---

## Projects

### `aputze-priority-ai-lab`
The lab where this architecture was built and validated. Developer Agent, dual MCP gateway, iterative learning log, 12 documented failure patterns with fixes.
`Priority ERP` `MCP` `Claude` `Node.js` `Docker` `OData` `WebSDK`

### Prime Spend Portal
Procurement intelligence portal with AI-driven analysis and approval workflows, running against a live Priority instance.
`FastAPI` `Claude` `OData` `WebSDK` `Docker`

### `Priority-RAG`
Hebrew-first semantic search over Priority community knowledge. Built to feed the Knowledge Layer above.
`Python` `FAISS` `FastAPI` `LangChain`

### MedOp
Voice-first medical documentation. Whisper transcription in Hebrew, English, and Russian - structured clinical reports out. FastAPI backend, React frontend, live on HuggingFace Spaces.
`Python` `Whisper` `FastAPI` `React` `TypeScript` `Docker`

### `mcp-gateway`
Unified MCP routing layer - OData and WebSDK under one interface with routing intelligence built in.
`Node.js` `MCP` `REST`

### `aputze-smart-home`
Home Assistant + Zigbee2MQTT + Docker + MCP servers, running on a local NUC. Six-plus years in production.
`Home Assistant` `Zigbee2MQTT` `MQTT` `Docker` `Python`

---

## Stack

```
Orchestration    Claude · LangGraph · LangChain · Groq · DeepSeek
Transcription    Whisper (custom: aputze/Whispr) - HE / EN / RU
Vector search    FAISS
Backend          FastAPI · Node.js · Python
Frontend         React · TypeScript · Vite · Tailwind · Gradio
ERP              Priority ERP · OData · WebSDK · DBI · MCP
Infrastructure   Docker · Home Assistant · Zigbee2MQTT · MQTT
                 NUC · Tailscale · Caddy · HuggingFace Spaces
```

---

## Contact

sergei.lerner@yahoo.com  
[linkedin.com/in/sergei-lerner](https://linkedin.com/in/sergei-lerner)  
[aputze.github.io](https://aputze.github.io)  
Ramat Gan, Israel

Open to consulting, collaboration, and the right full-time role.
