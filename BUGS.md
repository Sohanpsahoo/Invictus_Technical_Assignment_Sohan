# Bugs found

Add one section per issue. Bug 1 is filled in to show the format — fix it, then write what you changed. Copy the blank template for the rest.

Keep this file in the repo and **commit it** with your fixes.

---

## Bug 1

**How to reproduce:** : While opening the app , the expense list says “Newest first”. The first row is Wine (7 Mar). Board game (15 Mar) is further down.

**What is wrong:** : The list is showing oldest expenses first. Newest expenses should be at the top.

**What I changed:** : I have updated `dateValue` in `src/lib/format.js` to return the numeric timestamp (`new Date(date).getTime()`) so dates are correctly comparable, and updated the sort comparator in `src/components/ExpenseList.jsx` to sort in descending order (`dateValue(b.date) - dateValue(a.date)`) so newest expenses appear first.

---

## Bug 2

**How to reproduce:** :  If we look at the "Balances" panel on the right , Aisha Khan has paid more than her share, but the panel displays "Aisha Khan owes $38.67". Meanwhile, Carlos Mendes has paid less than his share, but the panel displays "Carlos Mendes is owed $17.34".

**What is wrong:** The logic in `src/components/BalancesPanel.jsx` has been inverted: members with positive net balances (who paid more than they spent) were shown as "owes", while members with negative net balances (who spent more than they paid) were shown as "is owed".

**What I changed:** : In `src/components/BalancesPanel.jsx`, I have updated the condition so that positive balances (`bal > 0.005`) render `is owed ${formatMoney(bal)}` with CSS class `"owed"`, and negative balances (`bal < -0.005`) render `owes ${formatMoney(-bal)}` with CSS class `"owe"`.

---

## Bug 3

**How to reproduce:** : If we look at the expense "Uber to airport" ($60 paid by Diya, split between Aisha and Ben). Diya was not part of the split. However, Diya's balance is penalized by deducting $30 ($60 / 2) from her balance. If you sum all members' balances, the total is -$30.00 instead of cancelling out to $0.00.

**What is wrong:** : In `src/lib/balances.js`, if the payer was not included in `shares`, the code subtracted `amount / splitWith.length` from the payer's balance. A payer who is not on the split should receive their full payment back without any deductions.

**What I changed:** : I have removed the erroneous penalty deduction block (`if (!(exp.paidBy in shares)...)`) from `computeBalances` in `src/lib/balances.js` so that payers not included in the split are credited for the full payment amount.

---

## Bug 4

**How to reproduce:** : In a scenario where a debtor's owed amount matches a creditor's owed amount exactly, the Settle Up panel fails to display a suggested transfer between them, leaving their debts and credits unresolved.

**What is wrong:** :  In `src/lib/settle.js`, the `else` branch of the settlement matching loop (which executes when `d.amount === c.amount`) only advanced the loop indices `i` and `j` without pushing the settlement transfer to the `transfers` array.

**What I changed:** : I have refactored the settlement algorithm in `src/lib/settle.js` to compute `const payment = Math.min(d.amount, c.amount)`, correctly creating and pushing the transfer for all cases (including exact matching amounts) and decrementing remaining balances.

---

## Bug 5

**How to reproduce:** In the "Filter" panel, select any member (e.g. "Aisha Khan") from the "Paid by" dropdown. The entire expense list becomes empty and shows "No expenses match these filters.", even though that member paid for multiple expenses.

**What is wrong:** In `src/App.jsx`, the filter comparison `if (paidBy !== "" && e.paidBy !== paidBy)` used strict inequality (`!==`) between `e.paidBy` (a number) and `paidBy` (a string from the `<select>` input), which always evaluated to true and filtered out all expenses.

**What I changed:** Updated the filter condition in `src/App.jsx` to compare string values (`String(e.paidBy) !== String(paidBy)`), ensuring accurate filtering regardless of numeric or string type representation.

---

## Bug 6

**How to reproduce:** Click the "Delete" button on the first item in the list ("Board game"). Instead of "Board game" being deleted, "Groceries" is removed from the list, while "Board game" remains. Similarly, editing the amount of any sorted or filtered expense modifies a different expense.

**What is wrong:** `ExpenseList.jsx` passed the visual array index of the filtered and sorted list to `onDeleteAt(index)` and `onUpdateAt(index, patch)`. The reducer in `src/state/store.js` spliced/mutated `state.expenses` by that raw index, mutating whatever item happened to be at that position in the underlying array rather than the intended expense.

**What I changed:** Updated `src/components/ExpenseList.jsx`, `src/App.jsx`, and `src/state/store.js` to identify and mutate expenses by their unique `id` rather than array index (`filter(e => e.id !== action.id)` for delete, and `map(e => e.id === action.id ? ... : e)` for update). Also updated list item keys to use `expense.id` for stable rendering.

---

