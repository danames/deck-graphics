---
description: Generate cute doodle graphics for a slide deck, evaluate them, and place them on the slides
argument-hint: [path/to/deck.pptx]
---

Illustrate the PowerPoint deck at `$1` (or, if no path is given, the single
`.pptx` in the current working directory).

Use the **deck-graphics** skill for the full procedure. In short:

1. Read the deck with `markitdown` and decide which slides genuinely benefit
   from a doodle — skip dense-data slides.
2. For each chosen slide, write an image `subject` and a `requirement`.
3. Generate each graphic, then **look at the saved PNG yourself** and re-roll
   any that miss on style, clarity, or spelling.
4. Place the keepers, then QA with the script's `render` command — view each
   illustrated slide's render and fix any collision before reporting back.

Prefer the MCP image connector if one is enabled; otherwise use the API path in
`${CLAUDE_PLUGIN_ROOT}/scripts/deck_graphics.py`. Confirm the plan with me
before generating if the deck has more than ~6 candidate slides.
