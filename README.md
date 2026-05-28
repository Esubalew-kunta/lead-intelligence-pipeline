# Candid — Lead Intelligence Pipeline

> n8n automation that turns a raw form submission into a scored lead, a pre-call brief, a HubSpot record, and a Slack-approved confirmation email — before the sales rep opens their laptop.

---

## Watch it run

[![Watch the walkthrough](https://img.shields.io/badge/▶_Watch_Demo-Video-red?style=for-the-badge)](YOUR_VIDEO_LINK_HERE)

---

## What it does — STAR


### Situation

Candid's sales team receives strategy session requests through a Webflow form. Before this pipeline, each submission meant manual research: Google the company, check HubSpot for duplicates, decide how urgent the lead is, write a follow-up email from scratch. A rep could spend 30–45 minutes on a single lead — before knowing if it was even worth the call.

### Task

Every new form submission needed to be:
- validated and GDPR-checked
- deduplicated against the existing CRM
- enriched with company-level data (headcount, industry, revenue, technologies)
- scored against Candid's ideal customer profile across six weighted dimensions
- accompanied by a written pre-call brief covering who the person is, why they applied, which product fits, and what objections to expect
- logged in HubSpot as a contact and a deal
- escalated to the right Slack channel based on score tier
- sent through a human-approval step before any confirmation email goes out

### Action

Two n8n workflows handle this end to end:

**Workflow 1 — Lead Intelligence Pipeline**

1. **Webhook** receives the Webflow form POST
2. **Prepare Lead Data** extracts and sanitizes all fields; throws if required fields are missing
3. **GDPR Gate** stops processing if the applicant did not tick consent — no data flows forward, no record is created
4. **HubSpot Duplicate Check** queries the CRM by email; duplicate submissions fire a Slack alert to the team and exit cleanly
5. **Apollo.io Enrichment** pulls company headcount, industry, country, revenue, description, and tech stack from the email domain
6. **Google News** fetches the five most recent headlines about the company from the RSS feed
7. **Build AI Context** assembles two structured prompts — one for scoring, one for the brief — combining form data, Apollo fields, and news headlines
8. **Score Lead** (Groq / GPT) returns a JSON object with scores across six dimensions — contact seniority (25%), company size (20%), industry match (20%), geography (15%), buying signals (15%), red flags (5%) — plus a challenge-keyword bonus. Output: a weighted total out of 10 and a tier label: Hot / Warm / Cold
9. **Write Brief** (Groq / GPT) produces a structured pre-call document: who this person is, what their company does, recent news that's relevant, why they applied, which Candid product fits, a word-for-word opening line, and two to three likely objections with counters
10. **HubSpot Create Contact + Deal** writes the enriched record and opens a deal at the right priority level
11. **Prepare Slack Alerts** builds tier-specific messages — Hot leads get a full breakdown with score evidence; Warm leads get a condensed version
12. **Slack Team Alert** posts to the priority channel for Hot leads, the general channel for Warm leads
13. **Slack Email Approval** posts the draft confirmation email as an interactive Block Kit message with Approve, Edit, and Reject buttons

**Workflow 2 — Email Approval Handler**

Listens for Slack button interactions on the approval message:

- **Approve** — posts a thread confirmation, then sends the email via Gmail
- **Edit** — opens a Slack modal pre-filled with the subject and body; on submit, sends the revised version and confirms in-thread
- **Reject** — logs the rejection in-thread; no email goes out

### Result

A rep receives a Slack message within seconds of a form submission — scored, researched, briefed, and with a one-click send on the confirmation email. Hot leads surface immediately with full context. Cold leads are logged in HubSpot without generating noise. No manual research, no copy-paste, no missed follow-ups.

---

## Stack

| Layer | Tool |
|---|---|
| Automation runtime | n8n |
| Form capture | Webflow webhook |
| CRM | HubSpot (Contacts + Deals) |
| Company enrichment | Apollo.io Organizations API |
| News | Google News RSS |
| LLM scoring + briefs | Groq (GPT OSS 120B) |
| Team notifications | Slack Block Kit |
| Email delivery | Gmail via OAuth2 |

---

## Workflows

| File | Description |
|---|---|
| `Lead Intelligence Pipeline.json` | Main pipeline — form intake to Slack alert |
| `email-approval.json` | Slack interaction handler — approval, edit, reject |

---

## Setup

1. Import both JSON files into your n8n instance
2. Set your credentials:
   - HubSpot private app token
   - Apollo.io API key
   - Groq API key
   - Slack bot token (with `chat:write`, `views:open` scopes)
   - Gmail OAuth2
3. Point your Webflow form to the webhook URL: `/webhook/candid-strategy-session`
4. Point your Slack app's Interactivity Request URL to: `/webhook/candid-email-approval`
5. Activate both workflows

---

## Lead Scoring Dimensions

| Dimension | Weight | Top Score Criteria |
|---|---|---|
| Contact Seniority | 25% | CMO, Head of Marketing, CEO, Founder |
| Company Size | 20% | 200–10,000 employees |
| Industry Match | 20% | Consumer & Retail, Financial Services, F&B, Automotive, Healthcare |
| Geography | 15% | Netherlands or UK |
| Buying Signals | 15% | Recent funding, growth announcements, digital transformation news |
| Red Flag Check | 5% | No competitor, student, or freelancer signals |

Challenge keyword bonus: +0.5 if the stated challenge mentions fragmented tools, campaign speed, briefing bottlenecks, or agency management.
