# Canonball: Deep Competitive Analysis (Goodreads, Fable) and Build Plan (v3)
_2026-10-13 MT._

**Sources:**
- Public pages and listings only.
- rea captures of public pages.
- The real canonball code: a read-only clone of `king-street-co/canonball-the-literary-companion` at v0.1.160.

No third-party code or assets were copied. Earlier versions: `report-v1-livesite.md`, `report-v2-code.md`. Raw rea output is in `rea2/`.

**Labels:** **[V]** = verified from the cited source. **[U]** = unverified or inferred.

## 0. What I could and couldn't access
- **rea (`analyze-web-bundle`, headless Chrome):**
  - Run with a realistic UA, `--disable-blink-features=AutomationControlled` and a 1366×900 window. rea has no stealth or headed mode, so this was done through Chrome flags.
  - **Goodreads book page:** 33 scripts; fingerprinted **Next.js + React** [V].
  - **Goodreads author page:** 19 scripts. **List page:** 27 scripts.
  - **Goodreads `/challenges`:** "Page not found". The challenge URL has moved, and the logged-in challenge needs auth.
  - **Fable home:** 1 script; club page: 3; book page: 4 (Next.js) [V].
  - **Fable web** is a thin JS shell ("You need to enable JavaScript"); its product lives in the mobile apps.
- **Fetched and read:**
  - App Store listings, release notes and featured reviews for both apps.
  - Google Play download and review counts: Goodreads 10M+ downloads, about 210–220K reviews. Fable 1M+ downloads, about 28–29K reviews. Play star ratings couldn't be parsed reliably [U].
  - Reddit threads.
  - Scribd/Everand press and PW, plus Fable help center article 185.
  - The Bookseller and other review-bombing coverage.
- **Couldn't access:**
  - Logged-in app flows. Mobile apps weren't installed, and I couldn't decompile them; that's also off-limits under the disclaimer.
  - Goodreads Help Center beyond search snippets.
  - r/books and r/FableApp at scale, so quote counts are thread-level, not statistical.
  - The YouTube video transcript (HTTP 429).

## 1. Market facts [V]
| | Goodreads | Fable |
|---|---|---|
| Owner | Amazon | **Scribd Inc. (Everand)** since June 2025 (Scribd newsroom; PW 2025-06-10) |
| Scale | "150M+ members"; iOS 4.8★ from 758K ratings; Play 10M+ downloads | "Community of 5M, ~200K book clubs" (Scribd, 2026-06-02); iOS 4.8★ from 82K, Editors' Choice; Play 1M+ |
| Monetization | Free with ads, Amazon affiliate; "Get the book" (US, Sept 2026) | **Fable Plus $5.99/mo or $49.99/yr** (stats, goals, cosmetics); à-la-carte ebooks ($3.99–14.99); **Everand bundle** (June 2026) |
| Release cadence (from notes) | Mostly "bug fixes"; half and quarter stars shipped 2026-10 after years of requests; DNF shelf June 2026; review drafts March 2026 | Weekly; Library UI, Goodreads import fixes, Scout "Pick for me", liquid-glass UI, streak widgets |
| Age rating | 13+ | **18+** (social, DMs, age assurance) |

## 2. User voice: loves and pain points
Counts are the number of distinct sources or threads I read that mention the point. They're indicative, not statistical.

**Goodreads pain points:**
1. **Slow, buggy, stagnant** (6):
   - "Takes a solid 20-30s"; "They have not updated this website since 2009" (r/goodreads 1rjiao8).
   - Broken updates and profile (1w904hs).
   - App Store review: "it's mostly bug fixes! This app NEEDS something new".
   - bestbooktracker 2026: "necessary evil".
2. **Unwanted layout churn** (2): "stop needlessly updating" (1wkleom).
3. **Half stars** (4): the most-requested feature for years, only now shipped (App Store reviews and release notes).
4. **Clunky bulk shelving and rereads** (2): "five clicks to add a book to a shelf… wish there was a select tool"; rereads don't count.
5. **Ratings trust and review bombing** (4):
   - The Bookseller.
   - r/selfpublish: "25,600 ratings, all 1-star".
   - Goodreads' own forum, June 2026.
   - Pre-publication checks are weak.
6. **Amazon data concerns** (3): kiveo, bestbooktracker, r/FableApp ("supporting an app that isn't Amazon owned").

**Goodreads loves:** the catalog, editions, series links, author follow and new-release alerts, Choice Awards, an established friend graph (2–3 sources).

