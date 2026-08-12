# Claim Assist – Use Cases & Feature List

## Use Cases

1. **Authorization / Pre-Certification Denial** – Validate authorization availability, status, date of service, CPT/HCPCS code, and authorized vs. billed units to identify the denial root cause.

2. **Eligibility / Validation Denial** – Validate patient eligibility, coverage status, member/payer details, and coverage effective dates against the claim date of service.

3. **Medical Necessity / Diagnosis–Procedure Mismatch** – Validate whether the billed procedure is supported by the diagnosis, physician/clinical documentation, and applicable payer medical-necessity criteria.

## Feature List

1. **Denial Classification** – Automatically classify the denial reason/code into Authorization, Eligibility, or Medical Necessity.

2. **Deterministic Workflow Routing** – Route each denial category to a predefined and approved workflow with only the relevant tools and data sources.

3. **Workflow-Level AI Agent** – Allow agentic decision-making within the selected workflow for investigation and additional checks.

4. **Claim Data Retrieval** – Retrieve required claim details such as denial reason, diagnosis codes, CPT/HCPCS codes, date of service, payer, and billed units.

5. **Authorization Validation** – Search and validate authorization number, status, service dates, CPT/HCPCS codes, and authorized units.

6. **Eligibility Validation** – Validate member eligibility, coverage status, payer/plan information, and coverage dates against the date of service.

7. **Clinical Document Processing** – Process physician and clinical documentation when required for Medical Necessity investigation.

8. **Diagnosis–Procedure Validation** – Evaluate the relationship between diagnosis (ICD) and billed procedure (CPT/HCPCS) using available clinical evidence.

9. **Conditional Payer Policy Retrieval** – Retrieve payer policies or medical-necessity criteria only when required by the selected workflow or when results are ambiguous.

10. **Investigation Timeline** – Show billers each investigation activity performed by the backend agent with its pass/fail/result status.

11. **Evidence-Based Findings** – Present the identified root cause with supporting claim, authorization, eligibility, clinical, or policy evidence.

12. **Recommended Action** – Suggest the appropriate next step, such as claim correction, documentation review, resubmission, appeal preparation, or manual review.

13. **Human Review / Escalation** – Route ambiguous or insufficient-evidence cases for manual review instead of making an unsupported automated decision.
