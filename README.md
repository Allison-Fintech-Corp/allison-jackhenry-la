# Allison + AgentScope: Orchestrating Fair Lending Decisions

## Overview
Allison is an **intent-first orchestration layer** built on [AgentScope](https://github.com/agentscope-ai/agentscope).  
It connects **LoanVantage (Jack Henry)**, **Stratyfy (bias-detection & explainable ML)**, and a bank’s **core (Core Director)** into a seamless workflow for **bias-aware, regulator-ready lending**.

---

## Business Context

**The Problem**
- Banks rely on **LoanVantage** for loan origination and credit decisioning.  
- **Stratyfy** provides advanced bias-detection algorithms and explainable ML models.  
- Today, Stratyfy struggles to integrate into LoanVantage due to limited API hook points.  
- Regulators demand **auditable, bias-aware decisions** in lending workflows.  

**The Solution**
- Allison sits in the middle, powered by AgentScope, orchestrating between LoanVantage, Stratyfy, and Core Director.  
- Every decision is enriched, bias-checked, explained, and written back to LoanVantage—complete with **audit trails**.  

**The Outcome**
- **Banks**: Compliance by design. Bias metrics and explanations are always attached to lending decisions.  
- **Stratyfy**: Plug-in distribution to Jack Henry’s LoanVantage banks without wrestling APIs.  
- **Jack Henry**: LoanVantage + Core Director gain new capabilities with zero changes required.  
- **Allison**: Becomes the **decision air-traffic controller**—auditor-ready, explainable, and intent-first.  

---

## High-Level Architecture

**Actors**
- LoanVantage (REST provider via jXchange REST)  
- Core Director (SOAP via jXchange)  
- Stratyfy (bias-detection & explainable ML APIs)  
- Allison (AgentScope orchestration + audit layer)  

**Flow**
1. Loan application reaches a decision stage in LoanVantage.  
2. Allison fetches the application (`lv_application_inquire`).  
3. Allison enriches with Core Director context (`cd_customer_context`).  
4. Allison calls Stratyfy (`stratyfy_assess`) with normalized features.  
5. Stratyfy returns decision + bias metrics + explanations.  
6. Allison writes results back into LoanVantage (`lv_document_create`) and stores an **evidence pack**.  
7. Compliance officers can query Allison (“why was this loan approved?”).  

---

## Why AgentScope

- **Intent-First Tools**: Define actions like `assess_loan_app` or `explain_decision`, not raw API calls.  
- **Adapters**: Swap mock adapters for real ones—demo and production use the same flows.  
- **Streaming & Async**: Real-time triggers and responses.  
- **Transparency**: Every tool call, input, output, and correlation ID is logged.  
- **Extensible**: Easy to add more vendors, rails, or compliance modules.  

---

## Core Intents (Tools)

| Intent                   | Description                                      | Backend Adapter            |
|--------------------------|--------------------------------------------------|----------------------------|
| `lv_application_inquire` | Fetch a loan application from LoanVantage         | LoanVantage REST (ApplicationInquire) |
| `cd_customer_context`    | Enrich app with Core Director customer/account data | jXchange SOAP (AcctInq, CustInq, AcctHistSrch) |
| `stratyfy_assess`        | Run Stratyfy bias-detection + explainable decision | Stratyfy REST API |
| `lv_document_create`     | Write decision + bias evidence back into LoanVantage | LoanVantage REST (DocumentCreate) |
| `audit_record`           | Persist an immutable evidence pack in Allison ledger | Postgres + object store |

---

## Example Data Contracts

**Normalized input to Stratyfy**
```json
{
  "app_id": "LV-12345",
  "customer": {"id": "CUST-555", "dob": "1987-04-02"},
  "loan": {"product":"Auto", "amount": 25000, "term_months": 36},
  "credit": {"bureau_score": 688, "nsf_12m": 1},
  "account_history": {"avg_balance_90d": 2300, "dda_tenure_months": 28},
  "application_meta": {"channel":"branch", "submitted_ts":"2025-09-16T14:36:00Z"}
}
Stratyfy output

json
Copy code
{
  "decision": "CONDITIONAL_APPROVE",
  "explanations": ["DTI slightly high", "Stable balances mitigate risk"],
  "bias_checks": {
    "metrics": {"pp_diff": 0.06, "four_fifths_rule": "PASS"},
    "protected_proxies": ["geo_tract", "age"]
  },
  "model_version": "stratyfy-uw-2.3.1"
}
