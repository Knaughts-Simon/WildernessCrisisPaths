# Stellaris Mod: Wilderness Crisis Paths

## Mod Config

- **Mod name**: Wilderness Crisis Paths
- **Stellaris install**: `C:\Program Files (x86)\Steam\steamapps\common\Stellaris`
- **Version file**: `<stellaris_install>\launcher-settings.json` (field: `rawVersion`)
- **Error log**: `%USERPROFILE%\Documents\Paradox Interactive\Stellaris\logs\error.log`
- **Main branch**: `main`
- **Mod tag format**: `v{mod_version}` (e.g. `v1.0.0`)
- **supported_version format**: `v{major}.{minor}.*` (e.g. `v4.3.*`)
- **File prefix**: `94_wildcrisis`
- **Namespace**: `wildcrisis`

## Version Bumping Rules

- **Minor bump** (1.0.0 -> 1.1.0): Each Stellaris version compatibility update, even if no mod code changes.
- **Patch bump** (1.1.0 -> 1.1.1): Bug fixes in the mod itself.
- Never bump major unless author explicitly says to.

## Vanilla Update Procedure

This is the full workflow for updating the mod after a Stellaris update. Run it as a conversation — do not silently execute everything.

### Phase 1: Detect Stellaris Version

1. Read `rawVersion` from `<stellaris_install>\launcher-settings.json`.
2. Parse the major.minor from it (e.g. `v4.3.1` -> `4.3`).
3. Check the current `supported_version` in `descriptor.mod`.
4. If the major.minor hasn't changed (just a patch bump like 4.2.3 -> 4.2.4), inform the author. They may want to skip the full process and just tag. Ask before continuing.

### Phase 2: Check for Breaking Changes

5. This mod relies on the following vanilla APIs/triggers. Check whether they still exist in the current Stellaris install:
   - `is_wilderness_empire` scripted trigger (in `common/scripted_triggers/00_scripted_triggers.txt`)
   - `is_player_crisis` scripted trigger (in `common/scripted_triggers/05_scripted_triggers_biogenesis.txt`)
   - `ap_become_the_crisis` ascension perk (in `common/ascension_perks/00_ascension_perks.txt`)
   - `ap_cosmogenesis` ascension perk (in `common/ascension_perks/00_ascension_perks.txt`)
   - `add_ascension_perk` effect
   - `remove_ascension_perk` effect
   - `activate_crisis_progression` effect (called by the vanilla perks' `on_enabled`)
6. Search the Stellaris install for these identifiers. If any have been renamed, removed, or changed semantics, flag the author with the details.
7. If no breaking changes, report that the mod should be compatible as-is.

### Phase 3: QA Testing

8. Read `QA.md` for this mod's test procedure.
9. Tell the author to boot Stellaris with a playset containing ONLY this mod. Remind them of the debug commands from QA.md. Print them so they can copy-paste.
10. Begin monitoring the error log: `Get-Content -Path "<error_log_path>" -Wait -Tail 0`
    - Filter for lines that reference this mod's files or object IDs (anything prefixed `wildcrisis` or containing filenames from this repo).
    - Also watch for generic errors that appear *only* when this mod is active (the author will know their baseline).
11. As errors appear, report them with:
    - The exact error line(s)
    - Which mod file likely caused it
    - A best-guess fix
12. Repeat until the author confirms QA is clean, or until they decide remaining errors are acceptable/known.

### Phase 4: Finalize

13. Update `descriptor.mod`:
    - Set `version` to the new mod version (minor bump from current).
    - Set `supported_version` to `v{major}.{minor}.*` matching the new Stellaris version.
14. Update `CHANGELOG.md`:
    - Add a new section at the top under the header, following the existing format.
    - If no mod code changes were needed: `Confirmed compatibility with v{stellaris_version}.`
    - If changes were needed: summarize what changed and why.
15. Commit: `git commit -am "Update for Stellaris v{stellaris_version}, mod v{new_mod_version}"`
16. Tag: `git tag v{new_mod_version}`
17. Push branch and tags: `git push --tags origin main`. The author may need to do this manually if auth switching is required.
18. Remind the author to publish via the Paradox Launcher, update the Steam Workshop changelog, and update the Steam Workshop description if anything player-facing changed. Note: the launcher may show "Create" instead of "Update" — this is a known launcher quirk. As long as `remote_file_id` is set in the `.mod` files, clicking "Create" will update the existing workshop item, not create a duplicate.
19. Claude can't reliably view the page on Steam, so don't ask it to.

## Mod Architecture

This mod uses a "trampoline" pattern to allow Wilderness empires to take the Galactic Nemesis and Cosmogenesis crisis paths without overriding any vanilla files.

### How it works

1. Two trampoline ascension perks (`ap_wildcrisis_become_the_crisis`, `ap_wildcrisis_cosmogenesis`) are visible ONLY to wilderness empires.
2. When a player picks a trampoline perk, its `on_enabled` calls `add_ascension_perk` to grant the real vanilla perk (`ap_become_the_crisis` / `ap_cosmogenesis`).
3. `add_ascension_perk` fires the vanilla perk's `on_enabled`, which activates crisis progression and all downstream systems.
4. A delayed event (1 day) calls `remove_ascension_perk` on the trampoline perk, freeing the ascension slot.
5. The player ends up with the vanilla perk using exactly 1 slot. All 200+ downstream checks pass because the perk key is the real vanilla one.

### Key objects

- `ap_wildcrisis_become_the_crisis` — trampoline for Galactic Nemesis (requires Nemesis DLC)
- `ap_wildcrisis_cosmogenesis` — trampoline for Cosmogenesis (requires Machine Age DLC)
- `wildcrisis.1` / `wildcrisis.2` — hidden events that remove the trampoline perks

### Vanilla dependencies

- `ap_become_the_crisis` — vanilla Galactic Nemesis perk (granted by trampoline)
- `ap_cosmogenesis` — vanilla Cosmogenesis perk (granted by trampoline)
- `is_wilderness_empire` — scripted trigger used in `potential` blocks
- `is_player_crisis` — scripted trigger (already includes `has_country_flag = is_player_crisis` for modded paths, but we don't need it since we grant the vanilla perk)
- `add_ascension_perk` / `remove_ascension_perk` — effects used in the trampoline mechanism

## QA Procedure

During QA testing, monitor the error log in the background:
```
tail -f "$USERPROFILE/Documents/Paradox Interactive/Stellaris/logs/error.log"
```
Filter for lines containing `wildcrisis` or this mod's file names. Report any errors immediately to the user with the exact error line, which mod file likely caused it, and a best-guess fix.

See `QA.md` for the full test procedure and console commands.

## Notes

- **Localisation files must be UTF-8 with BOM.** When writing them with `printf`, always use single-quoted format strings — double quotes cause bash to escape `!` into `\!`, corrupting `§!` color reset codes and producing "Illegal break character" errors at runtime.
- The Stellaris modding language is a Paradox-proprietary scripting format. Blocks use `= { }` syntax, conditions use triggers like `NOT`, `OR`, `AND`, `NOR`, `NAND`.
- `log = "..."` statements in Stellaris script produce output in the error log. These are used for debugging and should be preserved in mod files.
- All mod files use the `94_` prefix for load ordering and `wildcrisis` as the namespace/prefix.
- When committing, do not add a Co-Authored-By line.
