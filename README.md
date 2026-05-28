# Lead Intelligence Pipeline

> n8n automation that turns a raw inbound form submission into a scored lead, a pre-call brief, a CRM record, and a human-approved confirmation email — before a sales rep opens their laptop.

---

## Watch it run

[![Watch the walkthrough](https://img.shields.io/badge/▶_Watch_Demo-Video-red?style=for-the-badge)](YOUR_VIDEO_LINK_HERE)

---

## What it does — STAR

### Situation

B2B sales teams that run discovery or strategy sessions get inbound requests through a website form. Before a pipeline like this, every submission meant manual work: look up the company, check if the contact already exists in the CRM, judge how qualified the lead is, and write a follow-up email from scratch. A rep could burn 30–45 minutes per lead — before knowing if it was worth a call at all.

### Task

Every new form submission needed to be processed without manual intervention:
- validated and GDPR-checked before any data moves
- deduplicated against the existing CRM
- enriched with company-level data — headcount, industry, revenue, tech stack
- scored against a defined ideal customer profile across six weighted dimensions
- accompanied by a written pre-call brief covering who the contact is, why they reached out, which offering fits best, and what objections to expect
- logged in the CRM as a contact and a deal
- escalated to the right team channel based on score tier
- held in a human-approval step before any confirmation email goes out to the lead

### Action

Two n8n workflows handle this end to end:

**Workflow 1 — Lead Intelligence Pipeline**

1. **Webhook** receives the form POST from the website
2. **Prepare Lead Data** extracts and sanitizes all fields; throws a named error if required fields are missing
3. **GDPR Gate** stops the run entirely if the applicant did not tick consent — no data is stored, no record is created
4. **CRM Duplicate Check** queries existing contacts by email; duplicate submissions fire a Slack alert to the team and exit cleanly without creating a duplicate record
5. **Company Enrichment** pulls headcount, industry, country, annual revenue, description, and tech stack from the email domain via the Apollo.io API
6. **Google News** fetches the five most recent headlines about the company via RSS
7. **Build Context** assembles two structured prompts — one for scoring, one for the brief — combining form data, enrichment fields, and news headlines
8. **Score Lead** sends the scoring prompt to an LLM and gets back a JSON object with scores across six dimensions plus a weighted total out of 10 and a tier label: Hot / Warm / Cold
9. **Write Brief** sends the brief prompt to the LLM and gets back a structured pre-call document: who the person is, what their company does, relevant recent news, why they applied, which product fits best, a word-for-word opening line, and two to three likely objections with counters
10. **CRM Contact + Deal** writes the enriched record and opens a deal at the right priority level
11. **Prepare Slack Alerts** builds tier-specific messages — Hot leads get a full breakdown with score evidence and a suggested opening line; Warm leads get a condensed version
12. **Slack Team Alert** posts to the priority channel for Hot leads, the general channel for Warm
13. **Slack Email Approval** posts the draft confirmation email as an interactive Block Kit message with Approve, Edit, and Reject buttons

**Workflow 2 — Email Approval Handler**

Listens for Slack button interactions on the approval message:

- **Approve** — posts a thread confirmation and sends the email immediately
- **Edit** — opens a Slack modal pre-filled with the subject and body; on submit, sends the revised version and confirms in-thread
- **Reject** — logs the rejection in-thread; no email is sent

### Result

A sales rep gets a Slack message within seconds of a form submission — complete with a score, enriched company data, a full pre-call brief, and a one-click send on the confirmation email. Hot leads surface immediately with everything needed for the call. Cold leads are logged without creating noise. No manual research, no copy-pasting, no missed follow-ups.

---

## Stack

| Layer | Tool |
|---|---|
| Automation runtime | n8n |
| Form capture | Webhook (any form provider) |
| CRM | HubSpot (Contacts + Deals) |
| Company enrichment | Apollo.io Organizations API |
| News | Google News RSS |
| LLM scoring + briefs | Groq API |
| Team notifications | Slack Block Kit |
| Email delivery | Gmail via OAuth2 |

---

## Workflows

| File | Description |
|---|---|
| `Lead Intelligence Pipeline.json` | Main pipeline — form intake through Slack alert |
| `email-approval.json` | Slack interaction handler — approve, edit, or reject the outbound email |

---

## Setup

1. Import both JSON files into your n8n instance
2. Add your credentials:
   - HubSpot private app token
   - Apollo.io API key
   - Groq API key
   - Slack bot token (scopes: `chat:write`, `views:open`)
   - Gmail OAuth2
3. Update the scoring prompt in the **Build Context** node to match your ideal customer profile
4. Point your website form to the webhook URL: `/webhook/lead-intake`
5. Point your Slack app's Interactivity Request URL to: `/webhook/email-approval`
6. Activate both workflows

---

## Lead Scoring Dimensions

The scoring model is fully configurable in the **Build Context** node. The default weights are:

| Dimension | Weight | Logic |
|---|---|---|
| Contact Seniority | 25% | Decision-maker roles score highest |
| Company Size | 20% | Configurable headcount range |
| Industry Match | 20% | Define your target verticals |
| Geography | 15% | Define your target markets |
| Buying Signals | 15% | Recent funding, growth, or digital transformation news |
| Red Flag Check | 5% | Filters competitors, students, freelancers |

A keyword bonus applies if the stated challenge aligns with specific pain points you define.
