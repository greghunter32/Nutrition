# CLAUDE.md — Nutrition Calculator

This file documents the codebase structure, conventions, and development workflows for AI assistants working on this repository.

---

## Project Overview

A client-side **Personal Nutrition Calculator** deployed as a static site on GitHub Pages. Users build recipes by searching for foods, specifying quantities and units, and receive per-serving macronutrient and micronutrient breakdowns.

- **Live URL:** https://greghunter32.github.io/Nutrition/
- **Repository:** https://github.com/greghunter32/Nutrition
- **Deployment:** GitHub Pages (automatic on push to `master`)

---

## Repository Structure

```
Nutrition/
├── index.html    # Entire application: HTML structure, embedded CSS, embedded JS
└── README.md     # User-facing deployment and update instructions
```

This is a **single-file application**. All HTML, CSS, and JavaScript live inside `index.html` (656 lines). There is no build step, no package manager, and no external dependencies.

---

## Technology Stack

| Layer       | Technology                        |
|-------------|-----------------------------------|
| Markup      | HTML5                             |
| Styling     | CSS3 (embedded `<style>` block)   |
| Logic       | Vanilla JavaScript ES6+ (embedded `<script>` block) |
| Deployment  | GitHub Pages (static hosting)     |
| Build tools | None                              |
| Dependencies| None                              |

---

## File Layout: `index.html`

The file is divided into three logical sections:

| Lines     | Content                        |
|-----------|-------------------------------|
| 1–246     | `<head>`: metadata + all CSS  |
| 248–320   | `<body>`: HTML structure      |
| 322–655   | `<script>`: all JavaScript    |

---

## JavaScript Architecture

### Global State

```javascript
let currentRecipe = [];  // Array of ingredient objects for the active recipe
```

`currentRecipe` is the single source of truth. All UI updates derive from it.

### Ingredient Object Schema

Each element of `currentRecipe` has this shape:

```javascript
{
    food: 'chicken breast',           // Key into foodDatabase
    name: 'Chicken Breast (skinless)',// Display name from foodDatabase
    quantity: 200,                    // Raw user-entered amount
    unit: 'g',                        // Selected unit string
    grams: 200                        // Calculated weight: quantity * unitConversions[unit]
}
```

### Core Functions

| Function | Purpose |
|---|---|
| `setupFoodSearch()` | Attaches `input` and document `click` listeners for the autocomplete dropdown; called once on init |
| `selectFood(foodKey)` | Populates the search input and stores the selected `foodKey` in `dataset.foodKey` |
| `addIngredient()` | Reads search input, quantity, and unit; validates; pushes to `currentRecipe`; clears inputs |
| `removeIngredient(index)` | Splices `currentRecipe` at `index` and re-renders the list and nutrition panel |
| `updateIngredientsList()` | Re-renders the `#ingredientsList` DOM element from `currentRecipe` |
| `calculateNutrition()` | Sums nutrition across all ingredients (per-100g × grams/100), divides by servings, writes to DOM |
| `clearRecipe()` | Resets `currentRecipe` to `[]` and clears both display panels |

### Initialization (bottom of `<script>`)

```javascript
document.getElementById('servings').addEventListener('input', calculateNutrition);
setupFoodSearch();
calculateNutrition();
```

---

## Food Database

`foodDatabase` is a `const` object keyed by lowercase food name strings. All nutritional values are **per 100 grams**.

### Nutrient Fields

```javascript
{
    name: string,      // Human-readable display name
    calories: number,  // kcal
    protein: number,   // grams
    carbs: number,     // grams
    fat: number,       // grams
    fiber: number,     // grams
    calcium: number,   // mg
    iron: number,      // mg
    vitaminC: number,  // mg
    vitaminA: number   // IU
}
```

### Food Categories (25 entries)

| Category       | Keys |
|----------------|------|
| Proteins       | `chicken breast`, `salmon`, `eggs`, `tofu` |
| Carbohydrates  | `rice`, `quinoa`, `oats`, `bread` |
| Vegetables     | `broccoli`, `spinach`, `carrots`, `tomatoes`, `bell peppers` |
| Fruits         | `banana`, `apple`, `orange`, `blueberries` |
| Dairy          | `milk`, `greek yogurt`, `cheese` |
| Nuts & Seeds   | `almonds`, `walnuts`, `chia seeds` |
| Fats & Oils    | `olive oil`, `avocado` |

