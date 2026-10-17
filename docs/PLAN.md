# Wanlly Seen: build plan

Seen is a free directory at **seen.wanlly.africa** where Kenyan freelancers and small businesses get featured and found. Each one gets a personal portfolio site built from a template, with their work pulled in from links and buttons that let people reach them directly.

Seen only connects people. It makes no money, takes no commission, shows no prices, and has no inbox, bookings or payments. Clients and creatives talk on WhatsApp, Instagram, email or whatever the owner adds, and agree the work between themselves.

The UI reference is `prototype/index.html` (open it in a browser, no build step). It includes a guided walkthrough.

## 1. Who it's for

- **People looking for someone:** search by craft and town, look at real work, tap to reach the person.
- **Freelancers** (designers, illustrators, photographers, videographers, DJs, makeup artists, writers) get a site that looks like their work, replacing a link-in-bio, a PDF portfolio and a Google Drive folder.
- **Small businesses** (caterers, tailors, printers, studios, salons) get the same.

Start in Kenya with creative work, then add trades.

## 2. What a page is

| Part | How it works |
|---|---|
| **Template** | Picked from a gallery made for each craft, filtered by category. 23 so far: Street (hip hop designers), Gallery (photographers), Reel (filmmakers), Studio (agencies), Kitenge (fashion), Glow (beauty), Groove (DJs), Menu (food), Ink (tattoo artists), Canvas (painters), Mono (black and white photographers), Stage (dancers), Pages (writers), Blueprint (architects), Chalk (tutors), Garage (mechanics), Bloom (florists), Matatu (sign writers and matatu art), Spoken (poets and MCs), Frame (interior designers), Pixel (developers), Craft (beadwork and jewellery), Fade (barbers). Growing towards 100. Every template is editable: accent colour, fonts, which sections show and their order. Switching keeps all links. |
| **Work** | Links only. The owner pastes a link to an Instagram post or reel, TikTok, YouTube or Vimeo video, Behance or Dribbble project, SoundCloud or Mixcloud mix, Pinterest pin, or a public image link. Seen recognises the site and shows the post from where it lives. |
| **Contact buttons** | Whatever the owner adds: WhatsApp, Instagram DM, email, phone, TikTok, X, LinkedIn. Each button opens that app directly (`wa.me`, `ig.me`, `mailto:`, `tel:`). |
| **Proof** | Optional. A Google Business link shows the star rating and review count; a "worked with" line lists clients. |
| **Share** | A short link (`seen.wanlly.africa/kaz`) and a QR code to put in an Instagram bio, a WhatsApp status, on posters, stickers and shop walls. |

## 3. Storing nothing

The goal is to store no media at all, so storage costs stay near zero.

| Source | How it shows on the page | Notes |
|---|---|---|
| YouTube, Vimeo, TikTok, SoundCloud, Mixcloud | Their own embed player, from each site's free oEmbed endpoint | No key needed. Cache the oEmbed response (title, thumbnail URL) in the database for a day. |
| Instagram posts and reels | Instagram's embed (`<blockquote class="instagram-media">` plus `embed.js`) | Works for public posts without a key. Instagram's oEmbed API (for titles and thumbnails) needs a Meta app with the oEmbed Read feature approved. Apply in Phase 0, and fall back to the plain embed until it's approved. |
| Behance, Dribbble, Pinterest, any other page | A link card built from the page's Open Graph tags (title and image URL) | Fetched once by the Worker and cached as text. The image is loaded from their server, not ours. |
| A direct image link (`.jpg`, `.png`, `.webp`) | Shown as the image | Only from HTTPS links. If it breaks, the card shows the link instead. |
| Google reviews | Star rating and review count | Google's Places API has a free monthly allowance. Fetch each business's rating at most once a week and cache the numbers. Link out to Google for the reviews themselves. |

**The only exception:** a profile photo. Offer "use my Instagram or Google photo by link" first. If someone uploads one, resize it to 256 px WebP (about 15 KB), which is roughly 15 MB for 1,000 pages and fits easily in Cloudflare R2's free 10 GB.

**When a post is deleted or made private**, its card shows "This post is no longer available" and the owner gets a nudge in their dashboard to replace it.

## 4. Why people come back

There are no leads to send people back, so these do the job instead:

- **New work ranks first.** Pages that added a link in the last 30 days sort to the top of search. Posting on Instagram and pasting the link becomes a habit.
- **Monthly "you were seen" email** on the 1st: page views, search appearances, taps on each contact button, QR scans.
- **The page is worth sharing.** A good-looking site with a short link and QR code is something people put in their bio and on their stall, which brings new visitors and new owners.
- **Templates keep improving.** A few new templates a month gives owners a reason to come back and try one.

## 5. Getting the first pages: "claim your page" outreach

Same approach as Wanlly Sign's law firm outreach: import a CSV, send a short sequence of emails slowly from our own domain, stop on reply or unsubscribe. The email says: "Your business is on Seen. Claim it free to pick a design, link your posts and add your WhatsApp, or remove it."

### Rules (these protect the domain and keep it legal)

Kenya's Data Protection Act 2019 covers personal data about people, and a one-person freelancer's name and email are personal data.

