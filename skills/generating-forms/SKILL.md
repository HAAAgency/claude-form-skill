---
name: generating-forms
description: Use when building, generating, reviewing, or auditing any form — signup, login, checkout, intake, application, onboarding, survey, or multi-step wizard — or when deciding which fields are required vs optional, how to cut fields, how to prefill, or how to tell the user what they still have to fill in.
---

# Generating Forms

## Overview

A good form answers three questions for the user at every moment: **What do I fill in? What is required? Why can't I continue?** Almost all form pain comes from breaking one of three promises — making the user guess what's required, asking for the same thing twice, or asking for things that don't apply.

**Core principle: optimize for the user's understanding, not the database's schema.** The form is a conversation, not a table dump.

## When to use

- Building or generating any form or multi-step wizard.
- Deciding required vs optional, or trying to reduce the number of fields.
- A "Next"/"Submit" button is disabled and the user can't tell why.
- Users report the form is confusing, long, or errors only at the end.

## The five rules

1. **Required vs optional must be unmistakable — and never leave a disabled button unexplained.** When submit/next is disabled, show *exactly which required fields are still missing*, live, right by the button. Mark optional fields "(optional)"; don't make everything required. A disabled button with no reason is the #1 form defect.

2. **Never ask for the same information twice — pipe it.** If a value was already collected or is derivable, prefill it and let the user confirm or edit. (Sole shareholder → prefill as the sole director; "employees in province" defaults to total; mailing address seeds the province from the head office.)

3. **Never ask for what doesn't apply — progressive disclosure.** Hide/skip fields and whole steps that a prior answer makes irrelevant. (0 employees → hide the per-employee questions. Not registering for tax → skip the tax-detail step. Numbered company → the name field becomes optional.)

4. **The client gate MUST mirror the server schema.** If the UI accepts a value the server later rejects, the user fills the entire form and hits a generic "invalid" error at the end with no idea which field. Validate the same rules (formats, lengths, required-ness) at the field, and fail early with a field-level message.

5. **Smart defaults beat blank fields.** Pre-select the most common option (default country/province, "No" for a rare toggle) to cut effort and blank-slate anxiety. Every default stays editable.

## The disabled-button hint — highest-leverage pattern

Compute the missing-required list from the *same predicate* that gates the button, and render it when invalid. This single pattern implements rules 1 and 4 at once.

```tsx
// Gate and hint are built from ONE source of truth, so they can never disagree.
const need: string[] = [];
if (!draft.companyName) need.push("Company name");
if (!/^\d{3}\s?\d{3}\s?\d{3}$/.test(draft.sin ?? "")) need.push("SIN (9 digits)"); // mirrors the server regex
if (!draft.address?.postalCode) need.push("Postal code");
const valid = need.length === 0;

<button disabled={!valid}>Next →</button>
{!valid && <p className="hint">To complete: {need.join(", ")}</p>}
```

Now the user always knows the reason, and the reason is always the *real* one.

**Say *why*, and prevent it at the source.** If a field is present but malformed, the hint must explain — "SIN (9 digits)", not just "SIN", and never "to complete" on a field that already has a value. Better still, constrain the input so the bad value can't be typed in the first place: digits-only + `maxLength` for a SIN / postal code / phone, a picker for a date. Prevent early; validate as a backstop.

**Two states, two treatments.** Don't use one channel for both. An *empty* required field gets gentle guidance (the grey "to complete" summary) — never shout red at a field the user hasn't reached yet. A *filled-but-invalid* field gets a clear **red error directly below that field**: unmissable and precise. Red means "you entered something wrong," never "you haven't done this yet."

## Multi-step forms

- One idea per step; always show progress.
- Route conditionally: never render a step that a prior answer made irrelevant.
- Carry every answer forward; re-asking is a bug.
- Autosave the draft so a refresh never loses work.

## Input choices that reduce effort

| Instead of | Use |
|---|---|
| Free-text country / phone | Searchable picker with flags + dial codes, sensible default |
| Free-text date | Date picker; if only day+month matters, drop the year |
| Free-text that's easy to malform (SIN, postal code, phone) | Constrain at the source: digits-only + `maxLength`, or an input mask, so an invalid value can't be entered |
| "Enter it again to confirm" | Show what was entered and let them edit |
| One giant page | Progressive steps with a review screen at the end |

## Common mistakes

- Disabled Next/Submit with no explanation → user is stuck and blames you.
- Everything marked required (or nothing) → the user can't prioritize.
- Client validation looser than the server → a generic error at the very end.
- Re-asking a name/address already entered → piping was skipped.
- Wiping user input when they add/remove a row or flip a toggle.
- A conditional field left visible after its answer stopped applying.
- A filled-but-malformed field shown as "to complete" (or with no format hint) → the user can't tell what's actually wrong.
- Red-flagging empty fields the user hasn't reached yet, or dumping every error in one blob at the top → anxiety and noise; put the error under the field it's about.

## Accessibility & craft (baseline — always)

Labels tied to inputs · visible focus states · 4.5:1 text contrast · full keyboard nav · errors announced to screen readers · 44px touch targets · responsive (375/768/1024/1440). Respect `prefers-reduced-motion`.

## Testing this skill (recommended)

Per Anthropic's skill-authoring guidance, pressure-test it: give an agent a real form spec *without* this skill (baseline) and *with* it, and confirm the "with" version explains disabled buttons, pipes repeated data, and hides inapplicable fields. The patterns here are drawn from a production bilingual legal intake form, but micro-test the wording against your own stack before relying on it.
