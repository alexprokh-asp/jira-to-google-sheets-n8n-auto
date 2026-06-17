# Jira to Google Sheets AI-Powered Sync System (n8n + OpenAI)

🎥 **Demo:** 
![Architecture](doc/workflow.jpg)

---

## 📸 System Overview

### Problem Statement (Motivation)
In real QA workflows, maintaining a high-level summary spreadsheet of active bugs for stakeholders and management is critical. However, the traditional process introduces friction:

* QA engineers manually write and shorten structured bug reports to fit spreadsheet tables.
* Context is lost between highly technical JIRA tickets and business-oriented reports.
* Out-of-the-box integrations append duplicate rows instead of updating existing ones as bugs progress.
* Setting up real-time syncs usually requires global Jira Admin webhooks, which are blocked by corporate IT.

> ❌ **The Issue:** This process is time-consuming, repetitive, and does not scale well when the backlog grows.

To solve this, the reporting process was automated using n8n workflows, JQL polling, and AI-based summary evaluation.

### Solution Overview
This system transforms messy JIRA issues into a clean, structured Google Sheets bug tracking dashboard.

When the workflow is triggered, it:
1. Filters out everything except actual bugs using tight JQL.
2. Distills messy technical descriptions into a clean, 3-sentence executive summary via OpenAI.
3. Automatically classifies the bug category (*Functional* / *UI/UX* / etc.).
4. Extracts the specific system area or module (like *Deals* or *Agents*).
5. Normalizes all erratic LLM formats using a JS parser.
6. Updates existing rows in Google Sheets (by key matching) or creates new ones if they don't exist yet.

---

## 🏗️ Architecture

```text
Schedule Trigger (Every 2 Hours)
           ↓
Get Many Issues (Filter: Bug & updated >= -2h)
           ↓
Start Loop (Batch Size: 1)
           ↓
Jira JSON 2 MyFields (Extract key, priority, severity)
           ↓
AI Agent (OpenAI - Clean Description, Type, Area)
           ↓
Extract Add. Fields (Parse JSON & clean markdown)
           ↓
Edit Bugs Table (Google Sheets Upsert by 'Number')
```

---

## 🛠️ Key Technical Highlights

### 1. Bypass Admin Constraints (Polling with JQL)
Instead of waiting for IT to set up webhooks, the system relies on a **Schedule Trigger** combined with a tight, performance-friendly JQL filter:

```ini
project = "CRM" AND issuetype = "Bug" AND updated >= "-2h"
```

> 💡 **Benefit:** This reduces OpenAI API consumption and runs entirely on standard user-level API credentials.

### 2. High-Resiliency Loop Processing
By leveraging the **Loop Over Items** node, the workflow processes bugs one-by-one. If a network connection drops or a rate limit is reached, all previously processed bugs remain safely committed to your sheet.

### 3. Smart Parsing of LLM Outputs
A dedicated JavaScript node safely strips any Markdown formatting from the AI Agent response, handling potential JSON failures gracefully:

```javascript
// Очистка сырого ответа от LLM и парсинг JSON
aiRaw = aiRaw.replace(/```json|```/gi, '').trim();
parsedAi = JSON.parse(aiRaw);
```
