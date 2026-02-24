# CoolRiots Digital Employee Platform — Unified Data Model

> Generated from SYSTEM_DESIGN.md. Every entity below is a complete JSON schema.
> Fields marked `[LOCKED]` exist in current implementation — cannot be removed or renamed.
> Fields marked `[NEW]` are additions for the Digital Employee vision.

---

## Entity 1: Employee

The top-level entity. Everything else belongs to or serves an employee.

```json
{
  // ─── Identity (LOCKED) ───
  "assistantId": "string",
  "title": "string",
  "description": "string",
  "orgId": "string",
  "subOrgId": "string",
  "createdBy": "string",
  "updatedBy": "string",

  // ─── References (LOCKED) ───
  "opcodes": ["string"],
  "defaultOpcode": "string",
  "channels": ["string"],
  "collections": ["string"],
  "databases": ["string"],

  // ─── Streaming (LOCKED) ───
  "stream": false,
  "streamCumulative": false,
  "emitTokenEvents": false,
  "streamBufferSentences": false,

  // ─── Versioning (LOCKED) ───
  "version": 1,
  "status": "active | inactive",
  "disabled": false,
  "created": "ISO 8601",
  "updated": "ISO 8601",

  // ─── Persona (NEW) ───
  "persona": {
    "systemPrompt": "string",
    "role": "string",
    "domainExpertise": ["string"],
    "tone": "professional | friendly | formal | concise",
    "language": "string",
    "secondaryLanguages": ["string"],
    "fallbackBehavior": "apologize_and_escalate | use_default_opcode | reject"
  },

  // ─── Autonomy (NEW) ───
  "autonomy": {
    "level": "supervised | semi-autonomous | autonomous",
    "escalation": {
      "onConfidenceBelow": 0.6,
      "escalateTo": "string (user-id or assistant-id)",
      "maxConsecutiveAutonomousActions": 50,
      "requireApprovalFor": ["string"]
    },
    "boundaries": {
      "mustNever": ["string"],
      "mustAlways": ["string"],
      "allowedOpcodes": ["string"],
      "blockedFunctions": ["string"],
      "maxDailyExecutions": 1000
    }
  },

  // ─── Goals (NEW) ───
  "goals": [
    {
      "goalId": "string",
      "metric": "string",
      "target": 0.9,
      "measurement": "string",
      "window": "string (e.g. 7d, 30d)"
    }
  ],

  // ─── Memory (NEW) ───
  "memory": {
    "conversationHistory": {
      "enabled": true,
      "maxTurns": 100,
      "ttl": "string (e.g. 30d)"
    },
    "knowledgeMemory": {
      "enabled": false,
      "collectionId": "string (references Collection._id)",
      "autoLearn": false,
      "learnFrom": ["successful_interactions", "corrections", "feedback"]
    },
    "workingMemory": {
      "enabled": false,
      "persist": false,
      "scope": "per_user | global"
    }
  },

  // ─── Operational Context (NEW — V1) ───
  "operationalContext": {
    "currentObjective": "string",
    "activePlan": "string",
    "lastStatus": "string",
    "customFields": {},
    "writableByEmployee": false
  },

  // ─── State Machine (NEW — V3) ───
  "state": {
    "enabled": false,
    "currentState": "string",
    "transitions": {
      "<state_name>": {
        "on": "string (event name)",
        "next": "string (next state name)"
      }
    },
    "data": {}
  },

  // ─── Task Queue (NEW) ───
  "taskQueue": {
    "enabled": false,
    "maxConcurrent": 5,
    "priorityRules": [
      {
        "condition": "string (expression)",
        "priority": "critical | high | normal | low"
      }
    ],
    "overflow": "queue | reject | delegate",
    "maxQueueSize": 100
  },

  // ─── Onboarding (NEW) ───
  "onboarding": {
    "status": "training | probation | active | veteran",
    "shadowAssistantId": "string (references Employee.assistantId)",
    "graduationCriteria": {
      "minInteractions": 100,
      "minConfidence": 0.85,
      "minSatisfactionScore": 0.8
    },
    "autoGraduate": true
  },

  // ─── Supervision (NEW) ───
  "supervision": {
    "mode": "shadow | supervised | independent",
    "supervisorId": "string (user-id or assistant-id)",
    "reviewQueue": {
      "sampleRate": 0.1,
      "alwaysReviewWhen": ["string (expression)"]
    },
    "feedbackIntegration": {
      "source": "supervisor_ratings | customer_ratings | outcome_tracking",
      "autoAdjust": false
    }
  },

  // ─── Emotional Intelligence (NEW) ───
  "emotionalIntelligence": {
    "enabled": false,
    "detectSentiment": false,
    "adaptTone": false,
    "escalateOnFrustration": false,
    "frustrationThreshold": 3
  },

  // ─── Handoff (NEW) ───
  "handoff": {
    "toHuman": {
      "enabled": false,
      "method": "live_transfer | ticket | email",
      "contextTransfer": true,
      "summaryBeforeHandoff": true
    },
    "toAssistant": {
      "enabled": false,
      "rules": [
        {
          "condition": "string (expression)",
          "targetAssistantId": "string"
        }
      ]
    },
    "resumeAfterHandoff": false
  },

  // ─── Interrupt Handling (NEW) ───
  "interruptHandling": {
    "enabled": false,
    "pauseCurrentOnPriorityCritical": true,
    "maxPausedTasks": 3,
    "autoResumeAfter": "string (e.g. 300s)"
  },

  // ─── Triggers (NEW) ───
  "triggers": ["string (references Trigger.triggerId)"],

  // ─── Connected Assistants (NEW) ───
  "connectedAssistants": [
    {
      "assistantId": "string (references Employee.assistantId)",
      "role": "string (e.g. specialist, supervisor, peer)",
      "capabilities": ["string"],
      "protocol": "request_response | delegate | broadcast"
    }
  ],

  // ─── Router (NEW) ───
  "router": {
    "enabled": false,
    "modelConfigId": "string (references ModelConfig.modelConfigId)",
    "instructionId": "string (references Instruction.instructionId)",
    "skillManifest": "auto | manual",
    "confidenceThreshold": 0.7,
    "onLowConfidence": "ask_clarification | use_default_opcode | escalate",
    "onNoMatch": "use_default_opcode | escalate | reject"
  },

  // ─── Evaluation (NEW) ───
  "evaluation": {
    "enabled": false,
    "modelConfigId": "string (references ModelConfig.modelConfigId)",
    "criteria": ["relevance", "accuracy", "safety", "format_compliance"],
    "scoreThreshold": 0.8,
    "onFailure": "retry | escalate | deliver_with_warning",
    "maxRetries": 2
  },

  // ─── Outbound Config (NEW) ───
  "outboundConfig": {
    "formattingRules": {
      "<channel_type>": {
        "format": "markdown | plain_text | html | ssml",
        "maxLength": null
      }
    },
    "businessHours": {
      "enabled": false,
      "timezone": "UTC",
      "hours": { "start": "09:00", "end": "18:00" },
      "outsideHoursAction": "queue | reject | auto_reply"
    },
    "deliveryConfirmation": false
  },

  // ─── Orchestration (NEW) ───
  "orchestration": {
    "delegationRules": {
      "maxDelegationDepth": 3,
      "allowedDelegates": ["string (assistant-ids)"],
      "contextTransferPolicy": "full | summary | minimal",
      "onDelegateFailure": "retry | fallback_to_self | escalate",
      "timeout": "string (e.g. 30s)"
    }
  },

  // ─── Reporting (NEW) ───
  "reporting": {
    "dailySummary": {
      "enabled": false,
      "sendTo": "string (email or user-id)",
      "channel": "string",
      "includeMetrics": ["string"]
    },
    "alerts": [
      {
        "condition": "string (expression)",
        "notify": "string (user-id or role)",
        "channel": "string"
      }
    ]
  }
}
```

