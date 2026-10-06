# Case Study: Building a Finance Tracker with AI

**My role:** product owner, requirements, testing and validation
**Built with:** Claude Code (AI coding assistant)
**Timeline:** one extended working session, iterating on real data

## The problem

I wanted one place to see income, spending, budgets, savings goals and a debt payoff plan across checking, savings and several credit cards. My requirements:

- **Private:** financial data stays on my own computer.
- **Works with what banks actually provide:** CSV downloads for some accounts, only PDF statements for others, plus receipts from email and apps.
- **Low maintenance:** categorization should learn, not require hand-sorting every transaction.

## How I worked

I treated the AI like a development team I was directing. I set the requirements, reviewed each result, tested it against **real exports from my own accounts**, and reported problems back as specific issues to fix. It's the same loop I run in IT operations: define the need, verify the delivery, find the root cause when something breaks, and confirm the fix.

## Issues found in real data, and how they were resolved

Most of the value came from testing with real files instead of sample data. Each of these would have silently corrupted the numbers:

| What I observed | Root cause | Fix |
|---|---|---|
| Checking transactions showed "DEBIT"/"CREDIT" instead of merchant names | The bank's CSV has a *Details* column before *Description*; column detection picked the first match | Prioritized exact "Description" headers; added bank-specific cleanup of ACH payment strings into readable "who and what" |
| Card imports looked less detailed than the bank website | The website enriches data the export doesn't include; purchase dates were buried at the end of descriptions | Extracted the real purchase date; kept the original text searchable; used the bank's own category column as a fallback |
| Imported balances never changed | Account balances were only set manually | Balance = last confirmed statement balance + transactions since; warnings when no starting balance exists |
| Card payments counted as "income" | Money into a credit card looks like income without context | Any credit to a debt account is classified as a payment or a return, never income; fixed existing records with a one-time migration |
| A gas station fill-up was classified as food | The fuel receipt printed pump, gallons and price on separate lines from the dollar amount | Reader now recognizes multi-line fuel blocks; any fuel on a receipt means Gas |
| One card's export came back empty for older dates | Bank-side export limit (verified: recent ranges worked) | Clear "no transactions in this file" message; multi-file PDF statement import as the fallback |
| Interest showed as a credit, payments as charges (caught in testing before import) | A safety check that flips card signs misfired on files with separate Debit/Credit columns | Limited the check to single-amount files; re-tested with both banks' card formats |

## Operational practices built in

- **Data integrity:** duplicate detection across overlapping downloads, preview-before-commit on every import, amounts that must add up before a split is saved.
- **Change management:** versioned, non-destructive migrations. New rules merge into existing data without undoing choices the user already made.
- **Backup and recovery:** autosave to a user-chosen file, full backup/restore, clear warnings when storage isn't protected.
- **Security and privacy:** no server, no uploads, external libraries loaded only when needed. The public version was scrubbed of personal merchants and locations before publishing.
- **Verification:** every change was exercised in the browser against realistic data before being called done, and fixes were re-tested against the data that exposed them.

## What I took away

- AI makes building fast; **judgment and testing** make it correct. The bugs above were found by checking the output against reality, not by writing more code.
- Real-world data is messy in the same ways vendor data is messy at work: inconsistent formats, truncated fields and silent export failures. Designing for that is an operations skill.
- Clear, specific problem reports ("this column is wrong, here's the file") got better results than vague requests, just like working a vendor ticket.
