# AGENTS.md

Instructions for AI coding agents working in this repository.

## Goal
Convert a nutrition-plan PDF into a valid JSON payload consumed by the Nutrium app.

## Scope
- Input: one PDF file containing plan meals and substitution lists.
- Output: one JSON file (`plan.json`) that matches the API contract.
- Do not build UI, backend, or analytics.

## Required Output Contract
Always generate exactly these top-level fields:
- `schemaVersion` (must be `1`)
- `planVersion` (integer `> 0`)
- `id` (stable plan id, non-empty)
- `name` (non-empty)
- `updatedAt` (ISO date: `YYYY-MM-DD`)
- `meals` (array, at least 1 item)
- `substitutionGroups` (array)

### Meals
Each meal must contain:
- `id` (stable)
- `name`
- `order` (0-based)
- `startHour` (0-23)
- `endHour` (`> startHour`)
- `items` (array)

Each item must contain:
- `id` (stable)
- `order` (0-based)
- `label`
- `displayName`
- `displayQuantity`
- `isUnlimited` (`true` only for "À vontade")
- `groupId` (nullable; if present, must exist in `substitutionGroups[].id`)

### Substitution Groups
Each group must contain:
- `id` (e.g. `lista_01`)
- `listNumber` (integer)
- `name`
- `portions` (array)

Each portion must contain:
- `id` (stable)
- `foodName`
- `quantity`

## Stability Rules (Critical)
Never change existing IDs unless the underlying semantic entity truly changed.
- Keep `plan.id` stable across updates.
- Keep `meals[].id` stable.
- Keep `items[].id` stable.
- Keep `portions[].id` stable.

Changing IDs breaks user history/selection mapping.

## ID Convention
Use deterministic slug IDs:
- meal: `meal_<slug>` (e.g. `meal_cafe_manha`)
- item: `item_<meal_slug>_<slot_slug>`
- group: `lista_XX`
- portion: `food_lXX_<slug>`

Rules:
- lowercase
- ASCII only
- replace spaces with `_`
- remove punctuation and accents

## Extraction Rules from PDF
- Preserve original Portuguese meal and food names in `name`/`displayName`/`foodName`.
- Preserve quantities exactly as human-readable text.
- If the PDF says "À vontade", set `isUnlimited=true` and keep quantity text "À vontade".
- If a meal item references a list (e.g. "Lista 4"), map to corresponding `groupId` (e.g. `lista_04`).
- If no substitution list applies, use `groupId: null`.

## Defaults When PDF Omits Meal Time
Use these defaults unless explicit times are present in the PDF:
- Café da manhã: 7-9
- Lanche da manhã: 10-11
- Almoço: 12-14
- Lanche da tarde: 16-18
- Jantar: 19-22

## Validation Checklist (Must Pass)
Before finalizing `plan.json`, verify:
1. JSON parses.
2. `schemaVersion == 1`.
3. `planVersion > 0`.
4. Required fields exist and are non-empty when applicable.
5. `meals.length >= 1`.
6. Every `startHour/endHour` range is valid.
7. Every non-null `items[].groupId` exists in substitution groups.
8. No duplicate IDs in meals/items/groups/portions.

## Commands
Use these commands when available:
```bash
pdftotext <input.pdf> -
jq . plan.json
```

## Output Rules
- Update only `plan.json` unless explicitly requested otherwise.
- Do not output markdown as the final artifact.
- Final artifact must be raw JSON content in `plan.json`.
