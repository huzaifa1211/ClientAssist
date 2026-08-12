# RCM Denial Management – Proposed Architecture & Use Cases

## 1. Objective

The denial-management solution should **not process every denial through one generic AI/RAG workflow**.

Different denial categories require different:

* Data sources
* Validations
* Documents
* Business rules
* Payer policies
* Investigation steps

For the current scope, the solution will focus only on these three use cases:

1. **Authorization / Pre-Certification Denials**
2. **Eligibility / Validation Denials**
3. **Medical Necessity / Diagnosis–Procedure Mismatch**

The architecture should therefore route each denial to its appropriate workflow instead of invoking every available tool, RAG source, or document-processing pipeline.

---

# 2. Overall Architecture

The recommended architecture is a **hybrid deterministic + agentic approach**.

## Level 1 – Deterministic Routing

The system first identifies the denial using information such as:

* Denial code
* Denial reason
* Claim information
* Diagnosis codes
* Procedure codes
* Payer information

The denial is then classified into one of the approved categories:

**Denial → Classification → Denial Category → Approved Workflow**

For the current scope:

```text
Claim + Denial
      ↓
Denial Classification
      ↓
Deterministic Router
      ↓
 ┌─────────────────────────────────────┐
 │ 1. Authorization Workflow           │
 │ 2. Eligibility Workflow             │
 │ 3. Medical Necessity Workflow       │
 └─────────────────────────────────────┘
```

The orchestrator should **not independently invent workflows at runtime**.

The major denial categories and their permitted tools/data sources should be predefined.

---

# 3. Level 2 – Agentic Decision-Making

After Level 1 selects the appropriate workflow, an AI agent can make decisions **within that approved workflow**.

Therefore:

**Level 1 = Deterministic Routing**

**Level 2 = Agentic Decision-Making within the selected workflow**

For example, an Authorization agent may decide whether payer-policy retrieval is necessary after checking the authorization record.

However, it should not suddenly start processing clinical documents when clinical documentation has no relevance to that particular authorization denial.

This makes the system more controlled, predictable, efficient, and auditable.

---

# USE CASE 1 – AUTHORIZATION / PRE-CERTIFICATION DENIAL

## 4. Example Scenario

Suppose the denial reason is:

**“Authorization not found for billed services.”**

The denial classifier identifies:

**Category: Authorization / Pre-Certification**

The deterministic router then invokes the **Authorization Workflow**.

---

## 5. Authorization Investigation Flow

### Step 1 – Retrieve Claim Details

Retrieve the denied claim and relevant information, including:

* Claim ID
* Patient information
* Date of service
* CPT/HCPCS
* Billed units
* Payer
* Denial code/reason

### Step 2 – Search Authorization Records

Query the authorization database/table to determine whether prior authorization exists.

For example:

**Authorization Number: ABC123**

If no authorization exists, that itself may explain the denial.

If authorization exists, additional validation is performed.

### Step 3 – Validate Date of Service

Check whether the authorization was valid for the claim's date of service.

### Step 4 – Validate CPT

Compare the authorized procedure with the billed procedure.

Example:

**Authorized CPT: 99213**
**Billed CPT: 99213**

If both match, this validation passes.

### Step 5 – Validate Units

Compare:

**Authorized Units vs. Billed Units**

Example:

Authorized Units = 1
Billed Units = 2

This identifies a potential reason for the denial.

### Step 6 – Conditional Payer Policy Retrieval

Payer policy should **not automatically be retrieved for every authorization denial**.

If the authorization data itself clearly explains the denial, additional RAG/policy retrieval may not be necessary.

However, if the result is ambiguous or there is an exception such as a unit mismatch requiring payer-specific interpretation, the agent can retrieve the relevant authorization policy.

### Step 7 – Finding

Example:

**“Authorization existed; however, billed units exceeded the authorized units.”**

### Step 8 – Recommended Action

Example:

**Review the authorization documentation and applicable payer policy. Correct the claim where appropriate or prepare an appeal with the authorization record and supporting evidence.**

---

## 6. Authorization UI

The UI can display:

**Denial Identified**
Authorization / Pre-Certification

**Reason**
Authorization not found for billed services

**Investigation Performed**

✓ Claim details retrieved
✓ Authorization records searched
✓ Authorization found – ABC123
✓ Date of service validated
✓ CPT 99213 matched
✕ Unit mismatch identified
✓ Payer policy retrieved because additional investigation was required

