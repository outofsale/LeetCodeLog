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

## 2026-03-07

---

### Problem #876 — Middle of the Linked List
- **Difficulty:** Easy
- **Topic:** Linked Lists, Two Pointers

#### Solution
```cpp
ListNode* middleNode(ListNode* head) {
    auto slow = head, fast = head;
    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
}
```

#### Complexity
- **Time:** O(n) — **Space:** O(1)

#### Concepts Discussed
- **const-correctness:** mark parameter and method `const` when neither `this` nor the list is mutated
- **Explicit nullptr checks:** prefer `!= nullptr` over implicit bool conversion in low-latency codebases
- **std::exchange:** useful for move constructors and ownership transfer, not for simple pointer advancement
- **Thread safety:** `std::shared_mutex` for reader/writer lock; `std::atomic<ListNode*>` for lock-free traversal; hazard pointers for safe memory reclamation
- **Lock-free != safe:** atomics protect pointer reads but not memory reclamation
- **Branch prediction:** `[[likely]]`/`[[unlikely]]` (C++20); `__builtin_expect` pre-C++20
- **Prefetching:** `__builtin_prefetch` hides memory latency; useful in HFT, rarely in analytics
- **Cache levels:** L1 ~4 cycles, L2 ~12 cycles, L3 ~40 cycles, RAM ~200 cycles
- **Why step size 2:** gains exactly 1 on slow per iteration — guaranteed to converge on any cycle length

---

### Problem #141 — Linked List Cycle
- **Difficulty:** Easy
- **Topic:** Linked Lists, Two Pointers

#### Solution
```cpp
bool hasCycle(const ListNode* head) {
    const ListNode* slow = head;
    const ListNode* fast = head;
    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}
```

#### Complexity
- **Time:** O(n) — **Space:** O(1)

#### Concepts Discussed
- **O(1) space:** memory does not grow with n — variable count is irrelevant if fixed
- **count vs find vs contains:** `count` for existence; `find` when you need the element; `contains` (C++20) most expressive
- **Hash set is not faster:** both O(n) time; two pointers has better constant factor and cache behaviour
- **Compare after both move:** comparing before either pointer moves causes false positive

---

### Problem #206 — Reverse Linked List
- **Difficulty:** Easy
- **Topic:** Linked Lists

#### Solution
```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* cur = head;
    while (cur != nullptr) {
        auto next = cur->next;
        cur->next = prev;
        prev = cur;
        cur = next;
    }
    return prev;
}
```

#### Complexity
- **Time:** O(n) — **Space:** O(1) iterative, O(n) recursive

#### Concepts Discussed
- **Recursive alternative:** elegant but O(n) stack space — iterative wins for production
- **When recursion is good:** tree traversal, divide and conquer, tail recursion with TCO, backtracking
- **Tail call optimisation (TCO):** compiler converts tail-recursive calls to loops at -O2/-O3 — requires recursive call to be the absolute last operation
- **Recursion is bad when:** depth is O(n), no TCO, iterative is equally readable

---

### Problem #21 — Merge Two Sorted Lists
- **Difficulty:** Easy
- **Topic:** Linked Lists

#### Solution
```cpp
ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
    ListNode dummy(0);
    ListNode* tail = &dummy;
    while (list1 != nullptr && list2 != nullptr) {
        if (list1->val <= list2->val) {
            tail->next = list1;
            list1 = list1->next;
        } else {
            tail->next = list2;
            list2 = list2->next;
        }
        tail = tail->next;
    }
    tail->next = (list1 != nullptr) ? list1 : list2;
    return dummy.next;
}
```

#### Complexity
- **Time:** O(m+n) — **Space:** O(1)

#### Concepts Discussed
- **Dummy/sentinel node:** eliminates special-casing the first node — idiomatic C++ for linked list problems. Stack allocated, essentially free
- **Next follow-up:** #23 Merge K Sorted Lists — `std::priority_queue`, O(n log k)

---

### Problem #20 — Valid Parentheses
- **Difficulty:** Easy
- **Topic:** Stack, String

#### Solution
```cpp
bool isValid(std::string_view s) {
    static const std::unordered_map<char, char> match = {
        {'(', ')'}, {'{', '}'}, {'[', ']'}
    };
    std::stack<char> stk;
    for (const char c : s) {
        if (auto it = match.find(c); it != match.end()) {
            stk.push(it->second);
        } else {
            if (stk.empty() || stk.top() != c) return false;
            stk.pop();
        }
    }
    return stk.empty();
}
```

#### Complexity
- **Time:** O(n) — **Space:** O(n)

#### Concepts Discussed
- **Push expected closing bracket:** collapses matching logic to single comparison `stk.top() != c`
- **static const local:** initialised once on first call, thread-safe since C++11 (magic statics)
- **Magic statics (C++11):** hidden guard variable + double-checked locking — concurrent threads block until init completes
- **string_view:** zero-copy read-only string parameter — preferred over `const string&` in C++17
- **switch on char:** compiler generates jump table — O(1) dispatch vs O(n) chained if
- **if-with-initialiser (C++17):** `if (auto it = map.find(c); it != map.end())` — scopes iterator tightly
- **static storage duration vs static keyword:** namespace-scope variables have static duration without the keyword; adding `static` changes linkage to internal, not duration
- **Three meanings of static:** (1) internal linkage, (2) persists between calls, (3) class member shared across instances
- **Static initialisation order fiasco:** order across TUs is undefined — fix with construct-on-first-use idiom
- **Meyer's Singleton:** magic static inside `instance()` — thread-safe, lazy, no manual locking
- **Destruction order fiasco:** statics destroyed in reverse init order — avoid non-trivial destructors on globals

---


