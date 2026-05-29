---
title: Recipe Card
app_type: recipe-card
wallet: 0xbE6a50115b34543EB8Cf022A6031eb00D4B1A22E
---

A small page that holds a single recipe at a time. The visitor flips between recipes with a smooth view transition that animates the title, the photo, and the ingredient list across the swap, no router and no library needed. The page also scales every quantity proportionally when the visitor changes the serving size, prints cleanly to one sheet, and copies the recipe to the clipboard as both plain text and proper html so it pastes into emails with its formatting intact.

(¹) the look

A warm wheat background, the color of unbleached flour in a stoneware bowl. Cocoa for body and titles. Two accents only, paprika for the active serving size and the small paragraph counter on the method list, basil for the small leaf glyph on every herb in the ingredient list. Cooper Std at 84px for the single big number that shows the current serving size, the only place this typeface appears on the page. DM Serif Display at 38px for the recipe title and at 22px for the section labels. Inter at 17px 400 for body and at 14px 500 for the small button labels. Iosevka at 13px for the ingredient quantities and the small per step time on each method step. Four typefaces, four roles, no overlap. The page is one column at most 720px wide, centered, 24px gutter on phones.

The first thing the visitor sees on load is the recipe title in DM Serif Display, the cover photo below it (a small svg illustration drawn at boot from a few paths, no external image), the small serving size widget below the photo, and the two columns of the recipe (ingredients on the left, method on the right) on desktop, stacked on phones.

(²) the swap

The page holds an in source set of recipes (eight of them, written into src/lib/recipes.ts). The visitor moves between recipes with the left and right arrow keys, with a small text button below the cover photo reading next recipe, or by tapping the recipe name in a small drawer that opens from the top. The current recipe is mirrored into the url hash like #/recipe/four so the visitor can share the link.

Every swap runs through the View Transitions api. Each transitioned element receives a view transition name through inline style so the browser knows which old element pairs with which new element across the swap. The title, the cover photo, the serving size widget, the ingredient column, and the method column are all named. The browser captures the before state, mutates the dom to the new recipe, captures the after state, and runs the cross fade and translate animations between them. The View Transitions api takes care of the math, the page only has to name the elements correctly and call document.startViewTransition.

A small block of how the swap is invoked, written into src/lib/swap.ts.

```
function swapRecipe (next) {
  if (!document.startViewTransition) {
    setRecipe(next);  // no animation in older browsers
    return;
  }
  document.startViewTransition(() => {
    setRecipe(next);
  });
}
```

Recipes that share a cover photo style get a slightly different transition, because the api notices the same view transition name and slides the element across smoothly. Recipes with very different titles get a soft crossfade between the title strings. The result feels like turning the page of a printed cookbook.

(³) the ingredients

The left column holds the ingredients, one per row, in a CSS subgrid that aligns every quantity at the same vertical position across all rows. Each row has three cells. The first cell is the quantity in Iosevka 14px cocoa with the unit beside it. The second cell is the ingredient name in Inter 17px cocoa. The third cell is a small basil leaf inline svg if the ingredient is an herb (basil, rosemary, parsley, thyme, sage, mint), omitted otherwise. The basil leaf is the only decoration on the column.

A small serving size widget above the column shows the current serving size in Cooper Std 84px paprika, with two small text buttons in Inter 14px cocoa above and below the big number reading more and less. Tapping more increases the serving size by one, tapping less decreases it (minimum one). Every quantity in the ingredient list scales proportionally to the new serving size, with fractional units rounded to the nearest sensible value (half, third, quarter) when the result is close to a common fraction. The serving size persists across recipes per recipe id, written to localStorage under the key recipe.card.servings.v1 as a map from recipe id to serving size, so coming back to a recipe shows the size the visitor last left it at.

A small Inter 12px italic line below the widget reads scales every quantity, in cocoa muted, so the first time visitor knows what the big number is doing.

The shape of an ingredient row, written into src/lib/recipes.ts, looks like this.

```
{
  quantity: { value: 200, unit: 'g' },
  name:     'pasta',
  herb:     false
}
```

The herb flag controls the small basil leaf. The quantity unit is one of g, kg, ml, l, tsp, tbsp, cup, pinch, count. The scaling pass converts to a base unit per row before applying the ratio so that mixing grams and cups in the same recipe still scales correctly.

(⁴) the method

The right column holds the method steps, numbered in DM Serif Display 22px paprika on the left, with the step body in Inter 17px cocoa on the right, plus a small Iosevka 13px cocoa muted line below each step body showing the time the step usually takes, like 12 min. The numbers, step bodies, and times are aligned through a CSS subgrid so every step's three pieces stay in line down the column.

