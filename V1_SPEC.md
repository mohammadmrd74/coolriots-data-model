# V1 — The Working Digital Employee

## Implementation Specification

> Source of truth: BeX_Versioning_Roadmap.docx.pdf + DATA_MODEL.md + SYSTEM_DESIGN.md
> This document defines exactly what to build for V1, field by field, entity by entity.

---

## V1 Vision

**One question**: Can a non-technical enterprise user describe the employee they want in plain language, and have a working digital employee live within the same session?

V1 does one thing excellently: a digital employee that knows its domain through a knowledge base, can answer questions intelligently, can call specific external systems, and delivers responses through at least two channels. An enterprise can deploy this for customer support, internal helpdesk, HR FAQ, or compliance Q&A on day one.

---

## Two Parallel Tracks

Every feature must ship in both tracks simultaneously.

| Track | What It Means |
|-------|--------------|
| **Platform Track** | What capabilities the platform can execute |
| **Builder Track** | What the AI Builder Agent can configure from natural language |

**Critical Principle**: A platform capability that the Builder Agent cannot configure does not count as shipped.

---

## V1 Entity Scope

### Entity 1: Employee

Every field tagged with its target version. Fields without a version tag are V1.

```json
{
  // ─── Identity (LOCKED) ─── V1
  "assistantId": "string",
  "title": "string",
  "description": "string",
  "orgId": "string",
  "subOrgId": "string",
  "createdBy": "string",
  "updatedBy": "string",

  // ─── References (LOCKED) ─── V1
  "opcodes": ["string"],
  "defaultOpcode": "string",
  "channels": ["string"],
  "collections": ["string"],
  "databases": ["string"],           // V2 (BeXO not in V1 — field exists but unused)

  // ─── Streaming (LOCKED) ─── V1
  "stream": false,
  "streamCumulative": false,
  "emitTokenEvents": false,
  "streamBufferSentences": false,

  // ─── Versioning (LOCKED) ─── V1
  "version": 1,
  "status": "draft | active",        // V1: only draft and active (inactive deferred)
  "disabled": false,
  "created": "ISO 8601",
  "updated": "ISO 8601",

  // ─── Persona (NEW) ─── V1
  "persona": {
    "systemPrompt": "string",        // V1: injected as @context.identity.* into all skills
    "role": "string",                 // V1
    "domainExpertise": ["string"],    // V1
    "tone": "professional | friendly | formal | concise",  // V1
    "language": "string",            // V1
    "secondaryLanguages": ["string"], // V2
    "fallbackBehavior": "apologize_and_escalate | use_default_opcode | reject"  // V1
  },

  // ─── Behavioral Constraints (NEW) ─── V1
  "autonomy": {
    "level": "supervised",            // V1: fixed to supervised. V2: semi-autonomous/autonomous
    "escalation": {
      "onConfidenceBelow": 0.6,       // V1
      "escalateTo": "string (user-id)", // V1: human only. V4: assistant-id
      "maxConsecutiveAutonomousActions": 50,  // V2
      "requireApprovalFor": ["string"] // V3
    },
    "boundaries": {
      "mustNever": ["string"],        // V1: prohibited actions list
      "mustAlways": ["string"],       // V1: mandatory actions list
      "allowedOpcodes": ["string"],   // V1
      "blockedFunctions": ["string"], // V1
      "maxDailyExecutions": 1000      // V1
    }
  },

  // ─── Operational Context (NEW) ─── V1 (operator-writable only)
  "operationalContext": {
    "currentObjective": "string",     // V1: operator-writable only
    "activePlan": "string",           // V1: operator-writable only
    "lastStatus": "string",           // V2: employee self-write via _context_patch
    "customFields": {},               // V2: employee self-write via _context_patch
    "writableByEmployee": false       // V1: always false. V2: can be true
  },

  // ─── Router (NEW) ─── V1
  "router": {
    "enabled": true,                  // V1: auto mode or direct mode
    "mode": "auto | direct",         // V1: auto = employee decides, direct = caller specifies
    "modelConfigId": "string",        // V1: required for auto mode
    "instructionId": "string",        // V1: routing instruction
    "skillManifest": "auto",          // V1: auto-generated from registered skills
    "confidenceThreshold": 0.7,       // V1
    "onLowConfidence": "ask_clarification | use_default_opcode | escalate",  // V1
    "onNoMatch": "use_default_opcode | escalate | reject"  // V1
  },

  // ─── Memory (NEW) ─── V1 (basic)
  "memory": {
    "conversationHistory": {
      "enabled": true,                // V1: short-term turn retention
      "maxTurns": 100,                // V1
      "ttl": "string (e.g. 24h)",    // V1: session isolation per user
      "sessionIsolation": "per_user"  // V1: always per_user
    },
    "knowledgeMemory": {
      "enabled": false,               // V2
      "collectionId": "string",
      "autoLearn": false,
      "learnFrom": ["successful_interactions", "corrections", "feedback"]
    },
    "workingMemory": {
      "enabled": false,               // V3
      "persist": false,
      "scope": "per_user | global"
    }
  },

  // ─── Access Permissions (NEW) ─── V1
  "accessPermissions": {
    "allowedCollections": ["string"], // V1: scoped to org_id + sub_org_id
    "allowedApiConfigs": ["string"],  // V1: scoped to org_id + sub_org_id
    "allowedConnections": ["string"], // V2: BeXO not in V1
    "sensitivityLevel": "public | internal | confidential | restricted"  // V1
  },

  // ─── Handoff (NEW) ─── V1 (human only)
  "handoff": {
    "toHuman": {
      "enabled": false,               // V1
      "method": "email",              // V1: email notification only. V3: live_transfer, ticket
      "contextTransfer": true,        // V1
      "summaryBeforeHandoff": true    // V1
    },
    "toAssistant": {                  // V4: multi-employee
      "enabled": false,
      "rules": []
    },
    "resumeAfterHandoff": false       // V3
  },

  // ─── Evaluation (NEW) ─── V2 (flag stored in V1, NOT enforced)
  "evaluation": {
    "enabled": false,                 // V1: stored, never enforced
    "modelConfigId": "string",
    "criteria": ["relevance", "accuracy", "safety", "format_compliance"],
    "scoreThreshold": 0.8,
    "onFailure": "retry | escalate | deliver_with_warning",
    "maxRetries": 2
  },

  // ─── Outbound Config (NEW) ─── V1 (formatting only)
  "outboundConfig": {
    "formattingRules": {              // V1
      "webchat": { "format": "markdown", "maxLength": null },
      "email": { "format": "plain_text", "maxLength": null },
      "api": { "format": "json", "maxLength": null }
    },
    "businessHours": {                // V2
      "enabled": false,
      "timezone": "UTC",
      "hours": { "start": "09:00", "end": "18:00" },
      "outsideHoursAction": "queue | reject | auto_reply"
    },
    "deliveryConfirmation": false     // V2
  },

  // ─── Lifecycle (NEW) ─── V1
  "lifecycle": {
    "environment": "production",      // V1: single environment. V3: dev/staging/production
    "owner": "string (user-id)",      // V1
    "deployedAt": "ISO 8601"          // V1
  },

  // ──────────────────────────────────────────
  // DEFERRED FIELDS (stored in schema, not active in V1)
  // ──────────────────────────────────────────

  "goals": [],                        // V2
  "state": { "enabled": false },      // REPLACED by operationalContext in V1
  "taskQueue": { "enabled": false },  // V3
  "onboarding": {},                   // V3
  "supervision": {},                  // V3
  "emotionalIntelligence": { "enabled": false },  // V3
  "interruptHandling": { "enabled": false },      // V3
  "triggers": [],                     // V2
  "connectedAssistants": [],          // V4
  "orchestration": {},                // V4
  "reporting": {}                     // V2
}
```