### Field Count Summary

| Section | Fields | Status | Version |
|---------|--------|--------|---------|
| Identity | 7 | LOCKED | V1 |
| References | 5 | LOCKED | V1 |
| Streaming | 4 | LOCKED | V1 |
| Versioning | 4 | LOCKED | V1 |
| Persona | 1 (nested) | NEW | V1 |
| Autonomy | 1 (nested) | NEW | V1 |
| Operational Context | 1 (nested) | NEW | V1 (operator-write), V2 (employee self-write) |
| Router | 1 (nested) | NEW | V1 |
| Memory | 1 (nested) | NEW | V1 (basic), V2+ (knowledge, working) |
| Handoff | 1 (nested) | NEW | V1 (human only), V4 (to assistant) |
| Outbound Config | 1 (nested) | NEW | V1 (formatting), V2 (business hours) |
| Evaluation | 1 (nested) | NEW | V2 (stored in V1, not enforced) |
| Goals | 1 (array) | NEW | V2 |
| Triggers | 1 (array) | NEW | V2 |
| Reporting | 1 (nested) | NEW | V2 |
| State Machine | 1 (nested) | NEW | V3 |
| Task Queue | 1 (nested) | NEW | V3 |
| Onboarding | 1 (nested) | NEW | V3 |
| Supervision | 1 (nested) | NEW | V3 |
| Emotional Intelligence | 1 (nested) | NEW | V3 |
| Interrupt Handling | 1 (nested) | NEW | V3 |
| Connected Assistants | 1 (array) | NEW | V4 |
| Orchestration | 1 (nested) | NEW | V4 |
| **Total top-level keys** | **37** | 20 locked + 17 new |

