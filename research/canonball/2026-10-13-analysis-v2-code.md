# Canonball: Code-Grounded Analysis and Build Plan (v2)
_2026-10-13 MT._

**Sources:**
- A read-only clone of private `king-street-co/canonball-the-literary-companion` at v0.1.160, 706 commits. It was cloned to `/tmp/cb` with pushes disabled, and nothing was pushed to it.
- rea web-bundle capture of canonball.io.
- Public listings for Goodreads and Fable. rea couldn't capture either site.
- The YouTube video framework, from its description only, because the transcript was rate-limited (HTTP 429).

The v1 live-site report is kept in `report-v1-livesite.md`. No third-party code was copied.

## 1. Video framework
**plan → research → build → review → test**, with a reusable agent "skill" for each step. The repo already has a matching scaffold:
- **Plan:** `PLAN.md`, `ROADMAP.md`, `PRODUCT.md`, `MEMORY.md`, `AGENTS.md`/`CLAUDE.md`.
- **Agent rules:** `.cursor/rules` (8 rules).
- **Research:** goes to `king-street-co/rea/research/`, outside canonball.
- **Build:** feature branches.
- **Review and test:** Vitest plus Playwright (`playwright.config.ts`) and CI gates. The CI gates are missing today (see §3).

## 2. Stack (verified in code)
- **App:** Next.js App Router (`app/`), with a legacy Vite entry still present (`vite.config.ts`, `index.html`, `dev:vite`). The package is still named `vite_react_shadcn_ts`, a leftover from the Lovable/Vite origin.
- **UI and data fetching:** React, shadcn/Radix (`src/components/ui`, 52 files) and TanStack Query.
- **Other libraries:**
  - i18n for en/fr/es (`src/i18n/locales`).
  - framer-motion.
  - epubjs for the reader.
  - Capacitor 8 iOS/Android (`capacitor.config.ts`, `scripts/build-capacitor.mjs`).
- **Backend:**
  - **Supabase**: 59 migrations in `supabase/migrations`, a typed client in `src/integrations/supabase`, and edge functions `classic-text` and `classic-epub`.
  - **Sanity** CMS for the blog and resources (`sanity/`, `app/resources/[hub]/[slug]`).
- **Hosting:**
  - **Firebase App Hosting**, project `canonball-io` (`apphosting.yaml`, deployed by `.github/workflows`).
  - `vercel.json` is also present but looks vestigial.
- **Secrets:** SOPS-encrypted (`.env.enc`, `.sops.yaml`).
- **Telemetry:** PostHog and Sentry are wired up in `src/lib/telemetry.ts`, but **their keys aren't set yet** (PLAN.md, "Also now").
- **Book data:**
  - Open Library and Google Books (`use-google-books.ts`).
  - The canonical catalog is `public.books` plus `book_aliases`, one UUID per work (`src/lib/book-registry.ts`, `book-identity.ts`).

## 3. Data model, RLS and quality
**Tables (48), grouped:**
- **Catalog:** books, book_aliases, book_pairings, classic_editions, audio_recordings.
- **Library:** bookshelves, shelf_books, ratings, book_notes, reading_positions, reading_activities, reading_plans, reading_runs, daily_reading_checkins, session_recaps, get_it_entries.
- **Clubs:** clubs, club_members, club_book_nominations, club_nomination_votes, club_schedule_entries, club_book_posts, club_post_reactions, wrap_parties, wrap_party_questions, wrap_answer_drafts, book_wrap_results.
- **Social:** profiles, friendships, profile_posts, post_likes, post_comments, notifications, book_recommendations.
- **Gamification:** badges, user_badges, season_challenges, user_season_progress, pairing_bounties, bounty_submissions, reading_races, race_participants, daily_wrap_completions, vocab_sessions, user_memory_progress.
- **AI and moderation:** companion_qa, book_content_submissions, user_roles, role_requests.

