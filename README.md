# Lead Intelligence Pipeline

An n8n automation that takes a raw inbound form submission and turns it into a scored lead, a pre-call brief, a CRM record, and a human-approved confirmation email before a sales rep even opens their laptop.

---

## Watch it run

[![Watch the walkthrough](https://img.shields.io/badge/▶_Watch_Demo-Video-red?style=for-the-badge)](YOUR_VIDEO_LINK_HERE)

---

## What it does (STAR)

### Situation

B2B sales teams that run discovery or strategy sessions get inbound requests through a website form. Every submission used to mean manual work: look up the company, check if the contact already exists in the CRM, judge how qualified the lead is, write a follow-up email from scratch. A rep could burn 30 to 45 minutes per lead before knowing if it was worth a call at all.

### Task

Every new form submission needed to flow through a set of checks and actions without anyone touching it:

- Validate the data and confirm GDPR consent before anything moves
- Check for duplicates in the CRM
- Pull company data from a third-party enrichment API
- Score the lead against a defined ideal customer profile
- Write a pre-call brief with context on who the person is, why they reached out, and what to expect
- Log a contact and a deal in the CRM
- Alert the sales team in Slack with the right level of urgency
- Hold the confirmation email for human review before it goes out

### Action

Two n8n workflows handle this from start to finish.

**Workflow 1: Lead Intelligence Pipeline**

1. A webhook receives the form POST from the website
2. Lead data is extracted and sanitized. Missing required fields throw a named error
3. A GDPR gate checks for consent. If it is missing, the workflow stops and nothing is stored
4. The CRM is queried by email to catch duplicates. Returning leads trigger a Slack alert and the workflow exits without creating a duplicate record
5. The email domain is sent to Apollo.io to pull company headcount, industry, country, revenue, description, and tech stack
6. Google News RSS is queried for the five most recent headlines about the company
7. Two prompts are assembled using all the data collected so far: one for scoring, one for the brief
8. The scoring prompt is sent to an LLM which returns a JSON object with dimension scores, a weighted total out of 10, and a tier label: Hot, Warm, or Cold
9. The brief prompt is sent to the same LLM which returns a structured pre-call document covering who the person is, what their company does, relevant news, why they applied, which product fits, a suggested opening line, and likely objections with counters
10. A contact and a deal are created in the CRM with the enrichment and score data attached
11. Slack messages are built based on the tier. Hot leads get the full breakdown with evidence. Warm leads get a shorter version
12. The team is alerted in Slack in the right channel based on score tier
13. The draft confirmation email is posted to a Slack approval channel as a Block Kit message with Approve, Edit, and Reject buttons

**Workflow 2: Email Approval Handler**

Listens for button clicks on the approval message in Slack:

- Approve sends the email immediately and posts a confirmation in the thread
- Edit opens a Slack modal pre-filled with the subject and body. On submit, the revised version is sent and confirmed in-thread
- Reject logs the decision in-thread and no email goes out

### Result

A sales rep gets a Slack message within seconds of a form submission. It has the score, the enriched company data, the full pre-call brief, and a one-click send on the confirmation email. Hot leads surface right away with everything needed for the call. Cold leads get logged without creating noise. No manual research, no copy-pasting, no missed follow-ups.

---

## Stack

| Layer | Tool |
|---|---|
| Automation runtime | n8n |
| Form capture | Webhook (any form provider) |
| CRM | HubSpot (Contacts and Deals) |
| Company enrichment | Apollo.io Organizations API |
| News | Google News RSS |
| LLM scoring and briefs | Groq API |
| Team notifications | Slack Block Kit |
| Email delivery | Gmail via OAuth2 |

---

## Workflows

| File | Description |
|---|---|
| `Lead Intelligence Pipeline.json` | Main pipeline from form intake to Slack alert |
| `email-approval.json` | Slack interaction handler for approve, edit, and reject |

---

## Setup

1. Import both JSON files into your n8n instance
2. Add your credentials:
   - HubSpot private app token
   - Apollo.io API key
   - Groq API key
   - Slack bot token (scopes: `chat:write`, `views:open`)
   - Gmail OAuth2
3. Update the scoring prompt in the Build Context node to match your ideal customer profile
4. Point your website form to the webhook URL: `/webhook/lead-intake`
5. Point your Slack app Interactivity Request URL to: `/webhook/email-approval`
6. Activate both workflows

---

## Lead Scoring

The scoring model lives in the Build Context node and is fully editable. Default setup:

| Dimension | Weight | Notes |
|---|---|---|
| Contact seniority | 25% | Decision-maker roles score highest |
| Company size | 20% | Set your target headcount range |
| Industry match | 20% | Define your target verticals |
| Geography | 15% | Define your target markets |
| Buying signals | 15% | Funding, growth news, or digital investment announcements |
| Red flag check | 5% | Filters out competitors, students, and freelancers |

A keyword bonus adds points if the stated challenge aligns with the pain points you care about.


demo request 

https://esubalewk.app.n8n.cloud/webhook-test/candid-strategy-session


{
  "first_name": "Thomas",
  "last_name": "de Vries",
  "work_email": "m.vandenberg@coolblue.nl",
  "company": "Coolblue",
  "role": "CMO / Head of Marketing",
  "challenge": "We are running 14 different AI tools across the team with zero central governance. Our campaign briefing cycle takes 3 weeks and by the time content is live the trend has passed. We need to move at market speed without losing brand control.",
  "consent": true
}
