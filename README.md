# Prior Authorization Risk Engine

## 1. Executive Summary

The **Prior Authorization Risk Engine** is a healthcare claims and policy intelligence product designed to identify the likelihood that a requested medical service will require prior authorization, encounter documentation requirements, or create a high-risk utilization/coverage situation **before the service is performed or submitted**.

The product combines:

* CPT/HCPCS procedure codes
* ICD-10 diagnosis codes
* Provider information
* Place of service
* Patient/clinical context where available
* Payer and coverage policies
* CMS NCDs, LCDs, and Articles
* Historical claims patterns
* Procedure/diagnosis relationships
* Prior authorization rules where available
* Machine-learning risk signals
* Evidence-based LLM explanations

The core user experience is:

> **"Given this procedure, diagnosis, provider, payer, and context, what prior-authorization or documentation risks should I investigate before the service occurs?"**

The first POC should **not attempt to make an actual authorization decision**. Instead, it should function as a **risk and evidence engine** that identifies potential requirements and explains why a request may warrant additional review.

---

# 2. Product Concept

## Product Name

**Prior Authorization Risk Engine**

Possible portfolio-facing name:

> **PA Intelligence**

### Product tagline

> **Identify prior-authorization risk before the claim.**

Alternative:

> **Turn procedure, diagnosis, and coverage data into pre-service intelligence.**

---

# 3. Problem Being Solved

Prior authorization workflows frequently require users to bring together information from multiple places:

```text
Procedure code
      +
Diagnosis
      +
Payer
      +
Provider
      +
Place of service
      +
Coverage policy
      +
Clinical documentation
      +
Authorization rules
```

The problem is that this information is often fragmented.

A user may need to determine:

* Does this procedure require authorization?
* Is authorization dependent on diagnosis?
* Does the payer have a policy for this procedure?
* What documentation may be required?
* Does the patient's diagnosis appear consistent with the policy?
* Is this provider's utilization pattern unusual?
* Are there historical patterns associated with authorization problems?
* Which evidence should the reviewer examine?

The Prior Authorization Risk Engine brings these signals together.

---

# 4. Core User Question

The application should answer:

> **"If this procedure is being requested for this patient, what should I investigate before proceeding?"**

The output should be a **risk profile**, not an authorization determination.

Example:

```text
Procedure: MRI Lumbar Spine

Potential PA Risk: HIGH

Signals:
✓ Procedure commonly subject to coverage criteria
✓ Diagnosis does not clearly match selected policy criteria
✓ Documentation requirements identified
✓ Payer policy identified
✓ Provider utilization above peer benchmark

Recommended review:
→ Verify payer-specific authorization requirement
→ Verify diagnosis/indication
→ Review required documentation
→ Review applicable coverage policy
```

The application should always distinguish:

**"Potential risk"**

from:

**"Authorization is required."**

---

# 5. Target Users

Initial users:

* Health-plan analysts
* Utilization-management analysts
* Prior-authorization teams
* Healthcare data analysts
* Provider-network analysts
* Revenue-cycle analysts
* Claims analysts
* Product managers
* Clinical operations analysts

Future users could include:

* Provider organizations
* RCM companies
* Health plans
* TPA organizations
* Benefits administrators

---

# 6. Product Scope

The POC should focus on five capabilities.

## 1. Authorization Requirement Intelligence

Determine whether available policy/rule data suggests that authorization may be required.

## 2. Clinical/Policy Matching

Compare:

```text
Procedure
+
Diagnosis
```

against available policy relationships.

## 3. Documentation Intelligence

Identify documentation or clinical criteria referenced by applicable policies.

## 4. Historical Risk

Use claims and utilization patterns to identify characteristics associated with higher review risk.

## 5. Explainable Risk

Explain exactly why the engine generated the risk signal.

---

# 7. Product Architecture

