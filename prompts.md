# Prompt Engineering Log — CPA Email Outreach Workflow

---

## 1. Initial Version

```python
PROMPT_TEMPLATE = """
You are an AI assistant tasked with drafting an email to a CPA firm to inquire about tax filing services for a C-Corporation.
Your goal is to get a quote and potentially set up a meeting.

Here is the information about the startup:
- Exact tax filing service needed: {tax_filing_service}
- Shareholder information: {shareholder_info}
- State of operation: {state_of_operation}
- Business category: {business_category}
- 1-Phrase business introduction: {business_intro}

Draft a professional email suitable for sending to a CPA firm. The email should:
1. Clearly state the purpose of the email: inquiring about C-Corp tax filing services.
2. Include the provided details about the startup.
3. Ask for a quote for the specified services, considering the budget if applicable (but do not mention the budget directly in this initial email).
4. Suggest scheduling a brief meeting to discuss specific requirements and pricing, especially if there are foreign shareholders, and mention your availability. Provide a placeholder for the meeting time (e.g., 'your earliest convenience' or 'next week').
5. Maintain a professional and concise tone.

Subject: Inquiry: C-Corporation Tax Filing Services - {business_intro}

Body of the email:
"""

# User Inputs
# Please fill in the following details for your startup:
# These variables simulate command-line inputs.

budget = "$2,000 - $3,000"  # Example budget. This is for internal reference, not directly in the initial email.
tax_filing_service = "Federal and State C-Corp tax filing, including K-1s for investors."
shareholder_info = "We have both domestic and foreign shareholders (from Europe and Asia)."
state_of_operation = "Delaware (registered), New York (primary operation)"
business_category = "Software as a Service (SaaS) for small businesses, not containing sensible industries."
business_intro = "A B2B SaaS startup providing CRM solutions for small businesses."
```

---

## 2. Revision 1 — Assess Corporate Structure Before Assuming C-Corp

### What Changed
Point 1 of the prompt instructions is updated: instead of assuming the entity is always a C-Corp, the model must **first evaluate the entity type** declared in the user's input. If the registered entity is not a C-Corp (e.g., LLC, S-Corp, partnership), the email should pivot to request the appropriate tax filings for that structure rather than defaulting to C-Corp-specific services.

### Why
The original prompt hardcodes the assumption that the user is filing for a C-Corp. However, users filling in the intake form may:
- Accidentally select the wrong entity type, or
- Operate under a multi-entity structure where the filing entity is an LLC or S-Corp rather than a C-Corp.

Sending an email that requests C-Corp-specific filings (e.g., Form 1120) to a CPA on behalf of an LLC would generate an inaccurate quote and undermine the entire purpose of the workflow — price and service comparison. Catching the mismatch at the prompt level prevents garbage-in-garbage-out errors from propagating into outgoing emails.

### Revised Prompt

```python
PROMPT_TEMPLATE_V2 = """
You are an AI assistant tasked with drafting an email to a CPA firm to inquire about tax filing services for a newly registered U.S. business entity.
Your goal is to get a quote and potentially set up a meeting.

Here is the information about the startup:
- Exact tax filing service needed: {tax_filing_service}
- Shareholder information: {shareholder_info}
- State of operation: {state_of_operation}
- Business category: {business_category}
- 1-Phrase business introduction: {business_intro}

Draft a professional email suitable for sending to a CPA firm. The email should:
1. First, assess the entity type implied by the tax filing service requested (e.g., C-Corp → Form 1120; S-Corp → Form 1120-S; Partnership/LLC → Form 1065; Sole Proprietor → Schedule C).
   - If the entity is a C-Corp, state that you are inquiring about C-Corp tax filing services.
   - If the entity is any other structure, adjust the email subject and body accordingly to reflect the correct entity type and its associated filing requirements. Do NOT default to C-Corp language for non-C-Corp entities.
2. Include the provided details about the startup.
3. Ask for a quote for the specified services, considering the budget if applicable (but do not mention the budget directly in this initial email).
4. Suggest scheduling a brief meeting to discuss specific requirements and pricing, especially if there are foreign shareholders, and mention your availability. Provide a placeholder for the meeting time (e.g., 'your earliest convenience' or 'next week').
5. Maintain a professional and concise tone.

Subject: Inquiry: [Entity Type] Tax Filing Services - {business_intro}

Body of the email:
"""

# User Inputs
budget = "$2,000 - $3,000"
tax_filing_service = "Federal and State C-Corp tax filing, including K-1s for investors."
shareholder_info = "We have both domestic and foreign shareholders (from Europe and Asia)."
state_of_operation = "Delaware (registered), New York (primary operation)"
business_category = "Software as a Service (SaaS) for small businesses, not containing sensible industries."
business_intro = "A B2B SaaS startup providing CRM solutions for small businesses."
```

---

## 3. Revision 2 — Flag Sensitive Industries & Foreign-Based Entities

### What Changed
A new point is added to the prompt instructions: the model must **scan the business category for sensitive or regulated industries** under U.S. law, and **check whether any shareholders or related entities are physically based outside the United States**. Both conditions, if detected, must be explicitly surfaced in the email body so the CPA can respond with an appropriately scoped quote.

