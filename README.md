# Sergei Lerner - ERP & AI Integration Architect

One of the areas I've been focused on lately: building AI-native infrastructure on top of Priority ERP. An orchestration layer that understands ERP business logic, not just its API surface.

---

## What makes this different from "connecting an LLM to an ERP"

Most ERP+AI integrations treat the system as a flat database. Priority is not a flat database. It's a system with deeply embedded business logic. Updating a field is not a write operation. It's understanding which process owns that field, what preconditions apply, and what the ERP expects to happen before and after.

The infrastructure I built reflects that. It's a **4-layer architecture**:

```
User intent (natural language)
        │
        ▼
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LAYER 2: KNOWLEDGE (HYBRID)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STATIC (from Official Docs):
  • Common entities (CUSTOMERS, ORDERS, PART, USERS, AINVOICE)
  • Routing rules (OData vs WebSDK)
  • Business flows (Order -> Delivery -> Invoice -> GL)
  • Known failure patterns and resolution strategies
  Files: CLAUDE.md
  Load time: Startup (zero API calls)

DYNAMIC (from $metadata):
  • Unknown entities (custom tables)
  • Field schemas & types
  • Entity key fields & availability
  • Cache: Per-session
        │
        ▼
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LAYER 3: HEURISTIC
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  • Pre-flight validation
  • Error classification
  • Pattern matching
  • Tool routing decision
  • Composite key resolution
        │
        ▼
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
LAYER 4: OPERATIONAL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  • MCP Gateway
  • Subform operations
  • URL encoding
  • Business-logic-aware execution
  • Not "write field X" but "run the process that updates X"
  • Human approval, traceability, payload logging
        │
        ▼
Priority ERP (live)
  ✓ Constrained execution path
  ✓ Business rules respected
  ✓ No bypasses
```

The agent that executes doesn't write to Priority. It understands Priority's business logic and acts accordingly.

---

## Layer 2: HYBRID Knowledge Layer

The knowledge layer combines static domain rules with runtime discovery for optimal performance and flexibility.

### Static Component (CLAUDE.md from Official Docs)

Hardcoded from Priority's official SDK documentation:

| What | Source | Where |
|------|--------|-------|
| Common entity list | Priority SDK / REST API docs | CLAUDE.md |
| OData query rules | Official REST API docs | CLAUDE.md |
| Routing rules | SDK capability docs | CLAUDE.md |
| Screen Generator APIs (EFORM, FCLMN_SUBFORM) | Screen Generator + REST API docs | CLAUDE.md |
| DBI conventions | SDK + DBI Syntax docs | CLAUDE.md |
| Business flows (Sales: Order -> Delivery -> Invoice -> GL) | User Guide PDF | CLAUDE.md |
| Known issues and resolution strategies | Documented patterns | CLAUDE.md, KNOWN_ISSUES.md |

**Loading:** At agent startup (project-level rules). **Zero API calls needed.**

**Why static?** Speed (no discovery overhead), reliability (hardcoded safety rules), offline capability.

### Dynamic Component (Discovered at Runtime from $metadata)

For any entity not in static knowledge:

| What's Discovered | How | Tool | Cached |
|---|---|---|---|
| Unknown entities | Via schema discovery | OData MCP | Per session |
| Field names and types | From $metadata | schema tooling | Per session |
| Entity key fields | Live metadata | resolver | Per session |
| Subform structure | Via expand operations | query tools | Per session |
| Entity availability | Live checks | validation | Per session |

**Example:**
```javascript
// Discover unknown entity at runtime
const schema = await discoverEntity("CUSTOM_TABLE");
// Returns: { fieldName: 'string', fieldType: 'int', ... }
// Cached for the session - no re-fetch
```

**Why dynamic?** Accuracy (always in sync with live Priority), flexibility (handles unknown custom entities).

### The Hybrid Philosophy

From CLAUDE.md §1:
> "Schema: discover fields from $metadata per entity; do not assume universal field names (CUSTNAME, PARTNAME, … differ)."

**The rule:** Static when possible (speed), dynamic when needed (accuracy).

