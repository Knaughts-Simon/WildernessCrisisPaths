# QA Test Procedure

## Setup

1. Launch Stellaris with only this mod in the playset.
2. Start a new game as a **Wilderness empire** (Hive Mind + Origin: Wilderness, requires BioGenesis DLC).
3. Open the console with `~`.

## Console Commands

```
debugtooltip
research_all_technologies
activate_ascension_perk ap_wildcrisis_become_the_crisis
activate_ascension_perk ap_wildcrisis_cosmogenesis
```

- `debugtooltip` — enables debug tooltips. Hover over perks/objects to see internal IDs.
- `research_all_technologies` — unlocks all techs so ascension perks become available.
- `activate_ascension_perk` — directly grant a perk (useful if you don't want to pick 3 perks first).

## Test Cases

### 1. No errors in log
- **Monitor**: `Get-Content -Path "$env:USERPROFILE\Documents\Paradox Interactive\Stellaris\logs\error.log" -Wait -Tail 0`
- **Filter for**: `wildcrisis` — expect only the log messages confirming trampoline removal, no errors.
- **Also watch for**: Generic errors that only appear when this mod is active.

### 2. Become the Crisis (Nemesis path)
- Start as Wilderness empire.
- `research_all_technologies`, then open ascension perk picker.
- **Verify**: "Become the Crisis" appears in the list (it shows vanilla name via loc reference).
- Select it.
- **Verify**: Crisis progression UI appears (menace bar).
- Advance 1 day (spacebar or console `fast_forward 1`).
- **Verify**: The trampoline perk is gone — look in the perk list, you should see `ap_become_the_crisis`, NOT `ap_wildcrisis_become_the_crisis`.
- **Verify**: Only 1 ascension perk slot was consumed.
- **Verify**: Menace objectives work — destroy an enemy ship, check menace increases.

### 3. Cosmogenesis path
- Start as Wilderness empire.
- `research_all_technologies`, then open ascension perk picker.
- **Verify**: "Cosmogenesis" appears in the list.
- Select it.
- **Verify**: Crisis progression UI appears.
- Advance 1 day.
- **Verify**: Trampoline perk removed, `ap_cosmogenesis` present, 1 slot consumed.
- **Verify**: Cosmogenesis technologies appear in research options.

### 4. Non-Wilderness regression
- Start as a normal (non-Wilderness) empire.
- `research_all_technologies`, then open ascension perk picker.
- **Verify**: The vanilla "Become the Crisis" and "Cosmogenesis" perks appear as normal.
- **Verify**: The trampoline perks (`ap_wildcrisis_*`) do NOT appear.

### 5. Mutual exclusion
- As Wilderness, pick "Become the Crisis".
- **Verify**: "Cosmogenesis" (the trampoline version) no longer appears in the perk picker (because `is_player_crisis` is now true).
