# a11y-audit skill

A standalone Claude Code skill that audits a UI against WCAG 2.2 AA/AAA and
measures contrast on the real render. It is the `a11y-audit` skill from
[plugin87/ux-ui-agent-skills](https://github.com/plugin87/ux-ui-agent-skills)
v2.10.0, cut down to the files it reads so it works without the rest of the kit.

**설치 및 사용 설명서 (한국어): [docs/GUIDE.ko.md](docs/GUIDE.ko.md)**

## What changed from upstream
- Only the skill and the seven files it uses are kept under `a11y-audit/kit/`:
  the WCAG checklist, ARIA patterns, motion guide, `measure_render.mjs`,
  `verify_states.mjs`, `_targets.mjs` and `contrast.py`.
- Paths point inside the skill folder, so the "is the kit here?" step is gone.
- Render gates run with `DS_REQUIRE_BROWSER=1`, so a missing browser fails
  instead of printing SKIPPED and exiting 0.
- `python` instead of `python3`, which is often missing on Windows.
- No hooks, no plugin, no design systems.

## Install
Copy `a11y-audit/` into a project's `.claude/skills/` (or `~/.claude/skills/`
for every project), then install Playwright once:

```
npm install --prefix .claude/skills/a11y-audit/kit
```

The gates use the system Chrome. Without it, run `npx playwright install chromium`
from the `kit` folder.

## Use
Ask Claude Code for an accessibility check of a screen or component. The skill
returns a findings table: WCAG criterion, severity, what fails, and the fix.

## Limits
- The gates open pages over `file://`, where browsers do not run
  `<script type="module">`. A Vite or Next build measures as an empty shell and
  reports OK. Measure a static harness page or a saved copy of the rendered DOM.
- `--dark` sets `data-theme="dark"` on `<html>`; it does not emulate
  `prefers-color-scheme`.

See the guide for details.

## License
MIT, as upstream. Copyright (c) 2026 Thientan Soparat; see `LICENSE`.
