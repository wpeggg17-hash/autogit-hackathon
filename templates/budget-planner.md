---
title: Budget Planner
app_type: budget-planner
wallet: 0x3adbc52233ec00c2a96359e6d6298c72efad59ba
---

You are an expert React developer. Generate a complete, production-ready React application for a monthly budget planner.

Requirements:

- Header with app title "BudgetWise" and current month and year displayed
- A summary row at the top with 3 cards: Total Income (green), Total Expenses (red), Remaining Balance (blue or red depending on positive or negative)
- An Add Transaction form with fields: description, amount, category dropdown (Food, Transport, Entertainment, Bills, Salary, Other), and type toggle (Income or Expense)
- A transaction list below showing all entries with: description, category tag, date, amount colored green for income and red for expense, and a delete button
- Pre-populate with 5 hardcoded transactions (mix of income and expenses)
- A simple category breakdown section showing each spending category as a labeled bar using only div width percentages and Tailwind, no chart libraries
- Clean white card layout with subtle shadows

Technical:

- React 18 with TypeScript
- Tailwind CSS for all styling
- Use useState for all transaction state
- Export a single default App component
- No external UI libraries, icon packs, or chart libraries
- Use inline SVG for any icons needed
- All data stored in component state only
- Fully responsive layout
