# CoolRiots Digital Employee Platform — System Design Document

> This document is the comprehensive reference for building the unified data model.
> It merges grounded implementation schemas (from existing projects) with the architectural vision for Digital Employees.

---

## Architectural Framework

Every component in this system must answer at least one of five fundamental questions:

| # | Question | Components |
|---|----------|-----------|
| 1 | **Who is this employee and what is their job?** | Employee Management |
| 2 | **What do they know?** | Knowledge Platform, Structured Data Platform |
| 3 | **How do they think and decide?** | Intelligence Core |
| 4 | **What can they do and how?** | Skill Registry (Opcodes), Integration Platform, Channel Platform |
| 5 | **How do we trust them in an enterprise?** | Observability, Governance & Compliance |

Plus cross-cutting platforms: **Trigger & Event Platform**, **Orchestration Layer**, **Dynamic UI Platform**

### Component Dependencies

```
Employee Management (top-level container)
  ├── references → Skill Registry (opcodes)
  ├── references → Channel Platform
  ├── references → Knowledge Platform (collections)
  ├── references → Structured Data Platform (connections)
  ├── references → Trigger & Event Platform
  ├── uses → Intelligence Core (model configs, instructions)
  ├── uses → Integration Platform (API configs)
  ├── uses → Orchestration Layer (connected employees)
  ├── monitored by → Observability Platform
  └── governed by → Governance & Compliance
```

---

## Part 1: Employee Management

**Answers: Who is this employee and what is their job?**

The Employee is the top-level entity. Everything else belongs to or serves an employee. It holds identity, behavioral rules, operational state, and references to all other entities.

### 1A. Core Schema — LOCKED

These fields exist in the current system and cannot be removed or edited.

```json
{
  "assistantId": "string (unique ID)",
  "title": "string (required)",
  "description": "string (optional, can be empty)",
  "orgId": "string (required — organization ID)",
  "subOrgId": "string (required — sub-organization ID)",
  "createdBy": "string (required — user ID)",
  "updatedBy": "string (required — user ID)",

  "opcodes": ["opcode-id-1", "opcode-id-2"],
  "defaultOpcode": "opcode-id-1",

  "channels": ["channel-id-1"],
  "collections": ["collection-id-1"],
  "databases": ["db-id-1"],

  "stream": false,
  "streamCumulative": false,
  "emitTokenEvents": false,
  "streamBufferSentences": false,

  "version": 1,
  "status": "active | inactive",
  "disabled": false,
  "created": "ISO 8601 timestamp",
  "updated": "ISO 8601 timestamp"
}
```

### 1B. Identity Extensions — NEW

A digital employee needs a consistent identity beyond just a title.

