# Site audit — draft 8

Full pass through `index.html` (structure, CSS, JS, content), plus the repo's supporting files. Organized by priority. Nothing here has been implemented — this is the punch list to work through.

Baseline: the 3D spatial engine, the IBM Carbon-accurate design system, the guided tour, and the recruiter-mode timeline are all genuinely distinctive — most portfolios are a static scroll page, this is a real interaction model with a coherent visual language behind it. The gaps below are mostly things that don't show up until someone other than you uses the site: search engines, screen readers, LinkedIn's link preview, a recruiter's phone.

---

## Critical — fix before sharing this link widely

**No social preview (Open Graph / Twitter card) tags.** When this URL is pasted into LinkedIn, Slack, or a text message — which is exactly how a recruiter will receive it — it renders as a bare blue link with no image, no title, no description. This is the single highest-leverage fix on this list: a recruiter deciding whether to click is doing it from the preview card, not the URL. Needs `og:title`, `og:description`, `og:image` (a static screenshot of the home tile), `og:url`, and `twitter:card`.

**No deep linking.** Every section lives at the same URL (`/`). This means: refreshing the page always resets to Home and loses your place; the browser's back/forward buttons don't map to site navigation the way a user expects; and you can't send someone a link straight to "here's my IBM role" or "here's FlightFlow" — you can only say "go to my site and click around." For a portfolio whose entire pitch is specific project deep-dives, that's the most common sharing use case being unsupported. Fix is to sync `flyTo()` with `history.pushState` / `location.hash`, restore position on load.

**No print stylesheet.** Ctrl+P / Cmd+P is a real path recruiters use to save a resume for later or hand to someone else. Right now that would print the raw 3D canvas mid-transform — unusable. At minimum, a `@media print` rule that hides the world and shows a clean text summary (or redirects intent toward the resume PDF) prevents an embarrassing result.

**No `robots.txt` or `sitemap.xml`.** Without these, search engines have no explicit signal about what to index or where the canonical resume/site boundary is. Low effort, standard practice, currently absent.

## High priority — affects everyone who visits, not just edge cases

**No analytics.** There's currently no way to know if anyone's visited, which sections they open, whether they finish the guided tour, or whether they ever click "Hiring?". You are optimizing this site with zero feedback loop. A privacy-respecting option (Plausible, Fathom) or GA4 would tell you what's actually landing.

**One oversized image.** `splunk-office-boulder-saaif.png` is 1.3MB — a PNG being used for a photo, which should be a compressed JPEG or WebP. The other project screenshots are already reasonably sized (44–276KB), so this is an outlier, not a systemic issue, but on a mobile connection that one image alone is a meaningful chunk of load time. Converting the whole `/assets/photos` set to WebP with JPEG fallback would also shave 20-40% off the already-fine ones.

**Screen reader / keyboard-only experience is effectively unusable.** The site is a spatial 3D canvas navigated by camera flights — a screen reader has no meaningful way to convey "you are now looking at a tile in 3D space 750px back." Buttons are real `<button>` elements so raw tab-through technically works, but there's no `aria-live` announcement when the camera moves, so a screen reader user gets silence after every click. This is a real tradeoff of the format, not a small bug — worth deciding deliberately rather than by omission. A pragmatic fix that doesn't touch the 3D experience: a hidden, keyboard-reachable "text version" link at the very top of the DOM that serves the same content as a flat, linear page. That also happens to double as an SEO win, since search crawlers face the same problem the screen reader does.

**Theme choice doesn't persist.** `toggleTheme()` flips `data-theme` but nothing is saved to `localStorage` (which is already used elsewhere, for tour-seen state) — so every reload silently reverts to dark, even for a visitor who explicitly picked light. Two-line fix, currently inconsistent with the rest of the site's state-persistence pattern.

## Medium priority — polish, not urgent

**Structured data.** A `Person` JSON-LD block (name, jobTitle, alumniOf, sameAs → LinkedIn) helps Google understand this is a person's professional page and can surface richer results, e.g. a knowledge panel fragment or sitelinks.

**Resume PDF has no standalone SEO presence.** It's fetched via blob download, which is the right UX call for forcing a save-to-disk — but it also means the PDF's content is invisible to search engines and can't be deep-linked to directly (e.g., `/resume.pdf` opened in a new tab as a fallback). Worth confirming `resume.pdf` is still reachable as a direct URL for anyone who wants to open rather than download it — right now the primary path forces a download with no "view first" option, which is a bit presumptuous of the visitor's intent.

**Recommendations section only shows what's already been solicited.** Not a code fix, a content one: three recommendations across manager/mentor/peer categories is a fine number, but if you can get even two or three more before this ships to employers broadly, the pagination component you already built will just absorb them for free.

**Mobile touch behavior wasn't stress-tested.** The camera engine responds to click coordinates, which works for tap, but there's no evidence of specific touch-gesture handling (swipe-to-navigate, pinch). Worth an actual pass on a phone to confirm nothing feels like it's fighting the browser's native scroll/zoom gestures — 3D `perspective` transforms are exactly the kind of thing that behaves differently across mobile Safari vs. Chrome.

**No favicon `theme-color` meta tag.** A one-line addition (`<meta name="theme-color" content="#161616">`) themes the mobile browser chrome (Safari's address bar, Android's status bar) to match the site instead of showing the OS default — small detail, consistent with how much care went into the favicon itself.

## Feature opportunities — where this could genuinely stand out

These aren't fixes, they're additions worth considering because they play specifically to what this site already is (a spatial, interactive story) rather than bolting on generic portfolio features.

**A lightweight "compare tracks" mode for recruiters vs. engineers.** Recruiter Mode already exists as a narrative timeline — the natural complement is a technical mode that surfaces the stack, architecture decisions, and code-level detail for a hiring manager or engineer who wants depth instead of narrative. You already have the content bifurcated in spirit (dossiers have both a story side and a metrics/build side); making that an explicit second lens would let two very different audiences each get the version they actually want, instead of both getting the same blend.

**Live status line.** Something small and honest — "currently: 46 days into the IBM internship" or "next: Workiva, Aug 2026" — computed from today's date against the known role dates already in the data. Signals the site is maintained and current without you touching it again until the underlying facts change.

**A visible "last updated" or version marker.** Given how much this has iterated (draft 8, per the file's own header comment), a small, quiet timestamp signals active maintenance to anyone technical enough to notice — which, given the audience, is most of them.

**Case-study depth on one flagship project.** Right now every project gets roughly equal treatment. Picking the single strongest one (KDR Generator or SE Highspot V2, both have real before/after and measurable impact) and giving it a genuinely deeper page — problem framing, alternatives considered, what you'd do differently — does more for a technical evaluator than five projects at equal depth. Depth on one signals more than breadth across all.

---

## What NOT to change

Flagging this explicitly since the brief was "everything" — a few things that look like they could be "simplified" but shouldn't be:

- The 3D camera engine itself. It's the site's entire differentiator; simplifying it toward a conventional scroll page would trade away the thing that makes this memorable.
- The IBM Carbon-based color/type system. It's disciplined and consistent across every screen already — a rare thing in a solo-built site.
- The guided tour and recruiter mode. Both are already doing exactly what the top-bar fix did: solving a real navigation problem (a first-time visitor not knowing where to start) with an industry-standard pattern (product tours, executive-summary mode) adapted to this site's specific shape.
