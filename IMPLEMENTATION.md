# Portfolio Website — Implementation Notes

**Live site:** `https://<username>.github.io` (GitHub Pages) · **Owner:** Ribhav Bansal
**Stack:** Single static `index.html` (vanilla HTML/CSS/JS, zero dependencies, no build step)

This document records how the site is built, why each decision was made, and how to maintain and deploy it. The original design brief lives in `portfolio-website-plan.md`.

---

## 1. Files

| File | Purpose | Deployed |
|---|---|---|
| `index.html` | Entire site — markup, CSS, and JS in one file | ✅ |
| `resume.pdf` | Resume, linked from nav / hero / contact | ✅ |
| `assets/*.webp` | Optimized screenshots (6 images, ~231 KB total) | ✅ |
| `images/` | Raw original screenshots (source material) | ❌ gitignored |
| `portfolio-website-plan.md` | Design/content brief | ❌ gitignored |
| `IMPLEMENTATION.md` | This file | optional |
| `.gitignore` | Excludes `images/`, plan doc, `.DS_Store` | ✅ |

## 2. Why a single static file

The plan offered Next.js or a static-file fallback. Static won because:

- Zero dependencies and no build step — nothing to break, trivially portable to any host.
- Performance budget (Lighthouse ≥ 95) is easy: one HTML request, lazy-loaded WebP images, ~5 KB of JS.
- GitHub Pages serves it directly; every `git push` is a deploy.

## 3. Design system

Concept: **"Signal from noise"** — clean, instrument-like UI with one signature live moment (the typing transcript), echoing what MeetingMind does.

### Palette (CSS custom properties on `:root`, inverted on `html.dark`)

| Token | Light | Dark |
|---|---|---|
| `--bg` | `#FBFBF9` paper white | `#101014` |
| `--ink` | `#16161A` | `#ECECEF` |
| `--muted` | `#5C5C66` | `#9A9AA6` |
| `--accent` | `#5046E5` electric indigo | `#7B72FF` (brightened for contrast) |
| `--surface` | `#F1F1F4` | `#1A1A21` |
| `--line` | `#E3E3E8` hairlines | `#26262E` |
| `--card` | `#FFFFFF` | `#16161C` |

### Typography (Google Fonts)

- **Inter Tight** 600–800 — display/headings, tight tracking, oversized hero
- **Inter** 400–600 — body
- **JetBrains Mono** — metadata: tech chips, dates, kickers, transcript, badges

### Layout & motion

- Max-width 1100 px, single column; flagship and skills sections are full-bleed surface bands with hairline borders.
- Cards: hairline borders, 12 px radius, no drop shadows; hover = accent border.
- Motion: one typing animation + scroll-fade reveals (IntersectionObserver, 450 ms). `prefers-reduced-motion` renders everything static.

## 4. Page structure

```
nav (sticky, blurred)   RB · Work · Experience · Skills · Contact · GitHub · Resume ↓ · theme toggle
HERO                    H1, role line, value prop, 2 CTAs, live-transcript widget
FLAGSHIP #work          MeetingMind: pitch · 4 capability cards · 3-image gallery ·
                        architecture strip · 12 tech chips · GitHub link
EXPERIENCE #experience  PhonePe timeline (SDE Jul 2025–now, Intern Jan–Jun 2025)
PROJECTS #projects      2 cards with cover images: House Price Prediction, Easy Food Recipes
SKILLS #skills          4 grouped skill cards + achievements strip
CONTACT #contact        Email / GitHub / Resume buttons
footer                  © · IIIT Allahabad · email
```

## 5. JavaScript behavior (~70 lines, inline)

1. **Theme** — follows `prefers-color-scheme` by default; toggle stores override in `localStorage` (`rb-theme`); system changes still apply while no manual override exists.
2. **Scroll reveal** — IntersectionObserver adds `.in` to `.reveal` elements at 12 % visibility, unobserves after firing.
3. **Typing transcript** — hero widget types a meeting sentence char-by-char (34 ms/char); entity spans get a highlight fade after completing; loops every 6 s. Under reduced motion it renders instantly with highlights on.

## 6. Images

Originals in `images/`, optimized derivatives in `assets/` (Pillow: resize + WebP q82):

| Asset | Source | Used in |
|---|---|---|
| `mm-graph.webp` 1600px | Knowledge-graph force view | MeetingMind gallery (lead, full width) |
| `mm-insights.webp` 1600px | Insights / commitment-drift dashboard | gallery row |
| `mm-meeting.webp` 1400px | Meeting detail (summary, action items, exports) | gallery row |
| `mm-overlay.webp` 560px | In-meeting overlay popup | spare (unused) |
| `houseprice.webp` 1400px | Price-prediction web UI | project card |
| `foodapp.webp` 700px | Recipes app home screen | project card |

All `<img>` have `alt` text, `loading="lazy"`, and the gallery images carry explicit `width`/`height` to prevent layout shift. To add an image: drop the original in `images/`, create a resized WebP in `assets/`, reference it.

## 7. SEO, sharing, accessibility

- Title/meta description; Open Graph tags (title, description, type, url) for LinkedIn/iMessage previews.
- Favicon: inline SVG data URI ("RB" on indigo) — no extra request.
- Semantic landmarks (`nav/header/section/footer`), AA contrast in both themes, visible `:focus-visible` outlines, `aria-label` on the theme toggle, decorative transcript marked `aria-hidden`.
- Privacy: no phone number or address anywhere on the site (resume PDF may keep phone).

## 8. Deployment — GitHub Pages

One-time setup:

1. Create a **public** repo named exactly `<username>.github.io` at github.com/new (no README).
2. ```bash
   cd ~/Desktop/Projects/WebPortfolio
   git init && git add . && git commit -m "Portfolio site"
   git branch -M main
   git remote add origin https://github.com/<username>/<username>.github.io.git
   git push -u origin main
   ```
3. Wait 1–2 min → site is live at `https://<username>.github.io`.
   If not: repo → Settings → Pages → Deploy from a branch → `main` / `(root)`.

Why it stays live: Pages serves the repo's static files directly — no server, build, or expiry. **Updating the site = edit files → `git add -A && git commit -m "..." && git push`** (live in ~1 min).

Custom domain (optional later): buy domain → repo Settings → Pages → Custom domain → add the DNS records GitHub shows → enforce HTTPS.

## 9. Content updates cheat-sheet

| Change | Where in `index.html` |
|---|---|
| Role/value line | `<header class="hero">` |
| MeetingMind capabilities | `.cap-grid` cards |
| Gallery captions | `<figcaption>` in `.gallery` |
| Experience bullets | `#experience` `.xp-item` lists |
| Project cards | `#projects` `.proj` blocks |
| Skills / achievements | `#skills` `.skill-group` / `.ach` |
| Accent color | `--accent` in `:root` and `html.dark` |
| Resume | replace `resume.pdf` (keep filename) |

## 10. Known gaps / phase-2 ideas

- GitHub links point to the profile (`github.com/RibhavBansal`) — swap to real repo URLs when available.
- LinkedIn omitted until exact profile slug is confirmed.
- Optional `/meetingmind.html` case-study deep-dive (problem → architecture → key decisions → screenshots).
- Custom OG image (currently text-only preview) and `sitemap.xml` if a custom domain is added.
