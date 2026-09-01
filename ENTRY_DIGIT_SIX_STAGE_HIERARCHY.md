# Entry-Digit Six-Stage Hierarchy

This amendment sits after the existing Sentinel candidate/cell selection and does not create a second market ranking.

1. **Existing entry engines generate candidates/evidence.** The existing digit score, conditional transition evidence, stability, pressure/lifecycle, psychology, Entry-Condition Lab, operator learning, immediate guidance and Markov context remain intact.
2. **Exact DBot replay validates every digit.** Digits 0–9 are replayed with the exact DBot opening/settlement cadence: opening `T`, settlement `T+1`, next possible opening `T+2`. Settlement is never reused as an opening.
3. **Poor replay digits are rejected.** A digit with insufficient sample or replay performance below the DBot validation hurdle cannot become the preferred entry digit.
4. **Strong replay survivors are preferred.** `STRONGLY_VALIDATED` replay survivors outrank merely `VALIDATED` survivors.
5. **Existing statistical evidence breaks ties.** Among replay survivors, the existing entry-engine score and evidence resolve the ordering; this preserves all the pre-existing entry intelligence without allowing a rejected digit back into contention.
6. **One final entry digit is exposed.** `EntryPointReport.preferred` is the single selected entry digit for the already-selected cell. If no digit survives replay validation, `preferred` is null and the entry point is `UNVALIDATED` rather than falling back to a rejected digit.

## Validation policy

- Minimum completed DBot replay contracts: 20.
- Minimum held-out contracts: 8.
- Overall and held-out win rate: at least `theoretical - 0.05`, never below 50%.
- Expectancy must be non-negative.
- Strong validation additionally requires 40+ completed contracts, 16+ held-out contracts, overall and held-out win rates at or above theoretical, positive expectancy, and longest loss streak <= 4.

These are engineering validation gates, not a guarantee of future profitability.