```text
                    REQUEST
                       |
        ┌──────────────┼──────────────┐
        |              |              |
    Procedure       Diagnosis       Provider
        |              |              |
        └──────────────┼──────────────┘
                       |
                    Payer
                       |
                Place of Service
                       |
                       ▼
              Policy Retrieval
                       |
          ┌────────────┼────────────┐
          |            |            |
        NCD           LCD       Payer Policy
          |            |            |
          └────────────┼────────────┘
                       |
                Policy Matching
                       |
          ┌────────────┴────────────┐
          |                         |
     Clinical Risk            Utilization Risk
          |                         |
          └────────────┬────────────┘
                       |
                 Risk Engine
                       |
                ┌──────┴──────┐
                |             |
            Risk Score     Evidence
                |             |
                └──────┬──────┘
                       |
                       ▼
                  LLM Layer
                       |
                       ▼
                 Streamlit App
```

---

# 8. Technology Stack

Continue using the architecture from the CPT/HCPCS Procedure Intelligence project.

| Component        | Technology                |
| ---------------- | ------------------------- |
| Programming      | Python                    |
| Data processing  | pandas                    |
| Storage          | Parquet                   |
| Query engine     | DuckDB                    |
| Machine learning | scikit-learn              |
| Visualization    | Plotly                    |
| Application      | Streamlit                 |
| APIs             | requests                  |
| AI               | LLM API                   |
| Version control  | Git/GitHub                |
| Deployment       | Streamlit Community Cloud |

No Snowflake is required for the POC.

---

# 9. Data Sources

The engine requires several categories of information.

## Procedure Data

* CPT
* HCPCS
* Procedure categories
* Effective dates

## Diagnosis Data

* ICD-10-CM
* Diagnosis categories
* Effective dates

## Coverage Data

* NCD
* LCD
* Articles
* CMS coverage relationships

## Claims Data

* Procedure
* Diagnosis
* Provider
* Place of service
* Date
* Units
* Allowed amount
* Paid amount

## Provider Data

Where available:

* Provider specialty
* Provider type
* Geography
* Organization

## Payer Policy Data

This becomes increasingly important in future versions.

The POC can begin with CMS Medicare policy and clearly label it as such.

Commercial payer policies should not be represented as Medicare policy.

---

# 10. Key Data Model

Create a standardized request-level DataFrame.

## `pa_request_df`

```text
request_id
procedure_code
code_system
diagnosis_code
provider_id
provider_specialty
payer_id
place_of_service
request_date
patient_age
patient_sex
authorization_status
```

For the synthetic POC, some fields may be simulated.

That is acceptable if the application clearly labels them as:

> **Synthetic demonstration data**

---

# 11. Policy Data Model

Create:

## `policy_df`

```text
policy_id
payer_id
policy_type
policy_title
procedure_code
diagnosis_code
authorization_required
coverage_status
documentation_required
clinical_criteria
effective_date
termination_date
jurisdiction
source_url
source_text
```

Possible policy types:

```text
NCD
LCD
Article
Payer Policy
Medical Policy
PA Rule
```

---

# 12. Authorization Rule Model

Create:

## `authorization_rule_df`

```text
payer_id
procedure_code
procedure_category
requires_pa
pa_condition
diagnosis_condition
place_of_service_condition
provider_condition
effective_date
termination_date
source
```

Example:

```text
Payer: Example Payer
Procedure: XYZ
Requires PA: Yes
Condition: Outpatient
Diagnosis: Selected diagnoses
Documentation: Required
```

---

# 13. Evidence Model

Every risk signal should have evidence.

Create:

## `risk_evidence_df`

```text
request_id
risk_signal
evidence_type
evidence_value
source
source_url
source_date
confidence
```

Example:

```text
request_123
PA_REQUIREMENT
Procedure appears in policy
CMS LCD
LCD-XXXXX
High
```

This becomes extremely important when explaining the model.

---

# 14. Risk Categories

Do not start with one generic risk score.

Create multiple risk dimensions.

## A. Authorization Risk

Potential likelihood that PA requirements apply.

## B. Clinical Criteria Risk

Degree to which the procedure/diagnosis combination differs from identified policy criteria.

## C. Documentation Risk

Potential missing or unspecified documentation requirements.

## D. Coverage Risk

Potential coverage-policy mismatch.

## E. Utilization Risk

Whether the provider or patient pattern differs from relevant benchmarks.

## F. Policy Ambiguity

Whether multiple or conflicting rules exist.

---

# 15. Example Risk Output

