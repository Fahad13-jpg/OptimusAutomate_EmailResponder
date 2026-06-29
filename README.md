# 📧 AI-Powered Email Responder

An intelligent email automation system built with **n8n** that reads incoming Gmail messages, classifies them using AI, and auto-generates context-aware draft replies — all without human intervention.

Built as part of my AI Automation Internship at **OptimusAutomate**.

---

## 🚀 What It Does

- Polls Gmail inbox every minute for unread emails
- **Pre-filters** automated/transactional emails (bank alerts, noreply senders, OTPs) without wasting any API calls
- Sends genuine human emails to a **single Groq AI agent** that:
  - Classifies the email (inquiry, complaint, support, feedback, spam, other)
  - Generates a short, context-aware reply (max 3 sentences)
  - Decides whether to reply or skip
- Saves AI-generated replies as **Gmail Drafts** for human review before sending
- Marks spam and automated emails as read and skips them silently

---

## 🏗️ Architecture

```
Gmail Trigger
  → Extract & Pre-filter        (noreply / transactional pattern detection)
      → Should Skip?
          YES → Mark as Read & Skip
          NO  → Build Agent Prompt
                → AI Email Agent (Groq — single LLM call)
                → Parse Agent Response
                → Should Reply?
                    YES → Build Draft → Save as Gmail Draft
                    NO  → Mark Spam as Read
```

---

## 🧠 AI Agent Logic

One Groq API call handles both classification and reply generation. The agent returns structured JSON:

```json
{
  "action": "reply",
  "category": "complaint",
  "urgency": "high",
  "confidence": 0.92,
  "reply": "We sincerely apologize for the issue you experienced..."
}
```

**Reply rules enforced in the prompt:**
- Max 3 sentences
- Tone matched to category (empathetic for complaints, direct for inquiries, etc.)
- Never invents specific details it doesn't have
- Falls back to: *"Your request has been forwarded to our management team"* when unsure

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| n8n | Workflow automation |
| Gmail API (OAuth2) | Read emails, save drafts, mark as read |
| Groq API | LLM inference (llama-3.3-70b-versatile) |

---

## 📋 Nodes Overview

| Node | Type | Purpose |
|---|---|---|
| Gmail Trigger | Trigger | Poll inbox every minute |
| Extract & Pre-filter | Code | Parse email fields + detect automated senders |
| Should Skip? | IF | Route automated emails away from AI |
| Mark as Read & Skip | Gmail | Silently dismiss automated emails |
| Build Agent Prompt | Code | Build Groq request body as JS object |
| AI Email Agent | HTTP Request | Single Groq API call |
| Parse Agent Response | Code | Parse JSON + apply fallback logic |
| Should Reply? | IF | Route spam/low-confidence to skip |
| Build Draft | Code | Format final reply with greeting + sign-off |
| Save as Gmail Draft | Gmail | Save draft in correct thread |
| Mark Spam as Read | Gmail | Clean up spam silently |

---

## ⚙️ Setup Instructions

### 1. Prerequisites
- n8n instance running (local or cloud)
- Gmail account with OAuth2 credentials configured in n8n
- Groq API key (free at [console.groq.com](https://console.groq.com))

### 2. Import Workflow
1. Open n8n → **Workflows** → **Import from file**
2. Upload `email_responder.json`

### 3. Configure Credentials
- **Gmail nodes** → select your Gmail OAuth2 credential
- **Groq nodes** → select your Groq API credential

### 4. Activate
- Toggle workflow to **Active**
- Workflow will now poll every minute automatically

---

## 📁 Pre-filter Patterns

The following are skipped before any AI call:

| Pattern | Example |
|---|---|
| noreply / no-reply in sender | `noreply@google.com` |
| notifications@ in sender | `notifications@github.com` |
| Transaction alert in subject | `Allied Bank Transaction Alert` |
| OTP / password reset | `Your OTP is 4521` |
| Bank transaction in body | `A Debit transaction of PKR 400` |

---

## 🔄 How to Extend

- Add a label to complaint emails for priority tracking
- Connect to a CRM (HubSpot, Notion) to log emails
- Add Slack notification for high-urgency emails
- Replace Gmail draft with auto-send for low-risk categories

---

## 👨‍💻 Author

**Muhammad Fahad**  
AI Automation Intern @ OptimusAutomate  
[LinkedIn](https://linkedin.com/in/your-profile) | [GitHub](https://github.com/your-username)