### Adding a New Food

Add an entry to `foodDatabase` following the exact schema above. Keys must be **lowercase** strings (used in search matching and as `ingredient.food` identifiers).

---

## Unit Conversion System

`unitConversions` maps unit strings to a gram multiplier:

```javascript
const unitConversions = {
    'g': 1,
    'kg': 1000,
    'oz': 28.35,
    'lb': 453.59,
    'cup': 240,    // approximate; assumes water-like density
    'tbsp': 15,
    'tsp': 5,
    'ml': 1,       // approximate; assumes water-like density
    'l': 1000,
    'piece': 100   // default 100g per piece
};
```

The `<select id="unit">` options must stay in sync with these keys.

---

## Nutrition Calculation Logic

```
ingredient_grams = quantity × unitConversions[unit]
ratio            = ingredient_grams / 100
nutrient_total  += food[nutrient] × ratio   (summed over all ingredients)
per_serving      = nutrient_total / servings
```

Calories are rounded to the nearest integer; all others use `.toFixed(1)` (macros) or `Math.round` (micros).

---

## CSS Conventions

- **Box model:** `box-sizing: border-box` applied globally via `* { ... }`.
- **Layout:** CSS Grid for the two-column main content (`grid-template-columns: 1fr 1fr`); collapses to single column below 768px.
- **Color palette:**
  - Body background gradient: `#667eea` → `#764ba2`
  - Header gradient: `#4facfe` → `#00f2fe`
  - Buttons (primary): `#4facfe` → `#00f2fe`
  - Buttons (danger/remove): `#ff6b6b` → `#ee5a24`
  - Per-serving panel gradient: `#a8edea` → `#fed6e3`
  - Recipe section background: `#f8f9fa`
  - Nutrition section background: `#e8f5e8`
- **Typography:** `'Segoe UI', Tahoma, Geneva, Verdana, sans-serif`
- **Transitions:** `all 0.3s ease` on interactive elements; buttons lift with `translateY(-2px)` on hover.
- **Responsive breakpoint:** `@media (max-width: 768px)` — ingredient input and recipe info switch to single-column.

---

## HTML Conventions

- IDs used for JavaScript access: `foodSearch`, `searchResults`, `quantity`, `unit`, `recipeName`, `servings`, `ingredientsList`, `totalServings`, `macronutrients`, `micronutrients`.
- `onclick` attributes are used directly on buttons (inline handlers), not `addEventListener`.
- Food key for a selected item is stored in `document.getElementById('foodSearch').dataset.foodKey`.

---

## Development Workflow

### Making Changes

1. Edit `index.html` directly — there is no build step.
2. Open the file in a browser to test (or use a local HTTP server if needed).
3. Commit and push to `master` to trigger GitHub Pages deployment.

### Deployment

GitHub Pages serves the `master` branch root automatically. After pushing:
- Pages typically rebuild within 1–2 minutes.
- Hard-refresh the live URL (`Ctrl+F5`) to bypass caching.
- Check **Actions → pages build and deployment** for build status.

### No Testing Framework

There are no automated tests. Validation is manual in-browser. When making changes, verify:
- The food search autocomplete shows results after typing 2+ characters.
- Adding an ingredient updates the ingredients list correctly.
- Removing an ingredient recalculates nutrition immediately.
- Changing the servings input updates the per-serving nutrition in real time.
- The layout is usable on a narrow (mobile) viewport.

---

## Key Constraints and Gotchas

- **All nutrition data is per 100g.** Never store per-serving or raw values in `foodDatabase`.
- **Unit conversions are density-approximate.** `cup`, `ml`, and `l` assume water density (1g/ml). Volume measurements for dense or light foods will be inaccurate — this is a known limitation.
- **`piece` unit defaults to 100g.** This works reasonably for many foods but is a rough approximation.
- **Search matching is substring-based** (case-insensitive) across `foodDatabase` keys, limited to 8 results.
- **Food key must be stored in `dataset.foodKey`** before `addIngredient()` is called, or the add will fail silently with an alert.
- **No persistence.** The recipe state is in-memory only and is lost on page refresh.
- **Single `index.html` rule.** GitHub Pages requires the entry point to be named `index.html` in the repository root.