```text
Prior Authorization Risk

Overall: HIGH

Authorization Risk: HIGH
Clinical Criteria Risk: MEDIUM
Documentation Risk: HIGH
Coverage Risk: MEDIUM
Utilization Risk: LOW

Primary reasons:

1. Procedure is associated with a PA policy.
2. Selected diagnosis is not clearly represented in the identified criteria.
3. Policy identifies documentation requirements.
4. Policy effective date applies to the request date.
```

Again, this is a **risk assessment**, not an authorization determination.

---

# 16. Risk Scoring

Start with a transparent rules-based score.

For example:

```text
PA rule identified                 +40
Procedure requires PA              +30
Diagnosis-policy mismatch          +15
Documentation requirement          +10
Utilization signal                  +5
Policy ambiguity                    +5
```

Normalize:

```text
0–24   LOW
25–49  MODERATE
50–74  HIGH
75–100 VERY HIGH
```

However, these thresholds should be treated as **POC configuration**, not clinically validated thresholds.

---

# 17. Why Rules First?

A rules engine makes the first version:

* explainable
* testable
* easy to debug
* easy to demonstrate
* easier to validate against policy

Do not begin with a black-box neural network.

The first question is:

> **Can I reliably reproduce the policy logic?**

Only after that should machine learning be added.

---

# 18. Machine Learning Layer

After the rules engine works, add ML.

The ML model can estimate:

> **Likelihood that a request will require additional review based on historical patterns.**

Potential features:

```text
procedure
diagnosis
provider
specialty
place_of_service
historical procedure volume
provider procedure rate
diagnosis/procedure frequency
patient procedure history
procedure frequency
prior authorization history
historical claim outcome
```

---

# 19. Important Distinction

The ML target should not initially be:

```text
"Will this procedure be approved?"
```

That introduces significant complexity and requires appropriately labeled authorization decisions.

Instead, begin with:

```text
"Does this request resemble historical requests
that required additional review?"
```

That is much more appropriate for the initial POC.

---

# 20. Synthetic Data Strategy

The CMS synthetic claims data does not contain everything required for a genuine PA model.

Therefore create a synthetic authorization dataset.

Example:

```text
request_id
procedure_code
diagnosis_code
provider_id
payer_id
place_of_service
documentation_complete
pa_required
review_required
approval_status
```

Clearly label this data:

> **Synthetic demonstration data — not representative of actual payer authorization behavior.**

This allows you to demonstrate the architecture without pretending the model has real-world authorization performance.

---

# 21. Feature Engineering

Create features at three levels.

## Request-level

```text
procedure
diagnosis
POS
payer
patient age
```

## Provider-level

```text
procedure_volume
procedure_rate
peer_percentile
diagnosis_distribution
historical_review_rate
```

## Policy-level

```text
pa_required
criteria_match
documentation_required
policy_age
policy_jurisdiction
```

---

# 22. Initial ML Models

Test progressively.

### Model 1

Logistic Regression

Advantages:

* Explainable
* Fast
* Easy to evaluate

### Model 2

Random Forest

Useful for nonlinear relationships.

### Model 3

Gradient Boosting

Potentially stronger predictive performance.

Do not introduce complex deep learning until there is sufficient labeled data.

---

# 23. Model Output

The model should produce:

```text
risk_probability
risk_band
top_contributing_features
```

Example:

```text
Risk probability: 0.78

Risk band: HIGH

Top contributing factors:

+ Procedure historically associated with PA
+ Diagnosis-policy mismatch
+ Documentation requirement
+ Provider utilization above peer benchmark
```

Do not call the number a validated probability until the model has been properly calibrated and evaluated.

---

# 24. Explainability

For ML models, use:

* Feature importance
* Logistic regression coefficients
* SHAP in a future version if appropriate

The UI should explain:

> **Why did this request receive a high-risk signal?**

rather than simply displaying:

> `Risk = 0.83`

---

# 25. LLM Layer

The LLM should sit **after** the analytical engine.

Architecture:

```text
Request
   ↓
Rules
   ↓
Policy matching
   ↓
ML
   ↓
Evidence
   ↓
LLM
```

The LLM should not independently decide whether authorization is required.

---

# 26. LLM Prompt

Use an evidence-first prompt.

