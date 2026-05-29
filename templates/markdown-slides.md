---
title: Markdown Slides
app_type: markdown-slides
wallet: 0x069d5accd0bfb7b04b960D76136C03A474292c3A
---

A single page that turns a markdown file into a slide deck. The visitor types markdown into the editor pane on the left, the deck renders live on the right, slides split on a line of three hyphens. Press space to advance, b to go back, f to enter fullscreen present mode, n to pop a separate small window with the speaker notes through the Document Picture in Picture api. The whole deck is one text file, the visitor owns it.

## ▸▸ 1 surface

A charcoal page, the matte feel of a quiet conference hall before the doors open. Paper for the slide background so each slide reads like a printed handout against the charcoal frame. Candy red for the active slide indicator and the small page number on each slide. Hint blue for the small section labels and the link styles inside slides. Inter Tight at 16px 400 for the editor pane and 14px 500 for the small buttons. Recoleta serif (with Playfair Display as a fallback) at 36px 600 for slide titles, and at 22px 400 for slide body text inside the slide. Two typefaces, two roles, no overlap. The layout is two columns on desktop, the editor on the left at 40 percent of the width, the preview on the right at 60 percent. Stacks vertically on phones.

## ▸▸ 2 editor

The left pane is a single textarea in 14px Inter Tight charcoal on a soft paper background, padded 16px, with a 1px charcoal hairline at fifteen percent opacity. The textarea has spell check off and tab character handling, pressing tab inserts a literal tab character. The textarea wraps long lines visually but preserves the source as is. Above the textarea, a small text button row in 12px Inter Tight, three buttons reading add a slide, paste an image, save the deck.

Add a slide inserts a three hyphen separator followed by a fresh slide template at the current cursor position, the template is

```
---

## title of this slide

write the body here.

```

Paste an image opens a file picker, reads the picked image as a data url, and inserts a markdown image reference at the cursor with the data url as the source. The image is embedded in the deck source itself, no external file.

Save the deck downloads the textarea contents as a plain .md file through a temporary anchor click, with a filename of the visitor's chosen deck name.

## ▸▸ 3 preview

The right pane shows the deck as it would render. The current slide fills the available width with a 16 by 9 aspect ratio frame, scaled to fit. Below the slide frame, a thin horizontal row of small slide thumbnails, each thumbnail 80 by 45 pixels with the slide number in 11px Inter Tight charcoal centered below. The active thumbnail has a 2px candy red ring around it. Tapping a thumbnail switches the preview to that slide.

The markdown is parsed in source through a small line based parser in src/lib/md.ts. The supported subset is small.

A line starting with one to four pound signs and a space is a heading. The heading sizes are 48, 36, 26, 20 pixels for h1 through h4. The heading uses the Recoleta family.

A line starting with a hyphen or an asterisk plus a space is a list item. The marker stays visible as a hint blue dot. Nested lists are supported through indentation.

Text wrapped in single asterisks is emphasized in italic, double asterisks is bold in weight 600. Backticks are inline code in a soft hint blue background with a slight padding.

A run of three backticks on a line by itself opens a fenced code block, rendered in Inter Tight 14px on a soft charcoal band with a faint hint blue left border 2px wide.

A line containing only three hyphens is a slide break. The deck builder splits the source on these breaks and produces one slide per chunk.

A line like an image reference, embedded as the data url from the paste action, renders inside the slide at a maximum width of 80 percent of the slide, centered.

## ▸▸ 4 navigation

A small thin status strip across the very bottom of the page in 12px Inter Tight paper on charcoal shows the current slide number and the total, like 3 of 12. To the right of the count, a small text button row reads previous, next, present.

The keyboard shortcuts work in the preview pane and in fullscreen.

```
ArrowRight | Space | PageDown | n     -> next slide
ArrowLeft  | b     | PageUp           -> previous slide
Home                                  -> first slide
End                                   -> last slide
f                                     -> toggle fullscreen
Escape                                -> exit fullscreen
o                                     -> open speaker notes window
```

## ▸▸ 5 fullscreen

Pressing f calls element.requestFullscreen on the preview pane. The pane fills the screen, the editor and the thumbnails disappear. The arrow keys navigate, the slide number remains in the bottom right corner in small 11px paper. Escape exits.

The fullscreen pane respects the visitor's monitor aspect, the slide is scaled to fit while preserving the 16 by 9 frame. The background outside the slide fills with the same charcoal color used elsewhere on the page, so the transition into fullscreen is visually quiet.

## ▸▸ 6 speaker notes

