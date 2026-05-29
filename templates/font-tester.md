---
title: Font Tester
app_type: font-tester
wallet: 0x3057983B5DdAb5db0300c5336d8118126888e65b
---

Drop a font file on the page. The page registers the font through the FontFace api, sets a long sample paragraph in it, and lets the visitor try the font at several sizes, several weights when the font supports them, and across a small set of writing scenarios (headlines, body, captions, numbers). The page also peeks inside the font file to print the glyph count, the unicode ranges it covers, the design weight axis range when the font is variable, and the file size. The font never leaves the device, the file picker is local only.

drop

An ash colored page, the matte feel of recycled paper. Ink for the type. A single vermillion accent reserved for the drop hover state and the small badge that appears next to the active font name. Slate for the small captions and the section labels. Inter Tight at 14px 600 for section labels and small buttons. Iowan Old Style serif at 18px for body, and at 22px 36px 56px 92px for the live sample rendered in the dropped font (the page swaps the family to the dropped font, not the family that loaded the Iowan, the Iowan is the fallback while the dropped font is parsing). IBM Plex Mono at 12px for the metadata block. The page is one column at most 880px wide, centered, 28px gutter on phones.

The top half of the page is the drop zone. A full width rectangle, 200px tall, with a 1px ink hairline at twenty percent opacity. Inside the rectangle, centered, a Inter Tight 14px 500 line reading drop a font file here, with a small Iowan Old Style italic 13px subtitle beneath reading or click to pick from your computer. The rectangle accepts dragover events, switching its hairline to vermillion at full opacity while a file is being dragged over it. On drop, the file is read through FileReader as an ArrayBuffer, fed into a new FontFace constructor, added to document.fonts, and once the load resolves the live sample below swaps to the new family.

The FontFace shape the App builds for each dropped file is small and explicit, written into src/lib/font.ts.

```
new FontFace(
  'tester-' + slug,
  arrayBuffer,
  { display: 'swap' }
)
```

The slug is derived from the filename with a small in source normalization. Supported file types are otf, ttf, woff, woff2. Other extensions show a small italic 11px line under the drop zone reading this file type is not a font, in vermillion, for 1500ms.

render

Below the drop zone, the live sample block. It is split into four bands stacked vertically, separated by a thin ink hairline at ten percent opacity.

The first band is a single headline at 92px, the text is the dropped font's family name when the page can extract it from the file, otherwise the literal string the quick brown fox sits on the page. The line is left aligned, line height 0.95, letter spacing minus 1, lowercase, the kind of headline that appears in a magazine spread.

The second band is a paragraph at 18px, four short lines of plain modern english, the sample text is pangrammatic across the basic latin set so the visitor can see almost every letter. The line height is 1.5, letter spacing default, the paragraph reads naturally.

The third band is a caption row at 12px, with a small Inter Tight 12px label to the left reading caption, and a single italic line in the dropped font at 12px italic if the dropped font carries an italic style, otherwise the upright version at twelve percent slant emulated through CSS transform (a small italic 11px line below notes the italic was synthesized).

The fourth band is a number row at 36px, the line reading 0123456789 plus the typographic glyphs that often differ from the latin letters (lining figures, tabular figures, oldstyle figures), the page tries each OpenType feature available through font feature settings and shows them on three sub lines labelled lining, tabular, oldstyle in Inter Tight 11px slate.

The text inside each band is editable through a small text button at the right of each band reading edit this text in 12px Inter Tight slate, which turns the band into a contenteditable region with a one line undo through the standard browser undo. The sample text and the edits do not persist, every page load starts with the defaults.

inspect

Below the sample, a metadata block in IBM Plex Mono 12px ink, two columns on desktop and one column on phones. The fields, in order.

family. The family name from the font's name table.
style. The style name from the font's name table.
file size. The byte size of the dropped file in kilobytes.
file type. The file extension.
glyph count. The number of glyphs from the maxp table.
unicode ranges. A small list of the unicode blocks that have at least one mapped glyph, like basic latin, latin supplement, latin extended a, greek, cyrillic, hebrew, the list is in the IBM Plex Mono 12px.
variable. If the font is a variable font (the file contains an fvar table), the list of axes with min and max for each, like wght 100 to 900, opsz 8 to 144, slnt minus 12 to 0.
licence. The license string from the name table if present, truncated to one line.

The page reads these fields directly from the file's binary structure through a small in source binary reader using DataView, since installing an opentype library is overkill for the small subset the page needs. The fields the reader resolves are the four tables maxp, name, cmap, and fvar. Tables not present in the file are reported as absent in slate italic.