```text
You are a healthcare prior-authorization
intelligence assistant.

Use only the evidence supplied below.

Do not make a final authorization decision.
Do not infer medical necessity.
Do not invent payer requirements.
Do not state that authorization is required unless
the supplied policy explicitly supports that conclusion.

Explain:
1. What risk signals were identified.
2. What policy evidence supports them.
3. What information should be verified.
4. What limitations apply.
```

---

# 27. AI Response

Example:

## Risk Summary

The selected procedure is associated with a policy containing prior-authorization requirements.

## Evidence

* Procedure appears in policy XYZ.
* The policy identifies specific clinical criteria.
* The selected diagnosis does not clearly match the listed criteria.
* Additional documentation is identified.

## Recommended Verification

* Confirm the patient's payer.
* Confirm the policy is effective for the request date.
* Verify the diagnosis and clinical documentation.
* Confirm the payer's current authorization workflow.

## Limitation

The engine provides decision support and does not determine medical necessity or final authorization status.

---

# 28. Streamlit Application

Create the following pages.

```text
PA Risk Assessment
Procedure Intelligence
Coverage & Policy
Provider Intelligence
Evidence Explorer
Model Insights
Methodology
```

---

# 29. Page 1 — PA Risk Assessment

This is the primary product experience.

Inputs:

```text
Payer
Procedure
Diagnosis
Provider
Place of Service
Request Date
```

Optional:

```text
Age
Sex
Previous procedure
Documentation status
```

Click:

> **Assess PA Risk**

---

# 30. Risk Assessment Dashboard

Display:

```text
                 PA RISK
                  HIGH

Authorization       HIGH
Clinical Criteria   MEDIUM
Documentation       HIGH
Coverage            MEDIUM
Utilization         LOW
```

Then:

### Why?

Show the top evidence-backed signals.

---

# 31. Page 2 — Procedure Intelligence

Reuse your existing Procedure Intelligence work.

Display:

* Procedure description
* Common diagnoses
* Utilization
* Providers
* Place of service
* Historical trends

This page becomes a supporting intelligence layer for the PA engine.

---

# 32. Page 3 — Coverage & Policy

Display:

```text
Policy
Policy type
Effective date
Jurisdiction
Procedure
Diagnosis
Authorization requirement
Clinical criteria
Documentation requirements
Source
```

Allow the user to open the original source.

---

# 33. Page 4 — Provider Intelligence

Display:

```text
Provider
Specialty
Procedure volume
Peer comparison
Procedure rate
Diagnosis distribution
Historical utilization
```

The goal is not to label providers as problematic.

Instead:

> **"How does this request compare with observed provider and peer patterns?"**

---

# 34. Page 5 — Evidence Explorer

This is one of the most important pages.

For every risk signal:

```text
Risk Signal
     ↓
Evidence
     ↓
Source
     ↓
Policy
     ↓
Effective Date
```

Example:

```text
Risk signal:
Authorization requirement

Evidence:
Procedure XYZ appears in policy ABC.

Source:
CMS LCD ABC

Effective:
2026-01-01

Source document:
[View policy]
```

---

# 35. Page 6 — Model Insights

Display:

* Model version
* Training dataset
* Feature importance
* Performance metrics
* Calibration
* False positives
* False negatives

This makes the project much more credible as a data-science portfolio project.

---

# 36. Data Pipeline

The pipeline should be:

```text
CMS / Payer Sources
        ↓
Raw Files
        ↓
Validation
        ↓
Standardization
        ↓
Parquet
        ↓
Policy Matching
        ↓
Feature Engineering
        ↓
Risk Engine
        ↓
ML
        ↓
Evidence Store
        ↓
Streamlit
```

---

# 37. Directory Structure

Build on the existing project.

```text
prior-auth-risk-engine/
│
├── app.py
├── requirements.txt
├── README.md
├── .gitignore
│
├── data/
│   ├── raw/
│   ├── processed/
│   └── derived/
│
├── src/
│   ├── config.py
│   ├── ingest.py
│   ├── claims.py
│   ├── procedures.py
│   ├── diagnoses.py
│   ├── policies.py
│   ├── policy_matcher.py
│   ├── features.py
│   ├── rules_engine.py
│   ├── risk_engine.py
│   ├── model.py
│   ├── explain.py
│   └── llm.py
│
├── models/
│
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_policy_matching.ipynb
│   ├── 03_feature_engineering.ipynb
│   └── 04_modeling.ipynb
│
└── tests/
    ├── test_policy_matcher.py
    ├── test_rules_engine.py
    └── test_risk_engine.py
```

