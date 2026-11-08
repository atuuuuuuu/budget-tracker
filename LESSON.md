# Family budget: how it works, and how to build your own

Open it with `open ~/learning/budget-tracker/index.html`. Everything is saved in your browser, on this computer only.

## 1. The data: one list

Everything is one object, saved to `localStorage` as JSON:

```js
data = {
  entries: [ { id: 1731112345678, type: "out", amount: 32000000, category: "Food", note: "groceries", date: "2026-11-09" } ],
  limits: { "2026-11": 40000000 },     // monthly spending limit, per month
  currency: "MNT",
};
```

Everything on screen (totals, bars, the list) is **calculated from this list every time**, never stored
separately. That way the totals can never get out of sync with the entries.

## 2. The parts worth understanding

**Money is stored in cents, not decimals.** Computers store decimals in binary, which can't represent 0.1
exactly: in JavaScript `0.1 + 0.2` is `0.30000000000000004`. Add up a year of shopping and the total drifts.
So `parseMoney` turns `"12.50"` into the whole number `1250`, all maths is on whole numbers, and only
`money()` divides by 100 to display. Banks do the same.

**Tögrög are whole numbers.** The screen shows ₮ without decimals, so if you could type 45,000.50 the totals
on screen wouldn't add up (the tests caught this bug). So for ₮, decimals are rejected with a clear message.

**Filtering by month.** Dates are stored as text like `"2026-11-09"`, so "this month" is just
`e.date.startsWith("2026-11")`. That date format also sorts correctly as plain text.

**Totals with reduce.** `sum = (list) => list.reduce((total, e) => total + e.amount, 0)` walks the list,
adding as it goes. Income minus spending is what's left.

**Category bars.** Spending is grouped into an object (`{ Food: 500000, Transport: 45000 }`), sorted biggest
first, and each bar's width is its share of the biggest category.

**Never trust typed text (security).** If you put what someone typed into the page with `innerHTML`, a note
like `<img src=x onerror="alert(1)">` would *run as code*. That's called XSS (cross-site scripting). This app
only ever puts typed text in with `textContent`, which shows it as plain text. The tests try exactly that note.

**Delete with Undo, not "Are you sure?".** Deleting is instant, and a little bar offers Undo for 6 seconds.
Faster than a confirm box, and mistakes are free.

**Export to CSV.** The entries become text rows (`date,type,category,note,amount`). Text containing quotes or
commas is wrapped in quotes, and quotes inside are doubled (`""`), which is the CSV rule. A `Blob` turns the
text into a file and a temporary link downloads it. It opens in Excel or Google Sheets.

**Forms.** The form handles `submit`, so pressing Return in any field adds the entry. `preventDefault()` stops
the browser reloading the page, which is what forms normally do.

## 3. Build your own (new empty folder, one step at a time)

1. A form with amount and note, and an Add button that puts entries in a list on the page.
2. Store entries in an array and draw the list from it with a `render()` function.
3. Save and load with `localStorage` (`JSON.stringify` / `JSON.parse`).
4. Totals for income, spent and left. Try storing decimals first and adding 0.1 + 0.2 to see the bug,
   then switch to cents.
5. Categories and a month picker.
6. Delete. Then try putting a note with `<b>hi</b>` in it using `innerHTML` and see what happens.
   Then fix it with `textContent`.
7. Export to CSV.
