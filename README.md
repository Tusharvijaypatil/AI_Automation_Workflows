# AI Automation Workflows

A collection of production n8n workflows and automation tools I've built and shipped. Each folder has the workflow JSON (importable into n8n), a short writeup of the problem and what was built, and where relevant a small architecture diagram.

## What's in here

- **Image QC Automation** — n8n + LLM workflow that runs quality checks across 20+ languages. Replaced a 5-person manual team at ConsultBae.
- **Lead-Gen Outreach** — n8n + Apify scraping funded companies and running personalized outreach. Onboarded three clients through it.
- **Naukri Hiring Bot** — n8n + Gmail API + LLM agent. Cut candidate response time from 3 days to roughly 6 hours.
- **Apps Script Pipelines** — Google Apps Script data pipelines used as standard intake for new AI projects.

## Using a workflow

Import the `workflow.json` from any folder into your n8n instance (top right menu → Import from File). Credentials and client data have been stripped before publishing — look for `<REDACTED>` placeholders and swap in your own values.

---

By [Tushar](https://github.com/Tusharvijaypatil) · tusharpatil.w2001@gmail.com
