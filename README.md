# AI-Powered Donor Outreach & Content Automation System

> Two integrated agentic pipelines built for EPICS at Arizona State University, in partnership with NLEats — a community organization focused on food donation and distribution.

## Overview

This system automates two distinct but related workflows for a nonprofit community partner:

1. **Donor Outreach Email Generation** — AI-generated, personalized emails to potential donors, using both existing donor data and information extracted from LinkedIn, with every email scored across four quality dimensions before sending.

2. **Social Media Content Automation** — A calendar-triggered pipeline that turns upcoming events (speaker visits, company milestones, community events) into platform-ready social content, routed through a human approval workflow with automatic escalation and timeout handling.

Both pipelines are orchestrated using **Zapier**, with **Microsoft Copilot** and **Canva** integrated for content generation, and run in production as part of EPICS' ongoing work with NLEats.

---

## Branch 1: Donor Outreach Email Generation

### The Problem
NLEats needs to persuade potential donors to contribute — but writing genuinely personalized outreach for each donor doesn't scale manually.

### The Pipeline

```
┌──────────────────┐     ┌─────────────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│  Donor Data        │ --> │  LinkedIn Info        │ --> │  AI Email          │ --> │  Evaluation Agent     │
│  (existing CRM/     │     │  Extraction            │     │  Generation        │     │  (4-dim scoring,      │
│   contact info)     │     │  (professional         │     │  (personalized      │     │   1-10 scale)         │
│                     │     │   context, role,       │     │   draft using all   │     │                       │
│                     │     │   activity)            │     │   gathered context) │     │                       │
└──────────────────┘     └─────────────────────┘     └──────────────────┘     └─────────┬───────────┘
                                                                                            │
                                                                  ┌──────────────────────────┘
                                                                  │
                                                            Avg score ≥ 7/10
                                                            on all dimensions?
                                                             │              │
                                                            Yes             No
                                                             │              │
                                                             ▼              ▼
                                                      ┌────────────┐  ┌──────────────────┐
                                                      │ Send Email  │  │ Refine prompt with │
                                                      │             │  │ dimension-specific │
                                                      └────────────┘  │ feedback &         │
                                                                       │ regenerate         │
                                                                       └──────────────────┘
```

### Evaluation Dimensions (scored 1-10)

| Dimension | What it measures |
|---|---|
| **Persuasion** | Does this email give the donor a compelling reason to donate? |
| **Clarity** | Is the ask clear and easy to understand on first read? |
| **Personalization** | Does this feel written for *this specific* donor, using their actual context? |
| **Relevance** | Does it connect to something real and current about the donor (role, activity, interests)? |

See [`evaluation_framework.md`](./evaluation_framework.md) for the full scoring rubric.

---

## Branch 2: Social Media Content Automation

### The Problem
NLEats hosts and participates in many events (speakers, community events, milestones) and wants consistent social media coverage — without manually creating and posting content for every event.

### The Pipeline

```
┌────────────────┐     ┌──────────────────┐     ┌─────────────────────┐     ┌──────────────────────┐
│ Google Calendar  │ --> │  Linked Google     │ --> │  Content Generation   │ --> │  Slack Approval        │
│ Event Created     │     │  Drive File         │     │  (Canva + Microsoft   │     │  Workflow              │
│ (user enters       │     │  (JS extracts        │     │   Copilot, platform-  │     │  (4 authorized          │
│  event details)    │     │   event details      │     │   specific format)    │     │   reviewers)           │
│                     │     │   on trigger)         │     │                       │     │                        │
└────────────────┘     └──────────────────┘     └─────────────────────┘     └──────────┬─────────────┘
                                                                                            │
                                                              ┌─────────────────────────────┼─────────────────────────────┐
                                                              │                              │                              │
                                                          Approved                       Pending                        Rejected
                                                              │                              │                              │
                                                              ▼                              ▼                              ▼
                                                      ┌────────────────┐          ┌──────────────────────┐        ┌─────────────────┐
                                                      │ Post to          │          │ Reminder every 18hrs   │        │ No post           │
                                                      │ LinkedIn         │          │ (24-48hr window)       │        │ generated         │
                                                      │ (Instagram/       │          │                        │        │                   │
                                                      │  Twitter: WIP)    │          │ Still pending after      │        └─────────────────┘
                                                      └────────────────┘          │ 48hrs → dismantled       │
                                                                                    │ (no post generated)      │
                                                                                    └──────────────────────┘
```

### Key Design Elements

- **Trigger-based extraction**: Calendar events link to a Google Drive file; a JavaScript function extracts relevant event details only when the event is triggered, keeping content generation tied to real, current events.
- **Multi-tool content generation**: Microsoft Copilot handles text/copy generation, Canva handles visual asset creation, matched to the target platform's format.
- **Human-in-the-loop approval with escalation**: Content goes to 4 authorized reviewers via Slack. Three states — approved, rejected, pending. Pending posts get automatic reminders every 18 hours within a 24-48 hour window; if never approved, the post is dismantled rather than published or left in limbo indefinitely.
- **Platform status**: LinkedIn posting is fully live in production. Instagram and Twitter integrations are in progress.

See [`social_media_workflow.md`](./social_media_workflow.md) for the full approval/escalation logic.

---

## Tech Stack

- **Orchestration**: Zapier (multi-step workflows, conditional logic, scheduled reminders)
- **Content Generation**: Microsoft Copilot (text), Canva (visual assets)
- **Data Sources**: LinkedIn (donor personalization context), Google Calendar + Google Drive (event content), JavaScript (extraction logic)
- **Approval & Communication**: Slack (approval workflow, reminders)
- **Distribution**: LinkedIn API (live), Instagram/Twitter (in progress)

## Repository Structure

```
├── README.md                    # This file
├── evaluation_framework.md       # 4-dimension scoring rubric (1-10 scale)
├── social_media_workflow.md      # Calendar-to-LinkedIn approval/escalation pipeline
├── prompts/
│   ├── generation_prompt.md      # Donor email generation prompt template
│   ├── evaluation_prompt.md      # Scoring prompt template
│   └── refinement_prompt.md      # Iteration prompt template
└── sample_outputs/
    └── before_after_example.md   # Real example showing iteration improving a donor email
```

## My Role

As AI Agent Development Team Lead at EPICS, I led the design of both pipelines — the evaluation framework and prompt architecture for donor outreach, and the approval/escalation logic for the social media automation pipeline.

## About Me

I'm a Computer Science student at Arizona State University. I'm interested in agentic AI systems, LLM evaluation, and building automation that ships to production and serves real organizations.

[LinkedIn](https://www.linkedin.com/in/jashanpreet-kaur-2732a030a/)