**V1 Field Count**: 20 locked + 10 new active sections = **30 active fields**
**Deferred sections**: 10 (stored as empty/disabled defaults)

---

### Entity 2: Opcode (Skill) — V1 Scope

```json
{
  // ─── Core (LOCKED) ─── V1
  "opcode_id": "string",
  "type": "ChatOp | AutoOp",         // V1
  "version": "string",               // V1: single active version per skill
  "description": "string",           // V1
  "steps": ["(see Step)"],           // V1

  // ─── Skill Metadata (NEW) ─── V1
  "category": ["string"],            // V1
  "executionProfile": {
    "interactive": false,             // V1: supports Wait steps for interactive skills
    "durationClass": "real-time | short | long",  // V1
    "parallelizable": true,           // V2
    "idempotent": false               // V2
  },
  "inputContract": {                  // V1
    "required": ["string"],
    "optional": ["string"],
    "contextDependencies": ["string"],
    "requiredCollections": ["string"],
    "requiredConnections": ["string"]  // V2
  },
  "outputContract": {                 // V1
    "returns": {
      "<key>": "string (type description)"
    },
    "producesContextPatch": false,    // V2: _context_patch for employee self-write
    "producesUiSpec": false,          // V3: BeX3D
    "compatibleChannels": ["string"]  // V1
  },
  "quality": {                        // V1: flags stored, evaluation NOT enforced
    "riskLevel": "low | medium | high",  // V1: stored, used for V2 evaluation priority
    "evaluateBeforeDelivery": false,  // V1: stored, enforced in V2
    "maxTokenBudget": 10000,          // V1
    "maxCostBudget": 0.05,            // V1
    "retryPolicy": {
      "maxRetries": 2,                // V1
      "backoffMs": 1000               // V1
    }
  }
}
```

