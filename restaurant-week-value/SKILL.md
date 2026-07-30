---
name: restaurant-week-value
description: Research a city's Restaurant Week (or any prix-fixe / tasting-menu promotion) and build a verifiable value-for-money ranking of participating restaurants, comparing the prix-fixe menu against each restaurant's regular à la carte prices. Use this whenever the user asks for a "restaurant week" comparison, prix-fixe value ranking, "which restaurant week deal is worth it", a discount/性价比 leaderboard for a dining promotion, or wants to know if a fixed-price menu is a good deal versus ordering normally — even if they don't say "restaurant week" explicitly (e.g. "is this tasting menu deal actually cheaper than the normal menu", "rank these prix-fixe restaurants by discount"). Always push toward delivering a spreadsheet (xlsx) as the final output, since that's the format this workflow is built around.
---

# Restaurant Week Value Ranking

## What this skill produces

A restaurant-by-restaurant value analysis for a city's Restaurant Week (or
similar prix-fixe promotion), delivered as an Excel workbook. For every
restaurant in the sample, the workbook shows what the prix-fixe menu would
cost if ordered à la carte off the regular menu, the resulting discount
percentage, whether the deal is a genuine value or a shrunk/watered-down
version of the regular experience, and an overall score that blends the
discount with the restaurant's reputation. It ends in ranked leaderboards
(best overall, biggest discount, best Michelin-rated value, best per
cuisine) that a reader can act on immediately.

The output is only useful if the prices are real. The single biggest risk in
this workflow is silently inventing numbers to fill gaps — treat every price
as something that must be traced to a source, and treat every estimate as
something that must be labeled as an estimate.

## When to reach for this

