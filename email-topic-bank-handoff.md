# Endless Email Generator — Design & Build Handoff

*Updated after the brand pass and backend build. Supersedes the earlier
version, which described the tool as a Claude-artifact prototype with no
backend. Sections 1–4 are unchanged reasoning and still hold; sections 5–8
are new or rewritten.*

> **Renamed 2026-09-14.** The tool is now **Endless Email Generator** (was
> "Email Topic Bank") everywhere it's user-facing: page title, hero, PDF
> export, and in-tool labels. File names, the Supabase table
> (`topic_bank_sessions`), and the Edge Function (`topic-bank`) keep the old
> name deliberately — renaming them breaks live data and deployed URLs for
> no user-visible gain. The framework eyebrow is now "The Emails That Sell
> Framework".

---

## 1. What this is

An interactive tool implementing Whitney's two-step sales email framework
(from The Growth Show Ep 134, *The Two-Step Framework to Write Sales Emails
That Feel Genuine*):

1. **Brainstorm** — pick one service, generate 9 ideas across 3 buckets (3 each):
   - **Outcomes** — the transformation/result a client experiences
   - **Loved features** — what people love about *how* the service works
   - **Hesitations** — the real barriers/beliefs holding someone back (not "money" — one layer deeper)
2. **Match** — each of the 9 ideas is paired with one of 9 email structures,
   producing a reusable "bank" of topics. An AI-assisted "Get approach notes"
   button gives short directional guidance (never drafted copy) for how to
   approach that specific pairing.

The tool deliberately never writes the email itself — only what to write
*about*, and a few sentences of direction on *how* to approach it.

---

## 2. The 9 email structures — and why these nine

Started with Whitney's original 5 (from her own teaching), tested adding 6
more inspired by Mariah Coz's "17 Timeless Content Themes," found real
overlap, cut back to 9 non-redundant ones.

| Structure | Source | What it does |
|---|---|---|
| Myth-busting | Whitney (transcript) | Name a misconception, correct it directly |
| Topical / news tie-in | Whitney (transcript) | Connect research/news to your specific work |
| Case study / testimonial | Whitney (transcript) + merged | Anonymized client story *or* their own words |
| Tip / how-to | Whitney (transcript) | One tiny, doable action tied to the topic |
| Personal story / lesson learned | Whitney (transcript) + Mariah | The real, unpolished path |
| What's holding you back | Mariah, adapted | Name a real hesitation, validate it, explain why it's not the ending |
| FAQ / people always ask me | Mariah, adapted | The question you get constantly, answered straight |
| Painting the picture | Mariah, adapted | A specific moment a few months from now, vs. staying the same |
| Why now | Mariah, adapted (softened) | One concrete, real reason now is a good moment — no manufactured urgency |

**Cut, and why:** Testimonial spotlight (redundant with case study — merged),
Reader Q&A (redundant with FAQ), Comparison and Permission-giving (no real
source material — they were guesses, and it showed).

**Deliberately softened from Mariah's originals:** her "Why Now" leans on
countdown-timer urgency, which doesn't fit Whitney's anti-hype positioning —
rewritten to require a *real*, specific reason.

---

## 3. Bucket → structure mapping (and the reasoning trail)

Each bucket's 3 ideas are assigned to one of 3 pre-set structures — always
all three, no duplicates, no skips (a permutation, not independent random
picks — this was a bug we caught and fixed).

**Current mapping:**

- **Outcomes** → Why now, Topical/news, Case study/testimonial
- **Features** → Personal story, Tip/how-to, Painting the picture
- **Hesitations** → Myth-busting, FAQ, What's holding you back

**Why this grouping (in case it needs revisiting):**
- First pass loosely let any bucket suggest almost any structure — too vague,
  and testing showed uneven/duplicate assignments.
- Second pass over-indexed on "outcomes are the strongest hook" and put 6 of 9
  structures under outcomes — Whitney flagged this as lopsided and asked for
  even 3/3/3 coverage.
- Third pass balanced to 3/3/3, but initially placed "What's holding you back"
  under Outcomes. Testing showed the AI notes for that pairing didn't land.
- **Final correction:** "What's holding you back" moved to Hesitations — closer
  to Mariah's original definition (it's fundamentally objection-handling). Case
  study moved back to Outcomes to keep 3/3/3 balance, and because Whitney's own
  transcript example paired case study with an outcome.

