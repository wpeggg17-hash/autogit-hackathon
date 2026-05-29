---
title: Bug Triage Command Center
app_type: bug-triage-command-center
wallet: 0x09B1433D89d4a4583482268c445aec346F06512E
---

You are an expert React and TypeScript product engineer. Generate a complete, production-ready React application for a bug triage command center used by a small software team to review, prioritize, and prepare bug reports for fixing.

Core product idea:
- The app is a single-page workspace for triaging incoming software bugs.
- It should feel like an internal engineering tool, not a marketing page.
- It must work fully offline with in-source demo data.
- It must be immediately useful on first load without requiring login, backend calls, or external APIs.

Technical requirements:
- React 18 with TypeScript.
- Tailwind CSS for all styling.
- Export a single default `App` component.
- No external UI libraries, icon packs, chart libraries, routers, or state management packages.
- Use only React state, TypeScript types, native date formatting, and inline SVG icons.
- The generated app must compile in a standard Vite + React + TypeScript + Tailwind setup.
- Do not use browser-only APIs during module initialization; keep localStorage access inside safe functions or effects.

Data model:
Create an in-source array of at least 14 realistic bug reports. Each report should include:
- id, title, product area, severity, status, reporter, owner, created date, last updated date.
- customer impact summary.
- reproduction steps as a short array of strings.
- environment string such as "Chrome 125 on Windows" or "iOS 18 Safari".
- tags such as "billing", "auth", "mobile", "regression", "accessibility", "performance".
- confidence score from 1 to 5.
- affected customers count.
- linked release train such as "2026.05 hotfix" or "Backlog".

Main layout:
- Full viewport application shell with a compact top bar, a left filter rail, a central issue list, and a right detail panel.
- On mobile, stack the layout into top bar, filters, list, then detail panel.
- Keep the interface dense, calm, and scannable.
- Use a restrained palette with a light background, dark text, muted borders, and severity accents:
  - Critical: red
  - High: amber
  - Medium: blue
  - Low: green
- Do not use gradients, decorative blobs, landing-page hero sections, or marketing copy.

Top bar:
- Show the title "Bug Triage".
- Show three compact metrics calculated from the current data:
  - Open bugs
  - Critical and high bugs
  - Total affected customers
- Include a search input that filters by title, tag, product area, reporter, owner, or impact summary.

Left filter rail:
- Severity filter with toggle buttons for Critical, High, Medium, Low.
- Status filter with toggle buttons for New, Confirmed, In Progress, Blocked, Ready for QA.
- Product area dropdown or segmented selector.
- Minimum confidence slider from 1 to 5.
- "Only unowned" checkbox.
- Clear filters button.
- The filters must update the issue list and metrics live.

Central issue list:
- Sort controls for:
  - Most recently updated
  - Severity
  - Affected customers
  - Confidence
- Each issue row/card should display:
  - Title
  - Severity badge
  - Status badge
  - Product area
  - Owner or "Unowned"
  - Affected customer count
  - Updated date
  - First two tags
- The selected issue should be highlighted and drive the right detail panel.
- If filters produce no results, show a compact empty state with a reset button.

Right detail panel:
- Show the selected issue title, severity, status, owner, reporter, product area, and dates.
- Show customer impact in a clearly labeled section.
- Show reproduction steps as a numbered list.
- Show environment and release train.
- Show all tags as small pills.
- Include a "Triage note" textarea. Store notes in localStorage by issue id. Notes should persist after reload.
- Include action buttons:
  - Assign to me
  - Mark confirmed
  - Send to QA
  - Copy summary
- The actions should update local UI state only. For "Assign to me", use the name "You". For "Copy summary", copy a concise markdown summary to the clipboard if available and show a temporary "Copied" state; if clipboard is unavailable, select or display the summary text in the UI.

Derived logic:
- Severity sorting order is Critical, High, Medium, Low.
- Open bugs are any status except Ready for QA.
- Critical and high bugs count should include only currently visible bugs after filters are applied.
- A bug row should show a small "stale" indicator if it has not been updated in more than 7 days relative to the current date.
- The product area filter should be generated from the demo data, not hardcoded separately.

Interaction details:
- Search and filters should preserve the current selected issue if it remains visible; otherwise select the first visible issue.
- Keyboard accessibility: all buttons and form controls must be reachable by keyboard and have visible focus styles.
- Buttons should use familiar inline SVG icons where helpful, such as search, user, check, clipboard, and refresh.
- Avoid text overflow in badges and rows. Use truncation only where appropriate, with the full title visible in the detail panel.

Visual finish:
- Use 8px border radius or less on cards and panels.
- Use subtle shadows only for the active detail panel or sticky top bar.
- Keep typography compact: no oversized hero text.
- Use consistent spacing and clear section headers.
- The app should look polished at 1440px desktop, 1024px tablet, and 390px mobile.

Files the generated project should contain:
- `index.html`
- `src/main.tsx`
- `src/App.tsx`
- `src/components/TopBar.tsx`
- `src/components/FilterRail.tsx`
- `src/components/IssueList.tsx`
- `src/components/IssueDetail.tsx`
- `src/components/Badge.tsx`
- `src/lib/bugs.ts`
- `src/lib/storage.ts`
- `src/lib/triage.ts`
- `tailwind.config.ts`
- `package.json`
- `README.md`

The README should explain what the app is, how to run it locally, and that the data is demo data stored in source. Keep it short and practical.

Completion standard:
The app is finished when a first-time user opens it and sees a usable triage workspace with populated bugs, can filter and sort bugs, select one, write a note, assign it to "You", mark it confirmed, copy a markdown summary, reload the page, and see the note still attached to the same issue.
