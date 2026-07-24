# generating-forms — a Claude Code skill for building forms people actually understand

A portable [Agent Skill](https://agentskills.io) / Claude Code skill that teaches the assistant to generate and audit forms where **the user always knows what to fill in, what's required, and why they can't continue** — while never asking for the same thing twice or for things that don't apply.

## Why this exists

Most "UX skills" cover visual design (styles, palettes, typography) or broad UX frameworks. Very few cover the thing that actually makes or breaks a form:

- **Clarity** — required vs optional is obvious, and a disabled button always explains itself.
- **No redundant entry** — data already collected (or derivable) is piped forward, not re-asked.
- **No irrelevant fields** — anything a prior answer makes moot is hidden or skipped.
- **Fail early** — the client validates the *same* rules as the server, so users never fill the whole form only to hit a generic error at submit.

These patterns were extracted and hardened while building and re-auditing a production bilingual (FR/EN) legal incorporation intake form.

## What's inside

```
skills/
  generating-forms/
    SKILL.md   # the skill: five rules, the disabled-button-hint pattern, input choices, common mistakes
```

The skill's highest-leverage idea — build the disabled-button hint from the **same predicate that gates the button**, so the two can never disagree:

```tsx
const need: string[] = [];
if (!draft.companyName) need.push("Company name");
if (!/^\d{3}\s?\d{3}\s?\d{3}$/.test(draft.sin ?? "")) need.push("SIN (9 digits)"); // mirrors the server regex
const valid = need.length === 0;

<button disabled={!valid}>Next →</button>
{!valid && <p className="hint">To complete: {need.join(", ")}</p>}
```

## Install

**Copy into a project (recommended):**
```bash
mkdir -p .claude/skills
cp -r skills/generating-forms .claude/skills/
```

**Or make it available to all your projects:**
```bash
cp -r skills/generating-forms ~/.claude/skills/
```

Claude Code auto-loads the skill when a request matches its `description` (building/auditing a form, deciding required vs optional, a disabled button with no explanation, etc.), or invoke it explicitly with `/generating-forms`.

## Related / complementary

- [tommyjepsen/awesome-ux-skills](https://github.com/tommyjepsen/awesome-ux-skills) — broad UX frameworks (Nielsen heuristics, Rams, journey maps).
- [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) — visual design systems (styles, palettes, typography).

This skill fills the gap those leave: **form interaction & data flow**, not aesthetics.

## License

MIT
