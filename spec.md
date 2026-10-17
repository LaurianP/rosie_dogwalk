# spec.md — Rosie's Dog Walking quote page

Sources: the client brief (quoted below), `CLAUDE.md`, and a clarification interview held on 2026-10-17. The pricing rules below are settled. The only open items are Rosie's real values (§7), which go in the pricing block and don't change any behaviour.

> "Rosie's Dog Walking. People keep texting me asking how much it would cost and I have to work it out by hand every time. Rate depends on number of dogs (2nd dog is 50% off, not free). Same-day booking adds a surcharge. Holiday walks are double. No stacking discounts. I want one simple page with a calculator on it — customers can use it themselves, and I can pull up the same page when someone texts me."

## 1. Goal
One public page where anyone can get the price of one standard walk in a few seconds, worked out by Rosie's rules. That way Rosie never has to calculate a quote by hand, and customers and Rosie always get the same number.

## 2. User scenarios
**S1 — Customer quoting for themselves.** Sam has two dogs and wants a walk this afternoon. Sam opens the page on a phone, sets dogs to 2 and ticks "Walk is today". The total and a line-by-line breakdown appear at once, with no submit button. Sam reads the "How pricing works" note, sees why the price is what it is, and taps the booking line to text Rosie.

**S2 — Rosie answering a text.** Rosie gets a text: "How much for my 2 dogs on Christmas Day?" She opens the same page (no login, no special owner view), sets dogs to 2 and ticks "Holiday walk". She sees that Christmas Day is on her holiday list and texts back the total shown.

**S3 — Customer with too many dogs.** Pat has 5 dogs. The + button stops at 4, and a note says to text Rosie for a custom quote.

**S4 — Rosie changing her prices.** Rosie raises her base rate or edits her holiday list. One value changes in the labelled pricing block at the top of the script, and every price, label and note on the page reflects it. Nothing else in the file needs editing.

## 3. Functional requirements

### Pricing block
- **FR1** One labelled block at the top of the script holds every business value: `BASE_RATE`, `SAME_DAY_PERCENT`, `MAX_DOGS`, `CURRENCY`, `WALK_LENGTH` (e.g. "30-minute"), `HOLIDAYS` (a list of holiday names), and `PHONE`. Every number, label and note on the page that mentions these values is generated from the block, so none of them is typed into the HTML twice.

### Page
- **FR2** The page shows, top to bottom:
  1. the heading "Rosie's Dog Walking";
  2. a line saying what is being quoted: "Price for one `WALK_LENGTH` walk";
  3. the calculator inputs;
  4. the total and breakdown;
  5. the estimate note;
  6. "How pricing works";
  7. the booking line.

  There is no other content and no navigation.
- **FR3** The page works at 360px wide with no horizontal scrolling, and every control can be tapped with a thumb.

### Inputs
- **FR4** *Number of dogs (same household)*: a whole-number control with − / + buttons. It starts at 1. − is disabled at 1 and + is disabled at `MAX_DOGS`. At `MAX_DOGS` a note appears below it: "More than `MAX_DOGS` dogs? Text Rosie for a custom quote." The note is hidden below the max.
- **FR5** *Walk is today*: an on/off toggle labelled with the surcharge, e.g. "Walk is today (+25%)". Off by default. "Same-day" means the walk happens on the day it is booked, whatever the hour.
- **FR6** *Holiday walk*: an on/off toggle labelled "Holiday walk (2×)". Off by default. Directly beneath it, the page lists the holidays from `HOLIDAYS` as plain text. There is no date picker; the user decides whether the toggle applies.

### Pricing rules
Each step is applied in order. Every amount is rounded to a **whole dollar** (halves round up) at the moment it is calculated, and later steps use the rounded figures.

- **FR7 Dogs (normal day).** The first dog costs `BASE_RATE`. Every additional dog costs 50% of `BASE_RATE`, rounded. The second dog is never free.
- **FR8 Dogs (holiday).** On a holiday walk there is **no multi-dog discount**: every dog costs `BASE_RATE`. This is what "no stacking discounts" means.
- **FR9 Holiday doubling.** On a holiday walk, the dog subtotal is doubled. The breakdown shows this as one added amount equal to the subtotal.
- **FR10 Same-day surcharge.** If "Walk is today" is on, add `SAME_DAY_PERCENT` of the running subtotal (after any holiday doubling), rounded. On a holiday walk booked same-day, the surcharge is therefore worked out on the doubled price.
- **FR11 Total** = the sum of the breakdown lines. The total is never rounded separately from the lines, so the lines always add up to the total shown.

