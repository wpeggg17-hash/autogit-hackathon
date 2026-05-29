---
title: Birthday Reminder
app_type: birthday-reminder
wallet: 0x06804A48eFe7ABa9FeEE1fA4BfEaA6962B6ADB32
---

A small page that holds the birthdays of the people the visitor cares about and gently reminds them when one is coming up. Add a name and a date, the page shows who is up next, how many days away the day is, and what age that person is turning. Birthdays can be marked sent so the visitor knows whether they have already mailed a card. The page lives in one tab and forgets nothing.

what the page is

A dusty rose background, the warm color of an envelope a child might choose. Navy for body type. A mustard accent reserved for the next upcoming birthday card and for the small button that marks one sent. Ivory for the cards themselves so they read clearly against the rose. Fraunces serif at 28px for names and at 36px for the big number of days away on the next up card. Manrope at 15px 400 for body and 13px 500 for the small button labels. The page is one column at most 580px wide, centered, 24px gutter on phones.

The first thing the visitor sees on load is the page title in Fraunces 30px navy reading whose birthday is next, a small italic Manrope 14px navy muted subtitle below it reading the people you do not want to forget. Below the subtitle, the next up card, which shows the upcoming birthday in the biggest type on the page. Below that, the rest of the list, sorted by upcoming date.

the next up card

A single ivory card with a 1px navy hairline at fifteen percent opacity, padded 24px, with a small mustard ribbon glyph in the top right corner. Inside the card, four lines stacked vertically. The first line is the name of the person in Fraunces 28px navy. The second line is the big number of days away in Fraunces 36px mustard, like 14 days, with the word days in 16px Manrope navy muted to the right. The third line is the date in long form like Saturday the twelfth of June. The fourth line is the age turning, in Manrope 14px italic navy muted, like turning 31.

If the next birthday is today, the big number reads today in Fraunces 36px mustard. If it is tomorrow, it reads tomorrow. The card pulses with a soft 200ms scale up and back on first paint to draw the visitor's eye to it.

Below the four lines, a single small text button reading mark this sent in 13px Manrope 500 mustard. Tapping it records that the visitor has already mailed a card or sent a message for this year. The button text changes to sent for this year in mustard at sixty percent opacity, the small ribbon glyph at the top right turns into a small inline svg envelope, and the card stays visible until the actual birthday passes, at which point it drops into the rest of the list and the next up card refreshes.

the rest of the list

Below the next up card, a vertical list of every other person in the rolodex, sorted by upcoming birthday ascending. Each row is a smaller ivory card, padded 16px, with three lines. The first line is the name in Fraunces 20px navy. The second line is the date in 13px Manrope navy muted, in the same long form. The third line shows the days away, the age turning, and a small sent badge if the visitor has already marked this year sent, all in 12px Manrope.

To the right of each row, a small chevron in 12px navy. Tapping the row opens a small inline panel below the row with three small text buttons in 12px Manrope, the first reads edit, the second reads mark this sent (or unmark when already sent), the third reads remove this person. Each action is one tap, the panel closes itself after.

the add row

At the top of the page above the next up card, a small text button reads add a person in 14px Manrope 500 navy on a faint ivory band with a 1px navy hairline at fifteen percent opacity. Tapping it opens a small form with three inputs. The first is the person's name in 16px Manrope, required. The second is the date of birth, split into three small number inputs for day, month, and year. The day and month are required, the year is optional (the visitor may not know the year for a child of a friend, or may have decided to skip the age display). The third is a small text input labelled how do I know them, optional, free text for a small note like high school friend or my partner's sister. Saving the form inserts the person into the rolodex and sorts them into the list.

A small italic 12px line below the inputs reads names and dates are kept only on this device.

how the dates are computed

The days away for each person is the number of full days until their next birthday in the visitor's local timezone, computed at boot and refreshed every hour while the page is open. The age turning is the year of the next birthday minus the year of birth, omitted if the year of birth is not known. The page does not assume a region, the date is written in the visitor's language through Intl.DateTimeFormat with the default locale.

A person whose birthday falls on the leap day (the twenty ninth of February) is handled by celebrating on the twenty eighth in non leap years, with a small italic Manrope 11px line under the row reading celebrated on Feb 28 in non leap years.

the shape

Every person in the rolodex is one small object, kept in localStorage under the key birthday.reminder.list.v1 as an array.

```
{
  id:    'a4f9',                 // 4 char random hex
  name:  'Asuka Kim',
  day:   12,
  month: 6,
  year:  1994,                   // null if unknown
  note:  'high school friend',
  sentYears: [2024, 2025]        // years the visitor marked sent
}
```

The sentYears array tracks every year the visitor has marked the person sent so the page knows not to show the sent badge for the next year automatically. The badge clears for the new year on the first time the page loads after the person's birthday passes.

the import and export

A small text button at the bottom of the page in 12px Manrope navy reads export the list to a file. Pressing it downloads a single json file with the array of people through a temporary anchor click. A second small text button reads import a list from a file. Pressing it opens a file picker, the picked json is parsed, validated, and merged into the rolodex (people with matching id are updated, new people are added, none are removed). The visitor can carry their rolodex from one browser to another through this small file.

There is no cloud, no signup, no analytics, no ad.

the edges

A person added with a future date of birth (a child not yet born, or a typo) is accepted, the page shows the age turning as a negative number with a small italic 11px line under the row reading future date, are you sure, with a small text button reading edit and a small text button reading keep it. The visitor decides.

A person added with a date already past in the current year shows the date of next year in the long form date line, the days away counter counts to next year's birthday correctly.

A list with no people shows a single Fraunces 18px italic navy muted line in the center reading add the people you do not want to forget, with the add a person button just below.

A list of more than 200 people is accepted, the page does not paginate, but the rendering is virtualised through a simple windowing pass that only mounts the rows currently in the viewport plus a small buffer on each side, keeping the page fast even with a large rolodex.

the keys

Pressing the slash key opens the search input at the top of the list, where the visitor can filter the rolodex by name. Pressing the escape key closes the search. Pressing the a key opens the add a person form. Pressing the e key on a focused row opens the edit panel. Pressing the s key on a focused row marks it sent.

the build

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding rose, navy, navy muted, mustard, ivory. Vite as the build tool. State is plain useState and one useReducer for the rolodex and the edit panel state. No router, no global store, no context provider, no date library, no icon pack. Intl.DateTimeFormat is used directly. Every glyph including the small ribbon and the chevron is inline svg in the component that uses it.

Files. index.html with the Fraunces and Manrope links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the rolodex, the next up id, the search query, and the open edit panel. src/components/Header.tsx for the title and subtitle. src/components/NextUp.tsx for the big card. src/components/Row.tsx for one rolodex row. src/components/AddForm.tsx for the new person form. src/components/EditPanel.tsx for the inline edit. src/components/Search.tsx for the slash filter. src/lib/people.ts for the person shape and the localStorage. src/lib/dates.ts for the days away, age turning, and leap day rules. src/lib/io.ts for the export and import. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page in any browser. The dusty rose page paints, the title sits at the top in Fraunces navy, the empty state shows the add a person line. They tap add a person, type Asuka Kim, twelve, six, 1994, high school friend, save. A next up card appears with the name in Fraunces, the big 14 days in mustard, and the line turning 31 below it. They add three more friends, the next up card stays focused on the closest birthday, the rest of the list fills out below. They mark the next one sent. They close the tab. They open it next week, the days away counter has dropped to 7, the page is ready for them to send the card.
