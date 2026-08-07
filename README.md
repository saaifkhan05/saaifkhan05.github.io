# Saaif Khan — Interactive Portfolio (saaifkhan.com)

Complete project context. Read this first if you are picking up this project in a new chat.

---

## 1. The goal

A one-of-a-kind interactive portfolio that recruiters, peers, and anyone curious can explore. Saaif's priorities, in his order:

- **Interactive hub, not a scrolling page.** One entry point, then visitors dive into whatever they care about.
- **Capture attention in the first ~7 seconds.** No learning curve. Everything button-driven; keyboard is optional sugar.
- **Deep pages are narrative case studies, never resume bullets.** Story, photos, artifacts, metrics.
- **Must feel shocking, not AI-generated.** Earlier drafts were rejected explicitly for looking like generic AI output.
- **Dark theme by default**, light toggle available.
- **Free hosting + custom domain** (saaifkhan.com). ~$10–12/yr for the domain is the only cost.

**Audience note (important):** Saaif is going into **technical sales / solutions engineering / enterprise sales**, not software engineering. Copy is written for SE and AE hiring managers — discovery, demos, value translation, ROI. The site itself is the artifact: an AE intern who shipped a 3D spatial interface is a stronger proof than any bullet list.

## 2. The design — "The Fusion", draft 6

`index.html` is the working file. Single self-contained HTML. It fuses a Prezi-style spatial canvas with 3D depth navigation:

- **True 3D world.** All content lives as "frames" on one 9000×6400 spatial map at three real z-depths. Home is the front layer; L2 branches sit 750px deeper; L3 dossiers 750px beyond that. Deeper frames are visibly small and fogged (real CSS 3D perspective + parallax).
- **Camera flight navigation.** Clicking flies the camera through 3D space — pan + z-travel + rotation. Flying deep passes *through* intermediate layers, which blur past the camera.
- **Warp starfield** (see 2g), intensity scaled to travel distance.
- **Mouse parallax tilt** on the idle scene.
- **Navigation chrome (all visible, no keyboard needed):** top-left brand + **Back button** + clickable breadcrumbs; top-right Map / Index / theme; bottom-right depth gauge + clickable minimap. Esc and Backspace also go back.
- **Boot sequence:** name flash → overview of the whole map → auto-dive to Home (~1.5s).
- **Fonts:** Bricolage Grotesque (display), Hanken Grotesk (body), Red Hat Mono (labels).

### 2a. Colour system

Draft 5's cyan `#43d9ff` + hot red `#ff3b57` on navy was cut — that exact palette is the default of every AI-generated dev portfolio, and two saturated accents fighting each other read as arcade, not expensive.

**The shell is monochrome.** HUD, breadcrumbs, home frame, buttons, drawer carry *zero* accent hue. Bases are warm-neutral, not blue-black:

| | dark (default) | light |
|---|---|---|
| `--bg` | `#0b0a09` | `#efebe3` |
| `--surface` | `#141311` | `#fbf8f2` |
| `--line` / `--line-2` | `#2b2724` / `#3d3833` | `#d5cfc3` / `#bcb5a6` |
| `--ink` (bone, not white) | `#f4f0e8` | `#16130f` |
| `--dim` / `--dimmer` | `#8b8378` / `#5e574e` | `#6a6357` / `#948c7e` |

**All colour comes from one OKLCH variable.** Only the hue changes per context; lightness and chroma are locked, so no accent shouts louder than another and every one clears contrast on both themes:

```css
--ax: oklch(var(--ax-l) var(--ax-c) var(--ax-h));
/* dark: L .80 C .145   ·   light: L .50 C .155 */
```

Hues are **anchored to the real brand mark, then spaced to a 32° minimum**. Purely literal derivation broke as soon as the real company list arrived — Splunk green (135) collided with Workiva (144), Splunk pink (16) with eBay red (25), Cisco blue (234) sat too close to IBM (262). Anchor, then separate:

| context | source mark | derived | final | note |
|---|---|---|---|---|
| eBay | `#e53238` | 25 | **30** | +5 |
| UT Dallas | `#e87500` | 54 | **62** | +8 |
| Alpha Kappa Psi | — | — | **105** | assigned |
| Workiva | `#66cc66` | 144 | **148** | +4 |
| U.S. Renal Care | brand blue collided | — | **195** | assigned |
| Cisco (Splunk) | `#049fd9` | 234 | **228** | −6 |
| IBM | `#0f62fe` | 262 | **262** | anchor kept |
| Contact | — | — | **300** | assigned |
| Personal | — | — | **340** | assigned |

