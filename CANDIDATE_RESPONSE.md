# Candidate Response

## Your Name

Marius

---

## Section A: Top 5 Risks (Ranked)

1. **No role-based authorization on data access (broken access control)**
    The SQL generator does not check the SSO role, therefore it may expose the data for a user(role) which is not supposed to be exposed for that role.

2. **Sensitive data persisted in debug logs**
   The Debug Logger stores full prompts, SQL, and complete results (including PII) for 30 days, creating a large secondary breach surface and an unauthorized PII copy.

3. **Prompt injection via untrusted context above the system prompt**
   Top-5 RAG chunks and prior conversation are added above the system instructions, so retrieved text can override guardrails and direct llm response in unwanted direction.

4. **Unauthenticated admin `rerun_query` endpoint (IDOR)**
   `POST /admin/rerun_query` re-executes any prior query by ID with no authentication/authorization, enabling privilege escalation and replay of another user's PII queries.

5. **Direct LLM-generated SQL + hallucinated fallback**
   The LLM executes raw SQL against database with no read-only role, row limits, or timeouts (SQL-injection, cost, and latency risk), and a fallback that "estimates values from industry benchmarks" fabricates analytics as if real.

---

## Section B: Mitigations

1. **Authorization:** Enforce a deterministic policy layer between orchestrator and data — per-role table/column allow-lists plus Snowflake row/column-level security; a query validator rejects any disallowed table/column before execution. Queries 2 & 3 must be denied, not fulfilled.

2. **Logging:** Stop logging payloads. Log query shape/metadata only; redact PII and result sets; encrypt and access-control the log store.

3. **Prompt injection:** Move the system prompt to the top, wrap RAG/user content in delimiters as untrusted data, and add input/output filtering plus PII redaction before responses are returned.

4. **Admin endpoint:** Require authentication + RBAC (admin/ops role only), scope reruns to the original requester's permissions, and audit every call.

5. **SQL execution:** Replace free-form text-to-SQL with parameterized, pre-approved query templates. Run as a read-only role with statement timeouts and row caps. Remove the benchmark-estimate fallback — return "no data available" instead to avoid hallucinations.

---

## Section C: First Architecture Change

**Change:** Insert a deterministic authorization/policy-enforcement layer that gates every data access by SSO role.

**Rationale:** It directly satisfies the DPA/GDPR PII constraint and blocks the worst, most likely incidents (unauthorized data access) regardless of how the LLM behaves. Since the system is expected to be run inside the organisation, prompt injections may be treated as a less critical flaw.

---

## Section D: Clarifying Questions

1. Who exactly will use this tool, and what are the 5–10 questions they ask most often?

2. Should answers show where the number came from (a source/link), so people can verify it?

---

## Section E: Success Metrics

- Blocked all attempts to access data which the user/role does not have permissions to.
- High accuracy of responses, accurate data retrieved/no hallucinations.
- p95 latency and cost-per-conversation vs. the 3s / $0.50 budgets.

---

*End of response.*