---

# 38. Core Python Components

## `policy_matcher.py`

Responsible for:

```text
Procedure → Policy
Diagnosis → Policy
Procedure + Diagnosis → Policy criteria
```

---

## `rules_engine.py`

Responsible for:

```text
PA requirement
Documentation requirement
Coverage signal
Clinical criteria signal
```

---

## `risk_engine.py`

Combines:

```text
Rules
+
Policy matching
+
Utilization
+
ML
```

into the final risk profile.

---

## `llm.py`

Responsible only for:

```text
Evidence → Explanation
```

not:

```text
Raw request → decision
```

---

# 39. Example End-to-End Scenario

User enters:

```text
Payer:
Example Medicare Advantage Plan

Procedure:
MRI lumbar spine

Diagnosis:
Low back pain

Provider:
Provider 123

Place of service:
Outpatient

Request date:
2026-09-15
```

The system performs:

### Step 1

Identify procedure.

### Step 2

Identify payer policy.

### Step 3

Determine whether procedure appears in PA rules.

### Step 4

Match diagnosis to policy criteria.

### Step 5

Identify documentation requirements.

### Step 6

Calculate provider utilization signals.

### Step 7

Calculate overall risk.

### Step 8

Retrieve supporting evidence.

### Step 9

Generate explanation.

---

# 40. Example Final Result

```text
PA RISK ASSESSMENT
------------------

Overall Risk: HIGH

Authorization Risk: HIGH
Clinical Criteria: MEDIUM
Documentation: HIGH
Coverage: MEDIUM
Utilization: LOW


WHY THIS REQUEST WAS FLAGGED

1. The selected procedure is associated with an authorization
   requirement in the identified policy.

2. The policy specifies clinical criteria that should be verified.

3. Additional documentation is identified by the policy.

4. The selected diagnosis does not clearly establish that the
   policy criteria are satisfied.


WHAT TO VERIFY

□ Confirm current payer policy
□ Confirm authorization requirement
□ Verify diagnosis
□ Review clinical documentation
□ Verify policy effective date
□ Confirm place-of-service requirements


EVIDENCE

Policy: XXXXX
Effective: 2026-01-01
Procedure: XXXXX
Source: CMS/Payer policy
```

---

# 41. What Makes This Product Different

A basic PA tool might answer:

> "Does this CPT require authorization?"

Your product should answer a much richer question:

> **"Why might this request require additional review, what evidence supports that assessment, what information is missing, and what should the analyst verify?"**

That is the intelligence layer.

---

# 42. MVP Definition

Do not attempt to build everything initially.

The MVP should contain only:

### Input

```text
Procedure
Diagnosis
Payer
Date
Place of service
```

### Engine

```text
Policy lookup
+
Procedure/diagnosis matching
+
Rules-based risk
```

### Output

```text
Risk level
+
Risk reasons
+
Policy evidence
+
Recommended verification steps
```

### AI

One grounded explanation.

That is enough for a strong first demo.

---

# 43. Phase 2

Add:

* Provider intelligence
* Peer comparisons
* Historical utilization
* Documentation prediction
* More sophisticated policy matching
* Commercial payer policies

---

# 44. Phase 3

Add:

* Historical authorization outcomes
* ML risk model
* Model explainability
* Calibration
* Payer-specific models
* Provider-specific patterns

---

# 45. Phase 4

Add workflow integration:

```text
PA request received
       ↓
Risk Engine
       ↓
Policy retrieval
       ↓
Documentation checklist
       ↓
Analyst review
       ↓
Authorization workflow
```

Eventually the product could generate:

> **Pre-authorization submission checklist**

based on the applicable policy.

---

# 46. Future Product Expansion

The architecture can eventually support:

## Documentation Intelligence

> "What documentation is likely to be needed?"

## Medical Policy Intelligence

> "Which policy applies?"

## Denial Prevention

> "What characteristics are associated with downstream denial?"

## Provider Intelligence

> "How does this provider's utilization compare with peers?"

## Claims Risk

> "What downstream claim risk should be investigated?"

## Appeals Intelligence

