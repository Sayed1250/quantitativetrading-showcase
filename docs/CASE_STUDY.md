# Case study: real incidents and how they were solved

Each one follows the same pattern: **see the failure → collect evidence → find the cause in the code → make the smallest
fix → prove the failure is gone and nothing else changed.**

---

## 1. The night the bot stopped

**Seen.** No daily report arrived, and the scheduled jobs on the Mac seemed not to run.

**Evidence.** The system logs showed three separate problems on the same night:
1. A **router outage** while a VPN tool was managing DNS, so name lookups failed.
2. **macOS privacy rules** (TCC) denied the job launcher access to the external drive where the bot lived, so the jobs
   could not start.
3. The **backup** could not read back a database that was in WAL mode, so its own check failed.

**Fixes.**
- The jobs were redesigned so the system launcher **opens nothing on the external drive**: the program, the working
  folder and the logs are all on the internal disk.
- Backups now copy databases with **SQLite's backup API**, which is safe in WAL mode, and read every row back before
  accepting the copy.
- The day's close was run by hand, on time, and the ledgers were verified.

**Lesson.** "It worked yesterday" is not a test. Operating-system rules, networks and disks fail independently, so each
needs its own check and its own alert.

---

## 2. A button that did nothing

**Seen.** In the new operator app, pressing **Start** showed an empty message and nothing happened.

**Evidence.** The jobs folder had no job files, and the logs folder did not exist. So Start had failed before installing
anything. Reproducing Start read-only gave `paper-start: unknown command`.

**Cause.** The app copies its own runtime to the internal disk on first install. Opening a **newer build over an older
install** did not refresh that copy, so the new window was talking to old code that didn't know the new command. A
second bug hid the error: failed requests returned no message, so the page showed an empty bubble.

**Fix.** The runtime is brought up to date at every launch (never while a job is running), and every failure shows a
plain reason. A new test installs an old build, opens a new one over it, and checks that Start works with the ledgers
unchanged byte for byte.

---

## 3. An "unreliable" test that was perfectly reliable

**Seen.** "Check the code" failed once with *1 failed, 834 passed*. A manual rerun minutes later passed.

**Hypothesis 1, rejected:** load from parallel container builds made a timing-sensitive test flaky.

**Evidence.** A loop reproduced the failure on its **first** pass. The failing test checked that the bot refuses to
install jobs whose program lives on an external drive. It failed **every time** the Python running the tests was on the
external drive, which happened only when the app was opened from the build folder on that drive.

**Fix.** The certification now runs with the app's own Python on the internal disk. The full output of every
certification is kept, and failures name their tests. The test itself was made independent of where Python lives.

**Lesson.** "Flaky" is a hypothesis, not a diagnosis.

---

## 4. A false alarm every new install would have shown forever

**Seen.** On a fresh install, the safety checker reported the books and its own copy of the account differing by about
one to three dollars, and the gap only grew.

**Evidence.** The gap moved only on days a position was closed, by exactly the funding that position had collected
**before** the install.

**Cause.** Installs start from a compact backup that keeps only the newest saved state. The checker never learned about
the funding booked before the install, so each later close came out short by that amount.

**Fix, without loosening the tolerance.** The checker now starts from the ledger as it stands on the install day. That
"beginning" is decided **once**, so it cannot grow later, and a saved state missing **after** it raises a warning instead
of being skipped. Otherwise a future fault could have made the checker compare the ledger with itself and always say OK.
Result on the real data: **−0.4462 / −1.0943 → 0.0000**. The owner's own checker was byte-identical before and after.

---

## 5. "RESUME doesn't work"

**Seen.** After pausing from Telegram, pressing Start and typing `RESUME` left the app showing *paused*.

**Evidence** (from the records file): pause at 09:44:57, the executor's hourly check at 09:45:07 recorded *close-only*,
and RESUME at 09:46:13 **did** clear the pause. But the status kept showing what the executor had last confirmed, until
its next run an hour later.

**Fix.** After RESUME, the app runs that check once and waits for the bot's confirmation. In LIVE it does not run the
order-sending job early, by the owner's decision. Instead it says when the next check will confirm.

**Lesson.** The earlier tests had set the flag directly and skipped the executor. They missed the bug because they took a
shortcut the real system never takes.

---

## 6. A real-money risk found by asking "what happens on day one?"

**Question.** When a person switches to live trading, does their empty account copy the positions the model already
holds?

**Finding (read from the code).** Yes. On its first day the executor opened **every** position the model held, at that
day's prices, which the model never paid.

**Decision and fix.** A person's real account starts as **cash only**, and only **new** entries from their first day are
copied. Closes of positions they never held do nothing. The owner's own rehearsal mode was proven byte-identical in four
setups.

**Lesson.** The most valuable tests are often questions about the edges: the first day, the last day, a restart, an
upgrade.