Trigger on requests like: "帮我做一份芝加哥Restaurant Week性价比排行榜",
"rank NYC Restaurant Week by value for money", "is Dine Out Vancouver
actually a good deal", "compare these prix-fixe menus to the regular menu
prices", or any ask to quantify how much a fixed-price dining promotion
actually saves versus ordering normally. This applies to any city with an
organized restaurant-week-style program (NYC, Chicago, Boston, LA, SF, DC,
Vancouver, London's set menus, etc.) — the source website differs but the
method below is the same.

## Before doing any research: scope the sample with the user

Restaurant Week programs commonly have several hundred participants, and
many restaurants don't publish enough pricing detail to verify anything.
Building a fully rigorous per-dish comparison for every participant is not
realistic in one pass. Ask the user (via a clarifying-question tool if
available, otherwise just ask directly) before starting heavy research:

1. **Sample size** — a curated 30-50, a mid-size ~100, or "as many as
   possible"? Bigger samples mean either less depth per restaurant or much
   longer runtime — say so explicitly so the user can choose with eyes open.
2. **Geographic/cuisine scope** — one borough/neighborhood cluster, or
   citywide? Which cuisines must be represented (the user may want
   coverage across Italian, French, Japanese, steakhouse, etc.)?
3. **What to do when official pricing can't be verified** — exclude the
   restaurant entirely (higher rigor, smaller sample), or allow
   category-level estimates clearly labeled as such (bigger sample, some
   estimated cells)? Both are legitimate; the user should pick.

Don't skip this conversation even if the user's request sounds like it
wants "everything" — a request for full coverage almost always turns out to
mean "as much as is practical," and confirming saves you from doing (or
under-doing) the wrong amount of work.

## Phase 1: Find the official participant list

Restaurant week sites are almost always JavaScript-rendered client apps, so
a plain fetch tool will return an empty page shell — you need a real
browser tool (Chrome MCP or equivalent) to see the actual restaurant list.
Steps that worked well on nyctourism.com/restaurant-week and generalize to
similar sites:

1. Navigate to the official page and scroll to the restaurant browser /
   listing section — filters are usually below the hero banner.
2. Look for filter checkboxes: borough/area, cuisine, price tier, and
   critically a **"Has Menu"** filter — turn this on, since a restaurant
   without a published menu can't be analyzed anyway.
3. Filters are almost always client-side state, not URL query params —
   don't assume you can deep-link a filtered view; you have to click the
   checkboxes each time you load the page.
4. Use a "find element by natural language" tool (e.g. `find` in Chrome
   MCP) with a query like "restaurant name heading in result card" to pull
   many restaurant names/cuisines/neighborhoods out of the page at once —
   this is much faster than reading the full accessibility tree or taking
   screenshots of every page of results.
5. Each restaurant card usually links to: a PDF of the actual prix-fixe
   menu (often hosted on a CDN/S3 bucket — this is your primary source for
   Phase 2), a details/"learn more" page on the official site, and the
   restaurant's own website. Grab all three links per restaurant you plan
   to include.

If the sample needs to hit specific cuisines, apply the cuisine filter one
category at a time rather than paging through the unfiltered list —
that turns "page through 30+ pages of everything" into "page through 2-3
pages per required cuisine."

**If a menu PDF won't render, don't burn the budget retrying.** Some PDFs
(especially ones hosted on third-party blog CDNs rather than the official
site's own storage) come back blank in browser tools — empty canvas, "no
text content," repeated screenshot failures. This is a known environment
limitation, not a sign you're doing something wrong. After one or two
retries, drop to a secondary source instead of looping: a food blog or
local news write-up that transcribes the menu, or the restaurant's own
social media post announcing it. Cite it as a secondary source in the
notes column rather than pretending it came from the official PDF.

## Phase 2: Per-restaurant analysis (the core loop)

For every restaurant in the sample, work through these steps. This is the
part of the task that is naturally parallelizable — if you have subagents
available, splitting the restaurant list into batches and researching them
concurrently is far faster than doing it serially, and previous runs of
this workflow show a single restaurant done properly (PDF read, regular
menu found, dishes mapped, reputation checked) takes real research effort,
not a quick lookup.

**Step 1 — Extract the prix-fixe menu.** From the PDF or details page,
record: which meal periods it's offered for (lunch/dinner/weekend brunch),
the price, every course, and every choice offered within each course.

**Step 2 — Map every dish to the regular menu.** Go to the restaurant's own
website (or a reliable secondary source — the restaurant's Resy/Toast
online menu, a recent menu mentioned in press coverage) and find what each
dish costs as a regular à la carte order. When no identically-named dish
exists, use the closest equivalent and say so explicitly — "no exact match;
priced against the closest comparable dish because X" is a valid answer,
silently guessing is not.

Many restaurant websites — especially smaller or newer ones — are built on
Wix/Squarespace-style JS-rendered platforms that a plain fetch can't read
and that don't reliably respond to browser navigation either. When the
official site is a dead end, menu-aggregator sites (e.g. sagemenu.com,
menustic.com, or similar) and food-media write-ups that quote actual prices
are legitimate secondary sources — often more reliable in practice than the
restaurant's own site. Just say which kind of source each price came from.

If the prix-fixe menu offers choices within a course (e.g. "pick one of
three starters"), price the combination whose prices are *all independently
verifiable*, even if it isn't the most expensive or most-discounted
combination available. Reaching for the biggest-looking discount when some
of the underlying prices are unconfirmed undermines the whole point of the
exercise — a slightly smaller but fully-sourced discount is more useful
than a bigger but partly-guessed one.

**Step 3 — Compute the value.** Sum the regular-menu price of every course
in the prix-fixe menu to get the Normal Value, then:

```
Discount % = (Normal Value − Prix-Fixe Price) / Normal Value
```

**Step 4 — Handle bundled/sampler items honestly.** Dessert flights,
"chef's sampler" plates, or promotion-exclusive dishes that don't exist on
the regular menu can't be priced by direct lookup. Estimate a reasonable
value (e.g., a fraction of what a comparable full-size dessert costs, or
the average of the restaurant's other dessert prices) and state the
estimation method in the notes — never present an estimate as if it were a
looked-up price.

**Step 5 — Check for portion shrinkage.** If both menus specify portion
size (e.g. a steak's weight) and the promotion version is smaller, record
the reduction. If portion size isn't stated on either menu, say so and
assume parity rather than guessing a number.

**Step 6 — Check for signature dishes.** Look at what the restaurant, or
outlets like Eater/The Infatuation/Time Out/the Michelin Guide, call out as
must-order or chef-recommended, and note whether the promotional menu
actually includes any of those dishes. A promotion built entirely from
menu also-rans should score lower than one that includes the reason people
go to that restaurant in the first place.

**Reputation data** — for each restaurant, capture Michelin status
(star/Bib Gourmand/Selected/none), and note (with links) any coverage from
Eater, The Infatuation, Time Out, and similar outlets. If you can search
Google but don't have a way to pull a live ratings API, say so plainly
rather than presenting a made-up star rating.

## Phase 3: Scoring and rankings

Use a transparent, explainable weighting rather than a black-box number.
A formula that has worked well: **40% discount size, 35% reputation
quality, 15% whether a signature dish is included, 10% portion integrity**.
Whatever weights you use, write them down once and apply them consistently
so restaurants can be compared apples-to-apples, and show the reasoning
per restaurant in a comments/notes column rather than just a bare number.

Also produce a **"most reasonable order"** per restaurant: not the most
expensive combination, not the cheapest, but the choice a typical diner
optimizing for both value and dish quality would actually pick — and say
which dishes and why.

Build these leaderboards from the full dataset (recompute across the whole
sample, not just newly-added restaurants if extending prior work):

- Top N Overall (by score)
- Top N Biggest Discount (by discount %)
- Best Michelin-rated Value (restaurants with any Michelin recognition,
  ranked by value)
- Best by Cuisine (top pick per required cuisine category)

## Phase 4: Build the spreadsheet

Do the research first. Only once you have real data in hand, invoke this
environment's `xlsx` skill to learn the correct way to build/edit the
workbook — reading the xlsx skill before you have data to put in it just
anchors you on formatting instead of substance.

Structure:
- **All Restaurants** sheet: one row per restaurant, with the prix-fixe
  menu items, their regular-menu price equivalents, Normal Value, Discount
  % (as a live formula, not a hard-coded number, so edits propagate),
  Overall Score (also a live formula), signature-dish flag, portion-shrink
  flag, most-reasonable-order text, a plain-language "worth going?"
  verdict, and — this is important — a source/assumption column on every
  row stating exactly where each price came from and flagging any
  estimated values.
- One sheet per leaderboard (Top Overall, Top Discount, Michelin Best
  Value, Best by Cuisine), built with formulas that reference the main
  sheet so they stay correct if the main sheet is edited or extended later.

Before considering it done, spot-check a meaningful fraction of rows (more
if a lot of the data is estimated rather than sourced) by re-opening the
original PDF/website and confirming the numbers in the sheet actually
match. Report what you checked and what you found.

## Common failure modes to avoid

- **Filling gaps with plausible-sounding invented numbers.** If a
  restaurant's pricing genuinely can't be found, either exclude it (if the
  user asked for strict verification) or estimate transparently with the
  method stated (if the user has accepted looser rigor) — never present a
  guess as a fact.
- **Comparing a promotional menu to a restaurant's most expensive normal
  offering** (e.g. comparing a 3-course prix-fixe to a $300 tasting menu at
  a Michelin-starred restaurant) without flagging that the comparison
  is apples-to-oranges — this produces a misleadingly huge "discount"
  number. Call this out explicitly wherever it happens.
- **Hard-coding computed values.** Discounts and scores should be formulas
  in the spreadsheet so the workbook stays internally consistent if a
  price gets corrected later.
- **Treating "cover the whole city" literally.** Confirm actual scope
  (see "Before doing any research" above) rather than assuming the user
  wants a multi-hour, hundreds-of-restaurants research run.

## A negative discount is a valid, useful result

Not every prix-fixe deal is actually a deal. A past run of this workflow
found a restaurant whose promotional menu priced out *more expensive* than
ordering the same dishes à la carte (-7% "discount"). Don't second-guess a
result like this or quietly drop the restaurant — reporting "this one isn't
actually worth it" is exactly the kind of finding that makes the whole
exercise worth doing, and it's evidence the analysis is tracking real
prices rather than assuming every promotion is a bargain by default.