```
{ family:  'CoolSerif',
  style:   'Italic Bold',
  size:    218,   // kilobytes
  glyphs:  1342,
  ranges:  ['basic latin', 'latin supplement', 'symbols'],
  axes:    { wght: [100, 900], opsz: [8, 144] } }
```

save

A small horizontal row above the drop zone shows the recently dropped fonts as small pills, each pill is 12px Inter Tight slate with a vermillion dot in front when active. Tapping a pill switches the live sample to that font. The pills are kept across page reloads, the actual ArrayBuffer is stashed in IndexedDB under the store named fonts with a keyPath of slug. On reload the App opens the database, reads every stored buffer, registers each one with FontFace again, and rebuilds the pill row. The most recently dropped font is the active one on reload.

A small text button to the right of the pill row reads forget every font, with a confirm prompt that says remove every font from this device, which clears the IndexedDB store. There is no upload, no sharing, no export.

If IndexedDB is unavailable, the dropped fonts live only in the current session, the pills disappear on reload, a small italic 11px line at the bottom reads these fonts are not being kept on this device. The page still works for the session.

axes

If the active font is a variable font, a small extra block sits below the metadata, titled variable in Inter Tight 13px 600 slate. Inside the block, one slider per axis present in the file, each slider 24px tall with the axis tag on the left in IBM Plex Mono 12px (wght, opsz, slnt, wdth, ital), the slider track in ink at twenty percent opacity, the thumb in vermillion 14px round, the current value on the right in IBM Plex Mono 12px. Dragging a slider updates the matching CSS font variation setting on every band of the sample live, with no debouncing. The sliders persist their last values across reloads in localStorage under the key font.tester.axes.v1 keyed by the active font slug, so returning to a font shows the same axis values the visitor was tweaking last time.

The page also registers each axis as a CSS custom property through CSS Houdini's window.CSS.registerProperty api when available, with the syntax of number, the inherits set to false, and an initialValue picked from the axis default. The properties are scoped to the sample block so the visitor can see how the values flow through if they ever style the bands with their own classes through the dev tools.

keys

Pressing the keys 1 through 4 jumps the live sample to one of four preset scenes (display, body, caption, numbers), the band corresponding to the preset becomes the focus, the other bands dim to forty percent opacity. Pressing the e key on a focused band opens the inline edit. Pressing the escape key drops the edit and clears the dim.

edges

A dropped file that is not a valid font (the FontFace constructor throws on the buffer) shows a small italic 11px line at the bottom of the drop zone reading the browser could not parse this as a font for 1500ms. The pill is not added in this case.

A font with no italic style still shows the third band with a synthesized italic and the note next to it. The page never lies about which glyphs come from the file.

A font missing the maxp or the name table renders the bands fine but the metadata block shows the missing fields as slate italic dashes.

A font file larger than 16 megabytes is rejected before the FontFace call with a small italic 11px line reading this file is larger than expected, in vermillion, for 1500ms. Most real font files fit well under this size.

build

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding ash, ink, slate, vermillion. Vite as the build tool. State is plain useState and one useReducer for the font list, the active slug, and the axis sliders. No router, no global store, no context provider, no opentype library, no font loading library, no icon pack. FontFace, document.fonts, FileReader, DataView, IndexedDB, and CSS.registerProperty are all used directly. Every glyph including the small drop arrow and the vermillion dots is inline svg in the component that uses it.

Files. index.html with the Inter Tight, Iowan Old Style (with system fallback), and IBM Plex Mono links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the font list, the active slug, the live sample bands, and the axis values. src/components/DropZone.tsx for the top rectangle. src/components/PillRow.tsx for the small row of recent fonts. src/components/SampleBlock.tsx for the four bands. src/components/Metadata.tsx for the inspect block. src/components/AxesPanel.tsx for the variable slider block. src/lib/font.ts for the FontFace registration and the slug. src/lib/binary.ts for the small DataView reader (maxp, name, cmap, fvar). src/lib/db.ts for the IndexedDB store. src/lib/houdini.ts for the CSS.registerProperty wrapper. src/lib/storage.ts for the small localStorage helper used by the axis slider memory. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page in chrome on a laptop. The ash page paints, the empty drop zone sits at the top with the centered prompt. They drag a woff2 file from a folder onto the rectangle. The rectangle outlines in vermillion. They drop. The pill row shows a single pill with a vermillion dot, the four sample bands swap to the new font almost instantly, the metadata block fills in (family, style, file size, glyph count, unicode ranges). The font is variable so a small panel appears under the metadata with three sliders (wght, opsz, slnt). They drag wght to 700, the headline thickens live. They drag a second font into the page, a new pill appears, tapping it swaps everything. They close the tab, reopen the next day, both pills are still there, the active one is the most recent.