### Output
- **FR12** The total and breakdown update as soon as any input changes. There is no "Calculate" button, and the quote for the defaults (1 dog, no toggles) shows the moment the page loads.
- **FR13** The breakdown lists one line per dog, then holiday, then same-day. Lines that don't apply are hidden, not shown as $0.
  - Normal day: "1st dog $20", then "2nd dog (50% off) $10", "3rd dog (50% off) $10", and so on.
  - Holiday: "1st dog $20", "2nd dog $20", … with no "50% off" label, then "Holiday rate (2×) +$40".
  - Same-day: "Same-day (+25%) +$X".
- **FR14** All amounts show `CURRENCY` and whole dollars only, with no cents (e.g. `$38`, never `$37.50` or `$38.00`).
- **FR15** Under the total: "Quote is an estimate; Rosie confirms your price when you book."
- **FR16** "How pricing works" states the rules in plain words, using values from the pricing block:
  - first dog `BASE_RATE`, each additional dog half price;
  - holiday walks are double, with no multi-dog discount;
  - walks booked for today add `SAME_DAY_PERCENT`;
  - prices are rounded to the nearest dollar.
- **FR17** Booking line: "Text Rosie at `PHONE` to book". The number is an `sms:` link, so it can be tapped on a phone.

## 4. Edge cases & rules
- **Holiday + same-day** both apply: full-price dogs, doubled, then the percentage of the doubled amount.
- **Rounding.** Halves always round up. Each line is rounded when it is calculated, and the surcharge is worked out on the already-rounded subtotal. Example (base $27, 2 dogs, today): $27 + $14 (13.50 rounded) = $41, plus 25% of $41 = 10.25, which rounds to $10. Total $51.
- **Dog count** can never be 0, negative, a fraction, blank, or above `MAX_DOGS`. If the field is typeable, invalid input snaps back to the nearest valid value.
- **Same household.** The dog count label says "same household". Separate households sharing a walk each get their own quote. The page can't check this; the label is the rule.
- **Reload** resets to the defaults. Nothing is remembered between visits.
- **Offline / from disk.** The page works when opened as a local file with no internet connection.

## 5. Out of scope
- Booking, scheduling or availability.
- Payment.
- Accounts, login, or a separate owner/admin view.
- Saving, copying or emailing quotes.
- A holiday calendar, date picker or automatic same-day detection.
- Choosing a walk length, or pricing multiple walks in one quote.
- Other services (sitting, boarding).
- Maps or service areas.
- Analytics.
- Anything that needs a backend, database, external API or framework (`CLAUDE.md`).

## 6. Acceptance criteria
The placeholder pricing block is `BASE_RATE = 20`, `SAME_DAY_PERCENT = 25`, `MAX_DOGS = 4`, `CURRENCY = "$"`. With those values, the page shows exactly:

| # | Dogs | Today | Holiday | Breakdown | Total |
|---|------|-------|---------|-----------|-------|
| A1 | 1 | off | off | 20 | **$20** |
| A2 | 2 | off | off | 20 + 10 | **$30** |
| A3 | 3 | off | off | 20 + 10 + 10 | **$40** |
| A4 | 1 | on | off | 20 + 5 | **$25** |
| A5 | 3 | on | off | 20 + 10 + 10 + 10 | **$50** |
| A6 | 2 | off | on | 20 + 20 + 40 | **$80** |
| A7 | 2 | on | on | 20 + 20 + 40 + 20 | **$100** |
| A8 | 4 | on | on | 20 + 20 + 20 + 20 + 80 + 40 | **$200** |

These tests change the pricing block and reload the page:
- **A9** Set `BASE_RATE = 25`. A2 then shows 25 + 13 = **$38**.
- **A10** Set `BASE_RATE = 27`. 2 dogs + today then shows 27 + 14 + 10 = **$51**.
- **A11** Set `SAME_DAY_PERCENT = 30`. The toggle label, "How pricing works" and A4 (20 + 6 = **$26**) all update, with no other edits.

Behaviour checks:
- **A12** On load, before any interaction, the page already shows A1's quote.
- **A13** Changing any input updates the total and breakdown with no button press.
- **A14** At 1 dog, − is disabled. At 4 dogs, + is disabled and the "More than 4 dogs? Text Rosie…" note is visible. At 3 dogs the note is hidden.
- **A15** A6's breakdown shows no "50% off" labels. A2's breakdown shows "2nd dog (50% off)".
- **A16** A1's breakdown shows exactly one line. No amount anywhere on the page shows cents.
- **A17** The holiday list from `HOLIDAYS` is visible under the holiday toggle. The estimate note, "How pricing works" and the tappable `sms:` booking line are present.
- **A18** At 360px wide there is no horizontal scroll. Opened from disk with Wi-Fi off, the page works and the console shows no errors.

## 7. Open items (values only — no behaviour depends on them)
Rosie's real values for these, which replace the placeholders in the pricing block:
- `BASE_RATE`
- `SAME_DAY_PERCENT`
- `MAX_DOGS`
- `CURRENCY`
- `WALK_LENGTH`
- `HOLIDAYS` (her holiday list)
- `PHONE`
