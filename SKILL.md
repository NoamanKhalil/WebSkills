---
name: launch-readiness-checklist
description: Use before shipping a website or web app to production, or to re-audit one that is already live. Works out the site's stack and what it actually does (accounts, payments, email, user-submitted content, analytics), asks the user for the business facts it needs (domain, legal entity, analytics IDs, policies), then audits and fixes only the items that apply — legal pages and claims accuracy, consent, SEO tags/canonical/robots/sitemap/Open Graph/structured data, favicon and manifest, custom 404, accessibility and mobile, form states, analytics, security headers, exposed files, abuse protection, email authentication, payment terms, and backups/monitoring. Each item is explained in plain language so a developer or non-technical founder learns what it is and why it matters, not just whether it's done. Trigger on requests like "is this ready to launch", "pre-launch audit", "SEO checklist", or "go-live checklist".
---

# Launch Readiness Checklist

Most launches don't fail on the feature — they fail on the boring stuff around
it: a contact form that dies silently, a sitemap nobody generated, a cookie
banner that isn't actually legal, a `tests/` folder shipped straight to
production. None of it shows up in a demo. All of it shows up the week after
launch, as a support ticket, a bounced compliance review, or a domain
blocklisted for something a stranger uploaded through your own site.

This skill is the difference between "looks done" and "is done." Point it at
any project — static site, SPA, full-stack app, something already live — and
it works out what the site actually does, asks you the handful of business
facts only you know, then runs a real audit: not a generic checklist ticked
from memory, but grep results, `curl` output, and DNS lookups against *your*
code and *your* domain. It skips what doesn't apply instead of padding the
report, refuses to invent a business address or a tracking ID just to fill a
blank, and explains *why* each item matters in plain language — so whoever
reads the report walks away understanding the site better, not just holding a
longer to-do list.

An audit-and-fix skill for public-facing websites and web apps, meant to be worked
**together** by a developer and an LLM: the LLM handles the mechanical,
repeatable parts (grepping for missing tags, probing the live site, generating
boilerplate, wiring consistent markup across pages) and explains what it finds;
the human owns judgment calls the LLM shouldn't make unsupervised (legal text,
real business facts, brand/design decisions, final sign-off).

This skill is deliberately project-agnostic. It assumes nothing about the stack,
host, domain, or business — it **detects what it can and asks for the rest**.

Run it in four passes: **Kickoff (orient + ask) → Audit → Fix → Verify**, and
make every report teach, not just report — see [Report format](#7-report-format).

## 0. Orient before touching anything

Detect first; only ask about what the code can't tell you.

1. **Pages and routes.** List every publicly reachable page. Static sites:
   `find . -iname "*.html"` (ignore build output and vendor folders).
   Frameworks: page/route files, layouts, any sitemap generator. Note
   server-side endpoints too (form handlers, APIs, redirect/lookup routes).
2. **Stack and hosting.** Plain static files, a framework (Next.js, Vite, Astro,
   WordPress…), or server-side code (PHP, Node, Python, Ruby…)? Where does
   server config live — `.htaccess`, `nginx.conf`, `_headers`/`netlify.toml`,
   `vercel.json`, a CDN or hosting dashboard? That decides where redirect,
   404-status, header, and HTTPS fixes go. If you can't tell, ask (Kickoff Q2).
3. **Capability profile.** Skim the code and copy to work out which of these
   the site actually does — each one switches on extra items in section 3:
   - collects personal data or has accounts/logins
   - takes payments or subscriptions
   - sends email (transactional and/or marketing)
   - accepts user-submitted content or URLs (short links, uploads, profiles,
     comments)
   - runs analytics, tag managers, or ads
   - has forms or other public write endpoints

   Confirm your read of this with the user rather than assuming.
4. **Canonical host form.** Work out what the server actually serves and
   redirects to (`www` vs apex, `https`, trailing slash, `.html` vs clean URLs)
   from the server config and, if live, `curl -sIL`. Every URL you write into
   canonicals, sitemap, robots.txt, and `og:url` must use that exact form.
5. **Live or pre-launch?** If the site is already live, treat this as a
   regression audit and verify against it with read-only `curl` requests to the
   user's **own** domain (never anyone else's). Consider offering to turn the
   mechanical checks into a small script or CI step so they keep passing.
6. **Check rendered output, not just source.** Tags can be injected by
   JavaScript or templates. Grep the source to find them, but confirm against
   the rendered page (`curl` the URL, or a browser) before declaring a tag
   missing or present. The grep examples below assume static HTML — for
   framework sites, search the layout/template/component files instead.

## 1. Kickoff questionnaire — ask before fixing anything

A handful of facts gate a dozen of these items at once. Ask for everything you
couldn't detect **in one batch** (use an interactive question tool if you have
one, otherwise a single numbered message) instead of interrupting the fix pass
repeatedly. Skip questions the code already answered — but say what you
inferred so the user can correct it.

1. **Production domain and preferred form** (e.g. `example.com` vs
   `www.example.com`) — gates sitemap.xml, robots.txt, canonical tags, `og:url`.
