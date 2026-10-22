---
name: a11y-audit
description: Audit a UI or design against WCAG 2.2 AA/AAA and ARIA patterns, returning criterion-referenced findings with severity and specific fixes. Use when the user wants an accessibility check, contrast verification, keyboard/screen-reader review, or wants to confirm a component meets POUR.
---

# Skill: Accessibility Audit

Vendored from plugin87/ux-ui-agent-skills v2.10.0 (MIT, see `kit/LICENSE`). Only the
files this skill reads are kept, under `${CLAUDE_SKILL_DIR}/kit`. Render gates use
Playwright from `kit/node_modules` and the system Chrome; if `kit/node_modules` is
missing, run `npm install --prefix ${CLAUDE_SKILL_DIR}/kit` once.

Evaluate against WCAG 2.2 and the project's ARIA patterns.

## Steps
1. Read `${CLAUDE_SKILL_DIR}/kit/accessibility/wcag-checklist.md` (POUR-organized, P0/P1/P2) and `${CLAUDE_SKILL_DIR}/kit/accessibility/aria-patterns.md`.
2. Check the mandatory P0 set per component: keyboard navigable, focus visible (≥3:1), screen-reader name/role/state, contrast (4.5:1 text / 3:1 UI), target size ≥24×24, no color-only signaling.
3. Verify WCAG 2.2 additions: Focus Not Obscured (2.4.11), Target Size (2.5.8), Accessible Authentication (3.3.8).
4. **Contrast — measure, don't eyeball.** For rendered HTML (a built page or the running Vite dev server URL saved to a file), RUN the real-render gates with `DS_REQUIRE_BROWSER=1` so a missing browser fails instead of skipping, and report their actual output: `DS_REQUIRE_BROWSER=1 node ${CLAUDE_SKILL_DIR}/kit/scripts/measure_render.mjs <file> [--dark]` (every text element) AND `DS_REQUIRE_BROWSER=1 node ${CLAUDE_SKILL_DIR}/kit/scripts/verify_states.mjs <file> [--dark]` (every interactive element in default/hover/focus — catches hover-state failures). For loose color pairs, `python ${CLAUDE_SKILL_DIR}/kit/scripts/contrast.py "<fg>" "<bg>"`. Never state a ratio you did not measure.
5. Check reduced-motion handling (`${CLAUDE_SKILL_DIR}/kit/taste/motion-choreography.md`).

## Output
A findings table: WCAG criterion (e.g. 1.4.3) · severity (P0/P1/P2) · what fails · specific fix. Confirm passes explicitly. Accessibility may never be traded for aesthetics.
