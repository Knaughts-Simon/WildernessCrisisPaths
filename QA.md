# QA Test Procedure

## Setup

1. Launch Stellaris with only this mod in the playset.
2. Start a new game as a **Wilderness empire** (Hive Mind + Origin: Wilderness).

## Console Commands

```
debugtooltip
research_all_technologies
activate_ascension_perk ap_wildcrisis_become_the_crisis
activate_ascension_perk ap_wildcrisis_cosmogenesis
effect complete_crisis_objective = crisobj_destroy_empires
```

## Steps

1. `research_all_technologies`, pick 3 perks, verify Become the Crisis and Cosmogenesis appear.
2. Pick one. Verify crisis UI appears. Advance a day, verify trampoline removed and only 1 slot used.
3. Use `effect complete_crisis_objective` to verify menace/crisis currency is earned.
4. Check error log for `wildcrisis` — should only see the trampoline removal log lines.
