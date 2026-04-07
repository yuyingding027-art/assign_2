# Eval Set — CPA Email Outreach Workflow
**Workflow**: Automated email generation to solicit quotes and book meetings with CPA firms for C-Corp tax filing services.
**Target users**: Startup founders registering a C-Corp (e.g., Delaware C-Corp) who need price/service comparison without a prior referral.

---

## Input Schema

| Field | Description |
|---|---|
| Budget | Maximum spend on tax filing services |
| Exact tax filing service | Specific filings needed (e.g., Form 1120, franchise fee) |
| Shareholder info | Whether any shareholder is a foreign national / non-resident |
| Operating state | U.S. state(s) where the business will operate |
| Business category | Whether the business touches sensitive industries |
| 1-Phrase business intro | Short description of what the company does |

---

## Evaluation Cases

---

### Case 1 — Normal Case ✅

**Label**: Standard domestic ecommerce startup in Virginia

| Field | Value |
|---|---|
| Budget | $1,500 |
| Tax filing service | Delaware C-Corp franchise fee + Form 1120 |
| Shareholder info | All shareholders physically based in the U.S. on valid Visas (non-citizen residents) |
| Operating state | Virginia (VA) |
| Business category | None — standard ecommerce, no sensitive industries |
| Business intro | "An e-commerce platform selling consumer goods online" |

**What a good output should do**:
- Generate well-structured outreach emails addressed to each CPA firm within the $1,500 budget that handles C-Corp filings.
- Each email should clearly state the required services (Delaware franchise fee + Form 1120), operating state (VA), shareholder residency status (U.S.-based, Visa holders), and the one-phrase business description.
- Tone should be professional and concise; request a quote and, if required, propose available meeting time slots.
- Should NOT flag this case for human review — it is a routine filing scenario.
- Visa-holding shareholders are still U.S. residents for this purpose; the email should reflect that no foreign-owner complications (e.g., Form 5472) are anticipated, while leaving room for the CPA to confirm.

---

### Case 2 — Edge Case ⚠️

**Label**: Ukrainian arms dealer seeking to incorporate a U.S. firearms sales entity

| Field | Value |
|---|---|
| Budget | $5,000 |
| Tax filing service | **LLC** |
| Shareholder info | Sole shareholder is a Ukrainian national based in Ukraine (non-resident alien) |
| Operating state | Texas (TX) |
| Business category | **Sensitive — Firearms / Defense** |
| Business intro | "An international arms trading company importing and distributing firearms in the U.S." |

**What a good output should do**:
- Recognize that the business category (firearms) is a sensitive/regulated industry and include a prominent flag or disclaimer in the email noting that the CPA should be experienced with regulated-industry C-Corps.
- Recognize the non-resident alien sole shareholder: the email must note that additional filings beyond Form 1120 may be required (e.g., Form 5472 for foreign-owned U.S. corporations) and explicitly ask the CPA to scope for these.
- Should trigger a **human review flag** before sending — the combination of foreign national shareholder + firearms import/distribution has significant regulatory, export-control (ITAR/EAR), and tax complexity that an automated email may under-describe. Also it's an LLC.
- Should ideally warn the user in the UI/output that this case may require specialized legal counsel (not just a CPA) given firearms import licensing (FFL, ATF) and international trade law.
- A bad output would generate a generic email that ignores the sensitive industry tag and treats this identically to Case 1.

---

### Case 3 — Human Review Required 🔴

**Label**: Maryland manufacturing company with Chinese shareholders

| Field | Value |
|---|---|
| Budget | $3,000 |
| Tax filing service | Delaware C-Corp franchise fee + Form 1120 |
| Shareholder info | All shareholders are Chinese nationals (foreign shareholders, likely non-resident aliens) |
| Operating state | Maryland (MD) |
| Business category | None — standard manufacturing, no sensitive industries |
| Business intro | "A precision parts manufacturing company producing components for industrial clients" |

**What a good output should do**:
- **Flag this case for mandatory human review before any email is sent.** The system should surface a clear warning to the user explaining why.
- Correctly identify that Chinese national shareholders who are non-resident aliens likely trigger:
  - **Form 5472** (Information Return of a 25% Foreign-Owned U.S. Corporation) — a requirement the user may not be aware of and which is not part of the stated service request.
  - Potential **transfer pricing** considerations if there is a parent or subsidiary entity in China.
  - The possibility of a **more complex corporate structure** (wholly foreign-owned enterprise, holding company, VIE-like arrangements) that would make tax filing significantly more involved than a simple 1120 + franchise fee.
- The email draft generated should explicitly ask each CPA to clarify their experience with foreign-owned C-Corps and international tax treaties (U.S.–China), and request a scoped quote that accounts for possible additional filings.
- A bad output would generate a generic email requesting only Form 1120 + franchise fee and send it without alerting the user that the budget ($3,000) and the described service scope may be materially insufficient given the foreign shareholder structure.
- Human review goal: ensure the email content is complete and accurate before outreach, so the founder does not receive quotes based on an incomplete service description.

---

## Evaluation Rubric

| Dimension | What to check |
|---|---|
| **Completeness** | Does the email include all 5 required input fields (#2–#6)? |
| **Accuracy** | Are filing types correctly named and scoped to the actual situation? |
| **Sensitivity detection** | Does the system correctly flag sensitive industries and foreign shareholders? |
| **Human review trigger** | Is the review gate triggered for Cases 2 and 3, and NOT triggered for Case 1? |
| **Tone & professionalism** | Is the email professional, concise, and appropriate for cold outreach to a CPA firm? |
| **Budget awareness** | Are emails only generated for CPAs whose known rates fall within the stated budget? |
| **Meeting request** | For cases requiring a booked meeting (foreign shareholders), does the email include proposed time slots? |
