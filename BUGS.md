# Bugs found

Add one section per issue. Bug 1 is filled in to show the format — fix it, then write what you changed. Copy the blank template for the rest.

Keep this file in the repo and **commit it** with your fixes.

---

## Bug 1

**How to reproduce:** Open the app. The expense list says “Newest first”. The first row is Wine (7 Mar). Board game (15 Mar) is further down.

**What is wrong:** The list is showing oldest expenses first. Newest should be at the top.

**What I changed:** Updated `dateValue` in `src/lib/format.js` to return the numeric timestamp (`new Date(date).getTime()`) so dates are correctly comparable, and updated the sort comparator in `src/components/ExpenseList.jsx` to sort in descending order (`dateValue(b.date) - dateValue(a.date)`) so newest expenses appear first.

---

## Bug 2

**How to reproduce:** Look at the "Balances" panel on the right. Aisha Khan has paid more than her share, but the panel displays "Aisha Khan owes $38.67". Meanwhile, Carlos Mendes has paid less than his share, but the panel displays "Carlos Mendes is owed $17.34".

**What is wrong:** The logic in `src/components/BalancesPanel.jsx` inverted the display conditions: members with positive net balances (who paid more than they spent) were shown as "owes", while members with negative net balances (who spent more than they paid) were shown as "is owed".

**What I changed:** In `src/components/BalancesPanel.jsx`, updated the condition so that positive balances (`bal > 0.005`) render `is owed ${formatMoney(bal)}` with CSS class `"owed"`, and negative balances (`bal < -0.005`) render `owes ${formatMoney(-bal)}` with CSS class `"owe"`.

---

## Bug 3

**How to reproduce:**

**What is wrong:**

**What I changed:**

---
