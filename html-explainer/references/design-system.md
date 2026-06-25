---
version: "neuform-top-creators-featured"
name: "Platform Features - Remixed"
description: "Platform Features Section is designed for showcasing product surfaces and technical hierarchy with editorial calm. Key features include reusable bento layout, warm light surface, restrained shadow depth, and production-ready styling. Built with custom CSS, it is suitable for explainer pages, component libraries, and responsive product interfaces."
colors:
  primary: "#D97706"
  secondary: "#FAFAFA"
  accent: "#F59E0B"
  background: "#FAFAFA"
  surface: "#FFFFFF"
  surface-alt: "#F4F1EB"
  text-primary: "#111827"
  text-secondary: "#4B5563"
  text-muted: "#9CA3AF"
  border: "#E5E7EB"
  border-strong: "#D1D5DB"
  warm-1: "#FCD34D"
  warm-2: "#F59E0B"
  warm-3: "#EF4444"
  success: "#10B981"
typography:
  display-lg:
    fontFamily: "Newsreader"
    fontSize: "64px"
    fontWeight: 500
    lineHeight: "1.04"
    letterSpacing: "-0.01em"
  display-md:
    fontFamily: "Newsreader"
    fontSize: "32px"
    fontWeight: 500
    lineHeight: "1.1"
    letterSpacing: "-0.01em"
  body-md:
    fontFamily: "Newsreader"
    fontSize: "17px"
    fontWeight: 400
    lineHeight: "1.55"
  body-sm:
    fontFamily: "Inter"
    fontSize: "14px"
    fontWeight: 400
    lineHeight: "1.5"
  label-md:
    fontFamily: "JetBrains Mono"
    fontSize: "12px"
    fontWeight: 600
    lineHeight: "1.2"
    letterSpacing: "0.04em"
  code:
    fontFamily: "JetBrains Mono"
    fontSize: "13px"
    fontWeight: 500
    lineHeight: "1.55"
spacing:
  base: "8px"
  gap: "16px"
  card-padding: "32px"
  section-padding: "96px"
rounded:
  card: "16px"
  control: "10px"
  chip: "6px"
  pill: "9999px"
shadow:
  card: "0 1px 2px rgba(17, 24, 39, 0.04), 0 8px 24px rgba(17, 24, 39, 0.06)"
  card-hover: "0 2px 4px rgba(17, 24, 39, 0.05), 0 16px 40px rgba(17, 24, 39, 0.08)"
  chip: "0 1px 2px rgba(17, 24, 39, 0.05)"
components:
  card:
    background: "Pure white surface (#FFFFFF) on warm-cream page background. Use the card radius token, hairline 1px border in --border, and the card shadow for lift. Never flatten cards onto the page — the depth differential is the entire device."
    radius: "Match the declared card radius token"
  button:
    background: "Default to a white pill with thin border and subtle shadow; reserve primary/accent fills for the single most important action per surface."
    radius: "Use the pill radius for primary actions, control radius for in-card secondary buttons"
  chip:
    background: "Pill-shaped, white background, hairline border. Status chips (PASS / FAIL / Query executed!) carry a small leading dot icon coloured by --success or a neutral grey for negative states."
    radius: "Use the pill radius"
  code-block:
    background: "Inline on the card surface — no dark fill. Use mono type, a leading `>` prompt glyph per line, and tint identifiers/values in --primary."
---
# Platform Features - Remixed
Source: Neuform Featured templates from top creators. Author: Meng To (@mengto). Views: 26; favorites: 2; remixes: 7.
Tags: feature, section, bento, editorial, warm-light, animated.

## Overview
A warm-light editorial product showcase. Two-up bento cards sit on a cream page; each card pairs a serif heading and short body with one focal product moment — a tagged flow diagram on the left card and a code block plus a layered folder illustration on the right. The mood is calm, optimistic, and editorial — the opposite of dense dark dashboards.

## Composition
Use the attached HTML reference as the source of truth. Preserve the visible hierarchy, first-screen composition, section rhythm, density, and interaction tone before adapting copy or content.

