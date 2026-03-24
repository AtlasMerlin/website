# Atlas Merlin — Build Specification

## Stack
- **Framework:** Astro v6 (static site)
- **Styling:** Vanilla CSS with custom properties — no Tailwind
- **Fonts:** Google Fonts — Playfair Display (headings) + Lora (body)
- **Forms:** Formspree (endpoint: `xpqyvavy`)
- **Deployment:** Vercel, watching the `dev` branch
- **Repo:** GitHub — AtlasMerlin/website, branch: `dev`

---

## Brand Tokens

```css
--deep-navy:       #0F1A2E
--dark-navy:       #1A2940
--mid-navy:        #2E4057
--copper-light:    #D4873F
--copper-dark:     #DA9048
--electric-orange: #FF6B1A
--warm-white:      #F2F0EB
```

---

## Design Principles
- **Warm-white dominant** — navy and orange as accents only (not large background blocks)
- **No JS theme toggle** — pure CSS `prefers-color-scheme: dark` only
- **Clean and light** — generous whitespace, restrained colour use
- **Section spacing** — `.section { padding: 2rem 0 }` — all sections consistent

---

## Dark / Light Mode
Pure CSS via `@media (prefers-color-scheme: dark)` — no JavaScript, no localStorage, no toggle. Matches approach used on the live holding page at atlasmerlin.com. Theme variables are set in `:root` and overridden in the media query.

Key theme variables:
```css
--bg, --text, --heading, --border
--logo-main, --logo-diamond
--nav-bg, --nav-text, --nav-border
```

---

## Component Architecture

```
src/
  layouts/
    BaseLayout.astro        — HTML shell, meta, OG, JSON-LD, favicons
  components/
    Logo.astro              — Inline SVG logo, CSS variable fills, diamond animation
    Nav.astro               — Fixed nav, transparent → solid on scroll, login dropdown
    Hero.astro              — Full-screen video hero, parallax, scroll chevron
    Problem.astro           — Section 2
    Solutions.astro         — Section 3, alternating rows
    HowItWorks.astro        — Section 4, horizontal timeline (desktop) / vertical (mobile)
    WhyAtlasMerlin.astro    — Section 5, 2×2 grid
    Contact.astro           — Section 6, Formspree, spinner → tick on submit
    Footer.astro            — Deep navy, matches contact section
  pages/
    index.astro             — Imports and renders all sections
  styles/
    global.css              — Tokens, reset, typography, layout, buttons
```

---

## Logo
- SVG read at **build time** via `fs.readFileSync` in Astro frontmatter
- File: `public/darklogo_for_lightbackground_desktop.svg`
- Hardcoded fills replaced with CSS variables:
  - `#0f1a2e` → `fill="var(--logo-main)"`
  - `#d4873f` → `fill="var(--logo-diamond)"`
  - `#f2f0eb` → `fill="none"` (hidden background rect)
- Diamond (`#path13`) has `transform-origin: center; transform-box: fill-box`
- **Diamond animation:** rotates on scroll (`scrollY * 0.08` degrees), snaps back to nearest 90° with spring easing (`cubic-bezier(0.34, 1.56, 0.64, 1)`) 10ms after scroll stops

---

## Nav
- `position: fixed`, `z-index: 100`
- Starts **transparent** — hero video bleeds behind it seamlessly
- Gains `background-color` + `box-shadow` on scroll (`.scrolled` class via JS)
- Nav height: `--nav-height: 96px`
- Mobile breakpoint: `768px` — links/CTA hidden, hamburger shown
- **Login button:** person icon (SVG), opens dropdown modal anchored to button position via `getBoundingClientRect()`. On mobile, sits to the left of hamburger. On submit shows "Atlas Merlin needs to create your account" message.

---

## Hero
- `min-height: 100vh`, `padding-bottom: 2rem`
- **Video background:** `<video autoplay muted loop playsinline>`
- Current video: `public/videos/hero-preview-2.mp4` (converted from .mov via ffmpeg)
- Playback rate: `0.5` (half speed)
- **Overlay:** gradient — fully opaque at top 12% (matches nav), fades to 0.82 opacity below
  - Light: `rgba(242, 240, 235, ...)`
  - Dark: `rgba(15, 26, 46, ...)`
