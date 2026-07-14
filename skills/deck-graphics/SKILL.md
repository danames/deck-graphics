---
name: deck-graphics
description: >
  Use when illustrating a PowerPoint deck with generated graphics — the user
  asks to add doodles/cartoons/illustrations to slides, or runs
  /deck-graphics:illustrate. Claude reasons about the content, writes image
  prompts, generates via an image model, evaluates each result by viewing it,
  re-rolls the weak ones, and places the keepers on the slides.
---

# deck-graphics

Claude is the art director; an image model is the illustrator. Claude can't
draw, but it's good at judging a drawing and writing the prompt that gets a
better one. That judgment loop is the whole point of this skill.

## Inputs it expects
- a `.pptx` whose text/layout is already written (this skill only adds art)
- a style guide, resolved in this order: `--style <file>` on the CLI, then
  `Image Prompts.md` in the working dir (auto-detected), then the `STYLE_GUIDE`
  built into the script. One shared look so every graphic is consistent —
  there's a starter in `${CLAUDE_PLUGIN_ROOT}/examples/Image Prompts.md`.
- `${CLAUDE_PLUGIN_ROOT}/scripts/deck_graphics.py` — the engine
- deps: `pip install openai python-pptx pillow markitdown` (plus `google-genai`
  for Gemini, `anthropic` for unattended `--evaluate` runs)

## Procedure

1. **Read the deck.** `markitdown <deck>.pptx`. Understand each slide's point.

2. **Choose slides.** Not all of them. Flag slides where a doodle earns its
   place (a concept, a process, a hero image). Skip dense tables/charts. If more
   than ~6 slides qualify, show the user the shortlist before generating.

3. **Write a plan** — per chosen slide, a `subject` (the drawing, in plain
   words) and a `requirement` (how you'll grade it: style match, clarity,
   correct spelling of any label).

4. **Generate — pick the path:**
   - **MCP path (preferred if a connector is enabled):** call the image tool
     from the connector directly, saving each PNG to `art/slideN.png`. Ask it
     for a transparent background.
   - **API path:** run
     `python ${CLAUDE_PLUGIN_ROOT}/scripts/deck_graphics.py generate "<subject>" art/slideN.png`
     (add `--provider gemini` to switch models, `--style <file>` to point at a
     style guide that isn't `./Image Prompts.md`).

5. **Evaluate — this is the real work.** VIEW each saved PNG. Check: does it
   match the house style, is the subject instantly legible, is any text spelled
   correctly, does it sit well as a set with the others? If not, revise the
   subject with a specific fix and regenerate. (For unattended batch runs,
   `generate --evaluate` automates this with a Claude-vision critique.)
   CAUTION: a transparent PNG often *looks* like it has a dark background when
   viewed — that's alpha compositing, not a real background. Trust the
   transparency percentage the `generate` command prints, and only flag a
   background problem if it reports "no alpha channel".

6. **Place.** For each keeper:
   `python ${CLAUDE_PLUGIN_ROOT}/scripts/deck_graphics.py insert <deck>.pptx <slide_index> art/slideN.png out.pptx`
   Adjust `--left/--top/--width` (inches) so the doodle doesn't collide with
   text. Chain inserts by feeding `out.pptx` back in as the input deck.

7. **Visual QA.** Render every illustrated slide and check for overlap,
   overflow, or a graphic fighting the text:
   `python ${CLAUDE_PLUGIN_ROOT}/scripts/deck_graphics.py render out.pptx --slides 1,3,5`
   (macOS: per-slide PNGs via Quick Look, no installs; Linux/Cowork VM: falls
   back to LibreOffice and emits a PDF — view its pages instead). VIEW each
   render, fix and re-place only the slides that need it, then report what you
   changed.

## Notes
- gpt-image-1 and Gemini match the cute-doodle-with-clean-text look; FLUX/SD
  (easy to wire via MCP) are weaker on style and text — a real trade-off if you
  choose the connector for convenience.
- Keep `subject` about *what to draw*, never about slide layout. Layout is your
  job at insertion time, not the image model's.
