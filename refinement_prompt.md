# Refinement Prompt Template

When a generated donor email fails the 7/10 threshold on one or more dimensions, this prompt regenerates it — using the specific feedback from the evaluation step rather than starting from scratch.

## Refinement Prompt

```
Your previous draft of this donor email did not meet quality standards. Revise it using the specific feedback below.

ORIGINAL EMAIL:
{{generated_email}}

EVALUATION FEEDBACK:
{{#each failing_dimensions}}
- {{dimension_name}} (scored {{score}}/10): {{feedback}}
{{/each}}

CONTEXT (unchanged):
- Donor: {{donor_name}}, {{donor_role}} at {{donor_organization}}
- Existing relationship context: {{known_donor_history}}
- LinkedIn-sourced context: {{linkedin_professional_context}}
- Campaign goal: {{campaign_goal}}

INSTRUCTIONS:
- Address each piece of feedback specifically — don't rewrite randomly
- Preserve what was working (dimensions that scored 7+ should not be disrupted)
- Keep the same format and length constraints as the original generation request

Write the revised version now.
```

## Design Notes

- **Only failing dimensions get feedback** — this prevents the refinement step from "fixing" something that wasn't broken and accidentally lowering a passing score.
- This prompt explicitly asks the model to **preserve what scored 7+**, which reduces variance between iterations. Without this instruction, regenerated emails sometimes regress on dimensions that previously passed.
- In practice, **2-3 iterations** is the typical convergence point. If an email still fails after 3 iterations, it gets flagged for human review rather than looping indefinitely — this is a safeguard for cases where the available context (existing history + LinkedIn data) may genuinely be too thin for strong personalization, in which case more research input is needed rather than more prompt iteration.
