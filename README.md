# 🏦 Yellow Bank Agent – Agentic AI Banking Bot

## 📌 Overview

Yellow Bank Agent is an Agentic AI-powered banking assistant built on the Yellow.ai platform.

The bot securely authenticates users, retrieves loan details through multi-step API workflows, optimizes token usage using a projection (middle-man) pattern, and renders results using Dynamic Rich Media (DRM). It also includes a CSAT feedback agent.

This project was developed as part of the Yellow.ai Agentic AI evaluation.

---

## 🎯 Objective

Design and implement a GenAI Banking Agent that:

- Authenticates users (Phone + DOB + OTP)
- Handles multi-step API workflows
- Implements token optimization (projection pattern)
- Displays Dynamic Rich Media (Loan cards)
- Handles edge cases
- Restricts conversation to English only
- Collects CSAT feedback

---

## 🧠 Architecture Flow

User  
↓  
Intent Recognition  
↓  
Collect Phone Number  
↓  
Collect Date of Birth  
↓  
Trigger OTP Workflow  
↓  
OTP Validation  
↓  
Get Loan Accounts API  
↓  
Projection Script (Token Optimization)  
↓  
Dynamic Rich Media Cards  
↓  
Loan Details API  
↓  
Quick Reply + CSAT Agent  

---

## 🚀 Features

### 1️⃣ Intent Recognition
Recognizes:
- "I want to check my loan details"
- "Show loan details"
- "Check my loan"

---

### 2️⃣ Authentication Flow

User must provide:
- Registered Phone Number (10 digits)
- Date of Birth
- OTP Verification

Supported mock OTP values:
- 1234
- 5678
- 7889
- 1209

---

### 3️⃣ Multi-Step Workflow

#### 🔹 Workflow A – triggerOTP
- Calls mock OTP API
- Returns generated OTP
- Validates user input OTP

#### 🔹 Workflow B – getLoanAccounts
- Calls API returning massive JSON (15+ fields)
- Applies projection script
- Displays Dynamic Rich Media cards

#### 🔹 Workflow C – loanDetails
- Accepts selected Loan ID
- Returns:
  - Tenure
  - Interest Rate
  - Principal Pending
  - Interest Pending
  - Nominee

---

## 🔥 Token Optimization (Critical Requirement)

The getLoanAccounts API returns a large JSON response.

To reduce token usage and prevent hallucinations, a projection pattern is used:

```javascript
var accounts = workflow.getLoanAccounts.response.body.accounts;

var projectedAccounts = [];

for (var i = 0; i < accounts.length; i++) {
    projectedAccounts.push({
        loan_id: accounts[i].loan_id,
        type: accounts[i].type,
        tenure: accounts[i].tenure
    });
}

projectedAccounts;
