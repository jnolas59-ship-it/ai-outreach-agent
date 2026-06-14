# Social Media Content Automation: Approval & Escalation Workflow

## Overview

This pipeline turns calendar events into platform-ready social media content, but the most important design work isn't the content generation — it's the **human approval workflow with automatic escalation and timeout handling.** Generated content should never publish without human sign-off, but it also shouldn't sit in limbo forever. This document covers that logic in detail.

## The Full Pipeline

### Step 1: Event Creation
A user enters an event into **Google Calendar** — for example, an upcoming speaker visit, a community event, or a company milestone.

### Step 2: Linked Drive File
The calendar event is linked to a **Google Drive file** containing the event's details (description, relevant images, key information to highlight).

### Step 3: Trigger-Based Extraction
When the event is triggered (based on timing relative to the calendar entry), a **JavaScript function** extracts the relevant content from the linked Drive file. This keeps content generation tied to a specific, real event rather than running on a generic schedule.

### Step 4: Content Generation
The extracted information is passed to **Microsoft Copilot** (for text/copy) and **Canva** (for visual assets). Content is generated in a format appropriate to the target platform — currently LinkedIn (Instagram and Twitter formats are in progress).

### Step 5: Slack Approval Request
The generated content (text + visual) is sent via **Slack** to **4 authorized reviewers** for approval.

### Step 6: Approval State Machine

The request enters one of three states:

```
                    ┌─────────────┐
                    │   PENDING    │ ◄── Initial state when request is sent
                    └──────┬──────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
        Reviewer        No response    Reviewer
        approves        within 18hrs    rejects
              │             │             │
              ▼             ▼             ▼
        ┌──────────┐  ┌───────────┐  ┌──────────┐
        │ APPROVED  │  │ Reminder    │  │ REJECTED  │
        └─────┬────┘  │ sent (18hr  │  └──────────┘
              │        │ intervals)  │       │
              │        └──────┬─────┘       │
              │               │             │
              │        Still pending        │
              │        after 48hrs?         │
              │               │             │
              │              Yes            │
              │               │             │
              │               ▼             │
              │        ┌────────────┐       │
              │        │ DISMANTLED   │       │
              │        │ (no post     │◄──────┘
              │        │  generated)  │
              │        └────────────┘
              │
              ▼
      ┌───────────────┐
      │ Post to LinkedIn │
      └───────────────┘
```

### State Definitions

| State | Meaning | Next Action |
|---|---|---|
| **Pending** | Awaiting reviewer response | Reminder sent every 18 hours |
| **Approved** | At least one of 4 authorized reviewers approved | Post immediately to LinkedIn |
| **Rejected** | A reviewer explicitly rejected | No post generated, request closed |
| **Dismantled** | Pending state exceeded 48-hour window with no response | No post generated, request closed |

## Why This Design Matters

**The problem with most "human-in-the-loop" automation** is that the human step becomes a bottleneck — content sits waiting for approval indefinitely, or gets posted without real review because someone rubber-stamps a backlog.

This workflow solves both failure modes:

- **Reminders (every 18 hours)** keep the request visible without being so frequent that reviewers tune them out.
- **The 24-48 hour dismantle window** means content about a time-sensitive event (like an upcoming speaker visit) doesn't get posted *after* the event has already happened, which would look broken or out of touch.
- **4 authorized reviewers** rather than 1 means the workflow doesn't fully block on a single person's availability.

## Current Status & Future Work

- **LinkedIn**: fully implemented and running in production.
- **Instagram and Twitter**: extraction and generation logic is shared across platforms; the remaining work is platform-specific formatting and posting integration.
- **Planned improvement**: currently the linked Drive file is manually associated with each calendar event; a future iteration could automate this association based on event metadata.
