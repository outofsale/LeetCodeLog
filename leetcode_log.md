# LeetCode Practice Log

---

## 2026-03-06

### Problem #876 — Middle of the Linked List
- **Difficulty:** Easy
- **Topic:** Linked Lists, Two Pointers
- **Language:** C++17

#### Approach
Tortoise-and-hare slow/fast pointer.

#### Complexity
- **Time:** O(n) — **Space:** O(1)

#### Key Takeaways
- Explicit `!= nullptr` checks preferred over implicit bool conversion.
- `const`-correctness on method and pointer.
- `std::exchange` is for ownership transfer, not simple pointer advancement.
- Thread safety follow-up: `std::shared_mutex` / `std::atomic<ListNode*>`.

---