Two hooks:
- `.ax-{name}` sets the hue on a **frame** — its border, headings, buttons, metrics, quote rule.
- `data-ctx="{name}"` on a frame is copied to `<html>` by `setCtx()` on every flight, so **the HUD tints to match wherever you are** (breadcrumb, depth gauge, minimap dot and selection box, Back button hover). Home and overview set chroma to 0 and the shell goes neutral again.

**To add a company:** one `.ax-x{--ax-h:N}` rule, one `[data-ctx="x"]` rule, the class + `data-ctx` on the frame. Nothing else. Keep ≥30° from every existing hue.

### 2b. Frame shape language

No uniform rounded rectangles — that is itself an AI-default tell. Frames are **chamfered plates**: hard 90° corners with one or two cut on the diagonal, cut size and position varying by type (`.sh-wide`, `.sh-role`, `.sh-proj`, `.sh-hub`, `.sh-notch`). Widths and aspect ratios vary too — Home is a 1000px cinematic split panel, dossiers are 680–760px two-column plates.

**Silhouette encodes kind** (see 2i): roles cut the top-left corner, projects cut the bottom-right. You can tell what a frame is from across the room, before reading a word.

**Implementation note:** `clip-path` may not touch the frame itself (see 2e). The chamfer is drawn by two `pointer-events:none` pseudo-elements — `::before` is the 1px edge, `::after` is the surface — while `.frame` stays a plain unclipped rectangle.

### 2c. Cursor and hover affordance

**There is no custom cursor, deliberately.** A `cursor:none` reticle disappears whenever the pointer enters the page without a `mousemove` — windowed Chrome, tab switch, page load, returning from browser chrome — and at that moment there is *no cursor at all*. That is a bad failure on a page a recruiter opens once. The native pointer is guaranteed in every window mode, and `cursor:pointer` is already the universal "this is clickable" signal.

The affordance lives in **hover states** instead, and every clickable thing has one:

| element | hover |
|---|---|
| `.item` (list rows) | accent tint fills, 3px accent bar scales in on the left, name takes the accent, arrow slides right |
| `.frame.focusable` | edge turns accent, inner glow blooms across the surface |
| `.xlink` (role ⇄ project) | fills solid accent, text inverts, lifts 2px, outer glow |
| `.art` / `.cert` / `.ph` | accent border + tint + glow, lift |
| `.jump` / `.hbtn` / `.backbtn` | accent tint + ring + glow, lift |
| `.d-item` (drawer) | accent tint + accent border, slides left |

All of these animate `background`, `box-shadow`, `color`, and `transform` only — compositor-safe, no layout, no `filter`, no `clip-path`.

### 2d. Safe-area fitting and the readability floor

The HUD owns the top-left, top-right and bottom-right corners. Frames are fitted **and** centred against an inset safe box, not the raw viewport:

```js
safe() → {t:94, b:44, l:40, r:40}   // 70/36/14/14 under 900px wide
const MIN_S=.94, MAX_S=1.5, PAD=1.04;
```

Insets are deliberately **tight**, and asymmetric: the top carries the brand, Back button and breadcrumbs, while the bottom centre is now empty (the back pill moved to the top-left), so `b` is small. The minimap sits in a corner a centred frame never reaches and must not inflate `b`.

**`MIN_S` is the readability floor.** Fit-to-viewport alone let dossiers render at ~0.83, which pushed 13px body text to 10.8px and read as "zoomed out too far". A frame now never renders below 94%, even if that means it slightly exceeds the safe box (it still stays fully on screen — the safe margin absorbs it). The structural half of that fix is **two-column dossiers** (`.deep.two`): header spans, then story + photos in the main column, metrics + quote + cross-links in a narrow side column. That roughly halves dossier height, which is what actually lets them fit at 100%.

`fitScale()` also compensates for each frame's rotation (a 6°-rotated plate has a larger on-screen bounding box than its layout box). And frame centres are **measured from layout at runtime** via `centerOf()` rather than hand-typed `data-cx/cy`, so a frame can't drift off-centre when its content length changes.