**Fable loves:**
- Free half and quarter stars, tags, spice level and content warnings, spoiler toggle (App Store review).
- **Stats and the reader-type summary**: "compare your charts… they tell you what kind of reader you are".
- An Instagram-like feed.
- Streaks and widgets.
- "The best book club app, period" (bestbooktracker).

**Fable pain points:**
1. **Plus isn't worth it** (4 threads): "a tenner for a couple of graphs"; "put [requested stats] behind a paywall"; "unlock this! widgets I can't delete"; no regional pricing (₹600).
2. **AI trust** (2): AI reader summaries pulled in January 2025 (Overflowing Shelf).
3. **Ownership uncertainty** (1): the Scribd deal leaves "readers wondering… subscription upsells".
4. **Feed controls** (1): only mute user or mute book, no "show less like this".
5. **Outages at peak** (1): during year-end wraps (r/FableApp 1q2vq2a).
6. **Weak Android app** (1, same thread).
7. **Can't add missing books manually** (1).

## 3. Feature matrix (Canonball as built in `/tmp/cb`)
| Feature | Goodreads | Fable | Canonball (code) |
|---|---|---|---|
| Shelves, custom lists | Yes (tags + shelves) | Yes, plus followable lists | Yes, `components/shelves` |
| DNF | Yes (2026) | Yes | **Yes** (`MarkAsReadSheet`, `ShelvesView`) |
| Rereads counted | Partial | Yes (late 2024) | **No** (only in club `BookWrapEvent`) |
| Half / quarter stars | Yes (Oct 2026) | Yes | **No**: `ratings.rating INT CHECK 1..5` (migration `20260319231131`) |
| Tags, spice, content warnings on reviews | Partial | Yes | **No** |
| Spoiler-hidden reviews | Yes | Yes | Partial (companion only) |
| ISBN / cover scan | Yes | Yes [U] | **No** |
| Goodreads import | n/a | Yes, plus Kindle | **Yes** (`lib/goodreads-import.ts`); no Kindle import |
| Data export | CSV | [U] | **No** |
| Goals, streaks | Challenge | Goals, streaks, widget, editable past streaks | Goals and streaks (`components/goals`, `ReadingRunCard`); **no widget or push** |
| Stats / reader identity | Year in Books | Deep stats (Plus), Book Aura, monthly Wraps | `ReadingGoalWrapped`, `ReadingRecapCard` (shareable via html2canvas) |
| Feed, follows, DMs | Feed | Algorithmic feed, DMs ≤20 | Feed and friends (`components/feed`, `use-friends.ts`); no DMs |
| Feed controls | Minimal | Mute user or book | Mute and block in `PostActionsBar` [U depth] |
| Clubs | Forum groups | Chapter rooms, milestones, reflection room, influencer clubs | **Nominate, vote, snake draft, cadence, Gantt schedule, discussion, live wrap quiz** |
| In-book reading and annotation | No | Own ebook store, social highlights | **Classic Library companion reader** (`canon-text`, 13 PD packs) |
| AI | Recs | Scout recs; summaries removed | **BYOK opt-in companion Q&A, spoiler-aware** |
| Vocab / memory | No | No | **Yes, unique** |
| Kindle+Audible Whispersync pairings | No | No | **Yes, unique** |
| Synced ebook and audio | No | Everand sync | Planned (R2 audio, LibriVox) |
| Author pages, follow, giveaways | Yes | Partial | No (Author Hub is "later") |
| Awards / community voting | Choice Awards | No | Seasons and bounties (`gamification`) |
| Paid tier | No (ads) | Plus $49.99/yr | None yet |

