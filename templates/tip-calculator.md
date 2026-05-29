---
title: Tip Calculator
app_type: tip-calculator
wallet: 0x2af983462aDdc782B03fc5d4BeC5B64C40f9f6A0
---

A small page that calculates the tip on a bill and splits the total across the people at the table. The visitor enters the bill amount, picks a tip percentage from three quick chips or types one in, sets the number of people, and the page shows the tip per person, the total per person, and the grand total. No signup, no ads, no menu of features beyond what a table at a restaurant actually needs.

## the look

A mint cream background, the cool clean color of a cafe napkin. Deep teal for body type. Coral reserved for the active tip chip and the per person totals. Butter for the small currency selector and the round up nudge. Lexend at 16px 400 for body and 14px 500 for the small button labels. Bricolage Grotesque at 30px 600 for the big total per person number and at 22px for the section headings. Two typefaces, two roles, no overlap. The page is one column at most 480px wide, centered, 24px gutter on phones.

The first thing the visitor sees is the title tip calculator in Bricolage Grotesque at the top, the bill input below it in a generously padded card, the tip controls just under, the split row after that, and the totals block as the largest element near the bottom.

## the bill

A single large number input in 28px Lexend deep teal on a soft white card with a 1px deep teal hairline at fifteen percent opacity, with the currency symbol on the left in 22px Lexend deep teal muted. The placeholder reads the bill, like 64.50. The input accepts decimal numbers with up to two decimal places, anything past that is silently truncated.

To the right of the input, a small currency selector in 13px Lexend, a small dropdown of eight common currencies (dollar, pound, euro, yen, rupee, peso, real, won) with the matching symbol shown to the left of the bill amount. The currency choice persists in localStorage so the visitor's next visit picks up the same currency.

A small italic 11px line below the input reads enter the total before tip in deep teal muted.

## the tip

A horizontal row of four chips below the bill card. The first three are preset percentages, 15 percent, 18 percent, 20 percent, each chip rendered as a small rounded rectangle 64 by 36 with the percent in 14px Lexend 500. The fourth chip is a small text input that lets the visitor type their own percent, like 22, with a small percent sign to its right. The active chip is filled coral with mint cream text and a 2px coral ring around it.

Below the chips, a small italic 11px Lexend line reads tap a chip or type your own, in deep teal muted.

## the split

Below the tip row, a small stepper for the number of people. A minus button on the left, a coral 28px circle with a soft 1px deep teal hairline, and a plus button on the right of the same shape. Between them, the count in 22px Bricolage Grotesque deep teal, defaulting to one. The minus is dimmed when the count is one, the count can never drop below one. The maximum is twenty, after which the plus is dimmed.

To the right of the stepper, a small italic 12px Lexend line reads people at the table.

## the totals

Below the split, the totals block. The biggest element on the page, three numbers stacked vertically with their labels.

The first row is the tip per person, in Bricolage Grotesque 22px coral, with the label tip per person in 13px Lexend deep teal muted to its left.

The second row is the total per person, in Bricolage Grotesque 30px coral, with the label total per person in 13px Lexend deep teal muted to its left. This is the biggest number on the page because it is the number the visitor reads aloud at the table.

The third row is the grand total, in Bricolage Grotesque 18px deep teal, with the label grand total in 13px Lexend deep teal muted to its left.

All three numbers are formatted in the visitor's chosen currency through Intl.NumberFormat with style currency and the chosen currency code.

A small italic 11px line at the bottom of the block reads rounded to the nearest cent, in deep teal muted. The actual math is exact, the display is the rounded form.

The shape of the calculation, written into src/lib/calc.ts as a single function.

```
function calc (bill, tipPercent, people) {
  const tip   = bill * (tipPercent / 100);
  const total = bill + tip;
  return {
    tipPerPerson:   tip   / people,
    totalPerPerson: total / people,
    grandTotal:     total
  };
}
```

## the nudge

A small text button below the totals block in 13px Lexend 500 butter reads round up the total per person to the nearest dollar (or the nearest unit of the chosen currency). Tapping it bumps the tip up just enough to make the per person total a round number. The page recomputes the actual tip percentage and shows it in italic 11px Lexend below the chips, like the tip is now 19.4 percent. Tapping the button again returns to the visitor's original tip percent.

A second small text button next to it reads split the rounding evenly. Some people at the table sometimes pay one cent more than others when the math does not divide evenly, this button makes that explicit, showing a small italic 11px line under the per person total reading two people pay one cent more, when applicable. The default behavior is to round the per person total down and let one or two people pay the small extra, but the button makes the difference visible.

## the keys

Pressing the enter key in the bill input commits the bill (defocuses the input so the totals settle). Pressing the up and down arrow keys in the bill input bumps the bill by one. Pressing the plus key anywhere adds a person. Pressing the minus key anywhere removes a person. Pressing the digit keys one through nine sets the tip percent to that number times ten (so the 1 key sets 10 percent, the 2 key sets 20 percent, and so on).

## the save

The current state (the bill, the tip percent, the people count, the currency) is saved to localStorage under the key tip.calculator.last.v1 within 300ms of idle. On boot, if the saved state is from the same session day, the App restores it (so a quick reload at the table does not blank the page). If the saved state is from a previous day, the App starts fresh because last night's bill is not relevant to today's lunch.

If localStorage is unavailable, the state holds in memory and a small italic 11px line at the bottom of the page reads this calculation is not being kept on this device.

## the edges

A bill that is zero or empty shows the totals as zeros with the labels dimmed. A bill that is negative is rejected with a small italic 11px line under the input reading the bill cannot be negative.

A tip percent in the custom input that is below zero is clamped to zero. A tip percent above 100 is allowed (the visitor is generous, the math still works).

A people count of one shows the same numbers in the tip per person and the grand total rows minus the grand total label which becomes the same as total per person.

A currency with no decimal subunits (like the yen) shows the totals as integers and the round up button bumps to the nearest yen instead of the nearest cent.

## the build

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding mint cream, deep teal, deep teal muted, coral, butter. Vite as the build tool. State is plain useState for the bill, the tip percent, the people count, and the currency. No router, no global store, no context provider, no currency library, no icon pack. Intl.NumberFormat is used directly for the formatting.

Files. index.html with the Lexend and Bricolage Grotesque links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the bill, the tip, the people, the currency, and the round up state. src/components/Header.tsx for the title. src/components/BillInput.tsx for the big number input and the currency selector. src/components/TipChips.tsx for the four chips. src/components/PeopleStepper.tsx for the minus, count, plus row. src/components/Totals.tsx for the three lines. src/components/Nudge.tsx for the round up and split buttons. src/lib/calc.ts for the math. src/lib/currency.ts for the symbol table and the decimal subunits. src/lib/storage.ts for the localStorage. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page at a cafe table. The mint cream surface paints, the title sits at the top, the bill input waits with the placeholder. They type 64.50, tap the 18 percent chip, the totals block fills in. They tap the plus button on the people stepper twice to set the count to three. The total per person reads in coral, the tip per person in smaller coral, the grand total in deep teal. They tap round up the total per person, the tip jumps a few cents, the per person amount is now a round number. They show the screen to the table, everyone reads the same number, the bill gets paid.
