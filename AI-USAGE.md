# Using the WCAG Accessibility Button with AI tools

A guide for adding the widget to pages you build with ChatGPT, Claude, Copilot, Gemini, or any other AI assistant.

## Why paste the code instead of describing it?

If you ask an AI to "add an accessibility widget," it will invent one — usually a half-working panel with untested contrast, no keyboard support, no persistence, and no screen reader announcements. You'll get something that *looks* like accessibility.

This widget has already been built, used in real university courses, and checked against WCAG 2.1 AA. So the reliable workflow is:

> **Give the AI the finished code and instruct it to include the block unchanged.**

AI tools are very good at following "insert this verbatim" — and very unreliable at reinventing 700 lines of tested accessibility behavior.

## The ready-to-paste prompt

Copy the block below into your AI chat, then paste the **entire contents of [`accessibility-widget.html`](accessibility-widget.html)** where indicated, then describe the page you actually want.

```text
I'm going to give you a tested, self-contained accessibility widget
(CSS + HTML + JS in one block), and then describe a web page I want
you to build. Rules for the widget:

1. Include the ENTIRE widget block in the page, unchanged, pasted
   immediately before the closing </body> tag.
2. Do NOT rename, refactor, reformat, "clean up," or omit any part of
   it. Do not rename its IDs (a11yWidget, a11yToggle, a11yPanel,
   a11yHost...), its classes (.a11y, .fab, .panel...), or its CSS
   custom properties.
3. Do not add any other style or script that targets html[data-*]
   attributes or redefines --bg, --surface, --ink, --ink-soft, --line,
   --brand-1, --brand-2, --brand-3, --on-brand, --accent, --focus,
   --radius, --dur, --fs, --lh, --ls, --ws, or --para.

Rules for the rest of the page, so the widget's options control it:

4. Write the page's own CSS using the widget's design tokens:
   background: var(--bg); surfaces/cards: var(--surface); body text:
   var(--ink); secondary text: var(--ink-soft); borders: var(--line);
   brand/heading colors: var(--brand-1..3) with var(--on-brand) for
   text on them; links and highlights: var(--brand-1) or var(--accent).
5. Size text, spacing, and components in rem (not px) so the text-size
   setting scales everything.
6. Use var(--dur) as the duration of every CSS transition and
   animation so the motion setting controls them.
7. Follow normal accessibility practice in the content itself:
   semantic headings in order, alt text on images, labels on form
   fields, and no information conveyed by color alone.

Here is the widget block:

[PASTE THE FULL CONTENTS OF accessibility-widget.html HERE]

Now the page I want:

[DESCRIBE YOUR PAGE HERE]
```

## Retrofitting a page the AI already made

Already have a generated page? Use this instead:

```text
Here is an existing HTML page, and below it a tested accessibility
widget. Insert the widget block unchanged immediately before </body>.
Then update ONLY the page's CSS so colors come from the widget's
design tokens (var(--bg), var(--surface), var(--ink), var(--ink-soft),
var(--line), var(--brand-1..3), var(--on-brand), var(--accent)),
font sizes use rem, and animation durations use var(--dur). Do not
change the page's structure, text, or behavior, and do not modify the
widget itself.

[PASTE YOUR EXISTING PAGE]

[PASTE THE FULL CONTENTS OF accessibility-widget.html]
```

## Verifying what the AI gives back

AI tools sometimes "helpfully" rewrite things. After generation, check:

1. **The block survived intact.** Search the output for `a11yWidget` and `tip-a11y-widget-v1`. Spot-check that the `<style>`, markup, and `<script>` sections all made it.
2. **Open the page in a browser.** The round *Display settings* button appears bottom-right.
3. **Click through every panel section once.** Palettes recolor the whole page (not just the widget), text size scales the page, the spinner/any animation obeys the motion setting.
4. **Press keys.** Tab reaches the button, Enter opens the panel, Escape closes it and focus returns.
5. **Reload.** Your settings persist.
6. **If something broke**, the usual culprit is the AI redefining a token or hard-coding colors. Tell it: *"You modified the widget / hard-coded colors. Restore the widget block exactly as given and use the CSS variables instead."*

## Two honest notes

- **Fonts:** the OpenDyslexic option looks for `fonts/OpenDyslexic-Regular.woff` and `fonts/OpenDyslexic-Bold.woff` next to the page. AI chat tools can't give you font binaries — copy the [`fonts/`](fonts/) folder from this repository yourself, or let the option fall back to Comic Sans/Verdana.
- **The widget is not a page fixer.** It gives readers control over presentation. Heading structure, alt text, form labels, and reading order are still the page author's job — rule 7 in the prompt reminds the AI of that, but verify it.
