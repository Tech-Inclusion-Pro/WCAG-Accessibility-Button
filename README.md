# WCAG Accessibility Button

A self-contained **"Display settings" button** you can drop into any HTML page. Readers click it to change text size, text spacing, color palette, reading font, cursor, motion, and where the button sits — and their choices are remembered on their own device.

- **One copy-paste block.** Plain HTML + CSS + JavaScript. No frameworks, no build step, no dependencies, no network requests.
- **Private by design.** Settings are stored in the reader's own browser (`localStorage`). Nothing is sent anywhere, ever. No cookies, no analytics.
- **Built for WCAG 2.1 AA.** The widget itself is fully keyboard operable, screen-reader announced, 44 px+ touch targets, visible focus, and it survives Windows High Contrast mode.
- **LMS-aware.** It detects auto-height iframes (the way Canvas embeds HTML pages) and docks itself into the page flow instead of floating off-screen.

It was built for university course pages (UIC SPED 101) and is shared here so anyone — especially educators — can add it to their own pages, or hand it to an AI tool and say *"include this."*

**Try it:** download this repository and open [`demo.html`](demo.html) in any browser.

---

## Table of contents

1. [Features in detail](#features-in-detail)
2. [How it works](#how-it-works)
3. [How to add it to your page](#how-to-add-it-to-your-page)
4. [Using it with AI tools (ChatGPT, Claude, Copilot…)](#using-it-with-ai-tools)
5. [Making the palettes restyle your whole page](#making-the-palettes-restyle-your-whole-page)
6. [Customization](#customization)
7. [Canvas and other LMS notes](#canvas-and-other-lms-notes)
8. [WCAG mapping](#wcag-mapping)
9. [Download](#download)
10. [Privacy](#privacy)
11. [License and credits](#license-and-credits)

---

## Features in detail

### Text size
Four steps: Default, Large (115%), Larger (130%), Largest (150%). Implemented by scaling the root font size, so **everything sized in `rem` scales together** — text, buttons, spacing — without breaking layout or requiring horizontal scrolling (WCAG 1.4.4 Resize Text).

### Text spacing
Three steps: Default, Roomy, and Widest. **Widest meets every WCAG 1.4.12 metric**: line height 1.85 (≥ 1.5×), letter spacing 0.12 em, word spacing 0.16 em, and paragraph spacing 2 em. Useful for readers with dyslexia or low vision who need text to breathe.

### Color palettes
Four palettes, each verified for AA contrast:

| Palette | What it is |
|---|---|
| **Page colors** | Your page exactly as you designed it. The widget changes nothing. |
| **Color-vision friendly** | Blue and orange — hues that stay separable across protan, deutan, and tritan color vision. It does **not** simulate color blindness; it removes the ambiguity. |
| **Monochrome** | Grayscale only. For readers who find color distracting, or to test that your page never relies on color alone. |
| **High contrast** | Yellow on black, ≥ 7:1 contrast (AAA-level), for low-vision readers. |

### Reading font — OpenDyslexic
A checkbox switches body text, buttons, and form fields to [OpenDyslexic](https://opendyslexic.org). The option is honestly labeled in the panel: research on whether it improves reading speed is mixed, so readers are told to *try it and keep it only if it helps*. Fonts are bundled (SIL OFL) and load locally — if they're missing, it falls back to Comic Sans MS / Verdana.

### Pointer
- **Large / Extra large cursor** — white arrow with black outline, drawn as inline SVG (no image files).
- **High contrast cursor** — yellow with a black outline.
- **Cursor trail** — an optional fading trail of dots behind the pointer, which makes it much easier to find and follow. Automatically skipped on touch input.

### Motion
Three options: **Match my device setting** (respects `prefers-reduced-motion`), **Slow down animation** (4.5× slower), and **Stop animation**. Implemented through a single `--dur` CSS variable, so any animation on your page that uses `var(--dur)` obeys it instantly (WCAG 2.2.2, 2.3.3).

### Button position — it moves!
The floating button can end up covering something a reader needs. So:
- Pick any of the **four corners** from the panel.
- **Drag** the button or the panel's grip handle anywhere on screen.
- **Keyboard-move it**: focus the grip and press the arrow keys (Shift for larger steps), or hold Alt + arrows on the button itself.
- **Dock it** into the page flow ("Keep it at the top of the page") — essential inside LMS iframes.

The position is clamped to the viewport so it can never be dragged off-screen, and it's restored on reload.

### Persistence
Every choice is saved to `localStorage` under one key and reapplied on the next visit. Pages on the same site that share the key share the reader's settings — set it once, it follows them through the whole course. A single **Reset all display settings** button returns everything to default.

### Screen reader support
Every change is announced through an `aria-live` status region ("Large text", "High contrast", "Cursor trail on", "Panel moved"…). The toggle has proper `aria-expanded`/`aria-controls` state, opening the panel moves focus into it, Escape closes it, and closing returns focus to the button.

---

## How it works

The architecture is deliberately simple — three cooperating layers, no framework:

```
┌────────────────────────────────────────────────────────┐
│  <html data-palette="hc" data-textsize="2" ...>        │  ← JS writes state here
├────────────────────────────────────────────────────────┤
│  CSS custom properties (design tokens)                 │
│  html[data-palette="hc"] { --bg:#000; --ink:#fff; … }  │  ← CSS reacts to the attributes
│  html[data-textsize="2"] { --fs: 1.30; }               │
├────────────────────────────────────────────────────────┤
│  Page styles consume the tokens                        │
│  html { font-size: calc(100% * var(--fs)); }           │  ← the page updates instantly
│  body  { background: var(--bg); color: var(--ink); }   │
└────────────────────────────────────────────────────────┘
```

1. **State lives in one JavaScript object** (`{ textsize, spacing, palette, cursor, motion, corner, dyslexic, trail, x, y }`), serialized to `localStorage` on every change and loaded on startup.
2. **`apply()` writes the state onto `<html>` as `data-*` attributes.** That's the entire render step.
3. **CSS does the rest.** Attribute selectors like `html[data-spacing="2"]` swap the values of CSS custom properties (`--fs`, `--lh`, `--bg`, `--dur`…), and everything built on those variables updates in one paint. No DOM walking, no style recalculation in JS, no flicker.

Two pieces deserve a closer look:

**The iframe dock detection.** Canvas embeds uploaded HTML pages in an iframe sized to the full height of the document. Inside one of those, `position: fixed` pins to the bottom of the *document*, not the reader's window — the button exists but is never visible. The widget detects this (`window.innerHeight >= document.scrollHeight`) and moves itself into a `#a11yHost` element in the normal page flow instead. An explicit reader choice always beats the auto-detection.

**The drag system.** Pointer Events (so mouse, touch, and pen all work) with `setPointerCapture`, a 5-pixel threshold so a click is never mistaken for a drag, viewport clamping, and a keyboard equivalent on the grip — because a draggable-only control would itself be an accessibility failure.

---

## How to add it to your page

### Step 1 — copy the block

Open [`accessibility-widget.html`](accessibility-widget.html), copy **the entire file**, and paste it into your page **just before the closing `</body>` tag**:

```html
    ...your page content...

    <!-- paste everything from accessibility-widget.html here -->

  </body>
</html>
```

That's genuinely it. The block contains the stylesheet, the markup, and the script, in that order, and nothing else on your page needs to change.

### Step 2 — add the fonts (optional but recommended)

Copy the [`fonts/`](fonts/) folder so it sits next to your HTML file:

```
my-page.html
fonts/
  OpenDyslexic-Regular.woff
  OpenDyslexic-Bold.woff
```

If you skip this, everything still works — the OpenDyslexic option just falls back to Comic Sans MS / Verdana. You can also point the two `@font-face` rules at a CDN instead:

```css
src: url('https://cdn.jsdelivr.net/npm/open-dyslexic@1.0.3/woff/OpenDyslexic-Regular.woff') format('woff');
```

(Only do that if a network request is acceptable to you — the whole widget is otherwise offline.)

### Step 3 — choose where it docks (optional)

If the page is viewed inside an auto-height iframe (Canvas), the widget docks into the page flow. By default it creates its landing spot at the very top of `<body>`. To control where it lands, put this wherever you like — right under your page header works well:

```html
<div id="a11yHost"></div>
```

### Checklist after adding

- [ ] The round **Display settings** button appears in the bottom-right corner.
- [ ] Clicking it opens the panel; **Escape** closes it; focus returns to the button.
- [ ] Each palette visibly changes the page; text size visibly scales.
- [ ] Reload the page — your settings are still applied.
- [ ] Print preview — the widget does not appear.

---

## Using it with AI tools

If you build pages with ChatGPT, Claude, Copilot, or any other AI assistant, you don't need to describe accessibility features and hope for the best — **hand the AI this exact code and tell it not to touch it.**

### The short version

1. Open [`accessibility-widget.html`](accessibility-widget.html) and copy the whole file.
2. In your AI chat, write your page request, then add:

> Include the following accessibility widget in the page **unchanged**, pasted just before `</body>`. Do not rename its IDs, classes, or CSS variables, and do not "improve" it. Build the rest of the page's CSS using its design tokens — `var(--bg)`, `var(--surface)`, `var(--ink)`, `var(--ink-soft)`, `var(--line)`, `var(--brand-1)`, `var(--brand-2)`, `var(--brand-3)`, `var(--on-brand)`, `var(--accent)` — and size things in `rem`, so the widget's palettes, text size, and spacing options restyle the entire page. Use `var(--dur)` as the duration for any animation so the motion settings control it.
>
> ```html
> [paste the full contents of accessibility-widget.html here]
> ```

3. Verify the result against the [checklist](#checklist-after-adding) above.

**[AI-USAGE.md](AI-USAGE.md) has the full guide** — a complete ready-to-paste prompt, why "unchanged" matters, what to check in the output, and how to retrofit the widget into pages the AI already generated.

---

## Making the palettes restyle your whole page

Out of the box on any page: text size, spacing, the dyslexic font, cursors, and motion all work, and the non-default palettes recolor the page background, text, and links.

For the palettes to restyle *everything* — headers, cards, tables, buttons — build your page's own CSS from the widget's tokens:

| Token | Use it for |
|---|---|
| `--bg` | page background |
| `--surface` | cards, table stripes, wells |
| `--ink` / `--ink-soft` | body text / secondary text |
| `--line` | borders, dividers |
| `--brand-1` `--brand-2` `--brand-3` | your brand colors (dark → mid → accent) |
| `--on-brand` | text on top of brand colors |
| `--accent` | links, highlights (becomes yellow in high contrast) |
| `--dur` | every `transition`/`animation` duration |

Example:

```css
body    { background: var(--bg); color: var(--ink); }
.header { background: var(--brand-1); color: var(--on-brand); }
.card   { background: var(--surface); border: 2px solid var(--line); }
a       { color: var(--brand-1); }
.fade   { transition: opacity var(--dur) ease; }
```

Because the palettes only swap these variables, your layout is untouched — colors change, nothing moves.

> **Heads-up:** if your stylesheet already defines custom properties with these names, rename them on one side or the other before combining.

---

## Customization

**Brand colors.** Edit the three `--brand-*` values (and `--accent`) in the `:root` block. The default is indigo/purple/magenta. Keep your replacements at AA contrast against white (or adjust `--on-brand`).

**Storage key.** The line `var KEY = 'tip-a11y-widget-v1';` controls persistence. Keep one key across a whole site so settings follow the reader; change it per course/site if you want them separate.

**Default corner.** Change `data-corner="br"` on `#a11yWidget` to `bl`, `tr`, or `tl` (and the matching `checked` radio in the Button position fieldset).

**Removing a feature.** Each feature is one `<fieldset>` in the panel plus its CSS section (clearly commented). To drop, say, the cursor trail: delete the trail checkbox label, the `.trail-dot` CSS, and the "Cursor trail" JS section. Nothing else references it.

**Panel text.** All labels are plain text in the markup — translate or reword freely. The spoken announcements live in the `LABELS` object in the script.

---

## Canvas and other LMS notes

- **Works:** uploading the HTML file to Canvas Files and embedding/linking it, or serving it from any web host inside an iframe. The widget auto-docks when the iframe is auto-height (see [How it works](#how-it-works)), and readers can force this with "Keep it at the top of the page."
- **Does not work:** pasting into the Canvas Rich Content Editor directly — Canvas strips `<style>` and `<script>` from RCE content. The page must be a *file*, not RCE text.
- The widget never touches the LMS itself. As the course pages tell students: it changes how the page looks, only on their device, and is not reported to anyone.

---

## WCAG mapping

What the widget provides or supports (WCAG 2.1):

| Criterion | How |
|---|---|
| 1.4.3 Contrast (AA) | All four palettes verified AA; high contrast reaches 7:1+ |
| 1.4.4 Resize Text (AA) | 150% text size without loss of content or function |
| 1.4.8 Visual Presentation (AAA) | Reader control of colors, spacing, and line width behavior |
| 1.4.12 Text Spacing (AA) | "Widest" meets all four metrics |
| 2.1.1 / 2.1.2 Keyboard (A) | Fully keyboard operable, no traps, Escape closes |
| 2.2.2 Pause, Stop, Hide (A) | Motion can be slowed or stopped |
| 2.3.3 Animation from Interactions (AAA) | `prefers-reduced-motion` honored by default |
| 2.4.7 Focus Visible (AA) | 3 px offset outlines, recolored per palette |
| 2.5.5 Target Size (AAA) | Every control ≥ 44 × 44 px |
| 4.1.2 Name, Role, Value (A) | Native controls, correct ARIA on the disclosure |
| 4.1.3 Status Messages (AA) | `aria-live` announcements for every change |

> **Honest scope:** the widget adds reader control on top of your page. It does **not** make an inaccessible page accessible — it can't add alt text, fix heading structure, or label your forms. Build the page right; add this so readers can tune it.

---

## Download

Grab everything from the **[Releases page](https://github.com/Tech-Inclusion-Pro/WCAG-Accessibility-Button/releases)** — each release includes a zip with `accessibility-widget.html`, the fonts, the demo, and this documentation. Or clone the repo:

```bash
git clone https://github.com/Tech-Inclusion-Pro/WCAG-Accessibility-Button.git
```

---

## Privacy

- No network requests of any kind (fonts are local files).
- No cookies, no analytics, no fingerprinting, no external services.
- Settings live in the reader's own browser via `localStorage` and never leave it.
- Clearing site data in the browser removes everything.

---

## License and credits

- Widget code: [MIT License](LICENSE) © 2026 Tech Inclusion Pro.
- [OpenDyslexic](https://opendyslexic.org) typeface by Abbie Gonzalez, [SIL Open Font License 1.1](https://openfontlicense.org) — see [`fonts/README.md`](fonts/README.md).
- Built by Rocco Catrone (Tech Inclusion Pro) for SPED 101 at the University of Illinois Chicago, and shared so anyone can use it.

*If this widget helps your students or readers, that's the whole point. Issues and suggestions welcome.*