The mapping lives in one place in the code (`const SUGGEST = {...}`) — a simple
three-array object, editable without touching anything else.

---

## 4. Voice & tone rules for the AI-generated "approach notes"

Hardest part to calibrate; took several rounds. Rules baked into the prompt:

- **No corporate/B2B jargon** — explicit banned list: leverage, unlock, elevate,
  empower, seamless, journey, synergy, game-changer, dive in, unpack, resonate.
- **Plain, warm, direct** — modeled on Whitney's actual transcript language.
- **Positive framing only** — early drafts told the model what *not* to do and it
  read stiff and AI-generated. Every instruction now states what *to* do.
- **Real substance required** — the middle bullet must name one concrete piece of
  value the email actually delivers. Added after noticing the notes could
  produce a thin "opener → light gesture → pitch" email.
- **Output format is fixed:** exactly 3 short bullets — opening move, the concrete
  value/insight, how to pivot into the offer. Under 14 words each.
- **Few-shot examples** are drawn from Whitney's own transcript, using a fictional
  practitioner persona rather than Whitney herself (the tool is for her clients'
  businesses).

> ⚠️ **The prompt now lives in the Edge Function, not the HTML.** See section 6.
> The Edge Function is the single source of truth. This doc summarizes the
> *reasoning*; the function has the exact wording.

---

## 5. Brand implementation

Built on the WBDS system from `offers.whitneybateson.com/ads-made-simple`.
All tokens are in one `:root` block at the top of the file.

```
--cream #efeae3   --pink #ffcdcd   --coral #ff7f50   --lime #e3f696
--sky   #aed3dd   --teal #02525d   --peach #fdad8f   --text #262d32
```

Playfair Display for display type, Nunito Sans for body, 19px base.
Coral pill buttons, uppercase with letterspacing. Teal for headings and
active states.

**Bucket colour-coding.** Each bucket owns a tint — lime for Outcomes, sky for
Loved features, peach for Hesitations — carried through the brainstorm zones
and the topic cards. This replaced the prototype's nine arbitrary structure
colours, which encoded nothing. Card colour now tells you which bucket a topic
came from at a glance.

**Section background** is a cream→blush gradient that deliberately stops short
of full `--pink`. Full pink collides with the peach Hesitations tint and the
bucket stops reading. If revisiting, note that *any* end colour from the brand
palette will collide with one of the three bucket tints — keep the background
neutral and let the buckets carry the colour.

**Hero video.** 5-second loop of someone typing, sourced from Mixkit
(no attribution required). Original was 4K/24MB; transcoded to 1080p H.264 at
533KB with audio stripped and `+faststart`, plus a 471KB WebM and a poster
frame pulled from the video's own first frame.

⚠️ The current file has the video **inlined as a base64 data URI** so it works
as a single portable file. **For production, swap the `<video>` tag back to the
external version** and upload `hero-loop.mp4`, `hero-loop.webm`, and
`hero-poster.jpg` alongside the HTML. A 300KB+ base64 blob blocks first paint.
There's a comment in the file marking the spot.

Known open item: the clip is 5s and the hands don't return to their starting
position, so there's a visible jump each loop. A ping-pong (forward then
reversed) encode would fix it.

---

## 6. Backend — BUILT

Lives in the **DFY Funnel** Supabase project (`rguqefwhlzehpzgljbdo`),
alongside the `generate-ads` function it was modeled on.

### Edge Function: `topic-bank`

`POST {SUPABASE_URL}/functions/v1/topic-bank`, `verify_jwt: true`
(anon key in the Authorization header). Three modes:

| Mode | Payload | Returns |
|---|---|---|
| `load` | `email`, `first_name` | Saved `form_data` + `results`, or creates the row on first visit |
| `save` | `email`, `first_name`, `form_data`, `results` | `{saved:true}` — upsert on email |
| `notes` | `email`, `service`, `topic`, `bucketLabel`, `structureLabel`, `structureHint` | `{notes:"..."}` — 3 bullets |

Reuses the `generate-ads` conventions: same CORS block, same retry-with-backoff
on 429/529/503, same friendly error strings.

⚠️ **The Anthropic API key is read from a secret named `ads-made-simple`.**
Misleading name — it predates there being more than one tool — but four live
functions reference it, so renaming breaks them. Left as-is deliberately.

Model is pinned to current Sonnet. Note `generate-ads` still pins
`claude-sonnet-4-20250514`; the two are intentionally not matched.