---

### Entity 3: Step (embedded in Opcode) — V1 Scope

```json
{
  // ─── Core (LOCKED) ─── V1
  "id": "string (UUID)",
  "type": "LLM | Non-LLM | RAG | API | Memory | Condition | Loop | Parallel | Transform | Wait | Choose | Subflow",
  "name": "string",
  "description": "string",
  "input_params": {},
  "output_params": {},
  "routing": {
    "next": ["string"],
    "show_output": true,
    "accumulate_output": true,
    "true_branch": ["string"],        // used by Condition
    "false_branch": ["string"]        // used by Condition
  },

  // ─── Container steps (LOCKED) ───
  "loop_steps": [],                   // used by Loop
  "parallel_steps": []                // used by Parallel
}
```

#### V1 Step Types Available

| Step Type | Status | Purpose |
|-----------|--------|---------|
| **LLM** | LOCKED | Call an LLM with a prompt template |
| **Non-LLM** | LOCKED (backward compat) | Legacy name for RAG operations |
| **RAG** | NEW (V1) | Preferred name for knowledge retrieval |
| **API** | LOCKED | Call an external API |
| **Memory** | LOCKED | Retrieve conversation history |
| **Wait** | LOCKED | Pause for human input (interactive skills) |
| **Condition** | LOCKED | Binary branching (true/false) — maps to `when` |
| **Loop** | LOCKED | Iteration — maps to `for` and `repeat_until` |
| **Choose** | NEW (V1) | Multi-branch routing (multiple conditions) |
| **Subflow** | NEW (V1) | Call another opcode as a sub-routine |
| **Parallel** | LOCKED | Run steps concurrently — V2 enforcement |
| **Transform** | LOCKED | Data transformation — available but not Builder-configurable in V1 |

#### NEW: Choose Step input_params

```json
{
  "branches": [
    {
      "branchId": "string",
      "condition": "string (OEL expression)",
      "label": "string (human-readable)",
      "steps": ["string (step IDs)"]
    }
  ],
  "defaultBranch": {
    "steps": ["string (step IDs)"]
  }
}
```

Choose step `routing` uses branches array instead of `true_branch`/`false_branch`. Each branch has its own condition and step sequence. `defaultBranch` fires when no condition matches.

#### NEW: Subflow Step input_params

```json
{
  "opcode_id": "string → references Opcode.opcode_id",
  "inputMapping": {
    "<subflow_input_key>": "string (Jinja2 template from parent context)"
  }
}
```

#### Subflow Step output_params

```json
{
  "outputMapping": {
    "<parent_context_key>": "<subflow_output_key>"
  }
}
```

Subflow calls another opcode as a subroutine. Input/output mapping connects parent skill context to child skill context. The child opcode must exist and be in `active` status.

#### RAG Step input_params (alias for Non-LLM)

Same as existing Non-LLM `input_params`:

```json
{
  "function_call": {
    "function_name": "RAG_API",
    "function_params": {
      "org_id": "string",
      "sub_org_id": "string",
      "collection_id": "string → references Collection._id",
      "query_text": "string (Jinja2 template)",
      "top_k": 10,
      "top_n": 5,
      "use_rerank": true,
      "filters": {},
      "file_keys_filter": [],
      "search_type": "vector | keyword",  // V1: no hybrid. V2: adds hybrid
      "offset": 0
    }
  }
}
```

