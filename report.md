# Prototype Evaluation Report
**Workflow**: Automated CPA Email Outreach for C-Corp Tax Filing Services
**Date**: April 2026

---

## 1. Business Use Case

Startup founders registering a U.S. C-Corporation — particularly those unfamiliar with the American tax system — face a structurally inefficient problem: to find a suitable CPA, they must manually contact multiple firms, describe the same situation repeatedly, and compare quotes without a clear framework for what questions to ask. The situation is further complicated by special circumstances such as foreign shareholders, sensitive business categories, or multi-state operations, any of which can expand the required filing scope in ways the founder may not anticipate.

This prototype automates the first step of that process: generating a professional outreach email containing all relevant details (filing service needed, shareholder composition, operating state, business category, and a brief business description), and sending it to multiple CPA firms simultaneously to solicit quotes and, where necessary, schedule a meeting. The target user is a non-expert founder; the goal is to reduce friction, improve coverage, and surface the right questions before any CPA engagement begins.

---

## 2. Model Choice

**Selected model: `gemini-flash-latest` (Google Gemini 1.5 Flash)**

Gemini 1.5 Flash was chosen for this task primarily on grounds of latency and cost efficiency. Email drafting is a structured generation task with a well-defined input schema and a moderate-length output; it does not require deep multi-step reasoning. Flash-tier models handle this class of task reliably while keeping per-call cost low enough to support batch generation across multiple CPA targets.

**Observations on alternatives**: Gemini 1.5 Pro would offer stronger instruction-following for complex edge cases (e.g., correctly scoping foreign shareholder obligations without explicit enumeration in the prompt), but at meaningfully higher cost and latency per call. For the current prototype scope — single-email generation with a structured prompt — Flash is the appropriate trade-off. If the system is extended to include structured output parsing, compliance flagging logic, or multi-turn clarification dialogs, upgrading to a Pro-tier model would be warranted.

---

## 3. Baseline vs. Final Design — Prompt Iteration

| Dimension | V1 — Baseline | V3 — Final |
|---|---|---|
| **Entity type assumption** | Hardcoded as C-Corp regardless of input | Instructs model to infer entity type from the filing service field; adjusts email language accordingly |
| **Sensitive industry handling** | Not addressed; all business categories treated identically | Explicitly checks for regulated industries (firearms, crypto, cannabis, defense, financial services); flags in email if detected |
| **Foreign shareholder handling** | Mentions "schedule a meeting if there are foreign shareholders" generically | Detects foreign presence; instructs model to explicitly note potential Form 5472, transfer pricing, and treaty obligations in the email body |
| **Output accuracy risk** | High — a non-C-Corp input would generate a C-Corp email with wrong form types | Reduced — entity mismatch is caught at prompt instruction level |
| **Information completeness** | Low for complex cases — a foreign-owned company would receive a quote scoped only to Form 1120 | Improved — CPAs are asked to scope additional filings proactively |

**Observed improvement**: The V1 prompt reliably produces a well-formatted, professional email for the straightforward domestic case (Case 1 in the eval set). However, when tested against Case 3 (Chinese shareholders, Maryland manufacturing), V1 generates an email that requests only Form 1120 and the Delaware franchise fee — omitting any mention of Form 5472 or foreign-owner complexity. V3 surfaces these obligations explicitly, giving the CPA enough context to provide a materially more accurate quote.

---

## 4. Remaining Failures and Human Review Requirements

The prototype has several unresolved failure modes that make unsupervised deployment inadvisable. First, the current code contains a **critical ordering bug**: `PROMPT_TEMPLATE` is referenced on line 48 before it is defined on line 69, which will raise a `NameError` at runtime in any standard Python environment (it may appear to work in certain Colab execution orders only by accident). Second, the system performs **no budget-based CPA filtering** — the budget variable exists as a comment but is never used to gate which firms receive an email. Third, the prompt's foreign-entity and sensitive-industry detection relies entirely on the model's interpretation of free-text input fields; a user who describes a firearms business as "outdoor sporting goods retail" would produce no flag. Finally, and most critically, **the workflow has no human review gate**: for cases involving foreign shareholders, regulated industries, or ambiguous entity structures (as documented in eval Cases 2 and 3), an outgoing email that under-describes the filing scope is worse than no email — it generates CPA quotes that the founder may rely on despite being materially incomplete.

---

## 5. Deployment Recommendation

**Not recommended for unsupervised deployment in its current form.**

The prototype demonstrates a viable and genuinely useful concept: the core email generation quality is good, the prompt iteration meaningfully improved output completeness, and the business problem it addresses is real. However, deployment should be conditional on the following minimum requirements being met:

1. **Fix the runtime bug** — `PROMPT_TEMPLATE` must be defined before it is called.
2. **Implement a human review gate** — any case involving foreign shareholders, a sensitive/regulated industry, or an entity type other than a straightforward domestic C-Corp should be held in a draft state and reviewed by a qualified person before sending.
3. **Add budget-based CPA filtering** — the budget input should be used to exclude firms outside the user's range rather than being silently ignored.
4. **Harden the input parsing** — sensitive industry and foreign-entity detection should use a structured intake field (e.g., a checkbox or dropdown), not rely solely on the model parsing free-text descriptions.

If these conditions are met, the workflow is a strong candidate for deployment as a **human-in-the-loop tool**: it handles the mechanical work of drafting and parallelizing outreach, while a brief human review step before sending ensures that complex cases do not result in inaccurate or incomplete CPA engagements.
