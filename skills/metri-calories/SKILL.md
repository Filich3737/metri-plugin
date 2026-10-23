---
name: metri-calories
description: Use when the user wants to learn how metri works; log food or drink from freeform text, speech, or food images; review or correct diary history; check calories or macros; manage nutrition goals; or save a personal food or cooked batch. Trigger even without an explicit logging request or mention of metri.
---

# metri food diary

Turn freeform food input into a dietary log of calories, protein, carbohydrates, and fat. Input may be text, a speech transcript, a food name, a meal photo, a package or label photo, or any combination.

Interpret the evidence and resolve foods, quantities, and nutrition before calling metri. Use the live MCP tool descriptions as the source of truth for exact inputs and outputs.

## Explain metri

When the user asks how metri works, what it can do, or how to get started, give a concise metri 101 without calling a tool merely to demonstrate the product:

- Set a daily calorie goal and, optionally, protein, carbohydrate, and fat targets.
- Log naturally by typing or dictating what they ate, or by sharing one or more meal, package, or nutrition-label photos. A single request may contain several foods.
- Explain that the assistant interprets portions and nutrition using labels, web research, the user's saved foods and diary history, or a transparent estimate; metri privately stores the resulting structured diary.
- Mention that the user can review any period, see today's progress, reuse personal foods and cooked batches, and correct or delete past entries.

End with one simple next step: ask the user to share their daily goal or tell or show what they ate. Keep this explanation compact unless they ask for more detail.

## Decide whether to write

First decide whether the user wants a diary change at all. Record when they say or reasonably imply that they consumed food or drink; a meal, wrapper, package, or label image can communicate consumption without an explicit command.

Do not record something that is hypothetical, planned, part of a general nutrition question, shown only for identification, or mentioned while reviewing an existing entry. Answer or read the diary instead. If intent is genuinely ambiguous, ask whether they want it logged before writing.

## Reconstruct the eating event

Treat the conversation and all attached images as one evidence set, infer how the pieces relate, and reconstruct what the user most likely consumed.

- A meal photo followed by a cheese label usually means that cheese was used in the meal; the label is evidence, not another consumption.
- Several photos of a plate, packages, and labels usually describe one eating event unless the user separates them.
- Do not add a component the user explicitly ruled out.

Search with `metri_search_foods` when the wording points to something personal or reusable that may be in the user's private library, such as "my regular chicken curry," "the pasta I cooked yesterday," a saved cooked batch, or a specific saved product. A generic food such as an apple does not by itself require a library search merely because the user may have eaten one before. Omit the query only when the user wants to browse their library.

Before estimating leftovers or "the rest," check personal state that can determine the amount. Combine the package size from the label or the saved product's name with `metri_get_diary`; for example, 100 g already logged from a saved "Skyr, 450 g tub" means the remainder is 350 g.

Ask one targeted question only when a single missing fact would materially change the result; otherwise make a reasonable estimate rather than asking for structured input.

## Record nutrition data

New non-batch nutrition is per 100 g. Store kcal; if only kJ is shown, use `kcal = kJ / 4.184`. When a label is available, use the package's stated net weight and serving size rather than a remembered standard, and convert serving nutrition to per 100 g when necessary.

Wherever a tool takes a food, pass either the `foodId` of a saved food or an inline food with `name` and `nutritionPer100g`. An inline food is stored only where it is used, in that diary item or that batch, and never enters the library, so do not save a one-off food with `metri_save_food` first. The library gains foods only through `metri_save_food` and `metri_save_batch`.

## Log one eating event

Call `metri_log_consumption` once with every component consumed in the same occasion. Log recognizable components as separate items so their grams and nutrition remain visible. Supporting packages and labels never become separate eating events. The returned dashboard lets the attached card paint without another tool call.