Each slide can carry speaker notes through a hidden block at the end of the slide source, prefixed with a line that reads exactly notes after a single colon.

```
## the next product line

three skus shipping by autumn.

notes:
- skip the manufacturing slide if running short on time
- the autumn date is a soft target, do not commit
- if questioned on price, defer to the sales lead
```

The notes are not shown inside the slide preview. Instead, pressing o (or tapping a small text button reading open the notes window) opens a separate small window through the Document Picture in Picture api. The window holds the notes for the current slide in 16px Inter Tight paper on charcoal, and updates as the visitor advances. The window stays on top of other apps even when the visitor moves to a different desktop, useful when presenting to a projector with the visitor's laptop on a side table.

The api call is small.

```
const pip = await window.documentPictureInPicture.requestWindow({
  width:  420,
  height: 320
});
pip.document.body.appendChild(notesNode);
```

If the api is unavailable (firefox at most versions, safari), the notes open in a new browser window through window.open with the same content, falling back to the platform's window stacking. A small italic 11px line under the button reads notes opened in a separate window when this fallback is used.

## ▸▸ 7 save

The deck source is mirrored to localStorage under the key markdown.slides.last.v1 within 400ms of idle. On boot, the App reads the saved source and renders the deck. If localStorage is unavailable, the source lives in memory and a small italic 11px line at the bottom reads this deck is not being kept on this device.

A small text button below the editor reads export to pdf. Pressing it opens window.print with a print stylesheet that uses CSS @page rules to set the page size to a 16 by 9 aspect (A5 landscape rounded), and the slide breaks land at every three hyphen boundary in the source. The visitor's browser print dialog saves the result as a pdf or sends to a printer.

```
@page { size: 297mm 167mm; margin: 0; }
.slide { break-after: page; }
```

A second text button reads copy a share link. Pressing it builds a base64url string of the deck source (compressed first through CompressionStream gzip), copies a url with the string appended after a hash. Pasting the url on another device opens the page with the same deck loaded, no server required. The compression keeps short decks under the typical url length limits.

## ▸▸ 8 edges

A deck source with no three hyphen breaks renders as a single slide. The thumbnail row shows one thumbnail.

A slide source with no heading renders without a title, the body fills from the top with a small italic 11px caption in the top right reading no title, in hint blue muted.

A markdown image whose data url exceeds the typical url limit (very large pasted images) shows a small italic 11px line under the paste a image button reading the image is large, the deck will not fit in a share link.

A slide with notes but no body still renders an empty paper slide, the speaker can talk over it. A slide with body but no notes opens the speaker notes window empty, the visitor can type notes directly into the window which persist back into the source via a small live sync.

A fullscreen entry that the browser denies (some embedded contexts) shows a small italic 11px line at the bottom of the preview reading the browser denied fullscreen, present in the preview pane.

## ▸▸ 9 build

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding charcoal, paper, candy red, hint blue. Vite as the build tool. State is plain useState and one useReducer for the current slide index and the editor source. No router, no global store, no context provider, no markdown library, no syntax highlighter, no slide library beyond what is hand written, no icon pack. Fullscreen api, Document Picture in Picture api, CompressionStream, and the keyboard event handlers are used directly.

Files. index.html with the Inter Tight and Recoleta (Playfair Display fallback) links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the source, the current slide index, and the fullscreen state. src/components/Editor.tsx for the left pane and its three top buttons. src/components/Preview.tsx for the right pane and the slide frame. src/components/Thumbnails.tsx for the horizontal thumbnail row. src/components/StatusStrip.tsx for the bottom row. src/lib/md.ts for the small markdown parser. src/lib/slides.ts for the slide splitter on three hyphen breaks and the notes extractor. src/lib/pip.ts for the Document Picture in Picture wrapper with the window.open fallback. src/lib/print.ts for the CSS @page styles. src/lib/share.ts for the CompressionStream gzip encode and the base64url. src/lib/storage.ts for the localStorage. src/styles/print.css for the print stylesheet. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page in chrome on a laptop. The charcoal surface paints, the editor on the left holds a small example deck of three slides, the right pane shows the first slide rendered in paper with a Recoleta title and the body in Inter Tight. They press the right arrow, the second slide slides in. They press f, the preview goes fullscreen across the monitor. They press o, a small separate window pops up with the speaker notes for slide 2. They advance to slide 3, the speaker notes window updates. They press escape, fullscreen exits. They press copy a share link, paste into a chat, the recipient opens the link and sees the same deck.