- **Parallax:** video wrapper moves at `scrollY * 0.4` on scroll
- **Scroll chevron:** bouncing SVG arrow at bottom, links to `#problem`
- Heading font: `clamp(1.6rem, 2.6vw, 2.2rem)` (smaller than global h1 to fit long line)

---

## Solutions Section
- 5 rows alternating text/image (`.solution--reverse` flips image to left)
- "World First" badge on Foreign Object Detector — small orange pill
- Image placeholders: `aspect-ratio: 4/3`
- "Your Challenge" row has orange CTA button

---

## How It Works Section
- **Desktop:** horizontal timeline — 5 columns, connecting line via `::before`, orange-outlined circle nodes
- **Mobile:** vertical timeline — vertical line on left, nodes on line, content to right
- Line: `1px solid var(--border)`
- Node: `40px` circle, `1.5px solid var(--electric-orange)` border

---

## Contact Section
- Background: `public/atlas-office-logo.png` (office exterior with logo on rock) with `rgba(15, 26, 46, 0.88)` overlay via `::before` pseudo-element — photo subtly visible beneath the dark navy
- Two-column: form left, "Got a Challenge?" aside right (copper left-border accent)
- **Formspree:** AJAX submit with `Accept: application/json` header
- **Submit button states:**
  1. Default: orange, "Send Message"
  2. Loading: spinner (CSS animation)
  3. Success: tick SVG (same orange button, no colour change)
- Email field uses `type="email"` for native browser validation

---

## SEO
- **Title:** Atlas Merlin — AI-Powered Computer Vision for Mining & Construction
- **Meta description:** Real-time computer vision solutions for mining and large-scale construction. Edge-first AI monitoring — from hardware to alerts — fully managed, no technical team required.
- **Canonical:** `https://atlasmerlin.com`
- **OG + Twitter cards:** set up in BaseLayout
- **JSON-LD:** Organization schema — name, URL, email, sameAs (atlasmerlin.com + atlasmerlin.ai), knowsAbout
- **Sitemap:** `@astrojs/sitemap` installed, configured in `astro.config.mjs` with `site: 'https://atlasmerlin.com'`
- Social preview image: `public/index-socialpreview-1200x630.png`

---

## Domains
- Primary: `atlasmerlin.com`
- Also owned: `atlasmerlin.ai` (redirects to .com)
- Canonical points to `.com` — both listed in JSON-LD `sameAs`

---

## Favicons
- Light mode: `public/favicon-32x32-lightmode.png`
- Dark mode: `public/favicon-32x32-darkmode.png`
- Apple touch: `public/favicon-180x180-appletouch.png`
- Served via `<link media="(prefers-color-scheme: ...)">`

---

## Videos
- Source files (`.mov`) excluded from git via `.gitignore`
- Converted to `.mp4` (H.264) via ffmpeg for browser compatibility
- `hero-preview-1.mp4` — open pit mining, haul trucks
- `hero-preview-2.mp4` — currently active
- ffmpeg command: `ffmpeg -i input.mov -vcodec h264 -acodec aac -crf 23 -preset fast -movflags +faststart output.mp4`

---

## Key Design Decisions
- **Transparent nav + hero bleed:** nav starts transparent so the hero video fills behind it, gaining solid background on scroll. Creates seamless top-of-page feel.
- **Section spacing:** `.section { padding: 2rem 0 }` — all sections same padding. Hero has `padding-bottom: 2rem` to match. Total gap between sections = 4rem.
- **`scroll-margin-top: var(--nav-height)`** on all `.section` elements — prevents fixed nav from obscuring headings when clicking nav links.
- **Single page** — deliberate choice for this stage. Future phase: individual solution pages for SEO targeting specific search terms.
- **No manual dark/light toggle** — OS preference only, instant, no JS.

---

## Pending / Future Work
- Replace all image placeholders with real photography/renders
- Diamond logo animation already set up — `#path13` has transform-origin ready
- Individual solution pages (phase 2 — SEO value for specific search terms)
- Real login/auth system behind the login modal
- Case studies / proof section once customer results are documentable
