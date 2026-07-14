# deck-graphics

A Claude plugin that illustrates a slide deck: Claude picks which slides need a
graphic, writes the prompts, generates them with an image model (gpt-image-1 or
Gemini), **evaluates each result by looking at it**, re-rolls the weak ones, and
places the keepers on the slides.

```
deck-graphics/
├── .claude-plugin/plugin.json     # manifest (required)
├── commands/illustrate.md         # → /deck-graphics:illustrate
├── skills/deck-graphics/SKILL.md  # the workflow Claude follows
├── scripts/deck_graphics.py       # the engine (generate / insert / render / demo)
├── examples/Image Prompts.md      # starter style guide — copy into your project
├── .mcp.json                      # OPTIONAL image-gen connector
└── README.md
```

## Fast smoke test (no plugin needed)
Prove the pipeline works before wiring up Cowork:

```bash
pip install openai python-pptx pillow markitdown
export OPENAI_API_KEY=sk-...
cd deck-graphics/scripts

# 1) generate one doodle (prints the path + a transparency check)
python deck_graphics.py generate "cute cartoon stream gage on a riverbank, label 'stream gage'" test.png

# 2) drop it onto slide 2 (0-based index 1) of a real deck
python deck_graphics.py insert /path/to/talk.pptx 1 test.png talk_out.pptx

# 3) render that slide to a PNG for a visual check
python deck_graphics.py render talk_out.pptx --slides 1
```

Then try the full example:
`python deck_graphics.py demo /path/to/talk.pptx talk_illustrated.pptx`.

## Install as a plugin

**Local development / testing (CLI):**
```bash
claude --plugin-dir /path/to/deck-graphics
```
Then run `/deck-graphics:illustrate path/to/talk.pptx` in the session.

**Claude Desktop (Code tab) or Claude Code in VS Code / terminal:** plugins
install from a *marketplace* — there's no zip/folder upload. This repo doubles
as its own marketplace (`.claude-plugin/marketplace.json` points at the repo
root). In any session's prompt box, run:

```
/plugin marketplace add https://github.com/danames/deck-graphics
/plugin install deck-graphics@deck-graphics
```

(Use the full URL — Claude Desktop's marketplace add may not resolve the
short `owner/repo` form. The UI route is **+ → Plugins → Add plugin** and
paste the URL as a new marketplace.)

Note: plugins run in local sessions, not Claude cloud sessions.

## Choosing the image path
- **API path (default):** keys in the environment, best style match, cents/image.
- **MCP connector (`.mcp.json`):** keeps keys out of the script, but the HF
  Spaces models (FLUX/SD) are weaker on the cute style and on legible text.

## Customize the look
Copy `examples/Image Prompts.md` into your project dir and edit it — the script
auto-detects `./Image Prompts.md` (or point anywhere with `--style <file>`).
Falls back to the `STYLE_GUIDE` baked into `scripts/deck_graphics.py`. Define
the aesthetic once; every graphic inherits it.

## Visual QA notes
- `render` uses macOS Quick Look for per-slide PNGs (zero installs). On
  Linux/Cowork VMs it falls back to LibreOffice and produces a PDF instead.
- Transparent PNGs can *look* like they have a dark background in an image
  viewer — that's alpha compositing. `generate` prints the real transparency
  percentage after every image.
