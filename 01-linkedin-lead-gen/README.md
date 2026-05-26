# LinkedIn Lead-Gen & Outreach Engine

An end-to-end **n8n** automation that turns fresh LinkedIn job postings into a
qualified outreach pipeline: it scrapes recent tech job posts, finds the
decision-makers behind each hiring company, enriches their work emails, and
runs a multi-stage email sequence with automatic reply/bounce detection — all
backed by a Google Sheet as the source of truth.

Built as a real, in-production workflow and open-sourced here as a portfolio
reference. **All credentials, IDs, and message copy have been replaced with
placeholders** (see [Setup](#setup)).

---

## Problem

Manual top-of-funnel outreach is slow and leaky. Finding companies that are
actively hiring, identifying the right person to contact, finding a valid work
email, and then following up on the right cadence is hours of repetitive work
per day — and most of it gets dropped.

## Solution

A set of scheduled n8n flows that automate the whole funnel:

1. **Scrape** recent LinkedIn job posts (via an Apify actor) and write them to a sheet.
2. **Find decision-makers** at each hiring company using the Apollo API (founders, CxO, HR / TA leaders), plus the job's hiring manager where available.
3. **Enrich** each contact with a verified work email.
4. **Send** a personalized first-touch email, then **follow up** on a timed sequence.
5. **Detect replies and bounces** by reading the Gmail thread, and stop the sequence automatically when someone responds.

Every stage updates a status column in the sheet, so the pipeline is idempotent
and safe to re-run on a schedule.

## Architecture

The workflow is organized into four logical stages (marked with sticky notes in
the canvas):

```
Stage 1 — Scrape LinkedIn Jobs
  Manual/Schedule trigger
    -> Apify actor (recent job posts)
    -> Code: parse title, experience, budget, skills, hiring-manager URL
    -> Google Sheet (Job Data)

Stage 2 — Apollo People Finder
  Schedule trigger
    -> Read pending companies from sheet
    -> Clean website into a bare domain
    -> Apollo people search (titles: Founder/CEO/CTO/CHRO/HR Head/TA...)
    -> Code: pick one best contact per title category
    -> Apollo person-match (reveal work email)
    -> Code: map to clean contact fields
    -> Google Sheet (Employee Data)

  (parallel) Hiring-Manager enrichment
    -> Apollo person-match by hiring-manager LinkedIn URL
    -> Google Sheet (Employee Data)

Stage 3 — Email Sender
  Schedule trigger
    -> Read contacts with a verified email
    -> Loop with throttle (Wait node) to protect sending reputation
    -> Send first-touch email
    -> Record thread id + next-follow-up date in sheet

Stage 4 — Follow-up Sender
  Schedule trigger
    -> Read contacts whose follow-up is due
    -> Get Gmail thread
    -> Code: detect reply or bounce
        - bounced  -> mark BOUNCED, stop
        - replied  -> mark replied, stop
        - no reply -> Switch on sequence stage -> send next follow-up / final
```

A rendered architecture diagram is in [`architecture.svg`](./architecture.svg).

## Stack

n8n (orchestration) · Apify (LinkedIn scraping) · Apollo.io API (people search +
email enrichment) · Gmail API (sending, thread reads, reply/bounce detection) ·
Google Sheets (state + status tracking) · JavaScript Code nodes (parsing &
logic).

## Design notes

- **Sheet-as-state-machine.** Each contact carries an `Apollo Status`,
  `Sequence Stage`, `Replied?`, and `Stop Reason`. Flows read by status and
  write back on completion, so the whole thing is idempotent and re-runs
  cleanly on a cron.
- **Throttled sending.** A `Wait` node inside the send loop spaces out emails
  instead of blasting them, which is gentler on deliverability.
- **Reply/bounce detection from the thread itself.** Rather than trusting a
  label, a Code node walks the Gmail thread, ignores the user's own `SENT`
  messages, and flags a genuine inbound reply or a mailer-daemon bounce — then
  stops the sequence.
- **Defensive HTTP.** Apollo calls run with `onError: continueRegularOutput`
  and batch size 1, so one bad lookup doesn't kill the run.
- **Resilient parsing.** The job parser pulls structured fields (experience
  range, budget + currency, skills, education) out of messy free-text job
  descriptions with layered regex and sensible fallbacks.

## Setup

> This export ships with **placeholders instead of secrets**. You'll need to
> supply your own before importing.

1. **Import** `workflow.json` into your n8n instance (Workflows -> Import from File).
2. **Reconnect credentials** — the import will prompt for:
   - Google Sheets (OAuth2)
   - Gmail (OAuth2)
   - Apify API
3. **Replace placeholders:**

   | Placeholder | Where | What to put |
   |---|---|---|
   | `YOUR_APOLLO_API_KEY` | the three HTTP Request nodes (`x-api-key` header) | Your Apollo API key |
   | `YOUR_GOOGLE_SHEET_ID` | every Google Sheets node | Your spreadsheet ID |
   | `YOUR_APIFY_ACTOR_ID` | the Apify node | The LinkedIn-jobs actor ID you use |

4. **Customize the email copy.** The Set node and the Gmail nodes contain
   generic placeholder templates (`[ Your Name ]`, `[ Your Company ]`, etc.).
   Swap in your own.
5. **Set up the sheet tabs:** `Linkedin` (job data), `Linkedin Company`, and
   `Employee Data` (contacts + sequence state). Column names are referenced
   directly in the nodes.

> **Security note:** never commit a workflow export with live API keys. Apollo,
> Apify, and OAuth credentials all leak in raw exports — always redact before
> pushing, and rotate any key that was ever committed.

## Status

Running in production on a daily schedule. Open-sourced here as a sanitized
reference — it's a working system, not a polished product, and there's plenty
listed below I'd still improve.

## What I'd improve with more time

- Move state out of Google Sheets into a real DB (Postgres) for concurrency safety.
- Add a dedup/suppression list so the same contact is never emailed twice across runs.
- Replace regex job-description parsing with an LLM extraction step for messier posts.
- Add per-domain send caps and warm-up logic for deliverability.
- Proper secret management (n8n credentials / env vars) instead of inline values — already partly done, finish it.
- A small dashboard over the sheet for reply rate, bounce rate, and stage funnel.