---

## Entity 2: Opcode (Skill)

An executable pipeline of steps. Referenced by Employee.opcodes[].

```json
{
  // ─── Core (LOCKED) ───
  "opcode_id": "string",
  "type": "ChatOp | AutoOp",
  "version": "string",
  "description": "string [NEW]",
  "steps": ["(see Entity 3: Step)"],

  // ─── Skill Metadata (NEW) ───
  "category": ["string"],
  "executionProfile": {
    "interactive": false,
    "durationClass": "real-time | short | long",
    "parallelizable": true,
    "idempotent": false
  },
  "inputContract": {
    "required": ["string"],
    "optional": ["string"],
    "contextDependencies": ["string"],
    "requiredCollections": ["string"],
    "requiredConnections": ["string"]
  },
  "outputContract": {
    "returns": {
      "<key>": "string (type description)"
    },
    "producesContextPatch": false,
    "producesUiSpec": false,
    "compatibleChannels": ["string"]
  },
  "quality": {
    "evaluateBeforeDelivery": false,
    "maxTokenBudget": 10000,
    "maxCostBudget": 0.05,
    "retryPolicy": {
      "maxRetries": 2,
      "backoffMs": 1000
    }
  }
}
```

---

## Entity 3: Step (embedded in Opcode)

Not a standalone entity — always embedded in an Opcode's `steps` array.

```json
{
  // ─── Core (LOCKED) ───
  "id": "string (UUID)",
  "type": "LLM | Non-LLM | RAG | API | Memory | Condition | Loop | Parallel | Transform | Wait | Choose | Subflow",
  "name": "string [NEW]",
  "description": "string [NEW]",
  "input_params": {},
  "output_params": {},
  "routing": {
    "next": ["string"],
    "show_output": true,
    "accumulate_output": true,
    "true_branch": ["string"],
    "false_branch": ["string"]
  },

  // ─── Type-specific (LOCKED — only the relevant one is present) ───
  "loop_steps": [],
  "parallel_steps": []
}
```

### Step input_params by type

**LLM:**
```json
{
  "org_id": "string",
  "sub_org_id": "string",
  "unique_name": "string → references ModelConfig.uniqueName",
  "query": "string (Jinja2 template)",
  "model_type": "string",
  "stream": false,
  "placeholder_values": {},
  "image_urls": []
}
```

**Non-LLM (includes RAG):**
```json
{
  "function_call": {
    "function_name": "string (e.g. RAG_API)",
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
      "search_type": "vector | keyword | hybrid",
      "offset": 0
    }
  }
}
```

**API:**
```json
{
  "_id": "string → references ApiConfig.apiConfigId",
  "organizationId": "string",
  "subOrganizationId": "string",
  "query_params": {},
  "body": {},
  "auth": "string",
  "headers": {},
  "array_of_dicts_mode": false
}
```

**Memory:**
```json
{
  "n": 3,
  "limit_size": 3000,
  "org_id": "string",
  "sub_org_id": "string"
}
```

**Condition:**
```json
{
  "expression": "string (OEL expression)"
}
```

**Loop:**
```json
{
  "mode": "range | collection | repeat-until",
  "range_start": 0,
  "range_end": 5,
  "range_step": 1,
  "item_alias": "string",
  "collection": "string (Jinja2 template)"
}
```

**Transform:**
```json
{
  "expression": "string"
}
```

**Wait:**
```json
{
  "duration": 1000
}
```

