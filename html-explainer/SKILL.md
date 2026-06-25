---
name: html-explainer
description: "Build a self-contained HTML explainer or slide deck in the warm-light editorial house style (Newsreader serif, amber palette, bento cards). Use for /html-explainer, 'make an explainer', 'build a deck', or visualising a system, flow, or analysis as a styled page."
disable-model-invocation: false
---

# HTML explainer

Produce a single self-contained `.html` file (inline CSS, Google Fonts, no build step) that explains a system, flow, or analysis in the warm-light editorial house style. It opens straight in a browser.

The design system is the source of truth. Read `references/design-system.md` before writing and follow its tokens, components, and guardrails. Do not invent a palette or swap the typeface.

## Two modes

- **Explainer (default).** A scrolling page of bento cards. Each section is a full-sentence serif heading that states the takeaway, a short body paragraph, and one focal artifact. Two-up bento on desktop, single column on narrow. Concise by default; a "detailed" explainer is the same system with more sections and deeper artifacts, not a denser layout.
- **Deck.** One slide per concept. A mono kicker label, a serif heading, and a two-column layout with text beside a diagram. Entity and schema diagrams, one idea per slide. Use when the content is a presentation walkthrough rather than a page to scroll.

Pick the mode from how the content will be used. When it is unstated, default to explainer and say so.

## Interactivity archetypes

An explainer can be static or interactive. Three archetypes, cheapest first, all built with self-contained vanilla JS in the one file, no framework:

- **Static diagram.** A labelled figure with no time element. The default; reach past it only when motion or input genuinely adds understanding.
- **Phase machine.** A sequence that auto-plays through discrete steps, with play, pause, restart, and a clickable timeline. For protocols, race conditions, anything stepwise.
- **Interactive simulator.** User-driven, the reader changes an input and watches the output evolve (the blended-IRR slider is one). For exploring a relationship.

Pick the cheapest that tells the story; do not reach for a simulator when a static diagram says it.

Animation discipline: per-property transitions via inline style or a single class toggle, not blanket `transition: all`; asymmetric easing so a return motion hides behind a fade rather than visibly reversing; restrained throughout (a fade-and-rise on reveal, a small hover lift), no parallax or glow. Always verify in the browser, client JS can fail silently while the static markup still looks fine.

This skill builds the self-contained, portable artifact. If the figure must instead live inside an existing React or MDX site, a component-based approach fits better than a single file; that is a different tool, not this one.

## One focal artifact per card

The bento with a single focal artifact per card is the whole identity. Never crowd a card with two unrelated artifacts. Choose the artifact that carries the idea:

- **Flow.** White pill nodes joined by thin 1px faded connectors, with a status chip floating where a state matters. Never bold the connectors.
- **Code block.** On the card surface, no dark fill. Mono, a faint leading `>` per line, identifiers tinted in the primary amber. A confirmation chip below can echo the outcome.
- **Status chips.** Pill, white, a 6px leading dot coloured by success or neutral grey.
- **Schema or entity.** Labelled boxes with thin joins, used most in deck mode.
- **Illustration.** The warm yellow to orange to red gradient, large and slightly cropped, confined to this one focal moment. The rest of the page stays neutral.

## Voice

Headings are calm declarative sentences naming the takeaway, not labels ("Currency-correct by default", not "Currency"). Body is short serif prose. Follow the global writing rules: no em dashes, no colon as a mid-sentence connector, no inline-header lists that restate the line.

## Build steps

1. Read `references/design-system.md`.
2. Outline the sections (explainer) or slides (deck), one idea each, and pick the focal artifact for each.
3. Write one self-contained HTML file: the design tokens as inline CSS variables, Google Fonts for Newsreader, Inter, and JetBrains Mono, all styles in a single `<style>` block. No external CSS or JS frameworks.
4. One focal artifact per card. Keep the two-up bento rhythm; do not collapse to a generic 3 or 4 column grid.
5. Motion stays restrained: a fade-and-rise on first reveal (about 400ms ease-out, 60 to 80ms stagger) and a 2px card hover lift. No parallax, no glow, no neon, no canvas or WebGL.
6. Close with a dotted-grid base band as the floor.
7. Open it for review (`open <file>` on macOS).

## Guardrails

- Warm-light surface only. No dark mode, no saturated palette.
- Newsreader for every heading and body line. Inter only for tiny artifact labels, JetBrains Mono for code, tags, and metadata.
- Thin, faded connectors. One filled primary action per surface at most.
- Keep the warm gradient inside the focal illustration; the rest stays white-on-cream with grey text.
- Depth comes from the surface differential and a soft shadow, never heavy borders.

## Reference outputs

If you have existing explainers or decks in this house style, study one before building to match the rhythm and tokens. Otherwise the design system below is the source of truth.