> "Which policy criteria and documentation support an appeal?"

---

# 47. Important Guardrails

The product must not represent itself as a clinical or authorization decision-maker.

Avoid:

> "Authorization will be denied."

Instead:

> "This request has signals associated with additional authorization review."

Avoid:

> "The patient does not meet medical necessity."

Instead:

> "The available information does not establish that the identified policy criteria are satisfied."

Avoid:

> "This provider is high risk."

Instead:

> "The provider's utilization differs from the selected peer benchmark."

---

# 48. Model Evaluation

When real labeled data becomes available, evaluate:

### Classification

* Precision
* Recall
* F1
* ROC-AUC
* PR-AUC

### Calibration

* Calibration curve
* Brier score

### Operational

* Review rate
* False-positive rate
* False-negative rate
* Cases identified earlier
* Analyst workload reduction

### Fairness / subgroup analysis

Evaluate performance across relevant groups where appropriate.

---

# 49. Success Criteria

The POC succeeds if a user can enter:

```text
Procedure
+
Diagnosis
+
Payer
+
Date
```

and receive:

1. A transparent PA risk assessment
2. The policies that triggered the assessment
3. Procedure/diagnosis matching evidence
4. Documentation requirements
5. Utilization context
6. Clear reasons for the risk signal
7. A grounded AI explanation
8. Links back to source evidence

---

# 50. Eight-Week Development Plan

## Week 1 — Foundation

Build:

* Repository
* Data structure
* CMS claims ingestion
* Procedure reference
* ICD-10 reference
* Policy data model

**Deliverable:** standardized local datasets.

---

## Week 2 — Policy Intelligence

Build:

* Policy ingestion
* Procedure matching
* Diagnosis matching
* Effective-date logic
* Policy evidence

**Deliverable:** procedure → policy engine.

---

## Week 3 — Rules Engine

Build:

* PA rules
* Documentation rules
* Clinical criteria signals
* Coverage signals

**Deliverable:** deterministic PA risk engine.

---

## Week 4 — Streamlit MVP

Build:

* Request form
* Risk dashboard
* Policy evidence
* Explanation panel

**Deliverable:** working interactive application.

---

## Week 5 — Provider Intelligence

Add:

* Provider utilization
* Peer groups
* Procedure rates
* Historical patterns

**Deliverable:** provider context.

---

## Week 6 — ML

Build:

* Feature engineering
* Synthetic authorization dataset
* Logistic regression
* Random forest comparison
* Model evaluation

**Deliverable:** experimental ML risk model.

---

## Week 7 — LLM

Build:

* Evidence aggregation
* Structured prompt
* AI explanation
* Source references
* Guardrails

**Deliverable:** evidence-grounded PA assistant.

---

## Week 8 — Deployment

Build:

* Demo dataset
* GitHub repository
* README
* Architecture diagram
* Screenshots
* Streamlit deployment

**Deliverable:** public portfolio application.

---

# 51. Final Product Vision

The ultimate product is:

```text
              PRIOR AUTH REQUEST
                      |
        ┌─────────────┼──────────────┐
        |             |              |
     Procedure     Diagnosis       Payer
        |             |              |
        └─────────────┼──────────────┘
                      |
               Policy Intelligence
                      |
             Clinical Rule Matching
                      |
             Documentation Analysis
                      |
              Provider Intelligence
                      |
                Risk Engine
                      |
              Evidence Layer
                      |
                     LLM
                      |
                      ▼
          PRIOR AUTH INTELLIGENCE
```

The product should ultimately answer five questions:

> **1. Does this request potentially require authorization?**

> **2. What policy or rule supports that assessment?**

> **3. Does the procedure/diagnosis combination align with the identified criteria?**

> **4. What documentation or information should be verified?**

> **5. Why did the engine flag this request?**

### The strategic relationship to your existing project

Your original **CPT/HCPCS Procedure Intelligence** project becomes the foundation:

```text
                 CPT/HCPCS
                     ↓
          Procedure Intelligence
                     ↓
           Coverage Intelligence
                     ↓
        Prior Authorization Engine
                     ↓
          Provider/Utilization Risk
                     ↓
             Denial Prevention
```

So you would not be throwing away your first project. **The Prior Authorization Risk Engine is the next product layer built on top of it.**