2. **Hosting, and where server config lives** — gates HTTPS, 404 status, and
   security headers (T1.7, T2.8, T5.1).
3. **Legal business name, mailing address, jurisdiction, and the contact emails
   to publish** (privacy, legal, support, abuse) — gates privacy policy, terms,
   footer, and email footers. Jurisdiction and audience regions (EU/UK → GDPR
   and ePrivacy, California → CCPA, both if unsure) decide which privacy rules
   apply.
4. **Personal data and accounts** — what is collected (forms, sign-up,
   analytics, server logs with IPs, uploads) and how long it's kept. Decides
   whether the consent banner, privacy policy, and data-rights items are
   launch-blocking.
5. **Analytics / tag manager** — platform + IDs, if accounts exist. If not, ask
   whether to skip analytics entirely for now rather than inventing IDs.
6. **Payments** — does the site charge money? Which processor, which plan types
   (subscription / one-time / lifetime), and what are the refund, cancellation,
   and tax policies? (Gates T1.9.)
7. **Email** — does the site send email? Transactional only, or also
   marketing/reminders? From which domain and provider? (Gates T1.10, T5.7.)
8. **User-submitted content or links** — can visitors create anything public,
   shareable, or redirecting through your domain? Is there a way to report
   abuse, and does someone monitor it? (Gates T5.6.)