**Finding**

Authorization existed, but billed units exceeded authorized units.

**Recommended Action**

Review authorization documentation and payer policy before correcting the claim or submitting an appeal.

---

# USE CASE 2 – ELIGIBILITY / VALIDATION DENIAL

## 7. Example Scenario

Suppose the denial indicates that the patient was **not eligible or coverage could not be validated for the billed date of service**.

The classifier identifies:

**Category: Eligibility / Validation**

The deterministic router invokes the **Eligibility Workflow**.

---

## 8. Eligibility Investigation Flow

### Step 1 – Retrieve Claim Details

Retrieve relevant claim information such as:

* Patient/member information
* Payer
* Plan
* Date of service
* Claim details
* Denial code/reason

### Step 2 – Retrieve Eligibility/Coverage Information

Query the appropriate eligibility or coverage data source.

Determine whether the patient had active coverage.

### Step 3 – Validate Date of Service

Check whether coverage was active specifically on the billed date of service.

For example:

```text
Coverage Start: 01-Jan
Coverage End:   30-Jun

Date of Service: 15-Jul
```

The service occurred outside the active coverage period.

### Step 4 – Validate Member/Payer Information

Where applicable, validate:

* Member ID
* Insurance plan
* Payer
* Coverage status
* Effective/termination dates
* Other relevant eligibility attributes

### Step 5 – Additional Investigation When Required

If the eligibility information is ambiguous or contradictory, the workflow can perform additional approved checks.

However, **clinical-document processing or medical-necessity RAG should not automatically run for an eligibility denial** because those resources are unrelated to the primary investigation.

### Step 6 – Finding

Example:

**“Patient coverage was inactive on the billed date of service.”**

### Step 7 – Recommended Action

Depending on the evidence, the system can recommend:

* Verify updated insurance information
* Correct member/payer information
* Check whether another payer was active
* Correct and resubmit the claim where appropriate
* Route for manual review if eligibility remains ambiguous

---

## 9. Eligibility UI

**Denial Identified**
Eligibility / Coverage Validation

**Investigation Performed**

✓ Claim details retrieved
✓ Eligibility records retrieved
✓ Member information validated
✓ Coverage dates checked
✕ Coverage inactive for date of service

**Finding**

Patient coverage was not active on the billed date of service.

**Recommended Action**

Verify current insurance information and determine whether the claim should be corrected, submitted to another payer, or reviewed manually.

---

# USE CASE 3 – MEDICAL NECESSITY / DIAGNOSIS–PROCEDURE MISMATCH

## 10. Core Concept

This is the use case represented in the discussion by the example:

**The patient was being treated for the thumb, but a procedure associated with the knee was billed/performed.**

The important question for the payer becomes:

**Why was this procedure performed, and is there clinical evidence establishing that it was medically necessary?**

This is fundamentally different from Authorization or Eligibility.

Here, database validation alone may not be sufficient.

The system may need to understand the relationship between:

* Diagnosis
* Procedure performed/billed
* Physician documentation
* Clinical evidence
* Payer's medical-necessity criteria

---

# 11. Diagnosis Codes vs. Procedure Codes

Two important concepts are involved.

### Diagnosis Codes – ICD

Diagnosis codes represent the patient's:

**Disease / condition / diagnosis / clinical problem**

### Procedure Codes – CPT/HCPCS

Procedure codes represent:

**What service or procedure was performed or billed**

Therefore, the system may need to determine whether the billed procedure is reasonably supported by the patient's diagnosis and clinical documentation.

---

# 12. Medical Necessity Investigation Flow

### Step 1 – Retrieve Claim Details

Retrieve:

* Claim information
* Diagnosis codes
* CPT/HCPCS codes
* Date of service
* Payer
* Denial reason

### Step 2 – Identify Diagnosis–Procedure Relationship

Determine which diagnosis codes are associated with the billed procedure.

The system needs to investigate whether there is an expected clinical relationship between them.

### Step 3 – Retrieve Clinical Documentation

Unlike the Authorization and Eligibility workflows, this workflow may require processing:

* Physician notes
* Clinical documentation
* Medical records
* Procedure documentation
* Other relevant supporting documents

### Step 4 – Extract Clinical Evidence

