# Handover

- Name: Candidate
- Email used for this application: candidate@example.com
- Chosen track: Track A
- Why this track (one or two sentences): To showcase full-stack debugging and software repair skills across Python and JavaScript, tackling real-world small business reporting flaws.
- Approximate total time, including setup and handover: 2 hours

## Run and verify

Requires standard Python 3.10+ installation.
```
cd track-a
python app.py
```
To run tests:
```
python -m unittest discover -s tests -v
```

## What I delivered

I fixed the 6 seeded defects disrupting the application’s integrity:
1. **Partial Imports failing**: Moved row validation inside the transaction loop in `importing.py` so one bad row doesn't reject the whole file. 
2. **Duplicate Invoices**: `storage.py` lacked existence checks. Added logic to skip identical existing invoices or reject those with colliding IDs but different details.
3. **Misallocations**: Removed amount-only fallback matching in `matching.py` strictly enforcing `(customer_id, invoice_number)` matching.
4. **Export truncating cents**: Changed `int(item[key] * 100) / 100` to `round(item[key], 2)` in `reporting.py` so CSV exports match UI.
5. **Open Invoice Filter**: Corrected the filter map in `reporting.py` which was mapping `open` status to `paid`.
6. **UI swallows server response**: Updated `app.js` to parse HTTP responses, display rejected counts/errors, and no longer claim success on `400 Bad Request`.

### Improvement: Clear File Input After Import
Added `fileInput.value = '';` in `app.js` after a successful import. This is a crucial UX fix preventing the user from accidentally submitting the identical file multiple times, which avoids unnecessary backend validation spam.

## Evidence and limits

- **Failing-before / Passing-after Reproduction**: Before fixing `app.js`, uploading an invalid file reported "Import complete. Your records are ready." and silently swallowed rejection errors. Now, the frontend properly displays the exact rejected lines and reason.
- **Changed-input case**: Attempting to upload an identical invoice CSV previously inflated the total outstanding balance due to duplicates. Now, it reports `skipped: 1` and keeps balances intact.
- **Existing-register check**: I ensured `restore_fixture.py --replace` preserves existing IDs. The repair does not override or drop tables, it maintains schema compatibility while strengthening constraints and Python-layer matching.

## Tools and judgment

- **Parsing Response**: I decided to directly use `response.json()` in Javascript because the backend properly formats errors as JSON, avoiding ugly DOM parsing or blind success states.
- **Rounding Logic**: `int(amount * 100) / 100` failed due to IEEE-754 precision issues (truncation vs rounding). I switched to Python's builtin `round(x, 2)`, preserving the intended financial precision accurately for standard floating point use without rewriting everything to `decimal.Decimal` under time constraints.
