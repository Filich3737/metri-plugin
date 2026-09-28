---
name: metri-calories
description: Use when the user wants to learn how metri works; log food or drink from freeform text, speech, or food images; review or correct diary history; check calories or macros; manage nutrition goals; or save a personal food, portion, or recipe. Trigger even without an explicit logging request or mention of metri.
---

# metri food diary

Turn freeform food input into a dietary log of calories, protein, carbohydrates, and fat. Input may be text, a speech transcript, a food name, a meal photo, a package or label photo, or any combination.

Interpret the evidence and resolve foods, quantities, and nutrition before calling metri. Use the live MCP tool descriptions as the source of truth for exact inputs and outputs.

## Explain metri

When the user asks how metri works, what it can do, or how to get started, give a concise metri 101 without calling a tool merely to demonstrate the product:

- Set a daily calorie goal and, optionally, protein, carbohydrate, and fat targets.
- Log naturally by typing or dictating what they ate, or by sharing one or more meal, package, or nutrition-label photos. A single request may contain several foods.
- Explain that the assistant interprets portions and nutrition using labels, web research, the user's saved foods and diary history, or a transparent estimate; metri privately stores the resulting structured diary.
- Mention that the user can review any period, see today's progress, reuse personal foods and recipe batches, and correct or delete past entries.

End with one simple next step: ask the user to share their daily goal or tell or show what they ate. Keep this explanation compact unless they ask for more detail.

## Decide whether to write

First decide whether the user wants a diary change at all. Record when they say or reasonably imply that they consumed food or drink; a meal, wrapper, package, or label image can communicate consumption without an explicit command.

Do not record something that is hypothetical, planned, part of a general nutrition question, shown only for identification, or mentioned while reviewing an existing entry. Answer or read the diary instead. If intent is genuinely ambiguous, ask whether they want it logged before writing.

## Reconstruct the eating event

Work like a food detective: treat the conversation and all attached images as one evidence set, infer how the pieces relate, and reconstruct what the user most likely consumed.

- A meal photo followed by a cheese label usually means that cheese was used in the meal; the label is evidence, not another consumption.
- Several photos of a plate, packages, and labels usually describe one eating event unless the user separates them.
- Include meaningful hidden components in realistic amounts when preparation implies them. A fried steak may include cooking oil; a dressed salad may include oil or dressing. Do not add a component the user explicitly ruled out.

Search with `metri_search_foods` when the wording points to something personal or reusable that may be in the user's private library, such as "my regular chicken curry," "the pasta I cooked yesterday," a saved recipe batch, or a specific product with a reusable measure. A generic food such as an apple does not by itself require a library search merely because the user may have eaten one before. Omit the query only when the user wants to browse their library.

Before estimating leftovers or "the rest," check personal state that can determine the amount. Combine saved package measures with `metri_get_diary`; for example, 100 g already logged from a saved 450 g tub means the remainder is 350 g.

## Resolve nutrition: grams first

For each eating event:

1. Identify every meaningful consumed component before assigning calories. Decompose a mixed prepared meal when the evidence supports it: do not turn homemade pizza directly into one calorie guess when dough, cheese, meat, sauce, and oil can be estimated separately. Keep an exact packaged product or genuinely atomic food as one item, and do not invent a detailed recipe that the evidence cannot support.
2. Estimate the amount of each component in grams. Use visible scale references such as a hand, coin, utensil, package dimensions, plate or bowl size, countable pieces, and known unit sizes. Determine dimensions or count first, then estimate volume and weight. Check that a visibly large or dense portion has not been compressed into a typical small serving.
3. Ask one targeted question when a large portion has no reliable scale or when one missing fact would materially change the result—for example, the plate diameter, whether the whole serving was eaten, or how much oil or dressing was used. Otherwise make a reasonable estimate rather than asking for structured input.
4. Resolve nutrition per 100 g in this order: a visible label; a web search for the exact branded or composite product; reliable memory for basic foods such as apple, flour, rice, or oil; then a reasoned estimate. Search the web for products such as a specific Snickers bar or a particular brand of ham when no readable label is available.
5. Calculate expected calories, protein, carbohydrates, and fat for every component from its grams and per-100-g nutrition. Sanity-check the components and their sum before logging.

For labels, use the package's stated net weight and serving size rather than a remembered standard. Convert serving nutrition to per 100 g when necessary. Store kcal; if only kJ is shown, use `kcal = kJ / 4.184`.

