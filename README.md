# restaurant-week-value-skill

A Claude Skill (SKILL.md) for researching a city's Restaurant Week - or any prix-fixe / tasting-menu dining promotion - and building a verifiable value-for-money ranking of participating restaurants.

## What it does

For a chosen sample of restaurants participating in a Restaurant Week program, this skill drives a workflow that:

- Finds the official participant list and each restaurant's prix-fixe menu.
- Maps every dish on the prix-fixe menu to its regular a la carte price.
- Computes the discount percentage, flags portion shrinkage, and checks whether the promotion includes the restaurant's signature dishes.
- Gathers reputation signals (Michelin status, Eater, The Infatuation, Time Out, etc.), with sources cited.
- Scores every restaurant with a transparent, explainable formula and produces ranked leaderboards (best overall, biggest discount, best Michelin-rated value, best by cuisine).
- Delivers everything as an Excel workbook with live formulas and a source/assumption column on every price, so nothing is silently invented.

## How to use it

Copy this repository's restaurant-week-value/ folder into your Claude Skills directory, or install the packaged .skill file directly. Then ask Claude something like:

"Build a Restaurant Week value-for-money ranking for Chicago, about 20 restaurants, covering a mix of cuisines."

See restaurant-week-value/SKILL.md for the full methodology, including how it handles missing pricing data, choice-based menus, and bundled/sampler items.

## Origin

This skill was extracted from a real research workflow originally built to rank 2026 NYC Summer Restaurant Week, then generalized and validated against a second city (Chicago) to confirm it isn't NYC-specific.