### Why

**Sensitive / Regulated Industries**
Certain business categories — including but not limited to firearms, cryptocurrency/digital assets, cannabis, defense contracting, and licensed financial services — are subject to additional regulatory layers (e.g., ATF/FFL licensing, FinCEN/BSA compliance, SEC/FINRA oversight). A generic CPA outreach email that omits this context may:
- Reach firms that are not qualified to handle regulated-industry clients, resulting in wasted back-and-forth.
- Return quotes that do not factor in the compliance overhead associated with the industry.
- In worst cases, expose the client to risk if the CPA provides incomplete guidance because they were not informed of the industry.

Surfacing this information upfront ensures the CPA self-selects based on capability, and that any quote received reflects the true scope of work.

**Foreign Shareholders / Foreign-Based Entities**
If any shareholder is physically based outside the United States — or if the company has a parent or subsidiary incorporated abroad — the tax filing scope expands significantly beyond a standard Form 1120:
- **Form 5472** is required for any 25%+ foreign-owned U.S. corporation, regardless of income.
- **Transfer pricing documentation** may be required if there are intercompany transactions with a foreign parent or subsidiary.
- **FBAR / FATCA** disclosures may apply depending on financial account structures.
- **Tax treaty implications** (e.g., U.S.–China, U.S.–E.U.) may affect withholding rates and filing obligations.

A user who is new to the U.S. tax system — which is exactly the target user of this workflow — is highly unlikely to know about these additional obligations. If the email does not mention them, the CPA will quote only for the services explicitly requested, leaving the client with an incomplete engagement that may result in penalties or missed filings. Flagging foreign presence in the email forces the CPA to scope the full picture.

### Revised Prompt

```python
PROMPT_TEMPLATE_V3 = """
You are an AI assistant tasked with drafting an email to a CPA firm to inquire about tax filing services for a newly registered U.S. business entity.
Your goal is to get a quote and potentially set up a meeting.

Here is the information about the startup:
- Exact tax filing service needed: {tax_filing_service}
- Shareholder information: {shareholder_info}
- State of operation: {state_of_operation}
- Business category: {business_category}
- 1-Phrase business introduction: {business_intro}

Draft a professional email suitable for sending to a CPA firm. The email should:
1. First, assess the entity type implied by the tax filing service requested (e.g., C-Corp → Form 1120; S-Corp → Form 1120-S; Partnership/LLC → Form 1065; Sole Proprietor → Schedule C).
   - If the entity is a C-Corp, state that you are inquiring about C-Corp tax filing services.
   - If the entity is any other structure, adjust the email subject and body accordingly to reflect the correct entity type and its associated filing requirements. Do NOT default to C-Corp language for non-C-Corp entities.
2. Include the provided details about the startup.
3. Ask for a quote for the specified services, considering the budget if applicable (but do not mention the budget directly in this initial email).
4. Suggest scheduling a brief meeting to discuss specific requirements and pricing, especially if there are foreign shareholders, and mention your availability. Provide a placeholder for the meeting time (e.g., 'your earliest convenience' or 'next week').
5. Review the business category for sensitive or regulated industries under U.S. law (including but not limited to: firearms/weapons, cryptocurrency/digital assets, cannabis, defense/military contracting, licensed financial services, adult content).
   - If a sensitive industry is detected, clearly state this in the email and note that the firm should have specific experience with compliance requirements for that industry.
   - If no sensitive industry is detected, no action is needed for this point.
6. Review the shareholder information for any indication that shareholders or related entities (parent companies, subsidiaries) are physically based in foreign countries (i.e., non-resident aliens or foreign-incorporated entities).
   - If foreign presence is detected, explicitly state this in the email and note that additional filings may be required beyond the base service requested (e.g., Form 5472 for foreign-owned U.S. corporations, transfer pricing documentation, or tax treaty considerations). Ask the CPA to include these in their scoping and quote.
   - If all shareholders are U.S.-based, no action is needed for this point.
7. Maintain a professional and concise tone.

Subject: Inquiry: [Entity Type] Tax Filing Services - {business_intro}

Body of the email:
"""

# User Inputs
budget = "$2,000 - $3,000"
tax_filing_service = "Federal and State C-Corp tax filing, including K-1s for investors."
shareholder_info = "We have both domestic and foreign shareholders (from Europe and Asia)."
state_of_operation = "Delaware (registered), New York (primary operation)"
business_category = "Software as a Service (SaaS) for small businesses, not containing sensible industries."
business_intro = "A B2B SaaS startup providing CRM solutions for small businesses."
```

---

## Summary of Changes Across Versions

| Version | Key Addition | Problem It Solves |
|---|---|---|
| **V1 — Initial** | Base prompt: include startup details, request quote, suggest meeting | Generates a basic outreach email |
| **V2 — Revision 1** | Assess entity type before assuming C-Corp | Prevents wrong tax form types from appearing in emails for non-C-Corp entities |
| **V3 — Revision 2** | Flag sensitive industries + foreign-based shareholders/entities | Prevents emails from omitting critical compliance context, ensuring CPAs quote for the full actual scope of work |
