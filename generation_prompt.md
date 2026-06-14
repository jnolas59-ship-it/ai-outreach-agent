# Generation Prompt Template

This is the base template used to generate personalized donor outreach emails. Variables in `{{double_braces}}` are populated dynamically — some from existing donor data, others from LinkedIn-sourced professional context.

## Donor Outreach Email Template

```
You are writing a personalized donor outreach email on behalf of NLEats, a community organization focused on food donation and distribution.

CONTEXT:
- Donor: {{donor_name}}, {{donor_role}} at {{donor_organization}}
- Existing relationship context: {{known_donor_history}} (e.g., past donations, prior contact)
- LinkedIn-sourced context: {{linkedin_professional_context}} (e.g., recent role changes, posts, professional interests)
- Campaign goal: {{campaign_goal}}
- Tone: warm and mission-driven, not transactional

REQUIREMENTS:
1. Open with a connection point specific to {{donor_name}} — drawn from {{known_donor_history}} or {{linkedin_professional_context}}, not a generic greeting
2. Connect NLEats' mission to something {{donor_name}} would genuinely care about, based on the available context
3. Make a single, clear ask related to {{campaign_goal}}
4. Keep total length under 150 words
5. Close with a low-friction next step

AVOID:
- Generic openers like "I hope this email finds you well"
- Language that feels transactional rather than mission-driven
- Multiple calls to action
- Claims about the donor's interests that aren't grounded in {{known_donor_history}} or {{linkedin_professional_context}}

Write the email now.
```

## Design Notes

- The **REQUIREMENTS** section is intentionally specific and numbered — vague instructions ("write a good donor email") produce generic output that fails the Personalization and Relevance checks downstream.
- `{{linkedin_professional_context}}` is often the single most valuable input — for donors with limited prior history, LinkedIn-sourced context (role changes, recent activity, professional interests) is frequently what makes an email feel personalized rather than templated.
- The **AVOID** section exists because LLMs default to certain patterns (generic openers, hedging language, transactional framing) unless explicitly told not to use them — and a donation ask that reads as transactional tends to score lower on Persuasion.