## Bug 7

**How to reproduce:** Add an expense of $100 split equally among 3 people. Each share is calculated as $33.33, totaling $99.99 ($0.01 lost). Additionally, entering valid custom percentages (e.g., 33.33%, 33.33%, 33.34%) fails form validation due to floating point precision issues (`sum === 100.00000000000001`).

**What is wrong:** `splitEqual` and `splitByPercent` in `src/lib/money.js` simply rounded individual shares with `.toFixed(2)` without distributing remainder cents, causing the total shares to not equal the expense amount. Furthermore, `percentsSumTo100` used strict float equality (`=== 100`), causing false validation errors.

**What I changed:** Updated `splitEqual` and `splitByPercent` in `src/lib/money.js` to compute total cents and allocate remainder cents across shares so the sum of individual shares always matches the bill amount exactly. Updated `percentsSumTo100` to allow floating-point tolerance (`Math.abs(sum - 100) < 0.01`).

---

## Bug 8

**How to reproduce:** In the Summary panel on the right, use the "Add member" form to add a new person (e.g., "Elena"). The member count increases, but the "Paid so far" list below does not show the newly added member ("Elena $0.00") until an expense is added or modified.

**What is wrong:** In `src/components/SummaryCards.jsx`, the `useMemo` calculating `perPerson` only listed `[expenses]` in its dependency array and omitted `members`. Adding a new member did not trigger re-evaluation of the member paid totals. Additionally, `loadState` in `src/state/store.js` did not rehydrate date strings from `localStorage` into `Date` objects, and `formatDate` in `src/lib/format.js` did not format raw date strings consistently.

**What I changed:** Added `members` to the `useMemo` dependency array (`[members, expenses]`) in `src/components/SummaryCards.jsx`. Updated `loadState` in `src/state/store.js` to hydrate state loaded from `localStorage`, and updated `formatDate` in `src/lib/format.js` to ensure consistent date formatting.


---

## Bug 9

**How to reproduce:** View the app in a timezone west of UTC (e.g. America/New_York UTC-5). An expense recorded for "2026-03-12" displays as "11 Mar 2026" instead of "12 Mar 2026".

**What is wrong:** Date strings like `"2026-03-12"` from `seed.json` or `<input type="date">` are parsed as UTC midnight (`2026-03-12T00:00:00Z`). When `formatDate` in `src/lib/format.js` calls `.toLocaleDateString()` without specifying a timezone, the browser converts UTC midnight to local time, shifting the date back by one day in western timezones.

**What I changed:** Added `timeZone: "UTC"` to the `toLocaleDateString` options in `formatDate` in `src/lib/format.js`, ensuring dates render consistently regardless of the user's local timezone.

---

## Bug 10

**How to reproduce:** Enter a description (e.g. "Beach snacks") and an amount ($25), then click "Save expense". The expense is added to the list, but the description and amount fields still contain "Beach snacks" and "25". Clicking "Save expense" again creates an accidental duplicate.

**What is wrong:** In `src/components/AddExpenseForm.jsx`, the `submit()` function calls `onAdd(...)` but never clears the `description` or `amount` state afterwards, leaving stale values in the form.

**What I changed:** Added `setDescription("")` and `setAmount("")` after the `onAdd(...)` call in `submit()` in `src/components/AddExpenseForm.jsx`, so the form resets after a successful submission.

---

## Bug 11

**How to reproduce:** In the Summary panel, add a new member (e.g. "Elena"). Then look at the "Add expense" form — all 4 original members are selected in the "Split between" chips, but Elena's chip is unselected. If you switch to "Custom %", Elena does not appear in the percentage inputs either.

**What is wrong:** In `src/components/AddExpenseForm.jsx`, `splitWith` and `percents` are initialized with `useState(members.map(...))` only on component mount. When new members are appended to the `members` prop, the `splitWith` state is never updated, leaving newly added members excluded from bill splits by default.

**What I changed:** Added a `useEffect` in `src/components/AddExpenseForm.jsx` that watches the `members` prop and appends any newly added member IDs to `splitWith`, also recalculating `percents` via `evenPercents` to include them.

---

## Bug 12

**How to reproduce:** In any expense row, type an invalid value into the inline amount input (e.g. `-50`, `0`, or `abc`) and click outside the input (blur). The actual expense amount remains unchanged, but the input field continues to display the invalid text.

**What is wrong:** In `src/components/ExpenseList.jsx`, the `onBlur` handler only calls `onSaveAmount` when the input is valid. If the value is invalid, nothing happens — but `draft` state is never reverted to `String(expense.amount)`, so the invalid text stays visible in the input.

**What I changed:** Added an `else` branch to the `onBlur` handler in `ExpenseRow` in `src/components/ExpenseList.jsx` that calls `setDraft(String(expense.amount))` to revert the input to the actual amount when the entered value is invalid.

---
