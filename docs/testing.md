# Testing

An expense app earns trust with arithmetic that is always right and sync that never loses a change. Both are the kind of thing a test can prove and a demo cannot. two cats has four layers of automated tests, about 2,400 in all as of 2026-10-14, plus a few harnesses for the things only a device can measure. This page is what each layer covers and how it is kept fast enough to run on every change.

## The four layers

| Layer | Runs against | Count | Typical case |
|---|---|---|---|
| Swift package tests | The domain and persistence packages, with in-memory SQLite | 1,322 | The netting engine, the split allocators, a migration, a service's transaction |
| App unit tests | The app target, hosted in a test process | 512 | A view model, a hosted view's measured frame, a VoiceOver tree |
| UI tests | The real app on a simulator, through a hermetic seam | 169 in 33 classes | A flow from tap to saved row, a screen's hit regions |
| Go tests | The server packages, 47 of them against a real Postgres | 398 | A push's locking, a pull's visibility, the receipt mapping |

The package tests are the foundation and the fastest. The domain package has no database and no network, so the netting engine's fifteen mandatory cases, the split rounding sweeps over every total from one to three hundred cents, the recurring scheduler's month ends and leap days, the tax spread and the item allocator all run as pure functions. The persistence package opens an in-memory SQLite per test and exercises the services exactly as the app does, transaction by transaction. A frozen copy of the version-4 database is upgraded through every migration and compared, row for row and schema for schema, with a fresh one.

The app unit tests cover what needs the app's own types. Many of them host a view in a hosting controller and measure it: a 28-point visual under the hit-target modifier measures at least 44 by 44, a destructive icon button at least 48, a numeric field row at least 44 tall at the default and the largest accessibility size. Others switch accessibility on in-process and walk the resulting tree, which is how the card stack's VoiceOver behaviour is pinned, since UI tests cannot invoke custom actions.

## The hermetic seam

The UI tests drive the real app, but not against the real world. A launch argument puts the app into a hermetic mode: an in-memory database, an in-memory secure store instead of the Keychain, a URL protocol that answers every network request from stubs, and a seeded, onboarded account. Further arguments seed a scenario: a debt to settle, a failed outbox row, twenty contacts so the list overflows, a scanned receipt with printed tax lines, a database that fails to open. The demo dataset the marketing screenshots use is built the same way, through the real services rather than SQL, so a screenshot is also a test of the write path.

Speed is a discipline. Every wait is a tenth-of-a-second poll with a short timeout, never a fixed sleep or a framework expectation that re-checks after a second. A negative check runs only after a positive anchor. UIKit animations are off in the hermetic mode, SwiftUI springs still run, and the marketing tests keep motion on. The suite runs as three parallel clones on a simulator pool of its own, locked so that another session installing the app cannot kill a test host mid-run, and the script prints the ten slowest tests after each run so regressions in time are visible.

## The hit-region audit

One UI test class exists to enforce a design rule. For each main screen, Home, People, a group page, a person, Add Expense in every split mode, Settings, the recurring list and detail, and the receipt review in each of its layouts, it runs Apple's accessibility audit for hit regions and then a stricter check of its own: one snapshot of the screen, every on-screen button, link, field, switch, slider, stepper, segmented control or picker, and a failure for any frame under 44 points in either dimension, with half a point of tolerance for pixel rounding. A screen fails once, naming every finding. The exceptions, Apple's own 36-point bar items, the switch nested in a toggle, live in one allow list with a reason each, and a unit test fails if any entry has no reason.

## Convergence

Sync is tested as a conversation between devices. A convergence test builds two or three in-memory databases, each a complete phone with its own services, applier and identity map, records on one, pulls through a fake server into the others, and asserts that every ledger agrees: the same balance on every phone, a settlement that reached both sides, a void that reached a former link, a group page that shows the same numbers for three members. The same fixture runs the restore path, so a phone rebuilt from the log matches the phone that wrote it.

On the server side, 47 integration tests run against a real PostgreSQL in Docker, gated on an environment variable so the ordinary `go test` skips them. They cover what SQLite cannot stand in for: the row locks a push takes against a racing unlink, the deadlock a wrong lock order would cause, the held-back page when a long transaction is still in flight, the cursor reset after a restore, and the visibility arms a pull reads through. Packages truncate the shared database, so they run serially.

## Golden tests and fixtures

Some contracts are frozen on purpose. The receipt JSON the server returns is checked against a golden file, because shipped apps decode it. Money formatting has a table of expected strings per currency and locale. The pixel-cat generator dumps fixtures that the website's TypeScript port is tested against byte for byte, so the two can never drift. And the cat picker has a performance test with a 50-millisecond budget for a page, which is how its page size was chosen.

## On a device

Some things only a phone can tell you. A measurement harness replays the same card-stack gestures on a physical iPhone and records frame hitches through Instruments, so each change to the stack engine can be compared against a baseline. The marketing pipeline runs the demo dataset on an iPhone 17 Pro Max simulator in light and dark, captures the screenshots the website uses, records the App Store preview clips, writes out a hundred generated cats, and composes the icon, in about twenty minutes from one command. Every screenshot in this repository came from it.