1. **Businesses can get an unclaimed page. Individual freelancers can't.** Unclaimed pages are only made for registered businesses whose details they published themselves (their website, Google Business profile, a public association list). Freelancers get an invite email instead ("reserve seen.wanlly.africa/yourname") and have no page until they make one.
2. **Unclaimed pages show only public business facts:** name, category, town, website, and where the data came from with the date checked. No personal names, no photos, no embedded posts. Their contact email is never shown.
3. **The email only says what's true.** The page must exist when we say "you're on Seen". No fake view counts.
4. **One click to remove.** Every unclaimed page and every email has "Remove this page". Removal deletes the page and adds the email and domain to a do-not-list, so a later import can't bring it back.
5. **Unclaimed pages are `noindex`** in search engines until claimed.
6. **Slow sending:** at most 25 a day at first, weekdays 8am to 5pm Kenya time, a first email and two follow-ups 5 working days apart, from `seen@wanlly.africa` with SPF, DKIM and DMARC set. Reuse `src/lib/outreach.ts` and `src/jobs/outreach.ts` from `wandago/wanlly-sign`.

CSV columns: `business, email, category, town, website, phone, source` (with `business` and `email` required).

Claiming: the owner opens the link from the email (it carries a one-time token) or taps "Claim this page". We send a 6-digit code to the email already on the page, or by SMS to the published business phone. After that it's a normal account (Clerk, shared with the other Wanlly apps).

## 6. Templates, technically

- A template is **one HTML layout plus a set of design tokens** (colours, fonts, corner radius, section order), the same way the prototype restyles one markup with `data-tpl`. New templates are mostly new tokens and CSS, so reaching 100 is realistic.
- An owner's customisations are a small JSON object saved on their page (`{ template: "street", accent: "#FFE600", sections: ["work", "proof"] }`).
- Templates are server-rendered by the Worker, so pages load fast on slow phones and search engines can read them.
- Fonts come from Google Fonts. Each template loads at most two, so a page only pays for its own fonts.
- Templates are built in-house first. Later, designers in the community can submit templates, credited on the template card.

## 7. Stack

Same as Wanlly Sign, so the code and the deploy steps carry over.

- **Public pages (home, search, every personal site):** rendered as HTML by Hono on the Cloudflare Worker, cached at Cloudflare's edge for a few minutes.
- **Editor, dashboard and admin:** React (Vite), the same setup as Wanlly Sign. The editor shows a live preview using the same template code.
- **Data:** Neon Postgres with Drizzle. Search uses Postgres full-text search plus `pg_trgm` for typos.
- **Link previews:** the Worker calls oEmbed and Open Graph once per link and caches the result in the `links` table.
- **QR codes:** generated as SVG on request and cached at the edge. No files stored.
- **Sign-in:** Clerk, the same production instance as `wanlly.africa` and `sign.wanlly.africa`.
- **Email:** the jobs app on HostAfrica sends over SMTP from `seen@wanlly.africa`, as in Wanlly Sign.
- **Counting:** one row per page per day (`page_stats_daily`): views, search appearances, taps per contact button, QR scans. Bots are filtered by user agent and Cloudflare's bot score.

## 8. Tables

| Table | What it holds |
|---|---|
| `pages` | slug, kind (freelancer/business), name, trade, town, bio, avatar URL, status (unclaimed/claimed/hidden), source, template, template settings (JSON), last_link_at |
| `page_owners` | Which Clerk user owns which page |
| `categories`, `page_categories` | The fixed category list and each page's categories |
| `links` | page, position, URL, kind (instagram/youtube/…), cached title and thumbnail URL, last checked, broken flag |
| `contacts` | page, kind (whatsapp/instagram/email/phone/…), value, position |
| `templates` | id, name, who it's for, tokens (JSON), status (live/draft) |
| `page_stats_daily` | page, day, views, search appearances, taps per contact kind, QR scans |
| `prospects`, `outreach_settings` | Copied from Wanlly Sign; prospects link to the unclaimed page they created |
| `do_not_list` | Emails and domains that asked to be removed. Checked on every import. |

## 9. Step by step

Each phase ends with something you can click on.

### Phase 0: Repo and accounts
1. Copy the project setup from `wanlly-sign` into this repo (Vite, Hono, Drizzle, wrangler, the jobs app, CI).
2. New Neon database. Reuse Clerk and the HostAfrica mailbox.
3. Add `seen.wanlly.africa` as a custom domain on the Worker.
4. Apply for Meta's oEmbed Read feature (for Instagram titles and thumbnails). Set up a Google Places API key with a hard daily cap.

### Phase 1: Personal sites
The template renderer with all 23 templates from the prototype, link cards for every source in §3, contact buttons, share link and QR code. Pages made by hand from example data.

### Phase 2: Owner accounts and the editor
Sign up, claim a slug, pick a template, paste links, add contact buttons, change the accent colour, live preview, publish.

### Phase 3: Directory
Home, search, category and town pages, "new work first" ranking.

### Phase 4: Outreach and claiming
CSV import that creates unclaimed business pages, the do-not-list, the claim flow with email or SMS code, and the 3-email sequence. Start with 25 a day and watch replies and removals for two weeks before raising it.

### Phase 5: Retention
Stats in the dashboard, the monthly "you were seen" email, and broken-link checks.

### Later
- More templates every month, on the way to 100, then templates submitted by designers.
- Custom domains (`kazdrips.com` pointing at a Seen page) through Cloudflare for SaaS.
- More countries (Uganda, Tanzania, Rwanda), and town landing pages ("Photographers in Nakuru") for search engines.

## 10. Rules to keep from day one

- Free on both sides. No commission, no paid placement, no prices, no inbox. Seen only connects people.
- Store links, not files.
- Nothing about a person is published without their consent.
- Every email can be stopped with one click, and a removal is permanent.
- The only ranking signals are relevance, town, and how recently the page added work.