Always pass the required event `name`. Preserve a meal name supplied by the user. Otherwise create a short, natural name from the consumed foods in the user's language; for a single item, reuse its food name. Do not ask a separate question merely to obtain a name. A later quantity or item correction keeps the saved event name unless the user explicitly asks to rename it.

When the user counts in units, such as 2 cups or 1 medium apple, pass `serving` with that amount and unit alongside the grams; it is what the card shows and what later questions about servings are answered from.

Omit `consumedAt` for food being eaten now. For a past or explicitly timed event, give the user's local time as `YYYY-MM-DDTHH:mm`; metri reads it in the user's time zone.

## Batch rules

A saved batch represents one concrete cooked pot because metri tracks how much of it remains. A new preparation is a new batch. When same-name batches exist, match the user's time reference against their immutable `createdAt` values.

Reference a component by `foodId` when it is already a saved non-batch food in the library; otherwise describe it inline with `name`, `grams`, and `nutritionPer100g`. To change a saved batch's components, read the batch with `metri_search_foods` by `foodId`, change what the user corrected, and send the whole list back; to cook the same dish again, send that list as a new batch.

Updating a batch corrects it: every entry already logged from it follows the corrected composition, and a corrected finished weight recomputes entries logged in grams.

`fractionOfBatch` always means a fraction of the whole batch. If `consumedFractionOfBatch` is 0.4, 0.6 remains, and when the user eats half of what remains, log 0.3. Grams of a batch are stored as its fraction of the finished weight. Do not invent a finished batch weight; a batch without one must be logged by batch fraction rather than grams.

## Read and manage diary state

- For today, yesterday, or another period, give `metri_get_diary` the user's first and last calendar day, both included.
- Use `metri_show_today` for a visual card of today's calories, macros, allowance, or progress. Do not call app-only `metri_today_details` to answer in chat.
- Use `metri_set_daily_goal` for calorie and optional macro targets. Do not invent macro targets; omitted targets stay unchanged and `null` clears one.
- Use `metri_edit_consumption` with absolute corrected quantities. Quantity edits retain a food item's frozen historical nutrition basis; a batch item follows its batch. To correct the basis itself, remove the wrong item and add the corrected food in the same edit, leaving unrelated items untouched.
- Use `metri_delete_consumption` only to remove the entire eating event.
- Use `metri_save_food` for reusable non-batch data without consumption.
- Use `metri_save_batch` for cooked batches. Supplied components replace the whole component list; omitted components are preserved. Values supplied with a component's `foodId` override the library food's.
- Use `metri_delete_food` to remove a food or batch from the active private library. Historical diary snapshots remain.

After `metri_log_consumption`, `metri_edit_consumption`, or `metri_delete_consumption`, the attached card refreshes today's progress itself. Do not call `metri_show_today` merely to repeat the post-write summary.

## Report a newly logged meal

After `metri_log_consumption` succeeds, report what was written in the user's language. Use the returned totals and per-item nutrition, which comes back in the order you sent the items—not internal IDs or stray interface text—and follow the card's column order: Protein, Carbs, Fat, Calories.

For a multi-item event, use this compact shape and localize the labels:

| Logged: [event] | Protein | Carbs | Fat | Calories |
| --- | ---: | ---: | ---: | ---: |
| **Total** | [P] g | [C] g | [F] g | [kcal] kcal |
| [component], [amount] | [P] g | [C] g | [F] g | [kcal] kcal |

Add one component row per logged item. For a single-item event, show one row naming the food rather than duplicating a total and an identical component row. Round consistently; displayed component columns must reconcile with the displayed totals.

Confirm edits, deletions, saved foods, batches, and goal changes briefly without the meal table.

## Non-obvious failure modes

- Never log the same eating event again merely because the user later mentions or reviews it.
- Do not convert loose wording such as "about half and half" into fabricated exact macro percentages.
- Treat a manufacturer's serving size as specific to that product.
- Do not try to fix a wrong historical nutrition basis by changing only the quantity.
