# Evaluation Prompt Template

This prompt scores a generated donor outreach email across the four quality dimensions, each on a **1-10 scale**. It runs immediately after generation, before any email is approved to send.

## Evaluation Prompt

```
You are evaluating a donor outreach email for quality before it gets sent.

EMAIL TO EVALUATE:
{{generated_email}}

CONTEXT (what the email was supposed to achieve):
- Donor: {{donor_name}}, {{donor_role}} at {{donor_organization}}
- Existing relationship context: {{known_donor_history}}
- LinkedIn-sourced context: {{linkedin_professional_context}}
- Campaign goal: {{campaign_goal}}

Score the email on each of the following dimensions from 1-10, where 1 is "fails completely" and 10 is "excellent, ready to send as-is."

1. PERSUASION: Does this email give {{donor_name}} a compelling reason to donate?

2. CLARITY: Is the ask clear and easy to understand on a single read?

3. PERSONALIZATION: Does this feel written specifically for {{donor_name}}, using {{known_donor_history}} and/or {{linkedin_professional_context}}, or could it be sent to any donor?

4. RELEVANCE: Does this email connect to something real and current about {{donor_name}}, drawn from the available context?

For each dimension, provide:
- A score (1-10)
- A one-sentence justification
- If the score is below 7, specific feedback on what would need to change to improve it

OUTPUT FORMAT (JSON):
{
  "persuasion": {"score": X, "justification": "...", "feedback": "..."},
  "clarity": {"score": X, "justification": "...", "feedback": "..."},
  "personalization": {"score": X, "justification": "...", "feedback": "..."},
  "relevance": {"score": X, "justification": "...", "feedback": "..."},
  "overall_pass": true/false
}

overall_pass should be true only if ALL FOUR scores are 7 or higher.
```

## Design Notes

- **Structured JSON output** is critical — it lets the Zapier orchestration layer programmatically check `overall_pass` and route the email accordingly without parsing free text.
- **Per-dimension feedback** (not just a score) is what makes the refinement loop effective. Generic feedback like "make it more personal" doesn't give the next generation step anything actionable — feedback needs to point at *what context to use*.
- The **7/10 threshold across all four dimensions** was chosen after testing — see [`evaluation_framework.md`](../evaluation_framework.md) for how this was calibrated.
- This evaluation prompt is run by a **separate** LLM call from the generation step. Using the same context but a different "role" (evaluator vs. generator) produces more honest, critical scoring than asking a model to evaluate its own immediately-preceding output.