**Result:** You get the benefits of both:
- ✅ Fast responses for common entities (no discovery call overhead)
- ✅ Flexible handling of unknown custom tables (SERG_*)
- ✅ Always in sync with Priority's live metadata
- ✅ Per-session caching (no repeated fetches)

---

## Code Implementation

| Layer | File | Key Components |
|-------|------|-----------------|
| **2 (Static)** | CLAUDE.md | Priority domain rules |
| **2 (Static)** | knowledge.js | Routing and relationships |
| **2 (Dynamic)** | priority-client.js | Runtime discovery and caching |
| **3** | reasoning.js | Validation and routing |
| **4** | copilot.js | Execution orchestration |

---

## Projects

### `aputze-priority-ai-lab`
Core implementation of the 4-layer architecture for Priority ERP integration.
- Developer Agent with dual integration endpoints
- Layer 2 HYBRID knowledge layer
- Layer 3 heuristic validation
- Layer 4 operational execution
- Documented patterns and resolution strategies

`Priority ERP` `MCP` `Claude` `Node.js` `OData` `WebSDK`

### Prime Spend Portal
Procurement intelligence system demonstrating the 4-layer architecture in production. Integrates multiple ERP endpoints with Layer 2 hybrid knowledge enabling intelligent routing.

`FastAPI` `Claude` `OData` `WebSDK`

### `Priority-RAG`
Hebrew-first semantic search over Priority community knowledge. Feeds the static Knowledge Layer (Layer 2).
`Python` `FAISS` `FastAPI` `LangChain`

### MedOp
Voice-first medical documentation. Whisper transcription in Hebrew, English, and Russian. Structured clinical reports. FastAPI backend, React frontend, live on HuggingFace Spaces.
`Python` `Whisper` `FastAPI` `React` `TypeScript` `Docker`

### `mcp-gateway`
Unified routing layer with intelligent path selection for multiple ERP endpoints.

`Node.js` `MCP` `REST`

### `aputze-smart-home`
Home Assistant + Zigbee2MQTT + Docker + MCP servers, running on a local NUC. Six-plus years in production.
`Home Assistant` `Zigbee2MQTT` `MQTT` `Docker` `Python`

---

## Architecture: 4 Layers

| Layer | What | File(s) | Purpose |
|-------|------|---------|---------|
| **1** | User Intent | Agent prompt | Natural language input |
| **2** | KNOWLEDGE (HYBRID) | CLAUDE.md, knowledge.js, priority-client.js | Static rules plus runtime discovery |
| **3** | HEURISTIC | reasoning.js, priority-client.js | Validation and routing |
| **4** | OPERATIONAL | copilot.js, priority-client.js | Execution |

---

## Stack

```
Orchestration    Claude, LangGraph, LangChain
Backend          FastAPI, Node.js, Python
Frontend         React, TypeScript, Vite, Tailwind
ERP              Priority ERP, OData, WebSDK, DBI
Knowledge        HYBRID (Static rules + Runtime discovery)
Infrastructure   Docker, Home Assistant, Zigbee2MQTT
                 NUC, Tailscale, Caddy, HuggingFace
```

---

## Key Insights

### Transactional vs Procedural
Most integrations ask: "What field do I write?"

The right question is: "What process needs to run?"

Approving an order isn't updating a status field. It's running a procedure that posts GL entries, updates inventory, triggers workflows, validates business rules, and maintains an audit trail.

### Why Layer 2 is HYBRID
Most agents either hardcode everything or discover everything at runtime. That's a false choice.

Our approach: hardcode what's stable (Priority's official domain rules) for speed. Discover what's variable (custom tables, live metadata) at runtime for flexibility. Best of both.

---

## Contact

**Sergei Lerner**  
sergei.lerner@yahoo.com  
[linkedin.com/in/sergei-lerner](https://linkedin.com/in/sergei-lerner) | Ramat Gan, Israel

---

*Built on 14 years of Priority ERP expertise across regulated environments: defense manufacturing, financial services, and enterprise operations.*