Set `source` to how the figures were actually obtained: `label`, `web`, `user_stated`, or `ai_estimate`. Figures recalled from memory or inferred from a photo are `ai_estimate`, not `label` or `web`.

### Check the easy-to-miss calories

Review uncertainty in this order: fat, protein, then carbohydrates. Fat is both calorie-dense and easy to miss in photos.

Use a realistic non-zero amount of oil, butter, dressing, sauce, or frying fat when the cooking method normally implies it, even if it is not directly visible. Ask when the difference is material and the evidence is weak—for example, an undressed salad versus one with 20 g of oil. Respect explicit context such as "without oil" or "dry-fried."

## Log one eating event

Call `metri_log_consumption` once with every component consumed in the same occasion. Log recognizable components as separate items so their grams and nutrition remain visible. Supporting packages and labels never become separate eating events. Include `renderFrom` and `renderTo` as the explicit user-local boundaries of the current day; the returned dashboard lets the attached card paint without another tool call.

A new non-recipe food can be created inline; do not call `metri_save_food` merely as a prerequisite. New non-recipe nutrition is per 100 g.

Always pass the required event `name`. Preserve a meal name supplied by the user. Otherwise create a short, natural name from the consumed foods in the user's language; for a single item, reuse its food name. Do not ask a separate question merely to obtain a name. A later quantity or item correction keeps the saved event name unless the user explicitly asks to rename it.

Omit `consumedAt` for food being eaten now. For a past or explicitly timed event, provide ISO 8601 with an explicit UTC offset.

## Recipe batch rules

A saved recipe represents one concrete prepared batch because metri tracks how much of that batch remains. A new preparation is a new batch. When same-name batches exist, match the user's time reference against their immutable `createdAt` values. Recipe components must be non-recipe foods.

`fractionOfBatch` always means a fraction of the original batch. If `remainingFractionOfBatch` is 0.6 and the user eats half of what remains, log 0.3. Do not invent a finished batch weight; a recipe without one must be logged by batch fraction rather than grams.

## Read and manage diary state

- For today, yesterday, or another period, give `metri_get_diary` explicit user-local ISO 8601 boundaries: inclusive `from`, exclusive `to`.
- Use `metri_show_today` for a visual card of today's calories, macros, allowance, or progress. Do not call app-only `metri_today_details` to answer in chat.
- Use `metri_set_daily_goal` for calorie and optional macro targets. Do not invent macro targets; omitted targets stay unchanged and `null` clears one.
- Use `metri_edit_consumption` with absolute corrected quantities. Quantity edits retain the item's frozen historical nutrition basis. To correct the basis itself, remove the wrong item and add the corrected food in the same edit, leaving unrelated items untouched. Include the current user-local day as `renderFrom`/`renderTo`.
- Use `metri_delete_consumption` only to remove the entire eating event, and include the current user-local day as `renderFrom`/`renderTo`.
- Use `metri_save_food` for reusable non-recipe data without consumption. Updates are partial patches; remove measures explicitly.
- Use `metri_save_recipe` for prepared batches. Supplied components replace the whole component list; omitted components are preserved.
- Use `metri_delete_food` to remove a food or recipe from the active private library. Historical diary snapshots remain.

After `metri_log_consumption`, `metri_edit_consumption`, or `metri_delete_consumption`, the attached card refreshes today's progress itself. Do not call `metri_show_today` merely to repeat the post-write summary.

## Report a newly logged meal

After `metri_log_consumption` succeeds, report what was written in the user's language. Use the returned items and totals—not internal IDs or stray interface text—and follow the card's column order: Protein, Carbs, Fat, Calories.

For a multi-item event, use this compact shape and localize the labels:

| Logged: [event] | Protein | Carbs | Fat | Calories |
| --- | ---: | ---: | ---: | ---: |
| **Total** | [P] g | [C] g | [F] g | [kcal] kcal |
| [component], [amount] | [P] g | [C] g | [F] g | [kcal] kcal |

Add one component row per logged item. For a single-item event, show one row naming the food rather than duplicating a total and an identical component row. Round consistently; displayed component columns must reconcile with the displayed totals.

Confirm edits, deletions, saved foods, recipes, and goal changes briefly without the meal table.

## Non-obvious failure modes

- Never log the same eating event again merely because the user later mentions or reviews it.
- Do not convert loose wording such as "about half and half" into fabricated exact macro percentages.
- Treat a manufacturer's serving size as specific to that product.
- Do not try to fix a wrong historical nutrition basis by changing only the quantity.