**RAG [NEW — alias for Non-LLM]:**
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
      "search_type": "vector | keyword | hybrid",
      "offset": 0
    }
  }
}
```

**Choose [NEW — V1]:**
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

**Subflow [NEW — V1]:**
```json
{
  "opcode_id": "string → references Opcode.opcode_id",
  "inputMapping": {
    "<subflow_input_key>": "string (Jinja2 template from parent context)"
  }
}
```

### Step output_params by type

**LLM:**
```json
{
  "output_name": "string",
  "output_type": "string | dict | list | number",
  "extra_output": [{ "key": "string", "value": "string" }]
}
```

**Non-LLM / API:**
```json
{
  "output_names": { "<local_key>": "<source_key>" }
}
```

**Memory:**
```json
{
  "output_name": "string"
}
```

**Choose [NEW]:**
```json
{
  "output_names": { "<local_key>": "<source_key>" }
}
```
Output comes from whichever branch executed.

**Subflow [NEW]:**
```json
{
  "outputMapping": {
    "<parent_context_key>": "<subflow_output_key>"
  }
}
```

---

## Entity 4: Model Config

LLM model available to the system. Referenced by LLM steps via `unique_name`.

```json
{
  "modelConfigId": "string",
  "orgId": "string",
  "subOrgId": "string",
  "uniqueName": "string",
  "provider": "groq | ibm | sambanova | openai | anthropic",
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

**Storage:** Redis — key: `{Provider}Model:{orgId}:{subOrgId}:{uniqueName}`

---

## Entity 5: Instruction

Reusable prompt template paired with model parameters. Version-controlled.

```json
{
  "instructionId": "string",
  "orgId": "string",
  "subOrgId": "string",
  "name": "string",
  "description": "string",
  "systemPrompt": "string",
  "promptTemplate": "string (Jinja2 with {{ placeholders }})",
  "outputFormat": "string | json | list",
  "temperature": 0.7,
  "topP": 1.0,
  "maxTokens": 4096,
  "modelConfigId": "string → references ModelConfig.modelConfigId",
  "version": 1,
  "testCases": [
    {
      "input": {},
      "expectedOutputContains": ["string"],
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

**Storage:** Redis/DB — key pattern TBD

---

## Entity 6: Collection

Knowledge base for RAG. Referenced by Employee.collections[] and Non-LLM steps.

```json
{
  // ─── Core (LOCKED) ───
  "_id": "string",
  "name": "string",
  "organizationId": "string",
  "subOrganizationId": "string",
  "dimensions": 1024,
  "vectorStoreType": "MongoDB | Milvus",
  "chunkingStrategy": "string",
  "embeddingModel": "string",
  "created": "ISO 8601",
  "updated": "ISO 8601",

  // ─── Additions (NEW — locked into design) ───
  "description": "string",
  "status": "active | inactive",
  "createdBy": "string",
  "updatedBy": "string",

  // ─── Extensions (NEW) ───
  "accessControl": {
    "allowedAssistants": ["string"],
    "sensitivityLevel": "public | internal | confidential | restricted"
  },
  "freshness": {
    "retentionDays": 365,
    "reIngestionSchedule": "manual | daily | weekly",
    "lastIngested": "ISO 8601"
  },
  "language": "string",
  "tags": ["string"]
}
```

**Storage:** MongoDB — managed by KB service

---

## Entity 7: Connection

External data source bridge. Referenced by Employee.databases[].

```json
{
  // ─── Core (LOCKED) ───
  "org_id": "string",
  "app_id": "string",
  "connection_id": "string",
  "connection_name": "string",
  "description": "string",
  "source_type": "postgresql | mysql | sqlserver | mongodb | redis | rest_api | file | milvus | elastic",
  "settings": {},
  "advanced_options": {
    "retry_attempts": 0,
    "retry_backoff_secs": 0,
    "enable_logging": false,
    "connection_pool_size": null
  },
  "created_at": "ISO 8601",
  "updated_at": "ISO 8601",

  // ─── Additions (NEW — locked into design) ───
  "createdBy": "string",
  "updatedBy": "string",

  // ─── Schema Discovery (NEW) ───
  "schemaDiscovery": {
    "enabled": false,
    "tables": [
      {
        "name": "string",
        "businessDescription": "string",
        "fields": [
          {
            "name": "string",
            "type": "string",
            "businessName": "string",
            "businessDescription": "string",
            "sampleValues": [],
            "sensitive": false
          }
        ],
        "relationships": [
          {
            "field": "string",
            "relatesTo": "string (table.field)",
            "type": "one_to_one | one_to_many | many_to_many"
          }
        ]
      }
    ],
    "lastDiscovered": "ISO 8601",
    "autoRefresh": false
  },

  // ─── Action Templates (NEW) ───
  "actionTemplates": [
    {
      "templateId": "string",
      "name": "string",
      "description": "string",
      "queryTemplate": "string (parameterized query)",
      "requiredParams": ["string"],
      "validation": {},
      "requiresApproval": false,
      "rollbackCapable": false
    }
  ],

  // ─── Data Change Events (NEW) ───
  "dataChangeEvents": [
    {
      "eventId": "string",
      "table": "string",
      "watchFields": ["string"],
      "eventTypes": ["insert", "update", "delete"],
      "filter": "string (expression)",
      "emitTo": "string → references Trigger.triggerId"
    }
  ]
}
```

**Storage:** Redis — key: `bexo:{org_id}:{app_id}:connections:{connection_id}`

---

## Entity 8: API Config

External API configuration. Referenced by API steps via `_id`.

```json
{
  "apiConfigId": "string",
  "orgId": "string",
  "subOrgId": "string",
  "name": "string",
  "description": "string",
  "baseUrl": "string (URL)",
  "authType": "none | basic | bearer | api_key | oauth2",
  "authConfig": {},
  "defaultHeaders": {},
  "timeout": 30,
  "rateLimits": {
    "requestsPerMinute": 60,
    "requestsPerDay": 10000
  },
  "healthCheckEndpoint": "string",
  "endpoints": [
    {
      "endpointId": "string",
      "path": "string",
      "method": "GET | POST | PUT | PATCH | DELETE | HEAD",
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
  "status": "active | inactive",
  "createdBy": "string",
  "updatedBy": "string",
  "created": "ISO 8601",
  "updated": "ISO 8601"
}
```

**Storage:** Redis — key: `api:{orgId}:{subOrgId}:{apiConfigId}`

---

## Entity 9: Channel

Communication interface. Referenced by Employee.channels[]. **No changes — fully locked.**

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

**Storage:** Redis — key: `channel:{organizationId}:{subOrganizationId}:{address}`

---

## Entity 10: Trigger

Activation mechanism. Referenced by Employee.triggers[].

```json
{
  "triggerId": "string",
  "orgId": "string",
  "subOrgId": "string",
  "assistantId": "string → references Employee.assistantId",
  "name": "string",
  "description": "string",
  "enabled": true,
  "type": "schedule | event | threshold | agent",

  "scheduleConfig": {
    "cron": "string",
    "timezone": "string",
    "opcodeId": "string → references Opcode.opcode_id",
    "inputContext": {},
    "overlapPolicy": "skip | queue | parallel"
  },

  "eventConfig": {
    "source": "webhook | data_change | knowledge_update | channel_message",
    "filter": {},
    "debounceMs": 5000,
    "opcodeId": "string → references Opcode.opcode_id",
    "inputMapping": {}
  },

  "thresholdConfig": {
    "metric": "string",
    "condition": "string (expression)",
    "evaluationFrequency": "string (e.g. 60s)",
    "opcodeId": "string → references Opcode.opcode_id"
  },

  "agentConfig": {
    "fromAssistantId": "string → references Employee.assistantId",
    "protocol": "request_response | delegate",
    "opcodeId": "string → references Opcode.opcode_id"
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

**Note:** Only the config matching the `type` field is used (scheduleConfig for "schedule", eventConfig for "event", etc.)

**Storage:** DB (persistent)

---

## Entity 11: Execution Log

Audit record of every opcode execution. Write-once, immutable.

```json
{
  "executionLogId": "string",
  "orgId": "string",
  "subOrgId": "string",
  "assistantId": "string → references Employee.assistantId",
  "opcodeId": "string → references Opcode.opcode_id",
  "triggerSource": "channel | schedule | event | agent | manual",
  "triggerDetails": {
    "channelId": "string",
    "userId": "string",
    "triggerId": "string",
    "fromAssistantId": "string"
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
  "evaluationScore": null,
  "userFeedback": {
    "rating": null,
    "comment": null
  },
  "timestamp": "ISO 8601"
}
```

**Storage:** DB (persistent, immutable, indexed by assistantId + timestamp)

---

## Governance Configs (not standalone entities — org/sub-org level)

These are configurations at the organization or sub-organization level, not per-employee.

### Access Control (org level)

```json
{
  "orgId": "string",
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
```

### Content Governance (sub-org level)

```json
{
  "orgId": "string",
  "subOrgId": "string",
  "inputFiltering": {
    "enabled": true,
    "blockedPatterns": ["string"],
    "maxInputLength": 10000
  },
  "outputFiltering": {
    "enabled": true,
    "blockedTopics": ["string"],
    "piiDetection": true,
    "piiAction": "mask | block | warn"
  },
  "prohibitedTopics": ["string"]
}
```

### Compliance (org level)

```json
{
  "orgId": "string",
  "dataResidency": "string",
  "retentionPolicy": {
    "conversationHistory": "string (e.g. 90d)",
    "executionLogs": "string (e.g. 365d)",
    "auditTrail": "string (e.g. 7y)"
  },
  "rightToDeletion": {
    "enabled": true,
    "scope": "user_data | all_interactions"
  },
  "consentTracking": {
    "enabled": false,
    "requireConsentBefore": "first_interaction"
  }
}
```

### Change Management (org level)

```json
{
  "orgId": "string",
  "environments": ["development", "staging", "production"],
  "promotionRequiresApproval": true,
  "approvers": ["string"],
  "mandatoryTestGates": true,
  "rollbackEnabled": true
}
```

---

## Entity Reference Map (visual)

```
┌─────────────────────────────────────────────────────────────┐
│                        EMPLOYEE                             │
│  assistantId, title, orgId, subOrgId                        │
│                                                             │
│  ┌─ opcodes[] ──────────────► OPCODE (opcode_id)           │
│  │                             └─ steps[] ──► STEP          │
│  │                                  ├─ unique_name ──► MODEL CONFIG
│  │                                  ├─ _id ──────────► API CONFIG
│  │                                  └─ collection_id ► COLLECTION
│  │                                                          │
│  ├─ channels[] ─────────────► CHANNEL (_id)                 │
│  ├─ collections[] ──────────► COLLECTION (_id)              │
│  ├─ databases[] ────────────► CONNECTION (connection_id)    │
│  ├─ triggers[] ─────────────► TRIGGER (triggerId)           │
│  │                             └─ opcodeId ──► OPCODE       │
│  │                                                          │
│  ├─ router.modelConfigId ───► MODEL CONFIG                  │
│  ├─ router.instructionId ───► INSTRUCTION                   │
│  ├─ evaluation.modelConfigId► MODEL CONFIG                  │
│  ├─ memory.knowledgeMemory   │                              │
│  │  .collectionId ──────────► COLLECTION                    │
│  ├─ connectedAssistants[]    │                              │
│  │  .assistantId ───────────► EMPLOYEE (self-referencing)   │
│  └─ onboarding               │                              │
│     .shadowAssistantId ─────► EMPLOYEE (self-referencing)   │
│                                                             │
│  monitored by ──────────────► EXECUTION LOG                 │
│  governed by ───────────────► GOVERNANCE CONFIGS (org level)│
└─────────────────────────────────────────────────────────────┘
```

---

## Naming Convention Summary

| Entity | Convention | Reason |
|--------|-----------|--------|
| Employee | camelCase | New unified entity |
| Opcode | snake_case | LOCKED from existing implementation |
| Step | snake_case | LOCKED from existing implementation |
| Model Config | camelCase | New entity |
| Instruction | camelCase | New entity |
| Collection | full camelCase (organizationId) | LOCKED from existing implementation |
| Connection | snake_case | LOCKED from existing implementation |
| API Config | camelCase | New entity |
| Channel | full camelCase (organizationId) | LOCKED from existing implementation |
| Trigger | camelCase | New entity |
| Execution Log | camelCase | New entity |

**Rule:** Existing field names are never changed. New fields always use camelCase. The API translation layer maps between conventions at the boundary.

---

## Storage Summary

| Entity | Primary Store | Pattern |
|--------|--------------|---------|
| Employee | Redis / DB | `employee:{orgId}:{subOrgId}:{assistantId}` |
| Opcode | Redis | `opcode:{org_id}:{sub_org_id}:{opcode_id}` |
| Model Config | Redis | `{Provider}Model:{orgId}:{subOrgId}:{uniqueName}` |
| Instruction | Redis / DB | TBD |
| Collection | MongoDB | Managed by KB service |
| Connection | Redis | `bexo:{org_id}:{app_id}:connections:{connection_id}` |
| API Config | Redis | `api:{orgId}:{subOrgId}:{apiConfigId}` |
| Channel | Redis | `channel:{orgId}:{subOrgId}:{address}` |
| Trigger | DB | Persistent storage |
| Execution Log | DB | Append-only, indexed by assistantId + timestamp |
| Governance | DB | Per org / per sub-org |
