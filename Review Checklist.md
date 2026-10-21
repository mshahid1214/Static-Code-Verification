# Task 1: Review Checklist

## Static Code Verification (SVV Lab)

### Project: Monster Battle Game (`game.cpp`)

### Objective
To inspect the C++ source code without executing it and identify potential defects in initialization, pointers, arrays, loops, arithmetic, control flow, function calls, and classes.

## Review Checklist

| No. | Category | Review Finding | Result |
|---|---|---|---|
| 1 | Initialization | `sum` in `averageAttack()` is not initialized before use. | Issue |
| 2 | Pointers (Null/Dereference) | `attackMonster()`, `getMonsterLabel()`, and `defeatMonster()` do not safely handle a null pointer returned by `findMonster()`. | Issue |
| 3 | Arrays / Buffers | `spawnMonster()` can exceed the fixed array capacity. `readBoss()` also uses an unsafe character buffer, and `totalMonsterHealth()` accesses an invalid array index. | Issue |
| 4 | Loops | `totalMonsterHealth()` uses `i <= monsterCount`, causing an extra iteration. | Issue |
| 5 | Arithmetic | `attackMonster()` mixes signed and unsigned arithmetic. `averageAttack()` can divide by zero, and `calculateDamage()` performs integer division. | Issue |
| 6 | Control Flow | `findMonster()` uses assignment instead of comparison. `playerRank()` does not return a value for every possible score. | Issue |
| 7 | Function Calls | `defeatMonster()` accesses a deleted object, and `readBoss()` uses an incorrect form of `delete`. | Issue |
| 8 | Classes | The `Game` class owns raw pointers but does not implement safe copy construction or copy assignment. Copying a `Game` can cause double deletion. | Issue |

## Summary

- Total categories reviewed: 8
- Categories with issues: 8
- Categories marked OK: 0

### Conclusion

The static review identifies defects involving pointer safety, memory management, array bounds, initialization, arithmetic, and control flow. These defects should be corrected before the game is considered reliable.

**Note:** This checklist is based on the supplied source code. The results represent static inspection findings and do not require program execution.