### 2e. Never put `filter` or `clip-path` on a clickable element inside the world

**This is the single most load-bearing constraint in the file.** Both properties force an element to flatten out of its `preserve-3d` context, and Chrome then computes that element's hit region from the wrong coordinate space. Clicks silently stop registering — no error, no console warning, the element just becomes inert. Draft 5 used `border-radius` on frames and clicking worked; draft 6 switched to `clip-path` for the chamfer and **every dossier, list item, and in-world button became unclickable.** Only the Index drawer still worked, because it sits outside the 3D world.

Rules now enforced:

- **`.frame` itself is a plain, unclipped, unfiltered rectangle.** The chamfer is drawn entirely by `::before` (1px edge) and `::after` (surface), both `pointer-events:none`. The hit region is the frame's own box.
- **`filter` appears only on `.fog` and `.passed`** (distant or behind the camera). Depth is a gradient on `::after`, not a drop-shadow; the arrive pulse animates an inset `box-shadow`, not a filter glow.
- **In-world buttons** (`.jump`, `.art`, `.xlink`, `.ph`) use asymmetric `border-radius`, not `clip-path`. `border-radius` clips rendering without flattening, so it is safe. Fixed-position chrome outside the world (`.hbtn`, `.backpill`, `.minimap`, `.d-close`) may keep `clip-path` freely.

Anything outside the world can use whatever it likes. The test for "is this in the world" is simply: is it a descendant of `#world`.

### 2f. Navigation: delegated, with a geometric fallback

Because the failure mode above is silent and easy to reintroduce, navigation no longer trusts the browser's hit-test alone. Every clickable thing in the world carries `data-fly="frame-id"` instead of an inline `onclick`. One delegated listener on `document` resolves it in two stages:

1. `e.target.closest('#world [data-fly]')` — the normal path.
2. If that returns nothing, `geoHit(x, y)` resolves the click from screen geometry instead. `getBoundingClientRect()` reports correct screen-space boxes even for 3D-transformed elements, so this path works regardless of what the compositor does. Smallest containing rect wins, which naturally picks a list item over the frame containing it.

The reticle cursor uses the same two-stage resolution, so it never labels something inert that is actually navigable. Links (`a[href]`) and the fixed chrome are excluded from the handler so mailto, LinkedIn, and the resume still behave normally.

### 2g. Starfield

Real stars are not uniform white dots, which is exactly what the first version drew. Three things make the difference:

- **Colour temperature.** Six stellar classes with approximate sRGB values, weighted toward what actually dominates a real sky — mostly white and yellow-white (F/G), fewer blue (O/B) and red (M). `CLASSES` in the script.
- **Magnitude distribution.** `mag = Math.random() ** 2` — a power law, so there are vastly more faint stars than bright ones. Only stars above `mag > 0.92` (~6% of what's on screen) get the sprite variant with faint diffraction spikes. Spikes on everything reads as a video game.
- **Soft falloff.** Each star is a pre-rendered radial-gradient sprite (white-hot core → class colour → transparent), not a hard-edged `arc()`. Plus a slow, low-amplitude scintillation so the field feels alive without twinkling like a cartoon.

Warp now draws **actual streaks** — a stroked line from each star's previous projected position to its current one, width scaled to the star's size — instead of the old full-canvas alpha smear. A static `#nebula` div behind the canvas adds faint deep-field colour at zero per-frame cost (pure CSS, painted once).

In light theme the whole sprite ramp inverts so stars read as ink on paper; `buildSprites()` re-runs on theme toggle.

Tuning: 900 stars, ~233 visible at a time on a 1512×982 screen, median alpha 0.24.

### 2h. Performance

The rewrite above was also the fast version. Things that were fixed, worth not reintroducing:

| Problem | Cost | Fix |
|---|---|---|
| `getComputedStyle(root)` inside the starfield rAF loop | forced style recalculation **60×/sec** | theme colours cached, re-read only on toggle |
| `geoHit()` ran `querySelectorAll` + 42 × `getBoundingClientRect()` **on every mousemove** | forced layout per pointer event | target list cached once; rects re-measured once per flight, not per event |
| `ctx.arc()` path per star per frame | 460 path builds/frame | pre-rendered sprites blitted with `drawImage` |
| `will-change:transform` permanently on `#world` | compositor texture for an 8000×6400 element, all session | promoted on flight, released 1.4s later |
| `transition: all` on `.mm-cam` | engine watches every animatable property | enumerated to 5 properties |
| `.item:hover { padding-left }` | layout pass on every hover, inside the 3D subtree | `transform: translateX` (compositor only) |
| 5 × always-on `backdrop-filter` | backdrop readback per frame each | reduced to 2 (drawer, minimap) |
| Bricolage Grotesque 300/500/800 loaded | 2 unused variable-font weights | 800 only — it's the sole weight used |
| tilt loop wrote `transform` every frame forever | DOM write 60×/sec with the mouse stationary | idles once easing settles |
| `warp()` stacked `setTimeout`s | timers piling up on rapid navigation | single cleared timer |

Canvas renders at `min(devicePixelRatio, 1.75)` — sharp on retina without paying 4× the fill cost.

### 2i. Roles vs projects must never look alike

The complaint was exact: "when I press on the project it looks almost the same page." Four independent signals now separate them, so any one of them is enough:

| | Role dossier | Project dossier |
|---|---|---|
| silhouette | `.sh-role` — cut **top-left** | `.sh-proj` — cut **bottom-right** |
| banner | outlined, muted, reads `ROLE` | **solid accent block**, reads `PROJECT` |
| title face | Bricolage Grotesque 800 (display) | Red Hat Mono 700 (technical) |
| surface | plain gradient | faint **technical grid** overlay |

Plus the breadcrumb appends the kind (`Map ∕ Roles ∕ Workiva · Role`) and the drawer groups them under separate headers.

The **cross-link was invisible** before — a small chip in a row of artifact buttons. It is now `.xlink`: a full-width accent-filled block, placed **first** in the side column under a "What I built here" / "Built during" header, with the loudest hover in the file. Roles point to their projects; projects point back to their role.

### 2j. Back = history, not hierarchy

`goBack()` pops a **history stack**, it does not walk `data-parent`. Every `flyTo`/`overview` pushes the place you were leaving; back replays without re-pushing. This is the case that was broken:

```
Roles → Workiva → (its project) SE Highspot V2 → Back
  gives Workiva, not the Projects hub
```

`data-parent` still exists — it builds the breadcrumb trail, which *is* hierarchical. The two are different questions and now have different answers. The button is disabled and dimmed when the stack is empty, and its label always names the destination ("Back to Workiva"). It sits **top-left next to the breadcrumb**, where browsers and operating systems put back; it was previously bottom-centre, which is nowhere.

### 2k. Minimap

The selection outline and the tile it marks are both derived from a single `mmBox(frame)` function, so they cannot disagree — the previous version drew the camera *viewport* rectangle, which had no relationship to the tile size. Focused: the outline snaps to that frame's box + 2px. Overview: it outlines the whole map. Dots are tinted with each frame's context hue, and a caption above names what is selected.

## 3. Content structure (site map)

```
Overview (L0 — whole map visible)
└── Home (L1) — hook line, quick facts, 4 doors
    ├── The Career (L2) — 6 roles + certifications strip
    │   ├── IBM — Account Executive Intern (L3)
    │   ├── Workiva — SE Enablement Intern (L3)
    │   ├── Cisco (Splunk) — Solutions Engineer Intern (L3)
    │   ├── U.S. Renal Care — AI Solutions Consultant (L3)
    │   ├── eBay — Founder & Business Owner (L3)
    │   └── UT Dallas — BS CIS&T (L3)
    ├── Projects (L2) — 5 projects
    │   ├── KDR Generator (L3)          ←→ IBM
    │   ├── SE Highspot V2 (L3)         ←→ Workiva
    │   ├── FlightFlow (L3)             ←→ Cisco
    │   ├── U.S. Food Access Risk (L3)  ←→ Cisco
    │   └── Valor — watsonx × BofA (L3)
    ├── The Person (L2)
    │   └── [Hobby dossiers] (L3)
    └── Contact (L2) — email, LinkedIn, resume
```

17 frames total. **Rule: never more than 2 clicks from Home to any piece of information.**

**Cross-linking (`←→` above) is bidirectional and prominent.** Every role dossier carries a "Built here" block listing its projects as a full-width accented `.xlink` button; every project dossier carries a "Built during" block pointing back to its role. Both sit at L3, so the jump costs one click. This is why KDR Generator is reachable from inside the IBM dossier without leaving the Career branch — the Projects branch is an index, not the only door.

## 4. Current state

Real content is in for every professional frame — titles, dates, locations, narrative case studies, metrics, and pull-quotes from LinkedIn recommendations. Still placeholder:

- **All photos** (`.ph` slots say "add photo") — need real `<img>` handling with lazy-load and aspect ratios
- **All artifact buttons** in `[brackets]` — decks, demo videos, write-ups
- **The Person branch** — hobby names, story, photos. Pickleball and the community orgs are seeded from LinkedIn but unwritten
- **U.S. Renal Care metrics** — the only dossier without hard numbers; marked with a visible `.tbd` note in the UI

**Two data conflicts to resolve:** LinkedIn says Workiva ran Jan–Jun 2026 with a return as "SE Enablement II" from Aug 2026; the resume says Jan–May 2026. LinkedIn also shows IBM (May 2026–present) overlapping Workiva. The site currently reads "Jan – Jun 2026 · returned Aug 2026" — confirm and correct.

**Contact frame is fully live:** `mailto:` email, `tel:+14695161440` phone, LinkedIn (new tab), and a résumé row with a `download` attribute serving `resume.pdf`. Saaif chose to publish the phone number after being told the scraping tradeoff — that was his call, and it is reversible by deleting one `.cbtn`.

## 5. Next steps

1. **Photos and artifacts** — replace `.ph` slots with real images, wire the `[deck]` / `[demo video]` buttons to files or embeds.
2. **The Person branch** — the one frame recruiters remember. Needs Saaif's voice, not a summary.
3. **Build-out** — mobile QA (3D transforms on iOS Safari are the known risk area), accessibility pass (focus states, semantic headings), possibly split CSS/JS if the file gets heavy.
4. **Deploy** — GitHub Pages or Cloudflare Pages (free). Buy saaifkhan.com (~$10–12/yr, Cloudflare Registrar or Porkbun). Point DNS, enable HTTPS.

## 6. Folder contents

```
Portfolio/
├── index.html       ← THE working file (draft 6). At the ROOT because
│                       GitHub Pages serves index.html from the repo root.
├── resume.pdf       ← served by the Contact frame's download button
├── README.md        ← this file
├── .gitignore
└── archive/         ← rejected iterations. Git is the version history now;
                        this folder is redundant and safe to delete.
    ├── SITEMAP.md               (early structure doc, superseded)
    ├── mockup-a-cinematic.html  (v1: scroll-based — "looks AI generated")
    ├── mockup-b-os.html         (v1: fake OS w/ draggable windows — too much friction)
    ├── mockup-c-editorial.html  (v1: magazine style)
    ├── mockup-v2-hub.html       (v2: zoom-hub, right structure, wrong look)
    ├── draft-1-canvas-map.html  (v3: Prezi canvas — Saaif loved this)
    ├── draft-2-depth-dive.html  (v3: 3D z-dive — Saaif loved this too)
    ├── draft-3-orbit.html       (v3: orbiting satellites, rejected)
    ├── draft-4-hybrid.html      (v4: shallow 1+2 blend — "only the stars were combined")
    └── draft-5-fusion.html      (v5: right structure, wrong colour + shape)
```

## 7. Design principles agreed with Saaif (don't violate these)

- Information access beats spectacle: content reachable in ≤2 clicks, zero interface learning curve.
- Every navigation affordance has a visible button; keyboard is optional.
- Deep pages are narrative case studies with real artifacts, never bullet lists.
- Dark default, toggle available.
- One polished signature mechanic (the 3D map-dive). No sound effects, no cursor *trails*, no loading screens — deliberately rejected. (The reticle cursor is a targeting HUD, not a trail; approved separately.)
- The "AI-generated look" is killed by specificity: real names, numbers, photos, voice. Placeholder-looking content is the enemy.

## 8. Verification constraint

The Cowork sandbox has no root, so a headless browser cannot be installed, and Chrome blocks `file://`. **Visual changes cannot be verified by the assistant** — only static checks (JS syntax, nav-target integrity, CSS variable resolution, hue separation, frame-overlap geometry, world bounds). Saaif has to open the file to confirm anything visual.
