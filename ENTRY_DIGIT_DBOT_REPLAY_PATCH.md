# Entry-Digit DBot Replay — corrected cadence patch

This patch corrects the DBot replay model and adds an exact forced-entry-digit replay engine.

## Contract cadence

A DBot run is **opening tick T -> settlement tick T+1**. The settlement tick is consumed and can never also be the next opening tick. The next eligible opening is **T+2**.

There is no artificial cooldown between contracts. The legacy `waitTicks` option is retained only for API compatibility and is ignored by the exact simulator.

Therefore:

- `2,3,4,1` represents two consecutive contract slots only if `2` and `4` are the two opening ticks being tested: `2->3`, `4->1`.
- Four successful DBot runs require eight consumed ticks: `T0->T1`, `T2->T3`, `T4->T5`, `T6->T7`.
- A settlement digit that happens to equal the configured entry digit is still only a settlement and cannot trigger the next trade.

## New engine

`simulateBotForEntryDigit()` answers:

> If I choose entry digit X as my trigger, how does the DBot perform when entering through X?

`simulateAllEntryDigits()` evaluates all 0–9 for a fixed OVER/UNDER candidate. It preserves the DBot's real barrier map, recovery ladder, payout, and one-tick settlement while forcing the opening trigger to the tested digit.

The result includes trade count, wins/losses, win rate, expectancy, four-win runs, longest win/loss streak, drawdown, peak stake, fresh/recovery rates, and a chronological trade ledger. It also reports a held-out final-window replay for out-of-sample context.

## Important integration note

This patch intentionally does **not** replace the existing Sentinel ranking or invent a second ranking. The new replay is entry-specific evidence. A caller should use it to validate/select the entry digit for an already-selected OVER/UNDER candidate.

## Six-stage entry selection hierarchy

The entry point now uses the following authoritative order without creating a second market ranking:

1. Existing entry engines generate evidence for digits 0–9.
2. The exact DBot replay is run independently for every digit for the selected OVER/UNDER contract.
3. Digits with insufficient or poor DBot replay evidence are rejected and cannot become the preferred entry digit.
4. Strongly validated DBot replay survivors are preferred over merely validated survivors.
5. Existing statistical entry evidence (the pre-existing entry-digit score, Wilson lower bound, stability, context, psychology, pressure, operator evidence, etc.) breaks ties/order among DBot-replay survivors.
6. The selected survivor is exposed as `EntryPointReport.preferred` and can be displayed alongside the surfaced signal.

No fallback to a rejected digit is permitted. If no digit passes the DBot validation gate, `preferred` is null and the report remains `UNVALIDATED`; the system does not manufacture an entry digit.

### DBot validation thresholds

- Replay must contain at least 20 completed contracts.
- Held-out replay must contain at least 8 completed contracts.
- Overall and held-out win rate must be no worse than 5 percentage points below the contract's configured theoretical probability, with non-negative expectancy.
- `STRONGLY_VALIDATED` additionally requires at least 40 completed contracts, at least 16 held-out contracts, both win rates at or above theoretical, positive expectancy, and longest loss streak <= 4.

These are validation thresholds, not profitability guarantees.
