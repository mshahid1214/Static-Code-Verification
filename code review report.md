# Task 2: Code Review Report

## Static Code Verification (SVV Lab)

### Project: Monster Battle Game (`game.cpp`)

### Objective
To identify defects in the supplied C++ program, explain why each defect is problematic, recommend a solution, and assign a severity level.

## 1. Code Review Report

| Line Number | Defect Type | Why It Is a Problem | Proposed Solution | Severity |
|---|---|---|---|---|
| 41–42 | Array Bounds / Capacity | `spawnMonster()` adds monsters without checking whether the array is full. The program creates six monsters even though `MAX_MONSTERS` is 5. | Check `monsterCount < MAX_MONSTERS` before adding a monster. | Critical |
| 47 | Logical Error | `monsters[i]->id = id` assigns the ID instead of comparing it. This can modify monster IDs and return the wrong monster. | Replace `=` with `==`. | Critical |
| 54 | Null Pointer Dereference | `findMonster()` can return `nullptr`, but `attackMonster()` accesses `m->health` without checking it. Calling it with ID 99 can cause a crash. | Check `if (m == nullptr)` before accessing the monster. | Critical |
| 55–58 | Arithmetic / Signed-Unsigned Error | `damage` is converted to `unsigned int`. Subtracting it from signed health can produce unexpected results, and the negative-health check is unreliable. | Validate that damage is non-negative and use safe signed arithmetic. | High |
| 64–65 | Loop Boundary Error | The loop uses `i <= monsterCount`, so it accesses one element beyond the active monster range. | Use `i < monsterCount`. | Critical |
| 71–73 | Dangling Reference | `getMonsterLabel()` returns a reference to the local variable `label`. The variable is destroyed when the function returns. | Return `std::string` by value. Also check whether `m` is null. | Critical |
| 77, 79 | Uninitialized Variable | `sum` is not initialized before values are added to it, producing an indeterminate result. | Initialize it with `double sum = 0.0;`. | High |
| 80 | Division by Zero | `averageAttack()` divides by `monsterCount`, which can be zero. | Check whether `monsterCount == 0` before calculating the average. | High |
| 85–86 | Use After Free | `defeatMonster()` deletes the monster and then accesses `m->name`. Accessing a deleted object is undefined behavior. | Save the name before deletion and remove the monster safely from the array. | Critical |
| 90–91 | Arithmetic Precision | Both operands are integers, so division by 100 uses integer arithmetic before conversion to `double`. | Convert an operand to `double` before division. | Medium |
| 94–99 | Missing Return Statement | `playerRank()` does not return a value when `score <= 100`. Using its result in `main()` is undefined behavior for that path. | Add a return statement for all possible scores. | High |
| 102, 105 | Buffer Overflow | The dynamically allocated character buffer holds only 10 characters, including the terminating null character. A longer input can overflow it. | Use `std::string` and `std::cin >> name`. | High |
| 109 | Incorrect Memory Deallocation | Memory allocated using `new char[10]` is released using `delete` instead of `delete[]`. | Use `delete[] name`, or avoid manual allocation by using `std::string`. | High |
| 121 | Memory Leak | The `bonus` pointer owns a dynamically allocated `Monster`, but the object is never deleted. | Use a smart pointer such as `std::unique_ptr`, or release the allocated memory appropriately. | Medium |
| 133 | Shallow Copy / Double Deletion | Copying `Game` copies its raw pointers. Both objects can delete the same monsters, resulting in double deletion and undefined behavior. | Implement safe copy semantics or use smart pointers and suitable ownership design. | Critical |
| 35–38, 85–86 | Memory Management | `defeatMonster()` does not remove the deleted monster from the array or reduce `monsterCount`. The destructor can later delete the same pointer again. | Remove the entry from the array, update the count, and maintain clear ownership. | Critical |

## 2. Corrected Code Snippets

### Fix 1: Correct the Comparison

```cpp
Monster* findMonster(int id) {
    for (int i = 0; i < monsterCount; i++) {
        if (monsters[i]->id == id) {
            return monsters[i];
        }
    }
    return nullptr;
}
```

### Fix 2: Prevent Array Overflow

```cpp
void spawnMonster(int id, std::string name,
                  int health, int attack) {
    if (monsterCount >= MAX_MONSTERS) {
        std::cout << "Monster limit reached.\n";
        return;
    }

    monsters[monsterCount] =
        new Monster(id, name, health, attack);
    monsterCount++;
}
```

### Fix 3: Handle Missing Monsters and Damage Safely

```cpp
int attackMonster(int id, int damage) {
    Monster* m = findMonster(id);

    if (m == nullptr || damage < 0) {
        return -1;
    }

    m->health -= damage;

    if (m->health < 0) {
        m->health = 0;
    }

    return m->health;
}
```

### Fix 4: Correct Total Health

```cpp
int totalMonsterHealth() {
    int total = 0;

    for (int i = 0; i < monsterCount; i++) {
        total += monsters[i]->health;
    }

    return total;
}
```

### Fix 5: Return the Label Safely

```cpp
std::string getMonsterLabel(int id) {
    Monster* m = findMonster(id);

    if (m == nullptr) {
        return "Monster not found";
    }

    return "Monster-" + std::to_string(m->id)
           + ":" + m->name;
}
```

### Fix 6: Initialize the Sum and Handle Empty Games

```cpp
int averageAttack() {
    if (monsterCount == 0) {
        return 0;
    }

    double sum = 0.0;

    for (int i = 0; i < monsterCount; i++) {
        sum += monsters[i]->attack;
    }

    return static_cast<int>(sum / monsterCount);
}
```

### Fix 7: Correct Damage Calculation

```cpp
double calculateDamage(int attack, int armorPercent) {
    return static_cast<double>(attack)
           * armorPercent / 100.0;
}
```

### Fix 8: Add a Return for Every Rank

```cpp
int playerRank(int score) {
    if (score > 1000) {
        return 1;
    } else if (score > 100) {
        return 2;
    }

    return 3;
}
```

### Fix 9: Use a Safe Boss Name

```cpp
void readBoss(Game& game) {
    std::string name;
    int health;

    std::cout << "Enter boss name: ";
    std::cin >> name;

    std::cout << "Enter boss health: ";
    std::cin >> health;

    game.spawnMonster(100, name, health, 50);
}
```

**Important:** These snippets correct individual defects, but they are not a complete replacement for `game.cpp`. The memory ownership and copying problems in `Game` also require a proper design change.

## 3. Severity Levels

- **Critical:** Can cause crashes, invalid memory access, or serious memory corruption.
- **High:** Can produce incorrect results, undefined behavior, or unsafe input handling.
- **Medium:** Can cause resource leaks or incorrect calculations in particular situations.

## 4. Conclusion

The static verification of `game.cpp` reveals several defects in array handling, pointer safety, arithmetic, function return values, input handling, and memory management.

The most serious issues include the incorrect assignment in `findMonster()`, array overflow in `spawnMonster()`, null pointer access in `attackMonster()`, returning a dangling reference, use after deletion, and shallow copying of the `Game` object.

These defects should be corrected before the program is considered reliable. A complete fix should also ensure that every dynamically allocated monster has a clear owner and is deleted exactly once.
