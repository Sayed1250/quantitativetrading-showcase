# Testing and verification

The testing goal is simple: **the bot must do exactly what the research measured, and every change must prove that it
does.**

## Layers

| Layer | What it proves |
|---|---|
| **Unit and module tests** | Each rule and each piece of accounting does what the specification says |
| **Parity tests** | The bot's signals equal the research code's on the same data |
| **Truncation tests** | A decision on day *D* uses no data from after *D* (no look-ahead) |
| **Replay to the cent** | Several years of history through the bot's own code, with final balances equal to the research exactly |
| **Ledger replay** | The ledgers can be rebuilt from their own records with identical results |
| **Fail-closed tests** | Missing or bad data, an uncertified build, or an unreadable file all mean *no new positions* |
| **Safety scan** | No order endpoint, key or secret can be reached outside the one allowed path |
| **Message fingerprints** | 241 fingerprints of every message the owner receives; a change that should not touch them must leave all 241 equal |
| **Paired runs** | The same day run with the old code and the new code on copies of live data; the files must be byte-identical |
| **Cross-implementation check** | The bot's hand-written Ed25519 verifier is compared with a well-known library on 1,000 random signatures (all accepted) and 4,000 tampered ones (all refused) |
| **Container tests** | The Fedora package installs, self-checks, runs one paper hour and uninstalls inside a Fedora container |

## Gates

- **Certification before trading.** After every commit on `main`, the full suite and the replay run on the real machine.
  Until they pass, the service opens no new positions.
- **Certification before pushing.** A pre-push hook refuses any commit that the certificate does not name.
- **Certification on every operator's computer.** The app tests each update locally before switching to it. The
  compiled edition runs a compiled self-check of the replay instead, because it ships no test sources.

## Discipline

- Tests check **invariants and outcomes**, not the implementation restated.
- Tests are never deleted or skipped to get a green run. A harness fault is fixed separately and labelled as one, so it
  can't be confused with a defect in the application.
- **One problem at a time**: show the failure, find the cause in the code, make the smallest fix, prove the failure is
  gone, and prove that nothing else changed.

## A few numbers (November 2026)

- About **900** tests in the CI offline suite; about **420** in the certification run
- About **120** tests in the operator app, and about **60** in the compiled edition
- **0.00** difference in the replay finals after every merged change
