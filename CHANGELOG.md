# Changelog

All notable changes to this project are documented in this file.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), versioning is major.minor.hotfix.

## [1.60.1] - 2026-07-13

Teron fork of Atlas-CFM (upstream v1.60), with one critical fix.

### Fixed
- **Login disconnect**: `DataIndex.lua` automatically rebuilt the addon's full search index
  5 seconds after every `PLAYER_ENTERING_WORLD`, regardless of whether the addon was ever
  opened that session. Building that index walked every loot table in the entire game
  (all raids, dungeons, professions, quests) and, for profession recipes/enchants, forced
  a hidden-tooltip `SetHyperlink` query for each item's ID - including items the character
  had never seen in the world. Querying an unseen item this way is exactly what triggers
  the post-1.10 anti-exploit protection that disconnects the client. Fixed by:
  - Removing the automatic post-login index build entirely - every real consumer of the
    index (tooltips, wishlist, search, options) already builds it lazily on first actual
    use, so nothing is lost.
  - Guarding the remaining item-name resolution in `GetNamesFromID` to only use
    `GetItemInfo` (client-side cache, never touches the server) instead of forcing a
    `SetHyperlink` scan - an item not yet cached is simply skipped rather than queried.

### Changed
- Rebranded to "Teron's Atlas" / `TeronAtlas` folder per Terongorus's addon conventions.
  Internal `AtlasCFM.Name` identifier and the hardcoded `IsAddOnLoaded("Atlas-CFM")` /
  `arg1 == "Atlas-CFM"` checks were updated to match, as was `AtlasCFM.PATH`/`MAPPATH`
  (previously hardcoded to `Interface\AddOns\Atlas-CFM\`, feeding dozens of texture calls
  across the whole addon - left unfixed this would have silently broken almost all of
  Atlas's art). Saved-variable names (`AtlasCFMOptions`, `AtlasCFMCharDB`), the internal
  `AtlasCFM.*` Lua namespace/frame names, and the `/atlascfm` slash command are
  deliberately left unchanged so existing settings carry over.
- Removed the "report missing content" notice box/link that used to sit in the main
  frame's header - it always pointed at upstream's issue tracker regardless of which
  codebase was actually running.
- Version-check/broadcast module (`AtlasVersion.lua`): switched to a distinct broadcast
  abbreviation (`TRA` instead of upstream's `ATW`) so this fork's version announcements on
  the shared `LFT` channel never get cross-compared against independently-versioned
  upstream Atlas-CFM/Atlas-TW installs. Also fixed the version-number parser, which only
  understood two-segment `major.minor` strings, to handle this fork's three-segment
  `major.minor.hotfix` versions.
- Swept all remaining user-visible "Atlas-CFM" text (key binding names, minimap tooltip,
  first-run welcome popup, a chat debug message) and doc-comment headers across all 5
  locales to read "Teron's Atlas" instead.