Extract information explaining:

* Why the procedure was performed
* Patient's condition
* Clinical findings
* Physician's justification
* Supporting medical evidence

### Step 5 – Retrieve Medical Necessity Criteria / Payer Policy

Retrieve the applicable payer policy and medical-necessity criteria for the billed procedure.

### Step 6 – Compare Evidence Against Criteria

The workflow evaluates:

```text
Diagnosis
    +
Procedure
    +
Physician Documentation
    +
Clinical Evidence
          ↓
       Compare
          ↓
Payer Medical Necessity Criteria
```

### Step 7 – Determine Finding

Possible outcomes include:

**Criteria Met**

Clinical documentation supports the medical necessity of the procedure.

**Criteria Not Met**

Available clinical evidence does not satisfy the payer's medical-necessity criteria.

**Documentation Missing**

The procedure may potentially be justified, but required clinical documentation is unavailable.

**Ambiguous**

Available evidence is insufficient for an automated conclusion and requires human review.

### Step 8 – Recommended Action

Depending on the finding:

* Attach missing clinical documentation
* Obtain physician documentation
* Review diagnosis/procedure coding
* Review payer medical-necessity criteria
* Prepare supporting evidence for appeal
* Route to a human reviewer when necessary

---

# 13. Medical Necessity UI

**Denial Identified**
Medical Necessity / Diagnosis–Procedure Relationship

**Investigation Performed**

✓ Claim details retrieved
✓ Diagnosis codes identified
✓ Procedure codes identified
✓ Clinical documents processed
✓ Physician documentation reviewed
✓ Payer policy retrieved
✓ Medical-necessity criteria retrieved
✕ Required clinical criterion not supported

**Finding**

Available clinical documentation does not sufficiently support the medical necessity of the billed procedure under the applicable payer criteria.

**Recommended Action**

Review physician documentation and supporting clinical records. Obtain missing evidence where available before determining whether an appeal should be submitted.

---

# 14. UI Design Principle Across All Three Workflows

The biller should see **what the system did and what evidence it found**, rather than the LLM's hidden reasoning.

For every workflow, the UI should expose:

* Denial category
* Denial reason
* Investigation steps performed
* Data sources checked
* Pass/fail validation results
* Evidence found
* Relevant payer policy, where applicable
* Final finding/root cause
* Recommended action

The UI should **not expose**:

* Chain of thought
* System prompts
* User prompts used internally
* Internal LLM reasoning
* Model configuration
* Temperature/settings

The biller needs to understand:

**What happened → What was checked → What evidence was found → Why the denial likely occurred → What should be done next**

---

# 15. Final Pipeline for the Current Scope

```text
                    CLAIM + DENIAL
                          ↓
               DENIAL CODE / REASON
                          ↓
                DENIAL CLASSIFICATION
                          ↓
            LEVEL 1: DETERMINISTIC ROUTER
                          ↓
        ┌─────────────────┼──────────────────┐
        ↓                 ↓                  ↓
 Authorization       Eligibility       Medical Necessity
   Workflow           Workflow             Workflow
        ↓                 ↓                  ↓
 Claim/Auth DB       Eligibility DB     Claim + Clinical Docs
 validations         validations        + Payer Policy
        ↓                 ↓                  ↓
        └─────────────────┼──────────────────┘
                          ↓
             LEVEL 2: WORKFLOW AGENT
                          ↓
              Perform Allowed Checks
                          ↓
                 Collect Evidence
                          ↓
          Conditional Additional Checks
                          ↓
             Finding / Root Cause
                          ↓
              Recommended Action
                          ↓
              BILLER-FACING UI
```

# 16. Final Design Principle

The key architectural principle remains:

**“Deterministic at the routing level, agentic within the workflow.”**

For the current POC, we should therefore **not attempt to solve every possible denial category**.

The scope should focus specifically on:

1. **Authorization / Pre-Certification**
2. **Eligibility / Validation**
3. **Medical Necessity / Diagnosis–Procedure Mismatch**

Each category gets its own controlled workflow and access only to the tools/data sources relevant to that investigation.

This provides:

* Predictable behavior
* Better accuracy
* Reduced unnecessary RAG calls
* Lower latency
* Lower LLM cost
* Better auditability
* Easier debugging
* Clear biller visibility
* Controlled and safer AI decision-making