## 4. UX patterns worth emulating (rebuilt from scratch, never copied)
- **Covers as the UI:** a progress ring on the cover, plus one-tap "update progress" by page, percent or audio timestamp (both apps).
- **A home-screen streak widget** with editable past days, which forgives a missed log (Fable).
- **Reflection room at book end, and milestone pacing in clubs** (Fable). Canonball's Wrap Party is a more fun version.
- **Series rail and "next in series"** on the book page (Goodreads, praised in reviews).
- **Monthly Wrap stickers and shareable cards** (Fable). Canonball has `ReadingRecapCard`; make it a monthly ritual.
- **Review composer:** half stars, mood tags, spice and content warnings, a spoiler toggle, auto-saved drafts (both apps).
- **Bulk-select shelving** (Goodreads' missing feature; a quick win).

## 5. Growth loops
1. **SEO:** pairing and book pages → sign-up. Needs `app/sitemap.ts`, which is missing.
2. **Club invites:** one host brings in 5–15 people (`ClubInviteSheet`). Snake draft and live quizzes make inviting fun.
3. **Shareable artifacts:** monthly and yearly Wraps, quiz scores, vocab streak cards → social → installs.
4. **Contribution:** pairing verification bounties → better data → more SEO.
5. **Import-switch:** Goodreads CSV import, timed with Goodreads outages and layout changes. Target the "necessary evil" users.

## 6. Spectacular differentiators
1. **"Read it. Hear it. Never lose your place."** The only app built around Kindle+Audible pairing and unified read/listen progress. The pairings are verified; extend `reading_positions` with format.
2. **Classics companion reader with spoiler-aware AI**: opt-in, labeled, BYOK, never judging users. It's the inverse of Fable's AI failure.
3. **Clubs as a game:** snake draft, progress-gated chapter rooms, live Wrap Party quizzes, season standings. Nobody gamifies clubs.
4. **Literary memory:** vocab and quotes from what you read, with spaced repetition. A daily habit loop with more substance than streaks alone.
5. **Free stats and fair pricing.** Give away what Fable paywalls: stats, half stars, Wraps. Monetize synced classics audio, unlocks for premium companion models, club-host tools and affiliate.
6. **Trust by design:**
   - Ratings only after a "started reading" signal, plus weighting by verified readers.
   - Strong feed controls ("show less like this").
   - One-click full export.
   - Independent and ad-free.
7. **Speed:** a Next.js app with a fast cached catalog in `public.books`. Goodreads users wait 20–30s per page.

## 7. Updated phased plan
**Phase 0: Harden (1 wk)**, unchanged from v2:
- Fix the RLS holes in `pairing_bounties`, `reading_races` and `user_badges`.
- Add CI gates and set the PostHog and Sentry keys.
- Add `app/sitemap.ts`.

**Phase 1: Table stakes the competitors trained users to expect (2 wks)**
- Change `ratings.rating` to `numeric(2,1)` with a 0.25 step, and add review tags, spice, content warnings, a spoiler flag and auto-saved drafts.
- Rereads: add a `read_count` or separate read sessions to `shelf_books`.
- Bulk-select shelving.
- Full CSV and JSON export.
- Remove the Vite leftovers.

**Phase 2: Habit and mobile (2–3 wks)**
- Release the Capacitor apps with local push reminders and streak widgets (native plugins).
- Editable past streak days.
- ISBN barcode scan.
- Make the monthly Wrap a ritual by extending `ReadingRecapCard`.

**Phase 3: Club OS v2 (3 wks)**
- Progress-gated chapter rooms (`club_book_posts.chapter` + `reading_positions`).
- Reflection room.
- Milestone pacing and nudges via `notifications`.
- Live Wrap Party on Realtime.
- Season leaderboards.
- Host tools.

**Phase 4: Read/listen unification and Classics (3–4 wks)**
- Format-aware positions mapped across pairings.
- Synced public-domain audio (R2 + LibriVox).
- More companion packs.
- Spaced-repetition vocab from your own books.

**Phase 5: Trust, discovery and stats (3–4 wks)**
- Verified-reader rating weighting and anti-bombing heuristics.
- Feed "show less like this".
- Free deep stats and a reader-identity card.
- Explainable recommendations (pgvector).
- Series rails and author follow (the first slice of Author Hub).

**Phase 6: Monetization**
- Affiliate edition links (`affiliate.ts`).
- "Canonball Classic" subscription: synced audio, premium companion, club-host pro.
- Keep stats and ratings free, positioned against Fable Plus.

**Risks:**
- No Amazon progress API, so progress stays manual or community-verified.
- Modern-book AI stays parked (legal).
- Peak load during year-end Wraps (Fable outage lesson): precompute Wraps and cache them.

## 8. Sources
- App Store, Goodreads: apps.apple.com/us/app/goodreads-book-reviews/id355833469
- App Store, Fable: apps.apple.com/us/app/fable-track-discuss-books/id1488170618
- Google Play: co.fable.fable, com.goodreads
- r/goodreads: 1rjiao8, 1w904hs, 1wkleom
- r/FableApp: 1s7v75i, 1ppn5rz, 1r6o3aj, 1wrmjr3, 1q2vq2a
- r/selfpublish: 1r8d3r0
- Scribd newsroom: fable-joins-everand (2025-06-09); everand-and-fable-launch-a-single-shared-subscription (2026-06-02)
- Publishers Weekly, 2025-06-10
- help.fable.co/article/185
- The Bookseller, review bombing coverage; forreadingaddicts.co.uk
- bestbooktracker.com, kiveo.app comparisons (third-party blogs; treat as [U] opinion)
- overflowingshelf.com, Fable AI controversy, 2025-01-02
- rea raw output: `/workspace/canonball-analysis/rea2/*.json`
