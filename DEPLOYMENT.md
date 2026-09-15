# Photo Rolodex — Deployment Guide

**This game requires Rock RMS 18.0 or later**, no exceptions. That's the version that introduced both the Lava Applications feature and the built-in `{[ dropdown ]}` Lava shortcode used on the setup screen, and this project is built exclusively against that floor.

There are two ways to deploy it. **The Lava Application build below is the current, recommended version** and is what the rest of this guide covers. A simpler single-file alternative (`photo-rolodex.lava`, a plain HTML Content block) also exists in this folder if you'd rather not set up a full Lava Application with two separate endpoints — see the bottom of this document. Both require Rock 18.0+; the fallback file is a setup-complexity choice, not a version workaround.

## Before You Start: Confirm You're on Rock 18.0+

Go to **Admin Tools > CMS Configuration**. If there is no **Lava Applications** item, you're on an older Rock version and this game won't work yet — upgrade to 18.0+ before building either version out.

## Files in This Build

| File | Goes into |
|---|---|
| `rolodex-endpoint-setup.lava` | The `setup` LavaEndpoint's CodeTemplate |
| `rolodex-endpoint-game.lava` | The `game` LavaEndpoint's CodeTemplate |
| `rolodex-host-page.lava` | A **Lava Application Content Block** (not a plain HTML Content block) on a normal Rock page |

## 1. Create the Lava Application

1. Admin Tools > CMS Configuration > Lava Applications > Add.
2. **Name:** Photo Rolodex
3. **Slug:** `rolodex` (if you use a different slug, you must update it in all three files — each one has a `gameUrl`/`setupUrl` variable near the top with `rolodex` hardcoded into the path).
4. Save.

## 2. Create the Two LavaEndpoints

Inside the Lava Application you just created, add two endpoints:

**Endpoint 1**
- Name: Setup
- Slug: `setup`
- HTTP Method: `Get`
- Enabled Lava Commands: `sql`
- Security Mode: your preference (Application View is a reasonable default for internal staff use)
- CodeTemplate: paste the full contents of `rolodex-endpoint-setup.lava`

**Endpoint 2**
- Name: Game
- Slug: `game`
- HTTP Method: `Get`
- Enabled Lava Commands: `sql`
- Security Mode: same as Endpoint 1
- CodeTemplate: paste the full contents of `rolodex-endpoint-game.lava`

The "Enabled Lava Commands" setting here is per-endpoint — it's separate from the System Settings > Lava Engine toggle that HTML Content blocks use. Both endpoints need `sql` or every `{% sql %}` block in them will fail. Neither endpoint needs anything beyond `sql` — the list of who's in the game comes from Rock's Tag system, looked up with a plain SQL join, so there's no entity command to enable.

## 3. Create the Host Page

1. Admin Tools > CMS Configuration > Pages. Pick a parent page (internal staff area recommended).
2. Add a new page, name it "Photo Rolodex" or similar. Any full-width layout works.
3. Add a block to the main zone, block type **Lava Application Content Block** — not a plain "HTML Content" block. This specific block type is what auto-registers HTMX and resolves the `^/` endpoint shorthand used in this build.
4. Set the block's **Application** setting to the "Photo Rolodex" LavaApplication you created in step 1.
5. Paste the full contents of `rolodex-host-page.lava` into the block's content editor. Save.

## 4. Tag the People Who Should Be in the Game

There is no CONFIGURATION block to edit in either endpoint file, and no group or data view IDs to keep in sync across files. Sources are **Rock Tags** — the setup screen's dropdown is built live from a SQL query against `Tag`/`EntityType`, so adding a new "list" to the game is a Rock admin/staff action, not a code edit:

1. In Rock, tag the people you want in a given list. Two ways to do this:
   - **One person at a time:** open their Person profile, click the tag icon near the top, and add (or create) a tag.
   - **In bulk:** run a Grid (e.g., a Data View or a group's member list) that includes the people you want, select them, and use the Grid's "Tag" bulk action.
2. When creating a new tag, make sure it is:
   - **Scoped to Person** (not another entity type).
   - **Organization-wide**, not personal — personal tags (owned by a single staff login) are deliberately excluded from the dropdown so one person's private tags don't clutter everyone else's list.
   - **Active** — inactive tags don't appear in the dropdown.
3. That's it. The next time someone opens the setup screen, the new tag shows up in the dropdown automatically. No endpoint files need to be touched.

This is intentionally opt-in, not "every tag in Rock." Only organization-wide, active, Person-scoped tags appear — so a tag someone created for an unrelated purpose won't accidentally show up unless it happens to meet those three conditions.

Every tag used here should only be applied to people with `PhotoId` set — people without a profile photo are automatically excluded from the deck, so a tagged person with no photo just won't appear in the game (harmless, but worth knowing if a deck looks smaller than expected).

## 5. Test

Work through each of these before considering the game ready for real use:

1. **Load the host page.** You should see a brief spinner, then the setup screen with your dropdown populated with the tags from step 4.
2. **Tap to Reveal.** Pick a list, leave mode on Tap to Reveal, hit Start. Confirm a photo loads, tapping "Tap to Reveal Name" shows the correct name, and Next advances to a new person each time until you reach the "All done" screen.
3. **A second, differently-tagged list.** Tag a different set of people with a second tag, go back to the setup screen, and confirm that tag also appears in the dropdown and plays correctly. This confirms the dropdown and deck-building query are both reading tags live rather than anything cached.
4. **Multiple Choice, right and wrong answers.** Switch mode to Multiple Choice, keep "Track score" checked, and play through a few cards — tap the correct name at least once (confirms the green highlight and the score counter increments) and tap a wrong name at least once (confirms your pick turns red, the correct name turns green, and the score does *not* increment).
5. **Play Again / Change List.** Reach the "All done" screen (with score tracking on, confirm the X/Y correct percentage displays), then try both **Play Again** (same list/mode, freshly reshuffled) and **Change List** (returns to the setup screen).
6. **More-info panel, positive case.** With "Show more about each person" checked, find a person who has a campus, connection status, household members, and/or other group memberships on file, and confirm the "More about [Name]" panel expands to show them after answering/revealing, including "Tagged since" with the date they were tagged.
7. **More-info panel, negative case.** Find (or temporarily use) a person with none of those fields filled in and confirm the panel shows "No additional information on file" instead of an empty or broken-looking panel.
8. **Checkbox off.** Uncheck "Show more about each person" on the setup screen and confirm the panel doesn't appear at all during play.
9. **Empty list.** Pick (or create) a tag with no people on it, or where every tagged person lacks a profile photo, and confirm the game shows "Nobody tagged with this list has a profile photo yet" instead of erroring.

## Optional: "More About This Person" Panel

Both `rolodex-endpoint-setup.lava` and `rolodex-endpoint-game.lava` (and the fallback file) include an optional, collapsed-by-default panel that appears after a card is answered/revealed, showing campus, connection status, how long the person has been tagged with the list being played, household members, and up to 3 other non-family groups they belong to. It's controlled by a "Show more about each person" checkbox on the setup screen (default checked) and threaded through the game via a `minfo` query param, the same way the score-tracking checkbox works. No extra CMS Configuration setup is required — it's built entirely into the two endpoint files you're already pasting in.

If you'd rather exclude a particular group type from the "also part of" list (by default it excludes Family-type groups), the exclusion is a hardcoded GroupType GUID (`790E3215-3B10-442B-AF69-616C0DCB998E`, Rock's built-in Family group type) near the bottom of the "more-info lookups" section in `rolodex-endpoint-game.lava` — update it there if needed.

## Notes on How This Version Works

- Almost no JavaScript. Every interaction is either a native HTML element (the `<details>/<summary>` flip card and the "more about this person" panel) or an HTMX link, with one exception: the List field on the setup screen uses Rock's built-in `{[ dropdown ]}` Lava shortcode with `longlistenabled:'true'` for a searchable, enhanced dropdown. That shortcode's own markup includes a small inline `<script>` that initializes jQuery's Chosen plugin, which is why it works reliably even though the setup screen is delivered as an HTMX-swapped fragment rather than a normal page load. The fallback file (`photo-rolodex.lava`) uses the same shortcode for the same field, since this project targets Rock 18.0+ exclusively and that version floor already guarantees the shortcode exists.
- The host page renders the `setup` endpoint's content directly into `#rolodex-app` using Rock's `{% renderlavaendpoint %}` Lava command, so the setup screen is part of the page's own initial HTML — no spinner, no extra HTTP round trip for that first screen. Every link the two endpoints render afterward still targets that same div via `hx-get`, so the rest of the game plays out with normal HTMX swaps — no `hx-select` trick needed, because a LavaEndpoint's response is already just the raw fragment (Rock doesn't wrap it in page chrome at all). Unlike LavaEndpoints, the Lava Application Content Block does not need `renderlavaendpoint` separately enabled anywhere.
- Game state (deck order, current index, score) is still carried entirely in the query string of each request, same design as the fallback file, just read via the `QueryString` merge field instead of the `PageParameter` filter (Lava Endpoints don't expose `PageParameter`).
- **Trade-off vs. the fallback file:** a LavaEndpoint's own URL (`/api/v2/lava-app/1/rolodex/game?...`) is a bare API path with no site header, footer, or styling. Pushing that into the browser's address bar (like the fallback file did with `hx-push-url`) would mean a page refresh mid-game shows an unstyled raw fragment instead of your site. So this version does **not** update the address bar — refreshing the browser always returns to the setup screen, and there's no shareable "resume this exact game" URL. If you want that back, it's possible but needs an extra layer (the host page reading its own query string on load, and the game endpoint setting an `HX-Push-Url` response header back to the host page's URL instead of the API URL) — ask if you want that built out.
- Every link in this version uses `href="#"` with the real navigation handled by `hx-get`, instead of a working `href` as a fallback (like the plain-block version had). That's intentional: the "real" URL behind each link is the bare API endpoint, which would be a worse experience than a dead link if HTMX somehow failed to load, since it dumps an unstyled fragment with no way back.

## Alternative: Plain HTML Content Block

This is not a version workaround — it also requires Rock 18.0+. If you'd rather not set up a full Lava Application (one LavaApplication, two LavaEndpoints, a host page), `photo-rolodex.lava` in this folder is a complete, self-contained alternative: one file, one HTML Content block, no CMS Configuration setup beyond enabling `sql` Lava commands globally (Admin Tools > System Settings > Lava Engine). It has real `href` links (so it degrades to plain navigation instead of dead links if HTMX fails) and preserves progress across a page refresh, at the cost of not being a "real" Lava Application. It uses the same Tag-based sourcing as this version — tag people in Rock, and they show up in the dropdown — and the same `{[ dropdown ]}` shortcode for the enhanced List field. It has been through the same bug-check and Rock UI conventions pass.
