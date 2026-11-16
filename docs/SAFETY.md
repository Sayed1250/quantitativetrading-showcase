# Safety design

Real money is involved, so every safety rule assumes something **will** go wrong: a bug, a lost connection, a mistyped
key, a stolen phone, a tampered file.

## Paper by default

- The HTTP layer refuses order endpoints, authenticated calls and keys unless an install was explicitly set up for its
  own exchange.
- Even then, only the **executor** may use them, and only an **allowlist of exact method and path pairs** per exchange.
  Redirects are never followed.

## The Real trading switch

- One local switch per machine, **OFF by default**, stored outside the repository.
- It is turned **ON only at the machine**, by a person typing the confirmation: never from Telegram, SSH or a script.
- It can be turned **OFF anytime**, including from Telegram. With positions open, OFF means *close-only*.
- *(In progress)* The executor sends real orders only when LIVE was entered **the proper way**, as recorded in its own
  ledger. Editing configuration files by hand will not be enough.

## Exchange account checks

Before real trading, the app verifies:
- the API key **cannot withdraw** and is **bound to one IP address**,
- the account uses **isolated margin** at **1x**,
- the computer's clock agrees with the exchange's,
- the account holds only the expected cash.

The recommended setup is a **sub-account** for the bot, so its money and positions never mix with the person's own.

## Off-switches

| Command | Effect |
|---|---|
| `/closeonly` | no new positions; open ones are still managed and closed by the rules |
| `/kill` | the same, at once, for everything |
| `/realoff` | turns real trading off on that computer |

A phone can **stop** the bot but never **restart** it. Restarting needs the app on the bot's computer and a typed word.

## Signed builds

- Every file of a compiled build is fingerprinted, and the fingerprint is signed with an **Ed25519** key that never leaves
  the owner's Keychain. A recovery key (being set up) is kept offline.
- The bot opens new positions only when every file matches, the signature is valid, the build is for that kind of
  computer, and the self-check has passed on that computer.
- The updater refuses unsigned, wrongly signed, damaged and **older** versions.

## Secrets

- Keys live in one file per machine, readable only by its user. They are never logged, printed or copied into a backup,
  and a backup that contains any key value is refused.
- Messages show at most the last four characters of a secret.

## Honest limits

- Compiled code is very hard to read, but not impossible. Function names stay visible.
- Someone at the computer could put an older signed build back by hand. This is tracked as an open item to close
  before operators trade live.
- Trades are visible on Telegram and on the exchange by nature.
