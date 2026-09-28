# IvoryCare Dental & Implant Centre (ivorycare-dental)

React 19 + Vite 6 + TypeScript + Tailwind CSS v4 + motion + react-router-dom v7.
A demo/fictional clinic website for **IvoryCare Dental & Implant Centre**, Bengaluru.
All clinic data is fictional (see `IvoryCare_Dental_Demo_2.pdf`). Branding says
**IvoryCare** — never revert to the old "PearlSmile Dental Care" branding (this demo
was forked from that project) and never call it "AI/virtual dentistry" — it's a
physical Bengaluru clinic, not a SaaS.

> This is one of several demo sites used to present options to clients. Its whole
> job is to look deliberately different from the other demos (especially the green
> "Pearl & Pine" PearlSmile style) — treat every new build as a chance to pick a
> distinct identity, not to reuse this one.

## Repo & Live Deployment (current state)

- **Source repo:** `Rajveer-Adpaikar/ivorycare-dental` — `origin` already points
  here. This folder was forked from `pearlsmile-dental`; the PearlSmile demo still
  lives at https://rajveer-adpaikar.github.io/pearlsmile-dental/ in its own repo.
- **Live (GitHub Pages):** https://rajveer-adpaikar.github.io/ivorycare-dental/
  Pages serves the `gh-pages` branch root (legacy source, `gh-pages` branch).
- **Redeploy after changes:** `npm run build && npx gh-pages -d dist --dotfiles`,
  then push source to `main`. Both are required — source and live bundle are separate.
  CDN lag is real: right after publishing, poll `curl` for the new hashed asset name
  in the served HTML before concluding a deploy failed.
- `vite.config.ts` hardcodes `base: '/ivorycare-dental/'` and `App.tsx` passes it to
  `<BrowserRouter basename={import.meta.env.BASE_URL}>`. These two must stay in sync.
  If the repo/site name ever changes, change BOTH or you get broken assets or
  "No routes matched". The old base was `/pearlsmile-dental/` — it was changed.

## Stack & Run

- Install: `npm install`
- Dev server: `npm run dev` → **port 3200** (3000 and 3100 are taken by other apps).
  Vite auto-picks the next free port if 3200 is busy. Local URL: `http://localhost:3200/`
  (dev serves under the active base, so `/ivorycare-dental/` after a recent change).
- **Gotcha:** a `npm run dev` left running through a base-path change keeps the OLD
  base until restarted (it once served `/pearlsmile-dental/` after the base changed to
  `/ivorycare-dental/`). If a stale session ever shows a 404 at the logged URL, restart
  the dev server.
- Typecheck / "lint": `npx tsc --noEmit` (no test suite)
- Build: `npm run build` → `dist/`

## Data & Content

- `src/config.ts` — single source of truth: `CLINIC` object (name, tagline, address,
  phone, WhatsApp, email, maps embed + directions, `hours[]`, `dentists[]`,
  `services[]` (5 groups), `stats[]`, `reviews[]`, `faqs[]`). **All data lives
  here** — edit it to change the site's content, not the components.
- Phone numbers must be dummy values. Current: `+91 80 4123 6842` (fictional), WhatsApp
  digits `918041236842`. The `beforeAfter[]` gallery data was **removed** — do not
  reintroduce it unless the client asks for the gallery back.
