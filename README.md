# DirSync-Mongo-Agent-AD-MongoDB-AI-Sidecar
Synchronize users, groups, and memberships from customer Active Directory (LDAP/LDAPS) into MongoDB with idempotent bulk upserts, incremental watermarks, and an AI (RAG) sidecar that proposes per‑tenant mapping plans (human‑approved) to handle schema drift. Designed with Akamai‑style multi‑tenant needs in mind.


Features
Secure AD ingest: LDAPS bind, paged queries, retries.

MongoDB storage: deterministic _id from objectGUID, proper indexes, bulk upserts.

Incremental sync: simple ISO timestamp watermark.

AI sidecar (RAG): proposes AD → Mongo mapping plans, validated against strict JSON schemas and approved by you before use.

Ops‑friendly: structured logs, minimal config, Docker/K8s friendly.

Architecture (high‑level)
Active Directory (LDAP/LDAPS)
         │
         ▼
   AD Client (ldap3) ──► Mapper
         │                  │
         │              (AI plan? yes → apply_plan; no → default map)
         ▼                  │
     Bulk Upserts      ─────┘
         │
         ▼
      MongoDB
   ├─ users
   ├─ groups
   ├─ group_memberships
   └─ meta (watermark, AI plans)

Prerequisites
Python 3.10+

MongoDB 6/7 (local or managed)

Network access to target AD servers (LDAPS recommended)

AI Sidecar (RAG) Workflow
- Collect redacted AD samples (for a tenant). Keep attribute names; mask values if using a hosted LLM.
- Propose a plan (LLM drafts mapping rules → stored as status=proposed):
- Review mapping_plans doc in Mongo; edit if desired.

Approve:
python -m dirsync_agent.ai.cli approve --id <plan_object_id>

- Sync with plan: when you pass customer_id and dir_domain to your runner (or bake them into env/flags), the sync will apply the latest approved plan to map user attributes.

