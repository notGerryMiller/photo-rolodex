# Photo Rolodex

A "guess the name from the photo" game for Rock RMS, built with Lava and HTMX. It helps churches turn name-learning into a fast, repeatable habit for greeters, small group leaders, new staff, and volunteers.

Almost no JavaScript is used. Interactions are either native HTML (`<details>/<summary>` flip cards) or HTMX partial swaps, so photos and names refresh in place without a full page reload or a JavaScript framework.

## Requirements

Rock RMS 18.0 or later, no exceptions. That version introduced both the Lava Applications feature and the built-in `{[ dropdown ]}` Lava shortcode this project depends on.

## How it's sourced

There is no hardcoded list of people, groups, or data views to configure. Who's in the game is driven entirely by **Rock Tags**: tag the people you want (one at a time from a Person profile, or in bulk from a Grid), and any organization-wide, active, Person-scoped tag shows up automatically in the game's setup dropdown. Adding a new list is a Rock admin/staff action, not a code change.

People without a profile photo are automatically skipped, so the deck never shows a blank or placeholder image.

## Two ways to deploy

1. **Lava Application (recommended).** One Lava Application with two LavaEndpoints (`setup` and `game`) plus a host page using the Lava Application Content Block. Files: `rolodex-endpoint-setup.lava`, `rolodex-endpoint-game.lava`, `rolodex-host-page.lava`.
2. **Single-file fallback.** `photo-rolodex.lava` is a self-contained alternative that runs as a single plain HTML Content block, no Lava Application setup required. It preserves progress across a page refresh and degrades to real `href` navigation if HTMX fails, at the cost of not being a "real" Lava Application.

Both options require Rock 18.0+, use the same Tag-based sourcing, and have been through the same bug-check and Rock UI conventions pass.

Full setup steps, testing checklist, and configuration notes are in [DEPLOYMENT.md](DEPLOYMENT.md).

## Features

1. Two play modes: Tap to Reveal, and Multiple Choice with optional score tracking.
2. An optional "more about this person" panel showing campus, connection status, how long they've been tagged, household members, and other group memberships.
3. Live sourcing from Rock Tags, so lists can be added or changed without touching code.

## License

MIT. See [LICENSE](LICENSE).
