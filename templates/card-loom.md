---
title: Card Loom
app_type: card-loom
wallet: 0xf00dE0a49109769da9421504BF5824Fe9ecc137b
---

A page that holds a board of cards and lets them reflow themselves based on the width of the container they sit inside, not the width of the page. Every card knows whether it is currently narrow, medium, or wide, and switches its layout accordingly. Drop a panel divider across the board and the cards on each side rearrange themselves in real time. The board is one viewport, the layout is the work, the cards are the small living things inside.

› board

A rice colored page, the kind of warm cream a printer uses for a small cover. Soot for body type. Marigold reserved for the active drag handle and the small per card layout badge. Alpine for the panel divider line and the divider drag handle. Source Sans 3 at 18px 400 for body, at 22px 600 for the card titles. Outfit at 11px small caps for the layout state badge that sits on every card. Berkeley Mono at 12px for the live width readout on each card and the small ResizeObserver entries in the status strip. Three typefaces, three roles, no overlap. The layout is one full viewport with no scroll, the divider sits vertically in the middle, the two halves of the board flank it.

The board contains a small fixed set of cards that the visitor reorders and re organizes by dragging them between the two halves and inside each half. The cards themselves do the layout switching through CSS Container Queries, which gives the visitor a tiny direct view into how the modern web rebuilds itself around its container rather than its viewport.

› cards

Each card is a single CSS container, named tile, with container type inline size. Three breakpoints are defined inside src/styles/loom.css.

```
.tile {
  container-type: inline-size;
  container-name: tile;
}

@container tile (max-width: 280px) {
  .tile-body  { font-size: 14px; }
  .tile-image { display: none; }
  .tile-meta  { flex-direction: column; }
}

@container tile (min-width: 280.01px) and (max-width: 480px) {
  .tile-body  { font-size: 15px; }
  .tile-image { display: block; max-height: 96px; }
  .tile-meta  { flex-direction: row; }
}

@container tile (min-width: 480.01px) {
  .tile-body  { font-size: 17px; }
  .tile-image { display: block; max-height: 160px; }
  .tile-meta  { flex-direction: row; gap: 16px; }
}
```

Each card shows the card title in 22px Source Sans 3 600 soot at the top, a small image area below (a flat svg illustration in marigold and alpine with a few paths, drawn at card mount with a deterministic seed based on the card id), a small body paragraph in 14, 15, or 17 px depending on the container query state, a metadata row at the bottom with two small icons (drawn inline svg), and a marigold Outfit 11px small caps badge in the top right corner of every card reading either narrow, medium, or wide depending on the current container width.

A ResizeObserver attached to each card reports the live width to a small Berkeley Mono 12px line at the bottom right of the card, updating in real time as the card resizes, in the form 312 px wide. This readout is the only purely numeric element on the card.

› divider

A vertical bar, 6px wide, alpine, full board height, sitting at fifty percent of the board width by default. The visitor drags the bar left or right with the mouse or with a touch. The drag updates the CSS grid template columns of the board through a style update on a containing element, the two halves of the board resize themselves accordingly, the cards inside each half resize through the CSS grid template columns of their half, and the container queries fire on the cards as they cross the breakpoint thresholds. The page does not animate the resize, the resize is the animation.

A small marigold drag handle, 28px tall, sits in the middle of the divider with a pair of tiny vertical lines drawn inside it. Tapping the handle without dragging snaps the divider to the nearest of three preset positions, 30 percent, 50 percent, or 70 percent. Holding shift while dragging snaps to every five percent. The most recent divider position is saved to localStorage under the key card.loom.divider.v1 so the next visit comes back to the same split.

› cards inside the halves

Each half of the board is a CSS grid with grid template columns set to repeat auto fit minmax 240 px 1fr. The cards flow naturally into rows and columns inside the half. As the half shrinks, the cards reflow into fewer columns, and as the cards themselves shrink within their column they pass the container query thresholds independently. The result is that a single card at the wide breakpoint can sit next to another card already at the medium breakpoint, because each card responds to its own width, not the page's.

› drag

Cards can be reordered within a half and moved between the two halves. Each card has a small marigold drag handle in the top left corner. The visitor presses the handle, drags the card, drops it on a target slot, the cards reflow. The drag uses the standard HTML5 drag and drop api with DataTransfer, no library. The drag image is a soft semi transparent clone of the card with a 4px marigold ring.

When a card is being dragged, the layout state badge in the top right corner reads dragging in marigold instead of narrow medium or wide. On drop, the badge returns to its container width state.

› the status strip

At the very bottom of the page, a thin strip 32px tall, soot background, rice text, Berkeley Mono 12px. The strip shows a single line summarizing the current state of the board.