#### V1 Control Flow Mapping

| Roadmap Name | Maps To | Implementation |
|-------------|---------|----------------|
| `when` (conditional) | `Condition` step | Binary true/false via `routing.true_branch`/`false_branch` |
| `for` (loop) | `Loop` step, mode: `range` or `collection` | Iterates over range or collection items |
| `repeat_until` (retry) | `Loop` step, mode: `repeat-until` | Repeats until condition met |
| Multi-branch | `Choose` step (NEW) | N conditions, each routing to different step sequence |
| Sub-routine | `Subflow` step (NEW) | Calls another opcode with input/output mapping |

---

### Entity 4: Model Config — V1 Scope (Full)

No changes from DATA_MODEL.md. All fields are V1.

```json
{
  "modelConfigId": "string",
  "orgId": "string",
  "subOrgId": "string",
  "uniqueName": "string",
  "provider": "groq | ibm | sambanova | openai | anthropic | mistral | gemini | novita | cerebras | onprem",
  "modelIdentifier": "string",
  "capabilities": ["reasoning", "speed", "structured_output", "multilingual", "vision"],
  "contextWindow": 128000,
  "costPerInputToken": 0.001,
  "costPerOutputToken": 0.003,
  "supportedOutputFormats": ["string", "json", "list"],
  "status": "active | inactive",
  "createdBy": "string",
  "updatedBy": "string",
  "created": "ISO 8601",
  "updated": "ISO 8601"
}
```

V1 supports all providers: IBM, Groq, SambaNova, OpenAI, Anthropic, Mistral, Gemini, Novita, Cerebras, OnPrem.

---

### Entity 5: Instruction — V1 Scope

```json
{
  "instructionId": "string",         // V1
  "orgId": "string",                 // V1
  "subOrgId": "string",              // V1
  "name": "string",                  // V1
  "description": "string",           // V1
  "systemPrompt": "string",          // V1
  "promptTemplate": "string (Jinja2)",  // V1
  "outputFormat": "string | json | list",  // V1
  "temperature": 0.7,                // V1
  "topP": 1.0,                       // V1
  "maxTokens": 4096,                 // V1
  "modelConfigId": "string",         // V1
  "version": 1,                      // V1: stored, single active version
  "testCases": [                     // V1: stored, execution deferred to V3
    {
      "input": {},
      "expectedOutputContains": ["string"],
      "expectedFormat": "string"
    }
  ],
  "status": "active | inactive",     // V1
  "createdBy": "string",
  "updatedBy": "string",
  "created": "ISO 8601",
  "updated": "ISO 8601"
}
```

---

### Entity 6: Collection — V1 Scope

```json
{
  // ─── Core (LOCKED) ─── V1
  "_id": "string",
  "name": "string",
  "organizationId": "string",
  "subOrganizationId": "string",
  "dimensions": 1024,
  "vectorStoreType": "MongoDB | Milvus",   // V1: Milvus primary
  "chunkingStrategy": "by_paragraph | by_token_count",  // V1: these two only
  "embeddingModel": "string",        // V1: single embedding model per collection
  "created": "ISO 8601",
  "updated": "ISO 8601",

  // ─── Additions (NEW) ─── V1
  "description": "string",
  "status": "active | inactive",
  "createdBy": "string",
  "updatedBy": "string",

  // ─── V1 Extensions ───
  "accessControl": {
    "allowedAssistants": ["string"],  // V1
    "sensitivityLevel": "public | internal | confidential | restricted"  // V1
  },
  "language": "string",              // V1
  "tags": ["string"],                // V1

  // ─── V1 Ingestion Config ───
  "ingestion": {
    "supportedSources": ["file_upload", "url_crawl"],  // V1: file + URL only
    "chunkSize": 512,                 // V1: configurable
    "chunkOverlap": 50,               // V1: configurable
    "metadata": {
      "extractFileName": true,        // V1
      "extractDocumentTitle": true,   // V1
      "extractIngestionDate": true,   // V1
      "extractPageNumber": true       // V1
    }
  },

  // ─── V1 Query Config ───
  "queryConfig": {
    "defaultTopK": 10,                // V1
    "similarityThreshold": 0.7,       // V1
    "metadataFilters": true,          // V1
    "hybridSearch": false,            // V2
    "reranking": false                // V2
  },

  // ─── Deferred ───
  "freshness": {                      // V2
    "retentionDays": 365,
    "reIngestionSchedule": "manual",  // V1: manual only
    "lastIngested": "ISO 8601"
  }
}
```