**RLS:**
- Row-level security is enabled on all 48 tables, with 190 policies and 26 `SECURITY DEFINER` functions.
- **Holes found, as of the latest migration touching each table:**
  - `pairing_bounties`: `FOR UPDATE TO authenticated USING (true)`. Any signed-in user can edit any bounty (`20260320052306_…sql`).
  - `reading_races`: the policy named "Race creators can update races" actually uses `USING (true)`, so any user can update any race (`20260320052323_…sql`).
  - `user_badges`: the insert check is `auth.uid() = user_id`, so **users can award themselves badges**. Awarding should move to a SECURITY DEFINER function or the service role.
  - `profiles` and `ratings` can be read publicly with `USING (true)`. That's fine if intended, but confirm it doesn't expose private fields.
- `book_pairings` is now staff-write only, which is good.

**Tests:**
- Vitest has 17 spec files with about 119 cases in `src/test`, mostly catalog, identity, slug and scheduling logic.
- I found no Playwright specs, even though the config exists.
- CI only deploys (App Hosting, Sanity). There's **no lint, typecheck or test gate before a deploy**.
- I didn't run the tests, to avoid installing about 1 GB of dependencies. Unverified.

**Tech debt (per PLAN.md):**
- Leftover duplicated title columns on activity tables (`daily_reading_checkins`, `session_recaps`, `book_notes`, `ratings`, …).
- The dual `canonical_book_id` column still needs renaming.
- The Vite build path is still present.

## 4. Built versus live site
| Area | Code | Notes |
|---|---|---|
| Whispersync pairings (Find) | **Built** | `components/pairing`, `use-book-pairings.ts`, `pairing-merge.ts`, `app/pairings`, `pairings-seo.ts`, affiliate links in `affiliate.ts` |
| Classic Library and AI companion reader | **Built (not advertised on the homepage)** | `lib/canon-text/` (schema v2), 13 packs in `src/content/classics` (Frankenstein, Odyssey, …), `companion/CompanionReader.tsx`, BYOK OpenAI-compatible provider (`canon-text/providers.ts`), TF-IDF retrieval, history in `companion_qa`, edge functions for text and epub |
| Reading tracker / plans | **Built** | `PlanTab`, `components/planner`, `gantt-layout.ts`, `reading_plans`/`runs`/`positions`, daily check-ins, session recaps |
| Shelves, ratings, notes | **Built** | `components/shelves`, `ratings`, `use-book-notes.ts` |
| Goodreads import | **Built** | `lib/goodreads-import.ts`, `use-goodreads-import.ts` |
| Vocab and "memory builder" | **Built** | `components/vocab`, `vocab-data.ts`, `use-memory-builder.ts`, `user_memory_progress` |
| Clubs: nominations, voting, snake draft, cadence, discussion | **Built** | 24 files in `components/clubs` (`SnakeDraftSheet`, `draft-ordering.ts`, `SetCadenceSheet`, `DiscussionThread`) |
| Wrap Party quizzes | **Built** | `components/wrap-party`, `wrap-party-quiz.ts`, content packs `book-wrap-packs-*.ts` |
| Gamification: badges, seasons, bounties, races | **Built** (RLS holes above) | `components/gamification` |
| Social feed, friends, notifications | **Built (not on the homepage)** | `components/feed`, `use-friends.ts`, `use-notifications.ts` |
| Moderation / roles | **Built** | `components/moderation`, `admin`, `user_roles` |
| Recommendations | **Friend-to-friend only** | `use-recommendations.ts` reads `book_recommendations`; there's no algorithmic engine (pillar 5, "later") |
| Mobile apps | **Scaffolded** | Capacitor configured; store release status unverified |
| Sitemap / SEO | **Partial** | SEO helpers exist (`book-seo.ts`, `pairings-seo.ts`), but `/sitemap.xml` returned nothing live |

**Key takeaway:** the product is much further along than the homepage shows. The Classic Library AI companion, social feed, Goodreads import and roles are all built but not marketed.

