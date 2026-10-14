# By the numbers

Everything below was measured on 2026-10-14 from the app's repository and the records it keeps. The first commit is dated 2026-05-21, so the figures cover 146 days. Counts of lines are of source files only, with generated and fixture files left out, and are rounded in the text but exact in the tables.

## Work

| Measure | Count | Note |
|---|---|---|
| Commits | 1,457 | about ten a day, every day |
| Architecture decision records | 163 | numbered, dated, immutable once accepted |
| User stories with acceptance criteria | 107 | across 30 story files |
| Multi-slice plans completed | 15 | each with findings, behaviour clauses and decisions |
| Runbooks | 8 | deploy, restore, receipts cutover, screenshots, subscriptions, test users, VoiceOver, database access |
| Lines of project documentation | 27,695 | architecture, schema, decisions, stories, runbooks |

## Code

| | Files | Lines |
|---|---|---|
| Swift, the two packages and the app | 260 | 60,552 |
| Swift tests | 190 | 45,794 |
| Go, the server | 37 | 10,931 |
| Go tests | 45 | 15,330 |
| SQL, the server's 17 migrations | 34 | 670 |

The phone's own schema has been migrated 39 times, every migration additive, and a test upgrades a frozen version-4 database through all of them.

## Tests

| Layer | Count |
|---|---|
| Swift package tests | 1,322 |
| App unit tests | 512 |
| UI tests | 169, in 33 classes |
| Go tests | 398, of which 47 run against a real Postgres |
| **All** | **2,401** |

Every layer runs on every change, and the UI suite runs as three parallel clones on a simulator pool of its own. [Testing](testing.md) describes each.

## The app

| Measure | Count |
|---|---|
| Split modes | 4: equal, percentage, custom, itemized |
| Currencies | any ISO 4217 currency, with its own number of decimals |
| Sales-tax regions in the rate table | 82: Canada's 13, every US state and DC, and 18 other countries |
| Cats in the picker | 96, plus the one your name makes |
| Build number of the current release | 120, version 1.0.0, 2026-10-12 |
| Builds uploaded since the build ledger began on 2026-09-24 | 14 |

## How the numbers are measured

Commits are counted on the main branch. Decision records are the numbered files in the decisions directory. Stories are the distinct story ids across the story files. Tests are counted as test functions in the source: Swift Testing's `@Test` attributes in the packages and the app, XCTest methods in the UI target, and `Test` functions in the Go packages. Lines are counted by `wc` over the Swift and Go source trees, tests separately, with no generated code in either. The same commands are kept in a private runbook so that this page can be refreshed in a few minutes.