The line reads something like

left side, 632 px wide, 4 cards. right side, 448 px wide, 3 cards. divider at 58 percent.

The numbers update through a single ResizeObserver attached to each half of the board, with a single tick consolidating the readouts on every observer callback. The strip is the only place the page shows the board's overall geometry, the cards themselves show only their own width.

A small block of the shape the App stores for each ResizeObserver entry, written into src/lib/observers.ts.

```
{
  id:     'tile-3',
  inline: 312,      // px, contentRect width
  block:  186,      // px, contentRect height
  state:  'medium'
}
```

The mapping from inline width to state is the same as the container query breakpoints, so the badge and the line agree.

› scope

The board's styles are scoped through CSS @scope so the global page styles do not leak into the cards and vice versa. Each card's styles are scoped to a single .tile selector inside a single rule, which keeps the styles tight and lets the page introduce a second layout on the board in a future version without rewriting every selector.

The page also uses CSS subgrid where the browser supports it, with a graceful fallback to a plain grid when not. The fallback is detected through a single @supports query and the result is mirrored to a data attribute on the board so the App can choose to render an indicator if it wishes.

› motion

Reordering a card animates the position transition through the standard Element.animate api, using FLIP (first, last, invert, play) for smooth movement. The first measurement is taken before the dom mutation, the last measurement after, the inverse transform is applied, the animation plays the inverse back to zero over 240ms with an easing of cubic bezier 0.22, 0.61, 0.36, 1. The animations are short and quiet, the visitor feels them as inevitable.

› persistence

The board's full state, the divider position, the card order across both halves, the card content (which is fixed at boot but the order is editable), and the saved layout badges per card are written to localStorage under the key card.loom.state.v1 within 500ms of idle. On boot the App reads the saved state, restores the divider position and the card order, and the layout badges settle once the ResizeObservers fire.

The card content itself is shipped in src/lib/cards.ts as a small in source list of eight cards, each with a title, a body, and a deterministic seed for the inline svg illustration. The visitor cannot add or remove cards in this version, but the eight defaults are designed to span enough body length that the difference between the narrow, medium, and wide layouts is obvious at every breakpoint.

› edges

A divider dragged below 15 percent or above 85 percent shows a small marigold pulse on the divider and stops moving, with a small Berkeley Mono 11px line in the status strip reading the divider does not collapse a side. The minimum half width is enforced through CSS minmax inside the grid template columns.

A card dragged to a half that is too narrow for it to fit even at its narrow breakpoint (below 200 px) shows the same pulse on the divider, and a small Berkeley Mono 11px line in the status strip reading this card does not fit at this width, briefly, for 1500ms, then the card is returned to its original half.

If CSS Container Queries are unavailable (very old browsers), the cards fall back to a single fixed layout (the medium breakpoint), the badges show medium on every card, and a small Outfit 11px small caps line in the status strip reads container queries are not supported on this browser, falling back to the medium layout.

› build

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout primitives and color, plus a small set of plain CSS files for the container query rules and the @scope blocks (Tailwind does not yet expose container queries as fluent utilities at the version pinned). Custom palette in tailwind.config.ts adding rice, soot, marigold, alpine. Vite as the build tool. State is plain useState and one useReducer for the card order, the divider position, and the per card width map. No router, no global store, no context provider, no drag and drop library, no animation library, no CSS in JS library, no icon pack. ResizeObserver, IntersectionObserver, Element.animate, and DataTransfer are used directly. Every glyph including the small drag handle and the card image illustrations is inline svg.

Files. index.html with the Source Sans 3, Outfit, and Berkeley Mono links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the card order, the divider position, and the width map. src/components/Board.tsx for the two halves and the divider. src/components/Half.tsx for one side of the board. src/components/Tile.tsx for one card. src/components/Divider.tsx for the alpine bar and the marigold handle. src/components/StatusStrip.tsx for the bottom soot bar. src/lib/observers.ts for the ResizeObserver wrappers. src/lib/flip.ts for the FLIP helper using Element.animate. src/lib/cards.ts for the eight in source cards. src/styles/loom.css for the container query rules and the @scope blocks. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page on a desktop. The rice board paints, eight cards sit four and four across a divider in the middle. The marigold badges read medium on most of them. They drag the divider to thirty percent, the left side cards shrink and pop to the narrow badge, their images vanish, their bodies tighten. The right side cards grow and one of them passes the wide threshold, its body grows, its image grows, its badge reads wide. The status strip at the bottom updates the widths and the card counts. They drag a card from the left side over the divider into the right side, the FLIP animation slides every neighbor into place, the new card lands. They reload the tab and the same arrangement waits for them.
