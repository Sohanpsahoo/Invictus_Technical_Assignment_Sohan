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

**How to reproduce:** In a scenario where a debtor's owed amount matches a creditor's owed amount exactly, the Settle Up panel fails to display a suggested transfer between them, leaving their debts and credits unresolved.

**What is wrong:** In `src/lib/settle.js`, the `else` branch of the settlement matching loop (which executes when `d.amount === c.amount`) only advanced the loop indices `i` and `j` without pushing the settlement transfer to the `transfers` array.

**What I changed:** Refactored the settlement algorithm in `src/lib/settle.js` to compute `const payment = Math.min(d.amount, c.amount)`, correctly creating and pushing the transfer for all cases (including exact matching amounts) and decrementing remaining balances.

---

## Bug 5

**How to reproduce:**

**What is wrong:**

**What I changed:**

---