Key composition rules:
- **Two-up bento on desktop**, single column on narrow viewports. Cards are equal width with a generous gutter (24–32px).
- **Generous internal padding** (~32px). Heading sits at the top with a single body paragraph below; the focal artwork occupies the lower half of the card.
- **One focal artifact per card** — a flow with status chips, a code block + folders, a chart, etc. Never crowd a card with two unrelated artifacts.
- **Edge breathing room** — the page itself has a wide outer margin and a dotted-grid base band that reads as a subtle floor.

## Colors
Anchor the palette in primary `#D97706`, accent `#F59E0B`, and the warm trio (`#FCD34D` → `#F59E0B` → `#EF4444`) used only inside the focal illustration. The page background is warm off-white `#FAFAFA`; cards are pure `#FFFFFF` so depth comes from the surface differential plus a soft shadow, never from heavy borders. Text is near-black `#111827` for headings, slate `#4B5563` for body. Status uses `#10B981` (success) for PASS-style chips and neutral grey for FAIL/disabled chips.

Keep colour usage restrained: most surfaces are white-on-cream with grey text; the warm hues exist to make a single product moment per card sing.

## Typography
- **Display & body** use **Newsreader** (serif, low-contrast, optical) — both headings and prose. The serif is the signature; do not substitute a sans.
- **Inter** is acceptable for very small UI labels inside artifacts (chip labels, axis ticks) where the serif would feel too soft.
- **JetBrains Mono** for technical metadata, code blocks, kbd, and short tags.
- Headings ride a tight leading (1.04–1.1) with a small negative letter-spacing; body copy is comfortably loose (≈1.55).
- Code blocks render as mono on the card surface (no dark fill), each line prefixed with a faint `>` prompt; identifiers and string values pick up the primary orange.

## Layout
- Section padding is generous (96px vertical). Cards are 16px radius; controls and code chips are 10px; status chips are full pills.
- Grid is two equal columns with a wide gutter; do not collapse to three or four columns — the two-up rhythm is part of the identity.
- A subtle **dotted-grid base band** sits below the card row, indicating the page floor without competing for attention.
- All vertical rhythm follows the 8px base; gaps between heading, body, and artifact are 16/24px.

## Components
- **Card**: white fill, 1px hairline border in `--border`, soft 2-layer shadow for lift. Hover lifts the shadow slightly and may translate the card by 2px; nothing more dramatic.
- **Status chip**: pill, white, hairline border, leading 6px coloured dot (✓ success, ⊘ neutral, etc.). Type is mono or Inter at 12px.
- **Flow node** (inside the left card's diagram): white pill or rounded rect, faint border, optional small leading avatar/icon circle, status chip floating top-right when a state is meaningful. Connecting lines are thin (1px) and faded grey; never bold the connectors.
- **Code block**: lives directly on the card surface. Mono font, leading `>` glyphs in faint grey, identifiers tinted in primary orange. Below the block, a single confirmation chip ("Query executed!") can echo the outcome.
- **Illustration (right card)**: warm folder/document stack drawn with the warm trio gradient (yellow → orange → red), large and slightly cropped at the card edge so it reads as an object rather than an icon.
- **Buttons**: default white pill with hairline border and chip shadow; reserve a single filled primary/accent button per surface for the most important action.

## Motion
Preserve restrained editorial motion: staggered fade-and-rise on first reveal (≈400ms, ease-out, 60–80ms stagger), 2px card hover lift, gentle status-chip pop on state change. **No parallax, no glow trails, no neon pulses.** If an animation feels "techy," it is wrong for this aesthetic.

## WebGL & Effects
This aesthetic does **not** use canvas, WebGL, or Three.js. If atmospheric texture is needed, use a flat CSS dotted-grid or a faint radial wash behind the focal illustration — nothing more.

## Guardrails
- Do not flatten the source into a generic card grid — the two-up bento with one focal artifact per card is the device.
- Do not swap to a dark mode or to a saturated palette; the warm-light surface is the identity.
- Do not introduce serifs other than Newsreader, or replace it with a sans for headings.
- Do not bold the connector lines in flow diagrams, and do not let connectors compete with chips or labels.
- Do not stack more than one filled primary button per card surface.
- Do not let the warm gradient (yellow→orange→red) bleed outside the focal illustration; the rest of the page stays neutral.
- Preserve the first viewport signal, focal object, and visual density.
- Keep buttons, cards, and badges aligned to the same radius and border language.