Each step has a small text button on the right reading done in 12px Inter cocoa muted. Tapping it strikes through the step body and the time, the small number jumps to paprika at sixty percent opacity. The done state is a quality of life affordance for the cook standing at the stove, it does not persist across visits because the cook only needs it during one session.

When every step is done, a small Inter 14px italic basil line appears below the method column reading enjoy.

(⁵) the print

A small text button at the bottom of the page in Inter 13px cocoa reads print this card. Pressing it opens window.print with a print stylesheet that hides the swap controls, the serving widget buttons, and the drawer, and prints the recipe on a single A5 page in portrait. The print stylesheet uses CSS page break inside avoid on the ingredient and method columns so neither column breaks mid item. The print page is intentionally austere, the visitor can pin it to their fridge.

Below the print button, a second small text button reads copy this card. Pressing it writes both a plain text version and an html version of the recipe to the clipboard through navigator.clipboard.write with a ClipboardItem holding both text/plain and text/html mime types. The plain text is the simplest form, the html preserves the section headings, the lists, and the small italics. A small toast confirms with copied to clipboard.

(⁶) the lifecycle

The Page Lifecycle api is used to handle the case where the page is frozen by the browser (the visitor switched tabs for a long time and the browser decided to pause the page to save memory). The page listens for the freeze event and releases any timers that were holding the small done button states alive. The page listens for the resume event and re hydrates the state from localStorage so the visitor sees the recipe and the serving size they were on. The freeze and resume are quiet, the visitor does not see any banner.

The page also listens for the visibilitychange event and pauses the small live timer on any method step that has a per step timer started. The timer resumes on visibilitychange back to visible. There is no banner for this either, the visitor just sees the timer pick up where it left off.

(⁷) the edges

A recipe that has fewer than three ingredients renders normally but the ingredients column shows a small Inter 11px italic line above the list reading short recipe in cocoa muted, just so the layout does not look broken on a small list.

A recipe that requires a special unit not in the supported set (gallon, ounce, pound) is handled through a per recipe override table in src/lib/units.ts so the scaling still works.

The View Transitions api is not yet in every browser. The page detects support through the presence of document.startViewTransition and falls back to a plain setState on the swap when not. The page continues to work, the transition is just instant rather than animated.

The clipboard html write is gated behind the navigator.clipboard.write check. When it is not available, the copy this card button copies only the plain text version through the older clipboard writeText, and a small italic 11px line below the button reads this browser does not support copying the formatted version, in cocoa muted, for 2 seconds after the copy.

(⁸) the build

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding wheat, cocoa, cocoa muted, paprika, basil. Vite as the build tool. State is plain useState and one useReducer for the current recipe id and the per recipe done states. No router, no global store, no context provider, no animation library (the View Transitions api is the animation library), no clipboard library, no icon pack. Every glyph including the small basil leaf and the cover photo illustrations is inline svg in the component that uses it.

Files. index.html with the Cooper Std (or its closest free fallback), DM Serif Display, Inter, and Iosevka links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the current recipe id and the swap. src/components/Cover.tsx for the title and the photo. src/components/Servings.tsx for the big Cooper Std number and the more and less buttons. src/components/Ingredients.tsx for the left column. src/components/Method.tsx for the right column. src/components/Drawer.tsx for the top drawer with the recipe list. src/lib/recipes.ts holding the eight recipes. src/lib/swap.ts for the View Transitions wrapper. src/lib/scale.ts for the quantity scaling pass with fraction rounding. src/lib/clip.ts for the plain and html clipboard write. src/lib/lifecycle.ts for the freeze, resume, and visibilitychange listeners. src/styles/print.css for the print stylesheet. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page on a laptop. The wheat page paints, the title pasta with brown butter and sage sits in DM Serif Display at the top, the cover svg drawn in cocoa and paprika sits below it, the serving size widget reads 4 in Cooper Std paprika. The ingredient column lists 200 g pasta, 60 g butter, 8 leaves sage (with a basil leaf glyph), a quarter teaspoon black pepper, and a pinch of salt. The method column shows three numbered steps. They press the more button on the serving widget, the number rolls up to 5 with a small bounce, every quantity in the ingredient column re scales smoothly. They press the right arrow key, the page swaps to the next recipe with a soft cross fade through the View Transitions api, the title slides across, the cover photo morphs in place. They press print this card, the print dialog shows a clean A5 layout. They press copy this card, paste into an email, the recipe appears with its formatting intact.