**V1 Query Operations:**
- `basic_search` — semantic search against collection
- `search_with_files` — semantic search scoped to specific files
- `retrieve_files` — direct file fetch by key

---

### Entity 7: Connection — V2

**Not active in V1.** The Connection entity (BeXO / structured data platform) is deferred to V2. The schema exists in DATA_MODEL.md but the `databases[]` reference on Employee is unused in V1.

---

### Entity 8: API Config — V1 Scope

```json
{
  "apiConfigId": "string",           // V1
  "orgId": "string",                 // V1
  "subOrgId": "string",              // V1
  "name": "string",                  // V1
  "description": "string",           // V1
  "baseUrl": "string (URL)",         // V1
  "authType": "none | basic | bearer | api_key",  // V1: no oauth2 yet. V2: adds oauth2
  "authConfig": {},                   // V1: plain config. V3: vault references
  "defaultHeaders": {},              // V1
  "timeout": 30,                     // V1
  "rateLimits": {                    // V1
    "requestsPerMinute": 60,
    "requestsPerDay": 10000
  },
  "retryPolicy": {                   // V1
    "maxRetries": 3,
    "backoffMs": 1000
  },
  "healthCheckEndpoint": "string",   // V1
  "endpoints": [                     // V1
    {
      "endpointId": "string",
      "path": "string",
      "method": "GET | POST | PUT | PATCH | DELETE",
      "description": "string",
      "pathParams": [
        { "name": "string", "type": "string", "description": "string" }
      ],
      "queryParams": [
        { "name": "string", "type": "string", "required": false, "description": "string" }
      ],
      "requestBody": {},
      "responseSchema": {},
      "errorMappings": {}
    }
  ],
  "pagination": {                    // V1
    "type": "offset | page | cursor | link_header",
    "pageParam": "string",
    "limitParam": "string"
  },
  "status": "active | inactive",
  "createdBy": "string",
  "updatedBy": "string",
  "created": "ISO 8601",
  "updated": "ISO 8601"
}
```

---

### Entity 9: Channel — V1 Scope

**Fully LOCKED. No schema changes.**

```json
{
  "_id": "string",
  "organizationId": "string",
  "subOrganizationId": "string",
  "assistantId": "string",
  "address": "string",
  "type": "Webchat | Email | Voice | Whatsapp | Messenger",
  "access_token": "string (optional)"
}
```

**V1 Channel Scope:**

| Channel | Inbound | Outbound | Status |
|---------|---------|----------|--------|
| **REST API** | Direct call | JSON response | V1 |
| **Web Chat** | Embedded widget | Markdown | V1 |
| **Email** | - | Basic SMTP (plain text) | V1 |
| **Slack** | If capacity | If capacity | V1 stretch |
| **Voice** | - | - | V2+ |
| **WhatsApp** | - | - | V2 |
| **MS Teams** | - | - | V2 |

---

### Entity 10: Trigger — V2

**Not active in V1.** Schema exists in DATA_MODEL.md. The `triggers[]` reference on Employee is stored as an empty array in V1.

---

### Entity 11: Execution Log — V1 Scope (Full)

All fields active. This is non-negotiable from day one.

```json
{
  "executionLogId": "string",
  "orgId": "string",
  "subOrgId": "string",
  "assistantId": "string",
  "opcodeId": "string",
  "triggerSource": "channel | manual",  // V1: only channel and manual. V2: adds schedule, event, agent
  "triggerDetails": {
    "channelId": "string",
    "userId": "string",
    "triggerId": null,                // V2
    "fromAssistantId": null           // V4
  },
  "input": {},
  "output": {},
  "steps": [
    {
      "stepId": "string",
      "type": "string",
      "name": "string",
      "startTime": "ISO 8601",
      "endTime": "ISO 8601",
      "durationMs": 0,
      "status": "success | error | skipped",
      "input": {},
      "output": {},
      "confidenceScore": null,
      "tokenUsage": { "input": 0, "output": 0 },
      "cost": 0.0,
      "error": null
    }
  ],
  "totalDurationMs": 0,
  "totalCost": 0.0,
  "totalTokens": { "input": 0, "output": 0 },
  "status": "success | error | escalated",
  "evaluationScore": null,            // V2: when evaluation engine is active
  "userFeedback": {
    "rating": null,
    "comment": null
  },
  "timestamp": "ISO 8601"
}
```

