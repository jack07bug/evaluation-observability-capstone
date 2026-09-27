# Perturbation Log

For each system, make one deliberate change to an input or configuration, predict the outcome, run
it, and record what actually happened. See the starters in the Instructions, or design your own (your
own experiment earns more credit).

---

### System 1 — validated, routed pipeline

- **Change I made (file + what I changed):** Used the targeted missing-source edge case in `tests/test_us01_retry.py`, where a required field is null/missing from the source.
- **Command I ran:** `.venv/bin/pytest tests/test_us01_retry.py -v`
- **What I predicted:** The missing-source case should halt immediately instead of retrying.
- **What actually happened (paste the key output line):** `tests/test_us01_retry.py::test_ac_01_04_missing_source_halts_immediately PASSED`
- **How this differs from the unperturbed run:** The normal well-formed extraction path can continue/retry on validation failures, while the missing-source case is handled by stopping immediately.

---

### System 2 — schema-enforced two-pass extraction

- **Change I made (file + what I changed):** Used the bundled `income_sum_mismatch.txt` case with an intentionally inconsistent stated total and contrasted it with the clean `appraisal_informal_sqft.txt` fixture.
- **Command I ran:** `.venv/bin/mortgage-extract fixtures/documents/income_sum_mismatch.txt --mode replay`
- **What I predicted:** The arithmetic validator should detect the mismatch and report a discrepancy.
- **What actually happened (paste the key output line):** `"consistent": false` with `"calculated": 9642.17`, `"stated": 10892.17`, and `"delta": -1250.0`
- **How this differs from the unperturbed run:** The appraisal fixture returned `"consistent": true` with no discrepancies, while the mismatched income fixture was flagged.

---

### System 3 — multi-source synthesis

- **Change I made (file + what I changed):** Added the `--simulate-timeout` option to force the logistics source to fail.
- **Command I ran:** `.venv/bin/supply-chain-investigate meridian --offline --simulate-timeout`
- **What I predicted:** The investigation should complete and mark information from the failed logistics source as incomplete rather than aborting.
- **What actually happened (paste the key output line):** `> Sources unavailable: logistics unavailable (timeout)`
- **How this differs from the unperturbed run:** The normal run included logistics data and a contested on-time-delivery metric; the timeout run completed with logistics unavailable and moved affected information into the Incomplete section.