- Sections (homepage order): `Hero` → `WhyUs` (#why) → `Dentists` (#dentists) →
  `Treatments` (#treatments) → `Reviews` (#reviews) → `Faq` (#faq) →
  `FindUs` (#find-us, hours + map + directions).
- Legal pages at `/privacy-policy`, `/terms-of-service`, `/hipaa` — all source their
  branding from `CLINIC`.

## Booking & Enquiry

- `src/booking.tsx` — `BookingProvider` wraps the app in `App.tsx`; components call
  `useBooking()`. Exposes `openBooking(preset?)` (opens the appointment form, optional
  `{ dentist, service }` presets) and `openEnquiry()` (opens the cost-enquiry form).
  `useBooking()` returns an **object** — destructure it (`const { openBooking } =
  useBooking()`), don't call it as a bare function.
- `src/components/BookingModal.tsx` — **custom appointment form** (replaces the old
  Cal.com embed): dentist + treatment + date/time chips + name + phone, submitting to
  WhatsApp via `waLink()`. No Cal.com dependency anymore.
- `src/components/EnquiryModal.tsx` — treatment-cost enquiry that hands off to WhatsApp.
- `src/lib.ts` — `waLink(whatsapp, text)` WhatsApp deep-link helper.
- WhatsApp is wired across major calls-to-action (Hero, Dentists, FindUs, booking +
  enquiry handoff, floating button `WhatsAppFab.tsx`).

## Routing & 404

- `src/App.tsx` has a `<Route path="*" element={<NotFound />} />` catch-all →
  `src/components/NotFound.tsx` renders an on-brand "This page isn't on file." page
  (Back to home + Book an appointment + call clinic).
- GitHub Pages has no SPA fallback, so a hard refresh or direct link on a subpath used
  to 404. It now works via a redirect dance:
  1. `public/404.html` stores the requested path in `sessionStorage.redirect` and
     `location.replace`s to the home base. Pages serves this file for any unknown
     path (with HTTP 404 status — expected; the body is what matters).
  2. `index.html` has an inline restore script that reads `sessionStorage.redirect`,
     deletes it, and `history.replaceState`s the original URL before React mounts, so
     the router lands on the right page (legal pages) or the 404 page (nonsense URLs).
  - Keep the two halves in sync (404.html sets the key, index.html consumes it).
  - Do NOT hardcode `https://rajveer-adpaikar.github.io/ivorycare-dental/` inside
    404.html — it uses `location.replace('/ivorycare-dental/')` (base-relative).

## Design System ("Consultation Ledger")

- Palette (Tailwind v4 `@theme` tokens in `src/index.css`):
  - `rosewood` (deep claret-rose, primary brand; `rosewood-950` #230d18 → `rosewood-50`)
  - `ivory` (#fffdf8 near-white background — NOT cream/sand)
  - `peach` (action color — call, book, WhatsApp, accents; `peach-500` #e88a42)
- Type: `Newsreader` (`font-display`, editorial serif) + `Manrope` (`font-sans`, body)
  + `Fragment Mono` (`font-data`, clinical data like hours/stats/ledger labels).
  Imported in `src/index.css` via Google Fonts.
- Signature motif: the **tooth mark** (`src/components/Tooth.tsx` — exports `Tooth`,
  `ToothyRow`, `ToothRow`) as logo monogram and divider; the **consultation ledger
  card** in the hero. WhatsApp icon lives in `src/components/icons.tsx`.
- Design rules: no green/pine, no teal/slate, no gradient text, no cream/sand/beige bg,
  no card-grid-of-icons uniformity (services and why-us are editorial ledger rows),
  gold/green accents are banned — peach marks every action.

## Gotchas

- Section anchors: `#why`, `#dentists`, `#treatments`, `#reviews`, `#faq`, `#find-us`.
  There is NO `#gallery` (removed) or `#clinic` — link to the real sections only.
- Never use root-absolute hrefs (`/#services`) — use page-relative (`#services`) so
  anchors keep working under the Vite base on Pages.
- Touch targets: footer links use `py-2`+ padding to stay ≥40px tall — preserve when editing.
- TypeScript strictness: `motion` reveal props fire per-section — content is always
  visible by default (never gated on JS), wraps animate transforms only.
- Mobile QA method that works here: Playwright `browser_resize` + `browser_evaluate`
  measuring `getBoundingClientRect()` against `window.innerWidth` (skip elements under
  `pointer-events-none`). DOM measurement via `browser_evaluate` is the reliable check;
  screenshots saved to `.playwright-mcp/` are often unreadable in this environment.
- `.playwright-mcp/` is gitignored; the repo also ignores `.impeccable/`, `dist/`,
  `node_modules/`, `.env*`.

## State when this was last written

Deployed and live at https://rajveer-adpaikar.github.io/ivorycare-dental/ with the
404/SPA-fallback working. Local dev server is running on port 3200 (restart it if you
picked up this file after a session gap). All source is committed on `main` and pushed;
`gh-pages` carries the published bundle.