**V1 Retention**: 90 days. Append-only, immutable.

---

## V1 Governance — Minimum for Enterprise

### Access Control (org level)

```json
{
  "orgId": "string",
  "rbac": {
    "roles": ["admin", "operator"],   // V1: two roles only. V3: adds builder, viewer, etc.
    "permissions": {
      "admin": ["*"],
      "operator": [
        "employee.view", "employee.activate", "employee.deactivate",
        "employee.edit_context",
        "logs.view", "metrics.view"
      ]
    }
  },
  "orgIsolation": true,
  "subOrgIsolation": true
}
```

### Content Filtering (sub-org level)

```json
{
  "orgId": "string",
  "subOrgId": "string",
  "inputFiltering": {
    "enabled": true,
    "blockedPatterns": ["string"],    // V1: keyword/pattern list
    "maxInputLength": 10000
  },
  "outputFiltering": {
    "enabled": true,
    "blockedTopics": ["string"],      // V1: basic prohibited phrase list
    "piiDetection": false,            // V3
    "piiAction": "mask | block | warn"
  }
}
```

### API Key Management

```json
{
  "apiKeyId": "string",
  "orgId": "string",
  "subOrgId": "string",
  "assistantId": "string",           // scoped to specific employee
  "keyHash": "string",               // never store plaintext
  "permissions": ["execute"],         // V1: execute only
  "rateLimit": {
    "requestsPerMinute": 60,
    "requestsPerDay": 10000
  },
  "status": "active | revoked",
  "created": "ISO 8601",
  "expiresAt": "ISO 8601"
}
```

---

## V1 Observability

### Execution Logs
Full run record: employee, skill, trigger source, status, duration, token usage, estimated cost. Every execution logged.

### Basic Metrics Dashboard
- Success rate, failure rate per skill
- Average response time per skill
- Total cost per day — per employee and platform-wide

### Cost Tracking
- Per-execution cost calculation from ModelConfig cost profiles
- Daily and monthly aggregation per employee

### Basic Alerting
- Failure rate alert (configurable threshold)
- Cost alert (configurable budget)
- Email notification delivery

---

## Builder Agent — V1 Architecture

The Builder Agent is itself a BeX digital employee. It is built on V1 platform capabilities.

### Builder Agent as an Employee

```json
{
  "assistantId": "builder-agent-001",
  "title": "BeX Builder Agent",
  "description": "Meta-employee that builds other digital employees from natural language descriptions",
  "orgId": "platform",
  "subOrgId": "platform",
  "persona": {
    "role": "AI Platform Builder",
    "tone": "professional",
    "language": "en",
    "systemPrompt": "You are the BeX Builder Agent. You help users create digital employees by interviewing them about their needs and generating correct platform configurations."
  },
  "router": {
    "enabled": true,
    "mode": "auto"
  }
}
```

### Builder Agent Skills (Opcodes)

| Skill | Type | Interactive | Purpose |
|-------|------|------------|---------|
| **Discovery** | ChatOp | Yes (Wait steps) | Interview user: purpose, domain, data sources, audience, channels, autonomy level |
| **Mapping** | AutoOp | No | Maps discovery output to platform entities. Uses specialized instruction trained on full platform schema |
| **Generation** | AutoOp | No | Takes component list and generates valid JSON for each entity. Validates against schemas |
| **Review** | ChatOp | Yes (Wait steps) | Presents human-readable plan to user. Collects feedback. Routes changes back to Mapping or Generation |
| **Save** | AutoOp | No | Calls BeX Server provisioning API. Handles dependency ordering (instruction before skill, collection before employee) |

### Builder Agent V1 Can Configure