```json
{
  "persona": {
    "systemPrompt": "You are Alex, a senior customer support specialist at Acme Corp...",
    "role": "Customer Support Agent",
    "domainExpertise": ["billing", "technical_support", "onboarding"],
    "tone": "professional | friendly | formal | concise",
    "language": "en",
    "secondaryLanguages": ["es", "fr"],
    "fallbackBehavior": "apologize_and_escalate"
  }
}
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| persona.systemPrompt | string | yes | Core behavioral instruction applied across all interactions |
| persona.role | string | no | Human-readable role title |
| persona.domainExpertise | string[] | no | Tags for routing and capability matching |
| persona.tone | string | no | Communication style |
| persona.language | string | no | Primary language code |
| persona.secondaryLanguages | string[] | no | Additional supported languages |
| persona.fallbackBehavior | string | no | What to do when uncertain |

### 1C. Behavioral Extensions — NEW

Enterprise guardrails on what the employee can/cannot do.

```json
{
  "autonomy": {
    "level": "supervised | semi-autonomous | autonomous",
    "escalation": {
      "onConfidenceBelow": 0.6,
      "escalateTo": "human | assistant-id",
      "maxConsecutiveAutonomousActions": 50,
      "requireApprovalFor": ["send_email", "modify_database", "external_api_write"]
    },
    "boundaries": {
      "mustNever": ["share customer PII externally", "make financial commitments"],
      "mustAlways": ["cite sources", "confirm before destructive actions"],
      "allowedOpcodes": ["*"],
      "blockedFunctions": ["delete_record"],
      "maxDailyExecutions": 1000
    }
  },

  "goals": [
    {
      "goalId": "goal-1",
      "metric": "customer_satisfaction",
      "target": 0.9,
      "measurement": "post_interaction_rating",
      "window": "7d"
    }
  ]
}
```

| Field | Type | Notes |
|-------|------|-------|
| autonomy.level | string | Overall autonomy classification |
| autonomy.escalation | object | When and how to escalate to human/other employee |
| autonomy.boundaries.mustNever | string[] | Hard constraints — never violated regardless of instructions |
| autonomy.boundaries.mustAlways | string[] | Mandatory behaviors |
| goals | array | KPIs for self-evaluation and performance tracking |

### 1D. Operational Extensions — NEW

Living state and workload management.

```json
{
  "memory": {
    "conversationHistory": {
      "enabled": true,
      "maxTurns": 100,
      "ttl": "30d"
    },
    "knowledgeMemory": {
      "enabled": true,
      "collectionId": "mem-collection-1",
      "autoLearn": true,
      "learnFrom": ["successful_interactions", "corrections", "feedback"]
    },
    "workingMemory": {
      "enabled": true,
      "persist": true,
      "scope": "per_user | global"
    }
  },

  "state": {
    "enabled": true,
    "currentState": "awaiting_documents",
    "transitions": {
      "awaiting_documents": { "on": "documents_received", "next": "processing" },
      "processing": { "on": "complete", "next": "review" }
    },
    "data": {}
  },

  "taskQueue": {
    "enabled": true,
    "maxConcurrent": 5,
    "priorityRules": [
      { "condition": "source == 'vip_channel'", "priority": "critical" },
      { "condition": "age > 3600", "priority": "high" }
    ],
    "overflow": "queue | reject | delegate",
    "maxQueueSize": 100
  }
}
```

| Field | Type | Notes |
|-------|------|-------|
| memory.conversationHistory | object | Extended conversation retention (beyond current Memory step's last-N) |
| memory.knowledgeMemory | object | Self-improving: stores learned patterns in a dedicated RAG collection |
| memory.workingMemory | object | Persistent scratchpad (e.g., "this customer prefers formal tone") |
| state | object | State machine for multi-day workflows |
| taskQueue | object | Inbox, prioritization, capacity management |

### 1E. Lifecycle & Supervision Extensions — NEW

Training, oversight, and growth path.

```json
{
  "onboarding": {
    "status": "training | probation | active | veteran",
    "shadowAssistantId": "asst-senior",
    "graduationCriteria": {
      "minInteractions": 100,
      "minConfidence": 0.85,
      "minSatisfactionScore": 0.8
    },
    "autoGraduate": true
  },

  "supervision": {
    "mode": "shadow | supervised | independent",
    "supervisorId": "user-id or assistant-id",
    "reviewQueue": {
      "sampleRate": 0.1,
      "alwaysReviewWhen": ["confidence < 0.7", "sentiment == negative"]
    },
    "feedbackIntegration": {
      "source": "supervisor_ratings | customer_ratings | outcome_tracking",
      "autoAdjust": true
    }
  }
}
```

### 1F. Interaction Extensions — NEW

How the employee handles emotions, handoffs, and interruptions.

```json
{
  "emotionalIntelligence": {
    "enabled": true,
    "detectSentiment": true,
    "adaptTone": true,
    "escalateOnFrustration": true,
    "frustrationThreshold": 3
  },

  "handoff": {
    "toHuman": {
      "enabled": true,
      "method": "live_transfer | ticket | email",
      "contextTransfer": true,
      "summaryBeforeHandoff": true
    },
    "toAssistant": {
      "enabled": true,
      "rules": [
        { "condition": "topic == 'billing'", "targetAssistantId": "asst-billing" }
      ]
    },
    "resumeAfterHandoff": true
  },

  "interruptHandling": {
    "enabled": true,
    "pauseCurrentOnPriorityCritical": true,
    "maxPausedTasks": 3,
    "autoResumeAfter": "300s"
  }
}
```

### 1G. Relationship Extensions — NEW

References to other entities and employees.

```json
{
  "triggers": ["trigger-id-1", "trigger-id-2"],

  "connectedAssistants": [
    {
      "assistantId": "asst-sales",
      "role": "specialist",
      "capabilities": ["product_lookup", "pricing"],
      "protocol": "request_response | delegate | broadcast"
    }
  ],

  "reporting": {
    "dailySummary": {
      "enabled": true,
      "sendTo": "manager@company.com",
      "channel": "email",
      "includeMetrics": ["interactions_count", "resolution_rate", "avg_confidence", "escalations"]
    },
    "alerts": [
      { "condition": "error_rate > 0.1", "notify": "admin", "channel": "slack" }
    ]
  }
}
```

### What the Employee Replaces

In the old system, this was split across:
- **Assistant Instance** (MongoDB) — identity, ownership, status
- **Opcode Assistant** (Redis) — opcodes, streaming config, logic

Now unified into a single entity with extensions for Digital Employee capabilities.

---

## Part 2: Skill Registry (Opcodes / Steps)

**Answers: What can they do and how do they do it?**

**Source project:** `/home/mohammad/Public/CoolRiots/new/Project-O-New`

A skill (opcode) is a pipeline definition — a container of ordered steps. Each employee references one or more skills by ID. The `defaultOpcode` determines which skill runs first. Skills are the executable capabilities of the employee — each is one complete, testable unit of work.

### 2A. Opcode Core Schema — LOCKED

```json
{
  "opcode_id": "string (unique ID)",
  "type": "ChatOp | AutoOp",
  "version": "1",
  "description": "string (NEW — what this opcode does)",
  "steps": [
    {
      "id": "step-uuid",
      "type": "LLM | Non-LLM | API | Memory | Condition | Loop | Parallel | Transform | Wait",
      "name": "string (NEW — human-readable label for UI)",
      "description": "string (NEW — what this step does in the flow)",
      "input_params": { },
      "output_params": { },
      "routing": {
        "next": ["step-id"],
        "show_output": true,
        "accumulate_output": true,
        "true_branch": ["step-id"],
        "false_branch": ["step-id"]
      }
    }
  ]
}
```

### 2B. Skill Metadata Extensions — NEW

Metadata about the skill itself (not its steps), used by the router and orchestration layer.

```json
{
  "category": ["analysis", "communication", "data_retrieval", "action"],
  "executionProfile": {
    "interactive": false,
    "durationClass": "real-time | short | long",
    "parallelizable": true,
    "idempotent": false
  },
  "inputContract": {
    "required": ["user_input"],
    "optional": ["language", "customer_id"],
    "contextDependencies": ["org_id", "sub_org_id"],
    "requiredCollections": ["coll-1"],
    "requiredConnections": ["db-1"]
  },
  "outputContract": {
    "returns": { "answer": "string", "confidence": "number" },
    "producesContextPatch": true,
    "producesUiSpec": false,
    "compatibleChannels": ["Webchat", "Whatsapp", "Email"]
  },
  "quality": {
    "evaluateBeforeDelivery": false,
    "maxTokenBudget": 10000,
    "maxCostBudget": 0.05,
    "retryPolicy": { "maxRetries": 2, "backoffMs": 1000 }
  }
}
```

| Field | Type | Notes |
|-------|------|-------|
| category | string[] | Tags for what kind of task this skill handles — used by router |
| executionProfile.interactive | boolean | Can this skill pause and wait for human input mid-execution? |
| executionProfile.durationClass | string | Expected duration: real-time (<5s), short (<30s), long (minutes) |
| executionProfile.idempotent | boolean | Safe to run twice with same inputs? |
| inputContract | object | Declares what inputs the skill needs — enables validation before execution |
| outputContract | object | Declares what the skill returns — enables downstream planning |
| quality | object | Cost/token budgets and retry behavior |

### 2C. Step Types (9 active) — LOCKED

#### 1. LLM — Language Model Call

```json
{
  "type": "LLM",
  "input_params": {
    "org_id": "{{ context.org_id }}",
    "sub_org_id": "{{ context.sub_org_id }}",
    "unique_name": "model-unique-name",
    "query": "{{ context.user_input }}",
    "model_type": "groq | ibm | sambanova",
    "stream": false,
    "placeholder_values": {},
    "image_urls": []
  },
  "output_params": {
    "output_name": "result",
    "output_type": "string | dict | list | number",
    "extra_output": [
      { "key": "field_name", "value": "response.field" }
    ]
  }
}
```

**Notes:**
- `unique_name` identifies the specific model configuration (references Intelligence Core)
- `model_type` selects the LLM provider
- `output_type` coerces the LLM text output (default: "string")
- Streaming only supports `output_type: "string"`

#### 2. Non-LLM — Registered Function Call (includes RAG)

```json
{
  "type": "Non-LLM",
  "input_params": {
    "function_call": {
      "function_name": "RAG_API | KNOWLEDGE_RETRIEVAL_API",
      "function_params": {
        "org_id": "{{ context.org_id }}",
        "sub_org_id": "{{ context.sub_org_id }}",
        "collection_id": "collection-id",
        "query_text": "{{ context.query }}",
        "top_k": 10,
        "top_n": 5,
        "use_rerank": true,
        "filters": {},
        "file_keys_filter": [],
        "search_type": "vector | keyword | hybrid",
        "offset": 0
      }
    }
  },
  "output_params": {
    "output_names": {
      "rag_result": "compiled_text",
      "all_fields": "all_fields",
      "presigned_urls": "presigned_urls"
    }
  }
}
```

**Available functions in registry:**
- `RAG_API` — Vector/keyword/hybrid search with re-ranking
- `KNOWLEDGE_RETRIEVAL_API` — Same as RAG_API without presigned URLs

#### 3. API — External HTTP Call

```json
{
  "type": "API",
  "input_params": {
    "_id": "api-config-id",
    "organizationId": "{{ context.org_id }}",
    "subOrganizationId": "{{ context.sub_org_id }}",
    "query_params": {},
    "body": {},
    "auth": "override_auth",
    "headers": {},
    "array_of_dicts_mode": false
  },
  "output_params": {
    "output_names": {
      "api_result": "*"
    }
  }
}
```

#### 4. Memory — Conversation History

```json
{
  "type": "Memory",
  "input_params": {
    "n": 3,
    "limit_size": 3000,
    "org_id": "{{ context.org_id }}",
    "sub_org_id": "{{ context.sub_org_id }}"
  },
  "output_params": {
    "output_name": "memory_context"
  }
}
```

#### 5. Condition — Boolean Branching

```json
{
  "type": "Condition",
  "input_params": {
    "expression": "{{ output.score }} > 0.8"
  },
  "routing": {
    "true_branch": ["step-id-if-true"],
    "false_branch": ["step-id-if-false"]
  }
}
```

#### 6. Loop — Iteration

```json
{
  "type": "Loop",
  "input_params": {
    "mode": "range | collection | repeat-until",
    "range_start": 0,
    "range_end": 5,
    "range_step": 1,
    "item_alias": "i",
    "collection": "{{ output.items }}"
  },
  "loop_steps": []
}
```

#### 7. Parallel — Concurrent Execution

```json
{
  "type": "Parallel",
  "parallel_steps": []
}
```

#### 8. Transform — Data Transformation

```json
{
  "type": "Transform",
  "input_params": {
    "expression": "..."
  }
}
```

#### 9. Wait — Time Delay

```json
{
  "type": "Wait",
  "input_params": {
    "duration": 1000
  }
}
```

### 2D. Data Flow Between Steps

Steps communicate through two mechanisms:

1. **`executor.context`** — Initial input data (from the API request's `context` field). Read-only.
2. **`executor.accumulated_output`** — Mutable dict. Each step's output merges into this.

Steps reference data using **Jinja2 templates**:
- `{{ context.user_input }}` — from initial request
- `{{ output.rag_result }}` — from a previous step's output

### 2E. Routing Model

```
Step A → routing.next: ["step-B-id"] → Step B → routing.next: [] → END