9. **What's live vs. planned** — which features and claims in the copy are real
   today, and which are roadmap? (Gates T1.8. An audit can't tell "not built
   yet" from "built somewhere I haven't looked".)
10. **Brand assets** — logo, Open Graph image, favicon source — or should
    placeholders be generated from the existing colors and type?
11. **Launch date, or already live?** — helps prioritize the tiers below and
    decides whether live probing applies.
12. **Backups, monitoring, and rollback** — does anything back up the data,
    watch uptime and errors, and allow a bad deploy to be undone? (Gates T5.8.
    The LLM can't see the host, so this is the user's answer.)

Record the answers, then proceed. If the user wants to move fast and defers an
answer, mark the gated items `⚠️ blocked on input` in the report rather than
guessing.

## 2. Do NOT fabricate real-world facts

Never invent plausible-looking placeholders and leave them looking final —
either use the kickoff answers, or insert an unmissable
`TODO(real value needed)` marker:

- **Contact address, emails, business name, jurisdiction** — do not invent.
- **Analytics / tag-manager / payment IDs and keys** — must come from the
  user's actual accounts.
- **Production domain** — used in sitemap.xml, robots.txt, canonical URLs,
  `og:url`. A wrong domain baked in is worse than none.
- **Policies** — refund and cancellation terms, data-retention periods, the
  list of third-party processors. Draft only from what the user and the code
  confirm.
- **Legal page text as final** — generated privacy/terms copy is a *draft*.
  Say so explicitly; recommend human/legal review before it's treated as
  binding.
- **Claims about the product** — never write (or leave in) a feature, privacy,
  or security claim you can't trace to code or to the user. When copy and code
  disagree, report it; don't silently pick a side.

Everything else (markup, files, alt text, CSS, boilerplate scripts) can be
implemented directly without asking.

## 3. Which items apply

Not every item fits every site. Use the capability profile from section 0, and
mark items that don't apply `➖ N/A` **with a one-line reason** — don't force a
fix, and don't skip silently.

- **Every public site:** everything not marked *(conditional)*.
- **No forms or public write endpoints:** T1.5, T1.6, and T5.2 are N/A.
- **No conversion goal** (a pure tool, docs, or portfolio site): T3.1 and T3.2
  are N/A; instead check the primary action (e.g. the tool itself) is visible
  without scrolling.
- **Collects personal data / has accounts:** T1.11 applies.
- **Takes payments:** T1.9 applies.
- **Sends email:** T1.10 and T5.7 apply.
- **Accepts user-submitted content or URLs:** T5.6 applies.

## 4. The checklist, by priority tier

Each item lists: **What it is** (plain language) · **Why it matters** ·
**How to check** · **How to fix**. When you report status on an item, carry
the "what/why" through into the report — that's what makes the audit useful
to someone who's never heard of Open Graph tags, not just a pass/fail signal.

### Tier 1 — Legal & functional (launch-blocking)

Skipping these can mean broken checkout/signup flows, or real legal exposure
(FTC/CAN-SPAM require a real postal address in commercial email; GDPR/CCPA
require disclosure before tracking; consumer-protection law bars misleading
claims).

**T1.1 Privacy policy page**
- *What it is:* a page disclosing what user data you collect and how it's used.
- *Why it matters:* legally required almost anywhere you collect emails, use
  analytics, or run ads (GDPR in the EU, CCPA in California, and expected
  practice everywhere else). Its absence is also a trust signal visitors and
  app-store/ad-platform/payment-processor reviewers check. A policy that
  describes the wrong product, or omits tools you actually use, is nearly as
  bad as none.
- *Check:* the page exists and is linked from the footer. Then compare it to
  reality: does it name every analytics, tag-manager, payment, email, hosting,
  and embed provider the site actually loads or calls? Does it cover
  everything collected (forms, accounts, server logs/IPs, click or scan
  tracking, uploads), retention, user rights (access/delete/opt-out), and
  cookies?
- *Fix:* create or update it with standard sections (data collected, cookies,
  third-party services named, retention, user rights, contact, "last updated"
  date). Flag for human/legal review — this is drafted text, not legal advice.

**T1.2 Terms and conditions**
- *What it is:* the rules a user agrees to by using the site/product (liability
  limits, acceptable use, account termination).
- *Why it matters:* your main defense if a user misuses the product or disputes
  a charge; also expected by payment processors for anything transactional.
- *Check:* the page exists, is footer-linked and dated, and describes the
  *current* product (leftover text from a previous product or a template is
  common). If users can submit content or links, it has an acceptable-use
  clause. If the site takes payments, it covers refunds/cancellation (T1.9).
- *Fix:* same pattern as T1.1 — draft from confirmed facts, dated, flagged for
  review.

**T1.3 Cookie / tracking consent banner**
- *What it is:* a notice asking consent before non-essential cookies (analytics,
  ads, tag managers) run, with at least Accept/Reject — not just an "OK"
  dismiss.
- *Why it matters:* GDPR (EU) and similar laws require *opt-in* consent before
  tracking loads, not just disclosure after the fact. A banner with only
  "Got it!" and no real reject option is non-compliant in the EU.
- *Check:* is analytics/tag-manager code present with no consent gate in front
  of it? Is consent to marketing email bundled into "I accept the terms"?
- *Fix:* show a banner on first visit; don't fire analytics/tags until consent
  is granted; store the choice (e.g. `localStorage`); link to the privacy
  policy. For Google tags serving EU/UK visitors, wire Consent Mode so the tags
  honor the choice. Keep marketing-email consent as its own unticked option.

**T1.4 Real business identity & contact**
- *What it is:* an actual mailing address, legal entity name, and working
  contact emails.
- *Why it matters:* required by CAN-SPAM for commercial email and expected on
  privacy/terms pages; a missing address or dead mailbox reads as untrustworthy
  and can fail ad-platform/payment-processor review.
- *Check/Fix:* insert into the footer and legal pages, from the kickoff answer
  only — never fabricate. Confirm the published mailboxes actually exist.

**T1.5 Form error states** *(N/A if no forms)*
- *What it is:* visible, inline feedback when a form field is missing, invalid,
  or a submission fails — not silence or a raw browser default.
- *Why it matters:* a form that fails silently loses leads/signups with no
  trace; users assume it's broken and leave.
- *Check:* find `<form>`/`<input>` elements and async submit handlers; confirm
  on-blur/on-submit validation messaging exists for required/invalid fields.
- *Fix:* inline message near the field (e.g. "Enter a valid email") plus a
  visible style change (border/color); handle the network-failure case too,
  not just client-side validation.

**T1.6 Thank-you / confirmation state** *(N/A if no forms)*
- *What it is:* explicit confirmation after a successful submission.
- *Why it matters:* without it, users resubmit (duplicate leads/orders) or
  assume it failed.
- *Check:* does a successful submit navigate to a distinct page/state, or just
  clear the form / show a transient `alert()`?
- *Fix:* a dedicated thank-you page or an in-page confirmation state with a
  clear next step (e.g. "check your email"). If it's a separate page, mark it
  `noindex` (T2.10).

**T1.7 HTTPS / SSL active**
- *What it is:* the site is served over `https://`, not `http://`.
- *Why it matters:* browsers flag `http://` sites as "Not Secure," form
  submissions over plain HTTP can be intercepted, and many modern browser APIs
  (geolocation, camera, service workers) and ranking signals require it.
- *Check:* `curl -sIL http://<domain>/` — expect a valid certificate and a
  single permanent (301/308) hop to the canonical `https` URL. Multi-hop chains
  (apex → www → https) or temporary (302) redirects are worth fixing.
- *Fix:* managed hosts usually do this automatically (verify "force HTTPS" isn't
  off); on self-managed servers it's a redirect rule plus certificate renewal.
  This is a hosting config item, not an HTML change.

**T1.8 Copy, metadata, and legal text match reality**
- *What it is:* every public claim — page copy, meta descriptions, Open Graph
  text, structured data, FAQ, `llms.txt`, legal pages — describes what the
  product actually does today and how data actually flows.
- *Why it matters:* consumer-protection rules (e.g. the FTC in the US) treat
  false or unbuilt-feature claims as deceptive; visitors and search/AI systems
  repeat what you publish, and a "coming soon" feature sold as live becomes a
  refund request or a bad review. Sites that changed direction or started from
  a template are especially prone to stale leftovers.
- *Check:* grep public copy, structured data, `llms.txt`, and legal pages for:
  features and capabilities promised; privacy/security/encryption claims
  ("never stored", "no sign-up", "100% private"); pricing claims; old product or
  company names; template filler (lorem ipsum, sample emails/domains used as
  real values — input placeholders are fine); and a stale copyright year. For
  each promised feature, confirm it exists in code (kickoff Q9 says what's
  live vs. planned). Compare privacy claims against what the code actually logs
  and sends.
- *Fix:* propose corrected wording, "coming soon", or removal and let the owner
  choose — the LLM shouldn't decide what the business promises. Update or
  dynamically generate the copyright year.

**T1.9 Payment & billing terms** *(conditional: site takes payments)*
- *What it is:* terms of sale visible **before** purchase — price and currency,
  billing period, auto-renewal, refund and cancellation policy, tax handling —
  plus a way to manage or cancel a subscription and a receipt.
- *Why it matters:* payment processors and card networks expect clear terms;
  unclear renewal or cancellation is a top cause of chargebacks and is
  regulated in many places. Unverified payment webhooks also let anyone fake a
  paid upgrade.
- *Check:* the pricing/checkout page and terms state refund, cancellation, and
  renewal; a self-serve cancel/billing-portal link exists; server-side webhook
  handlers verify the processor's signature and tolerate replays.
- *Fix:* add the wording from the kickoff Q6 answers (never invent a refund
  policy), link the terms at checkout, add the portal link, and add signature
  verification if missing.

**T1.10 Email compliance** *(conditional: sends non-transactional email)*
- *What it is:* marketing, reminder, re-engagement, and newsletter emails carry
  a working unsubscribe and the sender's postal address, and — where
  GDPR/ePrivacy/CASL apply — go only to people who opted in.
- *Why it matters:* CAN-SPAM, GDPR/ePrivacy, and CASL require these, and major
  mailbox providers require one-click unsubscribe from bulk senders. Missing
  them gets mail filtered or fined.
- *Check:* find every code path that sends email and classify it as
  transactional (receipt, verification code, password reset) or not. For the
  non-transactional ones, confirm the unsubscribe link, a `List-Unsubscribe`
  header, the postal address, and that a suppression list is honored.
- *Fix:* add the footer, an unsubscribe endpoint, and the suppression check
  (address from kickoff Q3). Keep transactional email free of promotional
  content.

**T1.11 Data lifecycle & user rights** *(conditional: stores personal data)*
- *What it is:* a defined retention period for what you store, and a way for
  users to get a copy of their data and have it deleted.
- *Why it matters:* GDPR/CCPA give users these rights, and "we keep everything
  forever" multiplies the damage of any breach. Logs and analytics tables
  (clicks, scans, locations, security logs) grow silently.
- *Check:* which tables/logs hold personal data, including IP addresses and
  user agents? Is there a purge/expiry job or a documented retention period?
  Does "delete account" actually delete (or only deactivate)? Is there an
  export path or documented request process? Are stored IPs hashed with a
  secret salt? (An unsalted hash of an IP is reversible by brute force —
  pseudonymous, not anonymous — so don't call it anonymous in the policy.)
- *Fix:* add retention/purge, a real deletion path or a documented request
  process (with a contact address), and reflect all of it in the privacy
  policy. Retention periods are the owner's call.

### Tier 2 — SEO & discoverability

Determines whether the site shows up in search, and what it looks like when
shared or clicked.

**T2.1 Meta title on every page**
- *What it is:* the `<title>` tag — shown as the browser tab label and as the
  blue link text in Google results.
- *Why it matters:* it's the single strongest on-page SEO signal and the first
  thing a searcher reads; a missing or duplicate title across pages actively
  hurts ranking.
- *Check:* `grep "<title>" *.html` — present, unique per page, ~50–60 characters.
- *Fix:* `Primary value prop — Brand`, unique wording per page.

**T2.2 Meta description on every page**
- *What it is:* `<meta name="description">` — the gray summary text under the
  title in search results.
- *Why it matters:* doesn't directly affect ranking, but is the main lever
  over click-through rate from search results; a missing one lets Google
  auto-pick a random snippet from the page.
- *Check:* `grep 'name="description"' *.html` — present, unique, ~150–160 chars.
- *Fix:* action-oriented one-liner summarizing the page's value.

**T2.3 Canonical URL**
- *What it is:* `<link rel="canonical" href="...">` telling search engines the
  "official" URL for a page's content.
- *Why it matters:* prevents duplicate-content problems when the same page is
  reachable at multiple URLs (with/without `www`, trailing slash, `.html`,
  `?utm=` params, etc). A canonical that points at a URL which itself
  redirects sends mixed signals.
- *Check:* `grep 'rel="canonical"' *.html`; confirm each canonical uses the
  exact scheme/host/path form the server serves after redirects (section 0.4)
  and returns 200.
- *Fix:* add one per page pointing at the clean production URL, in that same
  form everywhere (canonical, `og:url`, sitemap, robots.txt).

**T2.4 robots.txt**
- *What it is:* a root-level file telling search-engine crawlers what they may
  crawl, and where the sitemap is.
- *Why it matters:* without it, crawlers guess; a misconfigured one can
  accidentally block the whole site from search. Note `Disallow` stops
  *crawling*, not *indexing*, and is not access control (see T2.10, T5.5).
- *Check:* does `/robots.txt` exist at the root, and does it avoid blocking
  pages you want indexed or the CSS/JS/images needed to render them?
- *Fix:*
  ```
  User-agent: *
  Allow: /
  Sitemap: https://<production-domain>/sitemap.xml
  ```

**T2.5 sitemap.xml**
- *What it is:* an XML list of every public URL on the site, for crawlers.
- *Why it matters:* speeds up and improves indexing, especially for new sites
  with few inbound links pointing search engines to them yet.
- *Check:* does `/sitemap.xml` exist and match robots.txt? Does every listed URL
  return 200 **without redirecting**, use the same URL style as the canonicals,
  and avoid `noindex` pages?
- *Fix:*
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
    <url><loc>https://<production-domain>/</loc></url>
    <url><loc>https://<production-domain>/<page></loc></url>
  </urlset>
  ```
  After launch, submit it in Google Search Console and Bing Webmaster Tools
  (manual step for the human — the LLM can't authenticate to those consoles).

**T2.6 Open Graph image & tags**
- *What it is:* `og:title`, `og:description`, `og:image`, `twitter:card` meta
  tags controlling how the link looks when shared on Slack, iMessage, X,
  LinkedIn, etc.
- *Why it matters:* a link with no OG tags — or an image URL that 404s — shows a
  blank/ugly preview, which measurably kills click-through when shared socially.
- *Check:* `grep -i "og:image\|og:title\|twitter:card" *.html`; then confirm the
  referenced image URL actually returns 200 as an image (`curl -sI`), at
  roughly 1200×630.
- *Fix:* add the tags; flag if no real image asset exists yet rather than
  inventing one.

**T2.7 Structured data (JSON-LD)**
- *What it is:* a `<script type="application/ld+json">` block describing the
  page's content in schema.org vocabulary (Organization, Product, FAQPage, etc).
- *Why it matters:* enables rich results in Google (star ratings, FAQ
  accordions, sitelinks) — pure upside for click-through, no ranking downside —
  as long as what it says is true (T1.8).
- *Check:* `grep "application/ld+json" *.html`.
- *Fix:* add a minimal `Organization` or `WebSite` block with name, URL, logo;
  expand to `Product`/`FAQPage`/`SoftwareApplication` if the content fits.

**T2.8 Custom 404 page (with real 404 status)**
- *What it is:* a branded "page not found" page **that the server actually
  returns with an HTTP 404 status code**, not a 200 and not a redirect.
- *Why it matters:* a broken link shouldn't dead-end on a generic browser
  error or, worse, a "success" status on a missing page (which confuses both
  users and search engines into indexing junk). Dynamic lookups (slugs,
  profiles, short links) that redirect unknown entries to the homepage create
  the same "soft 404".
- *Check:* does a 404 page exist? On the real host, does a bad URL return
  status 404 (`curl -I <domain>/nonexistent-page`)? Do dead/unknown entries on
  dynamic routes return 404/410 rather than a redirect?
- *Fix:* create a 404 page matching site branding with a link home, and wire
  the *status code* per host — this is a hosting/config step, not just a file:
  static hosts (GitHub Pages, Netlify, Vercel, Cloudflare Pages) pick up a root
  `404.html`; Apache/LiteSpeed use `ErrorDocument 404 /404.html`; nginx uses
  `error_page 404 /404.html;`; frameworks have a not-found route.

**T2.9 Favicon set & web manifest**
- *What it is:* the small icon shown in browser tabs, bookmarks, and
  home-screen shortcuts — ideally a full set (ico, svg, apple-touch-icon, and
  a web manifest for Android/PWA icons).
- *Why it matters:* a missing favicon shows a generic blank-page icon in every
  tab and bookmark — small, but it's the cheapest trust/polish signal there is.
- *Check:* `grep -i 'rel="icon"\|apple-touch-icon\|manifest"' *.html` (on
  **every** public page, not just the homepage); confirm the referenced files
  exist, that declared `sizes` match the real image dimensions, that the
  manifest is actually linked, that its name/description describe *this*
  product (not a template or previous project), and that it's served as
  `application/manifest+json`.
- *Fix:*
  ```html
  <link rel="icon" href="/favicon.ico" sizes="any">
  <link rel="icon" href="/favicon.svg" type="image/svg+xml">
  <link rel="apple-touch-icon" href="/apple-touch-icon.png">
  <link rel="manifest" href="/site.webmanifest">
  ```
  Generate placeholder icons from the existing brand mark/color if no assets
  exist, and say so explicitly in the report.

**T2.10 `noindex` on private and utility pages**
- *What it is:* telling search engines *not to list* pages like login,
  dashboards, admin, checkout, and thank-you pages, via
  `<meta name="robots" content="noindex">` or an `X-Robots-Tag` header.
- *Why it matters:* `robots.txt` `Disallow` only stops crawling — a blocked URL
  that's linked from elsewhere can still be indexed, and a crawler that's
  blocked can't see a `noindex` on it anyway.
- *Check:* list the private/utility pages (section 0.1) and check each for a
  robots meta or header.
- *Fix:* add `noindex` to those pages. For pages you specifically want removed
  from the index, allow crawling until they've dropped out, rather than
  `Disallow`-ing them.

### Tier 3 — Conversion & UX

**T3.1 CTA above the fold** *(N/A if no conversion goal)*
- *What it is:* the primary call-to-action (sign up, buy, book a demo, or the
  tool itself) visible without scrolling.
- *Why it matters:* most visitors never scroll past the first screen; if the
  CTA isn't there, they leave without converting.
- *Check:* render the hero at ~375×667 and ~1280×800 — is a high-contrast CTA
  visible with no scroll?
- *Fix:* move the CTA into the hero if buried below a long headline/animation.

**T3.2 Sticky mobile CTA** *(optional; conversion-driven landing pages only)*
- *What it is:* a CTA bar that stays fixed at the bottom of the screen while
  scrolling, shown only on mobile widths.
- *Why it matters:* on mobile, once the hero scrolls away the CTA usually goes
  with it — a sticky bar keeps the conversion action always one tap away.
- *Check:* is there an element with `position: fixed/sticky` scoped to a
  mobile `@media` breakpoint?
- *Fix:*
  ```css
  @media (max-width: 640px) {
    .sticky-cta { position: fixed; bottom: 0; left: 0; right: 0; z-index: 50; }
  }
  ```
  Add bottom padding to `body` so it doesn't cover footer content or inputs.

**T3.3 Mobile breakpoints**
- *What it is:* CSS that reflows layout at common screen widths instead of
  shrinking a desktop layout or forcing horizontal scroll.
- *Why it matters:* the majority of first visits to most sites are mobile; a
  layout that breaks on a phone loses the majority of traffic immediately.
- *Check:* confirm `<meta name="viewport" content="width=device-width, initial-scale=1">`
  is present on every page; `grep "@media" *.html *.css`; check for hardcoded
  fixed-width elements.
- *Fix:* add/verify `@media` rules at ≤480px, ≤768px, ≤1024px so text, hero,
  and nav reflow cleanly.

**T3.4 Loading states**
- *What it is:* visible feedback (spinner, disabled button, skeleton) while an
  async action — form submit, fetch — is in flight.
- *Why it matters:* without it, a slow request looks like a broken button;
  users click repeatedly or leave, sometimes causing duplicate submissions.
- *Check:* find async submit handlers; confirm a pending state renders and the
  trigger is disabled until it resolves.
- *Fix:* toggle a loading class/`disabled` attribute on submit; restore on
  both success and error.

**T3.5 Alt text & baseline accessibility**
- *What it is:* `alt="..."` describing every meaningful `<img>` (empty
  `alt=""` for purely decorative ones), plus basics: `<html lang>`, labeled
  form fields, sufficient color contrast, visible keyboard focus outlines, and
  every interactive element reachable with the Tab key.
- *Why it matters:* alt text is how screen-reader users and search-engine
  image crawlers understand images at all; contrast and keyboard nav are
  baseline requirements for users with low vision or motor impairments — and
  increasingly a legal requirement (ADA lawsuits over inaccessible sites are
  common in the US).
- *Check:* `grep -o "<img[^>]*>" *.html | grep -v "alt="` for missing alt text;
  confirm `<html lang="…">` and `<label>`s; tab through the page to confirm
  every interactive element gets a visible focus state; spot-check text/
  background contrast against WCAG AA (4.5:1). Run axe or Lighthouse if
  available.
- *Fix:* add descriptive alt text; add `:focus-visible` outlines where missing;
  darken/lighten low-contrast text; make form errors announce (`aria-live`).

### Tier 4 — Analytics & growth infrastructure

**T4.1 Analytics installed**
- *What it is:* a script reporting visits/behavior to a platform (GA4,
  Plausible, Fathom, etc).
- *Why it matters:* without it you're flying blind post-launch — no visibility
  into traffic, conversion rate, or where users drop off. Partial coverage
  (homepage only) or double-counting is nearly as bad as none.
- *Check:* `grep -i "gtag\|googletagmanager\|plausible\|fathom" *.html`. Is it
  on **every** page that matters, including the conversion pages (pricing,
  sign-up, checkout)? Is any property counted twice (e.g. `gtag.js` *and* a
  tag-manager container that also fires the same GA4 tag)? Are the key
  conversions (sign-up, purchase, primary action) defined as events?
- *Fix:* add the snippet for the platform the user specifies, gated behind
  cookie consent (T1.3) if it sets cookies. Never invent a measurement ID.

**T4.2 Google Tag Manager** *(if used)*
- *What it is:* a container script that lets you add/manage other tracking
  scripts (analytics, ads pixels, conversion tracking) from a web UI instead
  of editing code each time.
- *Why it matters:* decouples marketing/tracking changes from code deploys —
  useful the moment there's more than one tracking pixel to manage.
- *Check:* `grep "GTM-" *.html` — must appear once in `<head>` (as high as
  possible) and once as `<noscript>` immediately after `<body>`.
- *Fix:* standard snippet with the user's real container ID. If GTM is
  installed, route other analytics through it instead of a second hardcoded
  snippet, to avoid double-counting.

**T4.3 Search engine submission** *(manual, human step)*
- *What it is:* registering the site + sitemap with Google Search Console and
  Bing Webmaster Tools.
- *Why it matters:* speeds up initial indexing and surfaces crawl
  errors/security issues going forward.
- *Check:* look for existing verification first — a
  `google<token>.html` file, a `google-site-verification` meta tag, or a DNS
  TXT record. If one exists, the site is already registered; just confirm the
  sitemap is submitted.
- *Fix:* the LLM can't do this (it requires authenticating as the site
  owner) — call it out as a post-launch action item for the human.

### Tier 5 — Trust, security & resilience

**T5.1 Security headers & cookies**
- *What it is:* HTTP response headers like `Strict-Transport-Security`,
  `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `Permissions-Policy`,
  clickjacking protection (`X-Frame-Options` / CSP `frame-ancestors`), and a
  real `Content-Security-Policy` that harden the site against common attacks
  (XSS, clickjacking, protocol downgrade) — plus `Secure`, `HttpOnly`, and
  `SameSite` flags on session cookies.
- *Why it matters:* cheap to add, meaningfully reduces attack surface; many
  security scanners/partners check for these.
- *Check:* `curl -I https://<domain>` and inspect headers. A bare
  `Content-Security-Policy: upgrade-insecure-requests` is not a restrictive CSP.
  `X-XSS-Protection` is deprecated (omit it or set `0`).
- *Fix:* add via the host's mechanism found in section 0 (`.htaccess`
  `Header set`, nginx `add_header`, `_headers`, `vercel.json`, or app
  middleware). Roll a CSP out in report-only mode first, listing the analytics,
  payment, and CDN origins the site legitimately uses.

**T5.2 Public form & endpoint abuse protection** *(N/A if no public write endpoints)*
- *What it is:* any bot deterrent on public forms and endpoints that trigger
  work or email — sign-up, contact, password reset, "send me a code" — such as
  a honeypot field, CAPTCHA, rate limiting, or email verification.
- *Why it matters:* an unprotected public endpoint gets hit by bots within
  days of launch, filling your inbox/database with junk and turning your mail
  server into a spam cannon.
- *Check:* does each such endpoint have at least one deterrent? (Server-side
  rate limiting and email/OTP verification count — a honeypot isn't the only
  answer.)
- *Fix:* add a hidden honeypot input (bots fill every field; reject submits
  where it's non-empty) and/or per-IP rate limiting as a low-friction first line
  of defense.

**T5.3 Broken link check**
- *What it is:* a pass over every `<a href>` confirming it resolves.
- *Why it matters:* dead links are a bad first impression and a (minor) SEO
  signal; easy to introduce silently when content moves.
- *Check:* extract all hrefs and verify internal ones resolve to real
  files/anchors/routes; flag external links (including social profile links in
  the footer) for the human to spot-check.

**T5.4 Basic performance check**
- *What it is:* page weight and load time — images, fonts, and scripts sized
  reasonably; no render-blocking bloat.
- *Why it matters:* slow-loading pages lose visitors before the CTA even
  renders, and page speed is a direct Google ranking factor (Core Web Vitals).
- *Check:* `ls -la` on image assets for anything unusually large; check
  whether images are compressed/served in modern formats (WebP/AVIF); check
  font-loading strategy (`font-display: swap`); look for uncompressed
  responses, missing cache headers, and render-blocking third-party scripts.
  If minified/built assets are committed, confirm they're regenerated from the
  current sources. Run Lighthouse/PageSpeed if available.
- *Fix:* compress/convert oversized images; lazy-load below-the-fold images
  (`loading="lazy"`); defer non-critical scripts.

**T5.5 Exposed non-public files**
- *What it is:* files sitting in the deployed web root that were never meant
  for visitors — database schemas/migrations, test and debug scripts, internal
  docs/changelogs/roadmaps, config, backups, `.env`, `.git`.
- *Why it matters:* they hand attackers a map of your system (table names,
  endpoints, credentials) and leak internal strategy. `robots.txt` `Disallow`
  is **not** access control — it only asks polite crawlers to stay away, and
  it advertises the path.
- *Check:* list files in the web root that aren't pages or public assets. For
  each internal-looking one (plus `.git/HEAD` and `.env`), `curl -sI` it on the
  live site and expect 403/404 — a 200 is exposed, and even a 500 on a debug
  script means it's reachable. If a deny rule exists, confirm its pattern really
  matches the target: anchored regexes (`^\.(json)$` only matches a file named
  `.json`) and outdated directive syntax are common silent failures.
- *Fix:* move them out of the web root, or deny them at the server; delete
  debug/test scripts (especially any marked temporary). Don't publish anything
  you haven't read.

**T5.6 User-submitted links & content abuse** *(conditional: visitors can create public/shareable/redirecting things)*
- *What it is:* safeguards around anything visitors can publish or route through
  your domain — short links, profile pages, uploads, comments, QR codes.
- *Why it matters:* abuse arrives fast, and a single phishing or malware link is
  enough for browsers and search engines to blocklist your *entire* domain —
  taking down the legitimate site and every link or code already distributed.
- *Check:* at creation, is the destination validated (an `http`/`https`
  allowlist, not just "parses as a URL"; no loops back into your own
  redirector), rate-limited, and screened against a reputation list (Google Safe
  Browsing, URLhaus — at least on report)? Is there a public abuse-report path,
  a monitored abuse address, and a `/.well-known/security.txt`? Does the terms
  page have an acceptable-use clause (T1.2)? For uploads: type/size limits and
  no script execution in the upload directory. For redirect/lookup routes: do
  dead or unknown entries return a real 404/410 (T2.8), with
  `X-Robots-Tag: noindex` on redirects?
- *Fix:* add the validation and rate limits, an abuse-report path (address from
  kickoff Q3/Q8), `security.txt`, and terms wording. Choice of reputation
  service and its API keys is the user's.

**T5.7 Email authentication** *(conditional: sends email)*
- *What it is:* DNS records (SPF, DKIM, DMARC) proving your mail really comes
  from your domain.
- *Why it matters:* without them, mail lands in spam or is rejected, and anyone
  can spoof your domain in phishing emails. Verification codes and password
  resets that don't arrive are broken sign-up flows.
- *Check:* `dig +short TXT <domain>` (SPF starts `v=spf1`),
  `dig +short TXT _dmarc.<domain>` (DMARC), and DKIM at
  `<selector>._domainkey.<domain>`. The DKIM selector comes from the mail
  provider's docs or a sent message's headers — don't guess; ask if you can't
  find it.
- *Fix:* DNS records are a manual step at the registrar/DNS host — give the
  exact values from the mail provider. Suggest starting DMARC at `p=none` with
  a reporting address, then tightening once reports look clean.

**T5.8 Backups, monitoring & rollback** *(mostly manual)*
- *What it is:* the boring safety net — automated backups you've actually
  restored from, uptime and error monitoring, logs with a retention limit,
  secrets kept out of the repo, and a way to undo a bad deploy.
- *Why it matters:* sooner or later something breaks; whether it's a five-minute
  blip or lost data depends on this. Launch is the last cheap moment to set it
  up.
- *Check:* the LLM can verify the repo-level parts — secrets/env files are
  gitignored (with an example file committed instead), no keys in client-side
  code or git history, error output disabled in production. The rest is the
  user's answer to kickoff Q12: backups and a tested restore, uptime alerts,
  error tracking, rollback path.
- *Fix:* fix the repo-level items directly; list the rest under "Needs input
  from you" with concrete suggestions.

## 5. Fix pass

Work Tier 1 → Tier 5 in order. After Tier 1 (legal/functional), pause and
summarize what was generated or changed (privacy policy, terms, address,
consent banner, corrected claims, payment and email wording) for human review
before continuing — these carry legal and business risk the LLM shouldn't
sign off on alone. Tiers 2–5 can be implemented and reported in one pass.

## 6. Verify pass

- Re-run the audit table and confirm each fixed item now shows ✅.
- Manually check (or use a browser/preview tool if available): CTA visible above
  the fold on mobile, sticky CTA doesn't overlap content, form error states
  trigger correctly, the cookie banner blocks tracking until accepted.
- Validate `sitemap.xml` and `robots.txt` are well-formed and reachable, and that
  every sitemap URL returns 200 without redirecting.
- If deployed, probe the user's own domain with read-only requests:
  - `curl -sIL http://<domain>/` → one permanent hop to the canonical https URL
  - `curl -sI https://<domain>/<random-nonexistent-path>` → real 404
  - each `og:image` URL → 200 and an image content type
  - response headers (T5.1)
  - the exposed-file probes (T5.5)
  - DNS records (T5.7) and `/.well-known/security.txt` (T5.6)

## 7. Report format

Report by tier, and for every item include the one-line "what/why" — not just
a status — so whoever reads the report (developer or non-technical founder)
comes away understanding *what the item is*, not just whether it's checked.

Status legend: ✅ pass · ⚠️ partial or blocked on input · ❌ fail · ➖ N/A
(always give the reason).

For each item give: status, the what/why line, **evidence** (file:line or the
`curl`/`dig` result that proves it), **owner** (LLM-fixable vs. needs a human),
and the action taken or needed. Example row:

> **T2.6 Open Graph image** — ❌ missing. *What/why:* controls the preview
> shown when this link is shared on Slack/X/iMessage; without it, shared links
> show a blank card. *Evidence:* `og:image` points at `/img/og.png`, which
> returns 404. *Owner:* human (needs an asset). *Action:* added
> `og:title`/`og:description`, but no 1200×630 image exists yet — needs a real
> image from you.

Summary table template for the top of the report:

| ID | Item | Status | Evidence | Owner | Action |
|----|------|--------|----------|-------|--------|

End with a **"Needs input from you"** section listing every item blocked on
real business facts, brand assets, legal review, or a manual step (search
console submission, DNS records, backups) the LLM can't do on its own.
