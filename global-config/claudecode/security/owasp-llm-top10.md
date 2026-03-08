# OWASP Top 10 for LLM Applications:2025

_Source: https://github.com/OWASP/www-project-top-10-for-large-language-model-applications/tree/main/2_0_vulns_
_Last updated: 2026-03-08_

## LLM01:2025 — Prompt Injection

User inputs or external data sources manipulate LLM behavior in unintended
ways, leading to guideline violations, unauthorized access, or compromised
decision-making. Includes direct injection (malicious user input) and
indirect injection (compromised external content processed by the model).

**What to look for:**
- User input passed directly to LLM without sanitization or framing
- External content (web pages, documents, emails) ingested without
  content segregation from system instructions
- RAG pipelines that don't separate retrieved content from instructions
- Missing output validation on LLM responses before action execution
- No human-in-the-loop for high-impact operations triggered by LLM
- System prompts that rely on instructions alone for security boundaries

---

## LLM02:2025 — Sensitive Information Disclosure

LLMs risk exposing PII, financial details, health records, business secrets,
security credentials, or legal documents through their output. Can occur
through training data memorization, system prompt leakage, or insufficient
output filtering.

**What to look for:**
- Training data containing PII or credentials without scrubbing
- System prompts containing API keys, tokens, or connection strings
- Missing output filtering/redaction for sensitive patterns
- Verbose error messages exposing internal architecture
- No differential privacy or data minimization in fine-tuning
- Logging of full LLM inputs/outputs without redaction

---

## LLM03:2025 — Supply Chain

Vulnerabilities from compromised model components, training data, plugins,
or deployment dependencies that undermine system integrity.

**What to look for:**
- Models from untrusted sources without integrity verification
- Pre-trained models with unknown training data provenance
- Third-party plugins/extensions without security review
- Compromised or vulnerable dependencies in LLM application stack
- No SBOM for model components and dependencies
- Missing signature verification on model weights and updates

---

## LLM04:2025 — Data and Model Poisoning

Malicious data introduced during training or fine-tuning to manipulate
model behavior, inject backdoors, or degrade performance.

**What to look for:**
- Training data sourced from unvetted public datasets
- No data quality validation or provenance tracking
- Fine-tuning pipelines without access controls
- Missing monitoring for model behavior drift
- No adversarial testing against poisoning attacks
- Crowdsourced training data without review

---

## LLM05:2025 — Improper Output Handling

Flaws in managing LLM-generated content that risk downstream security
issues when outputs are consumed by other systems.

**What to look for:**
- LLM output directly rendered as HTML/JavaScript (XSS vector)
- Generated SQL/code executed without sanitization
- LLM responses used in template engines without escaping
- Generated commands passed to shell or system calls
- API responses including raw LLM output without validation
- Missing content-type headers on LLM-generated responses

---

## LLM06:2025 — Excessive Agency

LLM agents granted unchecked autonomy to take actions through tools,
plugins, or function calls without proper oversight or scope limits.

**What to look for:**
- Agents with access to tools beyond what their task requires
- Missing human approval workflow for high-impact operations
- Tools with overly broad permissions (full DB access when only SELECT needed)
- No rate limiting on tool invocations
- Open-ended tools (shell access, arbitrary URL fetch) without input filtering
- Missing audit logging of tool invocations and their parameters
- Actions executed in privileged context rather than user-scoped context
- No rollback mechanism for agent-initiated changes

---

## LLM07:2025 — System Prompt Leakage

System prompts containing sensitive configuration, credentials, or
architectural details that can be extracted by attackers. The system prompt
should not be treated as a secret or used as a security control.

**What to look for:**
- API keys, tokens, or credentials embedded in system prompts
- Internal architecture details exposed in prompts
- Authorization logic implemented solely in prompt instructions
- Content filtering rules visible in system prompts (reveals bypass paths)
- Role/permission structures defined only in prompts
- No external guardrails independent of the prompt

---

## LLM08:2025 — Vector and Embedding Weaknesses

Vulnerabilities in vector storage and embedding representations used
for RAG and semantic search that can be exploited for data access or
manipulation.

**What to look for:**
- Missing access controls on vector database queries
- No tenant isolation in shared vector stores
- Embedding inversion attacks (reconstructing source text from vectors)
- Poisoned embeddings injected into the knowledge base
- No input validation on vector search queries
- Missing encryption for stored embeddings

---

## LLM09:2025 — Misinformation

LLMs generating false, misleading, or fabricated information (hallucination)
that users act upon without verification.

**What to look for:**
- No fact-checking or grounding mechanism for LLM outputs
- LLM outputs presented as authoritative without disclaimers
- Missing confidence scoring on generated content
- No human review process for externally-facing LLM content
- Critical decisions based solely on LLM output
- Generated content without source attribution or references

---

## LLM10:2025 — Unbounded Consumption

Uncontrolled resource consumption by LLMs causing service disruptions,
excessive costs, or denial of service.

**What to look for:**
- No token/request limits per user or session
- Missing cost controls on API calls to LLM providers
- Recursive or looping agent chains without depth limits
- No timeout on LLM inference calls
- Missing rate limiting on LLM-powered endpoints
- Variable-length inputs without max-token enforcement
- No budget alerts or spending caps on LLM API usage