| What | From User Input | What It Generates |
|------|----------------|-------------------|
| Employee identity + constraints | "I want a customer support employee that is always polite and never discusses competitor pricing" | Employee identity block, behavioral constraints, tone setting |
| Skills from use case | "It should answer product questions, escalate complex issues, and check order status" | 3 skills with correct step types, input/output contracts, when conditions |
| LLM instructions | Use case description + domain knowledge | System prompts for each skill's LLM steps, output format specs, temperature settings |
| Knowledge collection setup | "I'll upload our product docs and return policy PDFs" | Collection config, chunking strategy recommendation, metadata fields |
| API integration | "It needs to call our order management API to check status" | API registry entry with endpoint definitions based on user's API description or schema |
| Channel configuration | "Deploy it on our website and also send updates to email" | Web chat inbound channel config, email outbound channel config |
| Routing logic | Derived from skill descriptions | Router manifest with descriptions, auto mode enabled, fallback configured |

### Builder Agent Knowledge Base

V1 Baseline content:
- Full JSON schema definitions for: Employee, Skill (LLM+RAG+API+Memory+Wait+Choose+Subflow), Instruction, Collection, API, Channel
- Interview patterns for reactive employee use cases
- 20+ example employee configurations as few-shot training data for generation skill
- Validation rules and anti-patterns

---

## Feature Flag Strategy

All V2+ capabilities are gated by feature flags. Single codebase serves all versions.

| Flag Category | Examples | Usage |
|--------------|----------|-------|
| **Version flags** | `V1_ENABLED`, `V2_ENABLED` | Enable all capabilities of a given version for an org |
| **Capability flags** | `EVALUATION_ENABLED`, `TRIGGERS_ENABLED`, `BEXO_READ_ENABLED` | Enable specific capabilities independently |
| **Beta flags** | `CANVAS_BETA`, `MULTI_AGENT_BETA` | Enable capabilities under development for test customers |
| **Compliance flags** | `HIPAA_MODE`, `FINREG_MODE`, `DATA_RESIDENCY_STRICT` | Enable compliance-specific features for regulated customers |

### V1 Default Flags

```json
{
  "V1_ENABLED": true,
  "V2_ENABLED": false,
  "EVALUATION_ENABLED": false,
  "TRIGGERS_ENABLED": false,
  "BEXO_READ_ENABLED": false,
  "BEXO_WRITE_ENABLED": false,
  "ORCHESTRATION_ENABLED": false,
  "CANVAS_BETA": false,
  "MULTI_AGENT_BETA": false
}
```

---

## V1 — What Is Explicitly Deferred

| Deferred Capability | Target Version |
|-------------------|----------------|
| Evaluation Engine (output quality enforcement) | V2 |
| Employee self-write to context (context patches) | V2 |
| Scheduled and event-based triggers (proactive execution) | V2 |
| Structured data platform (BeXO — database queries and write actions) | V2 |
| Slack / Teams inbound channels (full integration) | V2 |
| Hybrid search in knowledge queries | V2 |
| Instruction version history and rollback | V3 |
| Mandatory test cases for skills and instructions | V3 |
| Full secrets vault integration (HashiCorp Vault) | V3 |
| Change management and approval workflows | V3 |
| Dynamic UI (BeX3D) | V3 |
| Multi-employee orchestration and delegation | V4 |
| Inter-employee communication | V4 |

---

## V1 Entity Summary

| Entity | V1 Status | Notes |
|--------|----------|-------|
| **Employee** | Active | 20 locked + 10 new active sections, 10 deferred sections |
| **Opcode (Skill)** | Active | All fields active. New step types: RAG, Choose, Subflow |
| **Step** | Active | 12 step types (9 locked + 3 new) |
| **ModelConfig** | Active | All fields, all providers |
| **Instruction** | Active | All fields. Test case execution deferred to V3 |
| **Collection** | Active | Core + ingestion + query config. Hybrid search deferred |
| **Connection** | Deferred | V2 (BeXO) |
| **ApiConfig** | Active | All fields. OAuth2 deferred to V2 |
| **Channel** | Active | Fully locked. V1 scope: REST API, Web Chat, Email |
| **Trigger** | Deferred | V2 |
| **ExecutionLog** | Active | All fields. 90-day retention |
| **Governance** | Active | Basic RBAC (admin/operator), content filtering, API keys |

---

## V1 Enterprise Value Proposition

> A non-technical enterprise user describes their employee in a 10-minute conversation. The Builder Agent generates and deploys a working digital employee. The employee answers domain questions from a knowledge base, checks live data via API, escalates intelligently, and serves users through web chat and email. The enterprise has measurable ROI from day one.