Condition:
  true  → routing.true_branch: ["step-X-id"]
  false → routing.false_branch: ["step-Y-id"]
```

- `routing.next` is an array but currently only `[0]` is used
- Execution starts from `steps[0]`
- Ends when `routing.next` is empty/null
- Max 1000 iterations safety

### 2F. Execution Request

```json
{
  "org_id": "string",
  "sub_org_id": "string",
  "opcode_id": "string",
  "context": { "user_input": "...", "any_key": "any_value" },
  "memory_log_id": "string (optional)",
  "stream": false,
  "emit_token_events": true,
  "stream_buffer_sentences": false,
  "debug": false
}
```

### 2G. Streaming Events

When `stream: true`:
1. **Token events** — individual tokens from LLM steps
2. **Info events** — loop iteration progress
3. **Snapshot events** — accumulated state after each step
4. **Final event** — `status: "ok"` or `status: "error"`

### 2H. Error Handling

- **InputError** — invalid parameters
- **BusinessLogicError** — logic failures
- **ExternalAPIError** — external service failures
- **DatabaseError** — Redis/DB failures
- Errors bubble up and stop execution
- In streaming mode, error snapshot is yielded before stopping

### Storage

- Stored in **Redis** as JSON
- Key: `opcode:{org_id}:{sub_org_id}:{opcode_id}`
- CRUD managed by BeX-platform-server

---

## Part 3: Intelligence Core

**Answers: How do they think and decide?**

This is the LLM management layer — not just "which model" but the full configuration of how reasoning happens. Currently, LLM configs are stored in Redis by BeX-platform-server. This part elevates them to a proper platform.

### 3A. Model Config Schema

A model config represents a specific LLM model available to the system. Referenced by `unique_name` in LLM steps.

```json
{
  "modelConfigId": "string (unique ID)",
  "orgId": "string",
  "subOrgId": "string",
  "uniqueName": "string (what LLM steps reference)",
  "provider": "groq | ibm | sambanova | openai | anthropic",
  "modelIdentifier": "string (provider-specific model ID)",
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

**Current Redis key pattern:** `{Provider}Model:{org_id}:{sub_org_id}:{unique_name}`

### 3B. Instruction Schema

An instruction is a reusable prompt template paired with model parameters. This enables version-controlled, testable prompt management.

```json
{
  "instructionId": "string (unique ID)",
  "orgId": "string",
  "subOrgId": "string",
  "name": "string",
  "description": "string",
  "systemPrompt": "string (behavioral instruction)",
  "promptTemplate": "string with {{ placeholders }}",
  "outputFormat": "string | json | list",
  "temperature": 0.7,
  "topP": 1.0,
  "maxTokens": 4096,
  "modelConfigId": "string (which model to use)",
  "version": 1,
  "testCases": [
    {
      "input": { "user_input": "test query" },
      "expectedOutputContains": ["keyword1"],
      "expectedFormat": "string"
    }
  ],
  "status": "active | inactive",
  "createdBy": "string",
  "updatedBy": "string",
  "created": "ISO 8601",
  "updated": "ISO 8601"
}
```

| Field | Type | Notes |
|-------|------|-------|
| promptTemplate | string | Jinja2 template with {{ placeholders }} |
| testCases | array | Must pass before deployment — validates instruction quality |
| version | number | Incremented on each edit, enables rollback |

### 3C. Router Configuration

The router decides which opcode/skill to execute for a given input. It's configured per employee.

```json
{
  "router": {
    "enabled": true,
    "modelConfigId": "router-model-id",
    "instructionId": "router-instruction-id",
    "skillManifest": "auto | manual",
    "confidenceThreshold": 0.7,
    "onLowConfidence": "ask_clarification | use_default_opcode | escalate",
    "onNoMatch": "use_default_opcode | escalate | reject"
  }
}
```

This lives on the Employee entity. When `router.enabled: true`, incoming requests go through the router to select the right opcode instead of always using `defaultOpcode`.

### 3D. Evaluation Configuration

Quality gate before delivering output to the user.

```json
{
  "evaluation": {
    "enabled": false,
    "modelConfigId": "eval-model-id",
    "criteria": ["relevance", "accuracy", "safety", "format_compliance"],
    "scoreThreshold": 0.8,
    "onFailure": "retry | escalate | deliver_with_warning",
    "maxRetries": 2
  }
}
```

Also lives on the Employee entity. When enabled, the final output of an opcode is evaluated before delivery.

---

## Part 4: Knowledge Platform (Collections)

**Answers: What do they know? (Unstructured)**

**Source project:** `/home/mohammad/Public/CoolRiots/new/KB`

Collections hold documents/files for RAG search. Files are extracted, chunked, embedded, and stored across multiple databases.

### 4A. Core Schema — LOCKED

```json
{
  "_id": "string",
  "name": "string",
  "description": "string (NEW)",
  "organizationId": "string",
  "subOrganizationId": "string",
  "dimensions": 1024,
  "vectorStoreType": "MongoDB | Milvus",
  "chunkingStrategy": "",
  "embeddingModel": "jina",
  "status": "active (NEW)",
  "createdBy": "user-id (NEW)",
  "updatedBy": "user-id (NEW)",
  "created": "ISO 8601",
  "updated": "ISO 8601"
}
```

### 4B. Extensions — NEW

```json
{
  "accessControl": {
    "allowedAssistants": ["asst-1", "asst-2"],
    "sensitivityLevel": "public | internal | confidential | restricted"
  },
  "freshness": {
    "retentionDays": 365,
    "reIngestionSchedule": "manual | daily | weekly",
    "lastIngested": "ISO 8601"
  },
  "language": "en",
  "tags": ["product_docs", "faq"]
}
```

### 4C. Ingestion Pipeline

```
Files → COS (bucket: coll-{sub_org_id}-{collection_id})
  → Extract content (per file type)
  → Chunk (1000 chars, 200 overlap)
  → Embed (Jina AI, configurable dimensions)
  → Store in 3 databases:
    - Milvus/MongoDB → vectors (similarity search)
    - Elasticsearch  → full text (keyword search)
    - MongoDB        → metadata (chunk details)
```

### 4D. Supported File Types

| Type | Formats | Processing |
|------|---------|-----------|
| Documents | PDF, Word, PowerPoint | Text extraction + OCR fallback |
| Spreadsheets | Excel, CSV | Flattened to text rows |
| Images | JPEG, PNG, WebP, TIFF | OCR via Gemini |
| Audio | WAV, MP3, AAC, OGG | Transcription via Groq Whisper |
| Text | .txt, .md, .json | Direct UTF-8 reading |

### 4E. Search Capabilities

| Type | Method |
|------|--------|
| `vector` | Query → Jina embedding → Milvus cosine similarity |
| `keyword` | Query → Elasticsearch full-text search |
| `hybrid` | Vector + keyword via Reciprocal Rank Fusion (RRF) |
| `enhanced_vector` | Vector + Elasticsearch enrichment |
| `file` | Retrieve chunks for specific files |

Optional **reranking** via Jina Reranker.

### 4F. Data Isolation

| System | Isolation | Naming |
|--------|-----------|--------|
| COS bucket | Per collection | `coll-{sub_org_id}-{collection_id}` |
| Milvus | Per sub-org | `coll_{sub_org_id}` |
| Elasticsearch | Per sub-org | Index = `sub_org_id` |
| MongoDB | Document-level | Filtered by org/sub-org/collection |

### Storage

- Stored in **MongoDB**
- CRUD managed by KB service

---

## Part 5: Structured Data Platform (Connections)

**Answers: What do they know? (Structured) + What can they do with data?**

**Source project:** `/home/mohammad/Public/CoolRiots/new/BeXO`

A "database" in the employee context is a **Connection** — a configured bridge to an external data source.

### 5A. Core Schema — LOCKED

```json
{
  "org_id": "string",
  "app_id": "string",
  "connection_id": "string",
  "connection_name": "string",
  "description": "string (optional)",
  "source_type": "postgresql | mysql | sqlserver | mongodb | redis | rest_api | file | milvus | elastic",
  "settings": {},
  "advanced_options": {
    "retry_attempts": 0,
    "retry_backoff_secs": 0,
    "enable_logging": false,
    "connection_pool_size": null
  },
  "createdBy": "user-id (NEW)",
  "updatedBy": "user-id (NEW)",
  "created_at": "datetime",
  "updated_at": "datetime"
}
```

**Note:** `app_id` is a separate concept from `sub_org_id`.

### 5B. Schema Discovery Extension — NEW

Business-friendly metadata so the LLM can reason about data correctly. This is what makes natural-language-to-query work.

```json
{
  "schemaDiscovery": {
    "enabled": true,
    "tables": [
      {
        "name": "customers",
        "businessDescription": "All active and inactive customer accounts",
        "fields": [
          {
            "name": "annual_rev",
            "type": "float",
            "businessName": "Annual Revenue",
            "businessDescription": "Customer's annual revenue in USD",
            "sampleValues": [50000, 120000, 5000000],
            "sensitive": false
          }
        ],
        "relationships": [
          { "field": "id", "relatesTo": "orders.customer_id", "type": "one_to_many" }
        ]
      }
    ],
    "lastDiscovered": "ISO 8601",
    "autoRefresh": true
  }
}
```

### 5C. Action Templates Extension — NEW

Predefined safe operations for the LLM to invoke.

```json
{
  "actionTemplates": [
    {
      "templateId": "update-ticket-status",
      "name": "Update Ticket Status",
      "description": "Changes the status of a support ticket",
      "queryTemplate": "UPDATE tickets SET status = :status WHERE ticket_id = :ticket_id",
      "requiredParams": ["ticket_id", "status"],
      "validation": { "status": ["open", "in_progress", "resolved", "closed"] },
      "requiresApproval": false,
      "rollbackCapable": true
    }
  ]
}
```

### 5D. Data Change Events Extension — NEW

Watch for data changes and fire triggers.

```json
{
  "dataChangeEvents": [
    {
      "eventId": "evt-1",
      "table": "orders",
      "watchFields": ["status"],
      "eventTypes": ["insert", "update"],
      "filter": "status == 'urgent'",
      "emitTo": "trigger-id"
    }
  ]
}
```

### 5E. Supported Source Types (9)

| Type | Driver | Protocol |
|------|--------|----------|
| postgresql | asyncpg | SQL |
| mysql | aiomysql | SQL |
| sqlserver | aioodbc | SQL/ODBC |
| mongodb | motor | NoSQL |
| redis | aioredis | Key-value |
| rest_api | httpx | HTTP |
| elasticsearch | AsyncElasticsearch | Search |
| milvus | pymilvus | Vector |
| file | (planned) | — |

### Storage

- Stored in **Redis** as JSON
- Key: `bexo:{org_id}:{app_id}:connections:{connection_id}`
- In-memory cache with 60s TTL

---

## Part 6: Integration Platform (APIs)

**Answers: What can they do with external systems?**

Currently, API configs are stored in Redis by BeX-platform-server. API steps reference them by `_id`. This part elevates them to a proper registry with richer metadata.

### 6A. API Config Schema (existing + extensions)

```json
{
  "apiConfigId": "string (unique ID)",
  "orgId": "string",
  "subOrgId": "string",
  "name": "string",
  "description": "string",
  "baseUrl": "https://api.example.com",
  "authType": "none | basic | bearer | api_key | oauth2",
  "authConfig": {},
  "defaultHeaders": {},
  "timeout": 30,
  "rateLimits": {
    "requestsPerMinute": 60,
    "requestsPerDay": 10000
  },
  "healthCheckEndpoint": "/health",
  "endpoints": [
    {
      "endpointId": "ep-1",
      "path": "/customers/{id}",
      "method": "GET",
      "description": "Retrieve customer details by ID",
      "pathParams": [
        { "name": "id", "type": "string", "description": "Customer ID" }
      ],
      "queryParams": [
        { "name": "include", "type": "string", "required": false, "description": "Related objects to include" }
      ],
      "requestBody": null,
      "responseSchema": {
        "id": "string — unique customer identifier",
        "name": "string — full legal name",
        "tier": "string — bronze | silver | gold | platinum"
      },
      "errorMappings": {
        "404": "Customer not found",
        "403": "Insufficient permissions"
      }
    }
  ],
  "status": "active | inactive",
  "createdBy": "string",
  "updatedBy": "string",
  "created": "ISO 8601",
  "updated": "ISO 8601"
}
```

**Current Redis key pattern:** `api:{org_id}:{sub_org_id}:{api_id}`

| Field | Type | Notes |
|-------|------|-------|
| endpoints | array | **NEW** — typed endpoint definitions with business-friendly descriptions |
| endpoints[].responseSchema | object | Field descriptions so LLM understands what data means |
| endpoints[].errorMappings | object | How to interpret error codes |
| rateLimits | object | **NEW** — prevent the employee from overwhelming the API |
| healthCheckEndpoint | string | **NEW** — for connection testing |

---

## Part 7: Channel Platform

**Answers: How do they communicate?**

**Source project:** `/home/mohammad/Public/CoolRiots/new/BeX-FEO-V3`

### 7A. Core Schema — LOCKED (no changes)

```json
{
  "_id": "string",
  "organizationId": "string",
  "subOrganizationId": "string",
  "assistantId": "string",
  "address": "string",
  "type": "Voice | Whatsapp | Messenger | Email | Webchat",
  "access_token": "string (optional)"
}
```

**No fields added to the channel entity.** Channel-type-specific behavior is handled by FEO's transport layer.

### 7B. Channel Types (5)

| Type | Transport | Real-time | How it works |
|------|-----------|-----------|-------------|
| Voice | Twilio + WebSocket | Yes | STT (Groq Whisper) → assistant → TTS (ElevenLabs) |
| Whatsapp | Twilio webhook | No | Text + media. Audio auto-transcribed |
| Messenger | Twilio webhook | No | Facebook Messenger via Twilio |
| Email | Microsoft Graph API | No | Webhook → read → assistant → reply |
| Webchat | WebSocket | Yes | Browser widget. Streaming responses |

### 7C. Message Flow

```
User sends message → FEO receives (webhook/WebSocket)
  → Look up channel by address in Redis
  → Extract: assistantId, orgId, subOrgId
  → Call Project-O execute API
  → Receive response → Send back through same channel
```

### 7D. Outbound Configuration (conceptual — NEW)

Outbound behavior is currently hard-coded in FEO per channel type. For the Digital Employee vision, this should become configurable:

```json
{
  "outboundConfig": {
    "formattingRules": {
      "Webchat": { "format": "markdown", "maxLength": null },
      "Whatsapp": { "format": "plain_text", "maxLength": 4096 },
      "Email": { "format": "html", "maxLength": null },
      "Voice": { "format": "ssml", "maxLength": null }
    },
    "businessHours": {
      "enabled": false,
      "timezone": "UTC",
      "hours": { "start": "09:00", "end": "18:00" },
      "outsideHoursAction": "queue | reject | auto_reply"
    },
    "deliveryConfirmation": false
  }
}
```

**Note:** This config lives on the Employee entity (not the channel), since it's the employee's behavior, not the channel's identity.

### 7E. Conversation & Session Management

FEO manages conversations in Redis with 3 layers:

| Layer | Redis Key Pattern | TTL |
|-------|-------------------|-----|
| Conversation | `conversation:{orgId}:{subOrgId}:{address}:{id}` | Persistent |
| Session | `conversationSession:{orgId}:{subOrgId}:{address}:{sessionId}` | 15 min |
| Tracking | `conversationTrackingSession:{orgId}:{subOrgId}:{address}` | 15 min |

### 7F. Voice Processing

| Direction | Provider | Model |
|-----------|----------|-------|
| STT | Groq | whisper-large-v3-turbo |
| STT fallback | Microsoft Cognitive Services | — |
| TTS | ElevenLabs | configurable voice ID |

### Storage

- Stored in **Redis** as JSON
- Key: `channel:{orgId}:{subOrgId}:{address}`
- CRUD managed by BeX-platform-server

---

## Part 8: Trigger & Event Platform

**Answers: When do they act without being asked?**

Without this, you only have a reactive assistant, not a proactive digital employee.

### 8A. Trigger Schema — NEW

Triggers are standalone entities referenced by the Employee's `triggers` array.

```json
{
  "triggerId": "string (unique ID)",
  "orgId": "string",
  "subOrgId": "string",
  "assistantId": "string",
  "name": "string",
  "description": "string",
  "enabled": true,
  "type": "schedule | event | threshold | agent",

  "scheduleConfig": {
    "cron": "0 9 * * MON-FRI",
    "timezone": "America/New_York",
    "opcodeId": "daily-report",
    "inputContext": { "report_type": "daily_summary" },
    "overlapPolicy": "skip | queue | parallel"
  },

  "eventConfig": {
    "source": "webhook | data_change | knowledge_update | channel_message",
    "filter": { "event": "order.created", "priority": "high" },
    "debounceMs": 5000,
    "opcodeId": "handle-new-order",
    "inputMapping": { "order_id": "{{ event.data.id }}" }
  },

  "thresholdConfig": {
    "metric": "queue_size",
    "condition": "> 50",
    "evaluationFrequency": "60s",
    "opcodeId": "scale-alert"
  },

  "agentConfig": {
    "fromAssistantId": "asst-2",
    "protocol": "request_response | delegate",
    "opcodeId": "handle-delegation"
  },

  "safetyLimits": {
    "maxFiresPerHour": 100,
    "maxFiresPerDay": 1000,
    "cooldownMs": 1000
  },

  "lastFired": "ISO 8601",
  "fireCount": 0,
  "status": "active | paused | disabled",
  "createdBy": "string",
  "updatedBy": "string",
  "created": "ISO 8601",
  "updated": "ISO 8601"
}
```

| Trigger Type | How It Works |
|-------------|-------------|
| **schedule** | Cron-based — "run daily report at 9am" |
| **event** | External webhook or internal data change — "new order created" |
| **threshold** | Metric monitoring — "queue size exceeds 50" |
| **agent** | Another employee sends a request — multi-agent activation |

---

## Part 9: Orchestration Layer

**Answers: How do multiple employees coordinate?**

### 9A. Delegation Rules (on Employee entity)

```json
{
  "orchestration": {
    "delegationRules": {
      "maxDelegationDepth": 3,
      "allowedDelegates": ["asst-sales", "asst-billing"],
      "contextTransferPolicy": "full | summary | minimal",
      "onDelegateFailure": "retry | fallback_to_self | escalate",
      "timeout": "30s"
    },
    "interEmployeeCommunication": {
      "messageContract": {
        "from": "assistantId",
        "to": "assistantId",
        "type": "request | response | broadcast | delegate",
        "payload": {},
        "correlationId": "string",
        "timestamp": "ISO 8601"
      }
    }
  }
}
```

### 9B. Multi-Agent Patterns

| Pattern | Description |
|---------|-------------|
| **Request-Response** | Employee A asks Employee B a question, gets answer back |
| **Delegate** | Employee A hands off entire task to Employee B |
| **Broadcast** | Employee A notifies all connected employees |
| **Pipeline** | Employee A → Employee B → Employee C (sequential handoff) |

### 9C. Execution Graph (future)

Ability to define cross-employee workflows as a higher-order graph:

```json
{
  "executionGraph": {
    "nodes": [
      { "id": "n1", "assistantId": "asst-intake", "opcodeId": "classify" },
      { "id": "n2", "assistantId": "asst-billing", "opcodeId": "process-billing" },
      { "id": "n3", "assistantId": "asst-support", "opcodeId": "resolve-ticket" }
    ],
    "edges": [
      { "from": "n1", "to": "n2", "condition": "classification == 'billing'" },
      { "from": "n1", "to": "n3", "condition": "classification == 'support'" }
    ]
  }
}
```

---

## Part 10: Dynamic UI Platform

**Answers: How do they present complex information and collect structured input?**

This is the graphical-conversational synergy layer (BeX3D). Skills can produce UI specifications that the platform renders.

### 10A. UI Component Registry

Available component types:

| Component | Data In | User Interaction |
|-----------|---------|-----------------|
| Table | array of objects | Sort, filter, select row |
| Card | object | Click, expand |
| Form | field definitions | Fill and submit |
| Chart | datasets + config | Hover, zoom |
| Timeline | events array | Scroll, click event |
| Approval Panel | action + context | Approve / reject |

### 10B. UI Specification Contract

Skills produce a UI spec as part of their output:

```json
{
  "uiSpec": {
    "type": "table",
    "title": "Recent Orders",
    "data": [
      { "orderId": "ORD-123", "customer": "Acme Corp", "amount": 5000, "status": "pending" }
    ],
    "columns": [
      { "field": "orderId", "label": "Order ID", "sortable": true },
      { "field": "customer", "label": "Customer" },
      { "field": "amount", "label": "Amount", "format": "currency" },
      { "field": "status", "label": "Status", "format": "badge" }
    ],
    "actions": [
      { "label": "Approve", "opcodeId": "approve-order", "params": { "orderId": "{{ row.orderId }}" } }
    ]
  }
}
```

When the user clicks "Approve", it feeds back as a new opcode execution — closing the loop between graphical and conversational.

### 10C. Design Principle

The employee decides *what* to show. The UI platform decides *how* to render it. The channel determines *where* (web widget, full portal, embedded iframe). This separation allows one skill to work across multiple surfaces.

---

## Part 11: Observability Platform

**Answers: What happened, how well, and what should we improve?**

Non-negotiable for enterprise. Without it, you have a black box.

### 11A. Execution Log Schema — NEW

Every opcode execution produces a log entry:

```json
{
  "executionLogId": "string",
  "orgId": "string",
  "subOrgId": "string",
  "assistantId": "string",
  "opcodeId": "string",
  "triggerSource": "channel | schedule | event | agent | manual",
  "triggerDetails": { "channelId": "ch-1", "userId": "user-123" },
  "input": { "user_input": "..." },
  "output": { "result": "..." },
  "steps": [
    {
      "stepId": "step-uuid",
      "type": "LLM",
      "name": "Generate Response",
      "startTime": "ISO 8601",
      "endTime": "ISO 8601",
      "durationMs": 1200,
      "status": "success | error",
      "input": {},
      "output": {},
      "confidenceScore": 0.92,
      "tokenUsage": { "input": 500, "output": 200 },
      "cost": 0.003,
      "error": null
    }
  ],
  "totalDurationMs": 3500,
  "totalCost": 0.008,
  "totalTokens": { "input": 1200, "output": 400 },
  "status": "success | error | escalated",
  "evaluationScore": 0.88,
  "userFeedback": { "rating": 5, "comment": null },
  "timestamp": "ISO 8601"
}
```

### 11B. Metrics (aggregated)

| Metric | Aggregation | Purpose |
|--------|-------------|---------|
| Response time (p50, p95, p99) | Per skill | Performance monitoring |
| Success/failure rate | Per skill, per employee | Reliability |
| Cost per execution | Per skill, per employee | Budget control |
| Escalation rate | Per employee | Autonomy effectiveness |
| Satisfaction score | Per employee | Quality tracking |
| Token usage | Per skill, per employee | Resource planning |

### 11C. Alerting

```json
{
  "alerts": [
    { "metric": "error_rate", "condition": "> 0.1", "window": "1h", "notify": "admin" },
    { "metric": "cost_daily", "condition": "> 100", "window": "1d", "notify": "admin" },
    { "metric": "p95_latency", "condition": "> 10000", "window": "1h", "notify": "admin" }
  ]
}
```

### 11D. Audit Trail

Immutable log of all actions — especially write actions, API calls, data changes.

- **Cannot be disabled in production**
- PII handling: configurable masking rules
- Retention: configurable per entity type
- Required for enterprise compliance

---

## Part 12: Governance & Compliance

**Answers: Can we trust this employee in a regulated enterprise environment?**

This is what separates a product enterprises will buy from a demo.

### 12A. Access Control

```json
{
  "accessControl": {
    "rbac": {
      "roles": ["admin", "builder", "operator", "viewer"],
      "permissions": {
        "admin": ["*"],
        "builder": ["employee.create", "employee.edit", "opcode.create", "opcode.edit", "collection.manage"],
        "operator": ["employee.view", "employee.activate", "employee.deactivate", "logs.view"],
        "viewer": ["employee.view", "logs.view"]
      }
    },
    "orgIsolation": true,
    "subOrgIsolation": true
  }
}
```

### 12B. Content Governance

```json
{
  "contentGovernance": {
    "inputFiltering": {
      "enabled": true,
      "blockedPatterns": ["credit_card_number", "ssn"],
      "maxInputLength": 10000
    },
    "outputFiltering": {
      "enabled": true,
      "blockedTopics": [],
      "piiDetection": true,
      "piiAction": "mask | block | warn"
    },
    "prohibitedTopics": ["competitor_recommendations", "medical_advice", "legal_advice"]
  }
}
```

### 12C. Compliance Configuration

```json
{
  "compliance": {
    "dataResidency": "us-east | eu-west | ap-southeast",
    "retentionPolicy": {
      "conversationHistory": "90d",
      "executionLogs": "365d",
      "auditTrail": "7y"
    },
    "rightToDeletion": {
      "enabled": true,
      "scope": "user_data | all_interactions"
    },
    "consentTracking": {
      "enabled": true,
      "requireConsentBefore": "first_interaction"
    }
  }
}
```

### 12D. Change Management

```json
{
  "changeManagement": {
    "environments": ["development", "staging", "production"],
    "promotionRequiresApproval": true,
    "approvers": ["admin-user-id"],
    "mandatoryTestGates": true,
    "rollbackEnabled": true
  }
}
```

### 12E. Risk Classification

| Risk Level | Criteria | Requirements |
|------------|----------|-------------|
| Low | Read-only, no external calls | Standard logging |
| Medium | External API reads, user-facing responses | Evaluation gate, sample review |
| High | Write actions, financial data, PII handling | Approval workflow, full audit, human review |

---

## Cross-Cutting Concerns

### Naming Convention Reconciliation

| System | Convention | Examples |
|--------|-----------|---------|
| Employee (Assistant) | camelCase | orgId, subOrgId, assistantId |
| Opcodes/Steps | snake_case | org_id, sub_org_id, opcode_id |
| Collections | full camelCase | organizationId, subOrganizationId |
| Connections (BeXO) | snake_case | org_id, app_id, connection_id |
| Channels | full camelCase | organizationId, subOrganizationId |

**Decision for unified model:** All **new** fields use camelCase. All **existing** fields preserved as-is (additive only). API layer handles translation.

### Multi-Tenancy Pattern

```
Organization (orgId)
  └── Sub-Organization (subOrgId)
      ├── Employees (assistants)
      ├── Opcodes (skills)
      ├── Collections
      ├── Channels
      ├── Triggers
      ├── Model Configs
      ├── Instructions
      └── API Configs

BeXO uses a separate hierarchy:
  Organization (org_id)
    └── Application (app_id)  ← NOT the same as subOrgId
        └── Connections
```

### Versioning Strategy

| Entity | Versioned | How |
|--------|-----------|-----|
| Employee | Yes | `version` field, incremented on edit |
| Opcode | Yes | `version` field |
| Instruction | Yes | `version` field, enables rollback |
| Collection | No | Immutable config, content changes tracked by ingestion |
| Connection | No | Config changes, not versioned |
| Channel | No | Simple config |
| Trigger | No | Config changes |

---

## Entity Relationship Summary

### All Entities

| # | Entity | Storage | Standalone | Key Identifier |
|---|--------|---------|-----------|---------------|
| 1 | Employee (Assistant) | Redis/DB | Yes | assistantId |
| 2 | Opcode (Skill) | Redis | Yes | opcode_id |
| 3 | Step | — | No (embedded in Opcode) | id (UUID) |
| 4 | Model Config | Redis | Yes | modelConfigId |
| 5 | Instruction | Redis/DB | Yes | instructionId |
| 6 | Collection | MongoDB | Yes | _id |
| 7 | Connection | Redis | Yes | connection_id |
| 8 | API Config | Redis | Yes | apiConfigId |
| 9 | Channel | Redis | Yes | _id |
| 10 | Trigger | DB | Yes | triggerId |
| 11 | Execution Log | DB | Yes | executionLogId |

### Reference Map

```
Employee
  ├── opcodes[]         → Opcode.opcode_id
  ├── defaultOpcode     → Opcode.opcode_id
  ├── channels[]        → Channel._id
  ├── collections[]     → Collection._id
  ├── databases[]       → Connection.connection_id
  ├── triggers[]        → Trigger.triggerId
  ├── connectedAssistants[].assistantId → Employee.assistantId
  ├── router.modelConfigId → ModelConfig.modelConfigId
  ├── router.instructionId → Instruction.instructionId
  ├── evaluation.modelConfigId → ModelConfig.modelConfigId
  ├── memory.knowledgeMemory.collectionId → Collection._id
  ├── onboarding.shadowAssistantId → Employee.assistantId
  └── supervision.supervisorId → User or Employee

Opcode
  └── steps[].input_params
      ├── unique_name   → ModelConfig.uniqueName (LLM steps)
      ├── _id           → ApiConfig.apiConfigId (API steps)
      └── collection_id → Collection._id (Non-LLM RAG steps)

Trigger
  ├── assistantId       → Employee.assistantId
  ├── opcodeId          → Opcode.opcode_id
  └── agentConfig.fromAssistantId → Employee.assistantId

Execution Log
  ├── assistantId       → Employee.assistantId
  └── opcodeId          → Opcode.opcode_id
```

---

## Implementation Priority (from Jafar)

> The components that must be deep and robust from day one for enterprise buyers are: **Employee Management, Skill Registry, Intelligence Core, Knowledge Platform, Channel Platform, and Governance.** The rest can be phased — Orchestration and Dynamic UI especially can be lighter in v1 without losing the core value proposition. Observability is often skipped early and always regretted. Don't skip it.

### Suggested Phasing

| Phase | Components | Rationale |
|-------|-----------|-----------|
| **v1 — Core** | Employee, Opcodes, Intelligence Core, Collections, Channels, Observability | The working system — can already replace a human for defined tasks |
| **v2 — Autonomy** | Triggers, Connections (enriched), Governance, Supervision | Makes it proactive and enterprise-trustworthy |
| **v3 — Collaboration** | Orchestration, Connected Assistants, Handoff | Multi-agent and human-AI collaboration |
| **v4 — Intelligence** | Dynamic UI, Self-improving Memory, Emotional Intelligence | The "wow" features that differentiate |
