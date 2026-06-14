# Evaluation Framework

## Why Evaluation Matters More Than Generation

Any LLM can generate a plausible-sounding donor email. The hard problem in production AI systems is **knowing whether that email is actually good enough to send** — and doing it automatically, at scale, without a human reviewing every single donor outreach message.

This framework solves that by scoring every generated email across four dimensions, each on a **1-10 scale**, before it's allowed to send.

## The Four Dimensions

### 1. Persuasion (1-10)
**Question:** Does this email give the donor a compelling reason to donate?

**Scoring considerations:**
- Is there a clear emotional or logical case for why this donation matters?
- Does it connect NLEats' mission to something the donor would care about?
- Would a real person reading this feel moved to act, not just informed?

### 2. Clarity (1-10)
**Question:** Is the ask clear and easy to understand on a single read?

**Scoring considerations:**
- Is there one clear ask, not multiple competing requests?
- Is the structure logical (context → connection → ask)?
- Is the length appropriate — long enough to be personal, short enough to be read?

### 3. Personalization (1-10)
**Question:** Does this feel written specifically for *this* donor, using their actual context?

**Scoring considerations:**
- Does it reference the donor's actual role, industry, or activity (sourced from existing data or LinkedIn)?
- Would the donor recognize this as generic if they compared it to outreach sent to someone else?
- Does the tone match what's appropriate for this donor's background?

### 4. Relevance (1-10)
**Question:** Does this email connect to something real and current about the donor?

**Scoring considerations:**
- Does it reference recent professional activity, role changes, or interests visible on LinkedIn?
- Does the timing or framing make sense for this specific donor right now?
- Does it avoid feeling like a message that could have been sent to anyone, at any time?

## Pass Threshold

An email is approved to send when it scores **7 or higher on all four dimensions**. Any dimension scoring below 7 triggers the refinement loop.

## The Iteration Loop

```
Generate → Score on all 4 dimensions (1-10) → All dimensions ≥ 7?
                                                      │
                                          ┌───────────┴───────────┐
                                         Yes                      No
                                          │                        │
                                          ▼                        ▼
                                     Send Email          Refine prompt with
                                                          specific feedback on
                                                          the failing dimension(s)
                                                                   │
                                                                   └──────► Regenerate
```

This loop typically converges within 2-3 iterations. The key design decision is that the refinement step receives *specific* feedback about *which* dimension(s) scored below 7 and *why* — not just "try again."

## Example Refinement Feedback

If a generated email scores 5/10 on **Personalization**, the refinement prompt includes feedback like:

> "This draft uses generic language that could apply to any donor. Revise to incorporate [specific detail from donor's LinkedIn profile or existing data] and adjust tone to match their professional context."

This targeted feedback is what makes the iteration loop converge quickly rather than generating random variations and hoping one scores higher.

## Lessons Learned

- **Generic scoring prompts don't work well.** Each dimension needs its own evaluation criteria — a single overall "rate this" prompt produces inconsistent results across the 1-10 range.
- **Feedback specificity matters more than iteration count.** Two targeted iterations beat five generic ones.
- **The 7/10 threshold was calibrated through testing.** A lower threshold let too much generic content through; requiring near-perfect scores (9-10) caused excessive iteration without meaningful quality gains.
- **LinkedIn-sourced context is the highest-leverage input for Personalization and Relevance.** When the extraction step returns rich professional context, both scores tend to rise together — they often share the same underlying signal.