### Table: `topic_bank_sessions`

`id`, `email` (unique), `first_name`, `form_data` jsonb, `results` jsonb,
`source` text (`leadgen` | `course`), `created_at`, `updated_at`.

> ⚠️ **Upserts must use `?on_conflict=email`.** PostgREST resolves upsert
> conflicts against the PRIMARY KEY by default, which here is `id`, not
> `email`. Without that query param every save attempts a fresh INSERT and
> dies on the unique email constraint with a 409. This cost an afternoon;
> don't remove it.

Kept **separate from `sessions`**, which is owned by the ads tool and keyed on
the same email — sharing it would have overwritten people's ad copy.

RLS is enabled with **no policies**, so the Edge Function's service role is the
only way in. Nothing is reachable from the browser with the anon key. This
matches the pattern on `help_requests`, `meta_setup`, `reads`, and
`website_intakes`.

### Front end

Opens on a full-screen gate collecting first name + email. On submit it calls
`load`, which either restores a previous session (service, nine ideas, built
bank, structures, used flags, generated notes) and jumps to step 2, or creates
the row and shows the empty brainstorm.

The row is created **at gate submission**, so the lead is captured even if the
person never finishes the brainstorm.

Autosave is debounced 1.2s and fires on: bank build, shuffle, structure change,
used toggle, and notes generation.

---

## 7. Verified working (2026-09-09)

Full chain tested end to end: gate → brainstorm → build → approach notes →
autosave → Kit sync. Confirmed in the database, the Kit API, and the function
logs.

- Kit form is **9900591** ("Email Generator", uid `0a8ed833f0`).
- Secrets set: `KIT_API_KEY` (v4 key, `kit_` prefix), `KIT_FORM_ID`.
  `KIT_TAG_ID` is unset and optional.
- That form has **double opt-in off** — subscribers land `active` immediately.
- Kit sync fires on FIRST registration only. To re-test with an email that has
  already been through, delete its row from `topic_bank_sessions` first, or the
  function takes the returning-visitor path and skips Kit entirely.
- Not yet tested: creating a genuinely new Kit subscriber. The test address was
  already on the list since Jan 2025, so Kit added it to the form rather than
  creating it.

## 8. Still open
- **Conversion path — built.** Three exits now: the nav CTA
  ("Help me build my list"), the Growth Show callout on Ep 134 in the
  How-to-use section, and a teal DFY Funnel CTA block at the foot of Step 2,
  under the download buttons ("Grow Your Email List"). All three point at
  `offers.whitneybateson.com/dfy-funnel/`. Nothing links to Whitney's 1:1
  services or WPWS — worth revisiting if those need a route.
  The funnel CTA currently shows on **both** builds, course included.
- **Nav logo — done.** `logo.png` (the two-line teal script wordmark,
  1545x863 transparent PNG) sits next to the HTML and renders at 70px tall,
  44px on mobile. It needs that much height because it's a stacked two-line
  script; a single-line mark would sit around 42px. If it ever goes missing
  the "Whitney Bateson" text wordmark renders in its place, so the nav never
  breaks.
- **PDF exports don't include notes generated in a previous session** — they do
  now that notes persist, but this is worth re-testing end to end.
- **Unrelated security issue in the same project:** `clients` and
  `students_ads_made_simple` have RLS policies written but RLS never enabled,
  making both fully readable by anyone with the anon key — which ships publicly
  in this tool's HTML. `students_ads_made_simple` is the course roster.

---

## 9. Files

- `index.html` — the **public** build (`VERSION = "leadgen"`): opt-in copy
  shown on the gate, subscriber synced to Kit.
- `course.html` — the **Ads Made Simple student** build
  (`VERSION = "course"`): work still saves, nothing goes to Kit, no opt-in
  notice.

> **Renamed 2026-09-14.** These two swapped roles. `index.html` used to be
> the course build (it had been renamed from `email-topic-bank-COURSE.html`)
> and `email-topic-bank.html` was the public one. Since everything public
> facing lives at the default file name, `index.html` is now the public build
> and the student build moved to the self-describing `course.html`.
> **`email-topic-bank.html` no longer exists** — any link pointing at it
> needs updating.

The two files are byte-identical apart from that one `VERSION` line;
regenerate one from the other rather than hand-editing both.
- `hero-loop.mp4` / `hero-loop.webm` / `hero-poster.jpg` — hero assets for the
  production swap described in section 5.
- This document.
