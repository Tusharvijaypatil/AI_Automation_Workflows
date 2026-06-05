# AI Automation Workflows

Production n8n workflows I've built, sanitized and open-sourced as portfolio
reference. Every export has its credentials, IDs, and message copy replaced with
placeholders.

I work day-to-day in **n8n, LangChain, Apollo, Apify, and the Gmail / Google
Sheets APIs**. These are real systems that ran in production, not toy demos.

## Workflows

### 01 · LinkedIn Lead-Gen & Outreach Engine

**Folder:** [`./01-linkedin-lead-gen`](./01-linkedin-lead-gen)
**Stack:** n8n · Apify · Apollo · Gmail · Google Sheets

Scrapes fresh LinkedIn job posts, identifies decision-makers at the posting
company, enriches their work emails via Apollo, then runs a multi-stage email
sequence with reply and bounce detection. Results are logged to Google Sheets.

What's in the folder:
- the sanitized workflow JSON (import into n8n via copy/paste onto the canvas)
- a short walkthrough of the node graph and what each stage does

## Related repos

- [bookleaf-query-bot](https://github.com/Tusharvijaypatil/bookleaf-query-bot) —
  RAG customer-query bot (LangChain · FAISS · Supabase) with confidence scoring
  and human escalation.
- [AI-Resume-Screening-Agent](https://github.com/Tusharvijaypatil/AI-Resume-Screening-Agent) —
  resume-vs-JD screening with structured AI output.

## Contact

Tushar Patil · AI Automation Engineer · Delhi NCR
[LinkedIn](https://www.linkedin.com/in/tushar-patil47) · tusharpatil.w2001@gmail.com