## 5. Feature matrix (revised)
| Capability | Canonball (code) | Goodreads (public) | Fable (public listing) |
|---|---|---|---|
| Shelves / ratings / notes | Yes | Yes (deepest catalog) | Yes, plus tags/spice/emoji |
| Goodreads import | Yes | n/a | Yes, plus Kindle import |
| Goals / streaks / plans | Yes, plus Gantt-style plans | Annual challenge | Goals, streaks, Wraps |
| Clubs | Nominate, vote, **snake draft**, cadence, discussion, **live quizzes** | Forum-style groups | Thousands of clubs; spoiler-free chapter rooms; milestones |
| Spoiler gating | Partial (spoiler flags in UI); not tied to progress | No | Chapter rooms |
| In-book reader / annotations | **AI companion reader for public-domain classics** | No | Own ebook store and social annotations |
| AI | BYOK companion Q&A (opt-in) | No | Scout recommendations; AI summaries pulled after 2025 backlash |
| Vocab / memory | **Unique** | No | No |
| Whispersync pairings | **Unique** | No | No |
| Social feed / friends | Yes | Yes | Yes, plus DMs |
| Algorithmic recommendations | No | Basic | Yes |
| Year/month Wraps (shareable) | Daily/book wraps; no yearly wrap found | Year in Books | Yes |
| Mobile | Capacitor (status unverified) | Native | Native |

## 6. Top opportunities (revised)
1. **Market what's already built, and fix the homepage.**
   - Lead with the Classic Library AI companion, which is unique, plus the social feed and Goodreads import.
   - Ship a `app/sitemap.ts` from `public.books` and the pairings.
2. **Progress-gated spoiler-safe club rooms.**
   - Join `reading_positions` to `club_book_posts` by chapter.
   - This turns the existing `DiscussionThread` into something better than Fable's chapter rooms.
3. **Unified read/listen position.** Extend `reading_positions` with format and a pairing-based conversion, building on `book_pairings` and `classic_editions`/`audio_recordings`.
4. **Algorithmic, explainable recommendations (pillar 5).**
   - Use signals already present in ratings, shelves, `club_nomination_votes` and `vocab_sessions`.
   - Start with pgvector or co-occurrence.
5. **Shareable monthly and yearly Wraps** from `reading_activities`, `ratings` and `book_wrap_results`, as a growth loop.

## 7. Phased plan (revised against the code)
Each phase runs the video loop: plan in PLAN.md, research notes in rea/research, build on a branch, review in a PR, test in CI and on preview.

**Phase 0: Harden (≈1 week, highest priority)**
- Write an RLS fix migration:
  - `pairing_bounties` update → creator or staff only.
  - `reading_races` update → `created_by = auth.uid()`.
  - `user_badges` insert → revoke for users and award through a SECURITY DEFINER function.
- Audit the public `profiles` columns.
- Add CI with `lint`, `tsc --noEmit` and `vitest run` before the App Hosting deploy, plus a first Playwright smoke test for sign-in, add a book and join a club.
- Set the PostHog and Sentry secrets in `apphosting.yaml` / Secret Manager.
- Add `app/sitemap.ts` and `app/robots.ts`.

**Phase 1: Debt and positioning (1–2 weeks)**
- Finish PLAN.md's denormalization DROPs and the `canonical_book_id` rename.
- Remove the Vite path (`vite.config.ts`, `index.html`, `dev:vite`/`build:vite`) and rename the package.
- Rework the landing page in `components/landing` to showcase the Classic Library companion, feed and import.

**Phase 2: Club OS v2 (2–3 weeks)**
- Progress-gated chapter rooms: add a chapter column to `club_book_posts` and an RLS/view filter against `reading_positions`.
- Milestone reminders through `notifications`.
- Live Wrap Party over Supabase Realtime.

**Phase 3: Read/listen unification (2–3 weeks)**
- Add format and offsets to `reading_positions`.
- Map positions across a pairing.
- Classic Library synced audio, per the roadmap: R2 audio, LibriVox.

**Phase 4: Recommendations and Wraps (3–4 weeks)**
- Use pgvector embeddings on `books`, with explainable recommendations surfaced in `FindItTab`.
- Monthly and yearly Wrap pages with OG images.

**Phase 5: Mobile and monetization**
- Release the Capacitor builds.
- Turn on affiliate links (`affiliate.ts`), then the Classic/Pro subscription from ROADMAP.md.

**Unchanged from v1:**
- Amazon has no public progress API, so pairings stay community-verified.
- AI stays opt-in and labeled; modern-book AI is parked pending legal, per PRODUCT.md.
