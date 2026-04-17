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



## 2026-03-08

---

### Problem #704 — Binary Search
- **Difficulty:** Easy
- **Topic:** Binary Search

#### Solution
```cpp
int search(const vector<int>& nums, int target) {
    int left = 0, right = static_cast<int>(nums.size()) - 1;
    while (left <= right) {
        const auto mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        if (nums[mid] < target) left  = mid + 1;
        else                    right = mid - 1;
    }
    return -1;
}
```

#### Complexity
- **Time:** O(log n) — **Space:** O(1)

#### Concepts Discussed
- **do-while bug:** executes body before checking condition — unsafe for empty input. Standard while handles empty naturally via `left <= right` evaluating false immediately
- **Overflow-safe mid:** `left + (right - left) / 2` — naive `(left + right) / 2` overflows for large indices
- **static_cast<int>(nums.size()):** size() returns unsigned — subtracting 1 from 0 wraps to SIZE_MAX. Cast first to avoid unsigned underflow
- **Lower-mid vs upper-mid:** lower-mid = `left + (right-left)/2`, upper-mid = `left + (right-left+1)/2`. Choice is irrelevant in standard search (both boundaries always move past mid) but critical in boundary-finding variants
- **Why upper/lower mid matters:** when a boundary is set to mid itself (not mid±1), wrong mid choice causes infinite loop in two-element window. Rule: `right=mid` → must use lower-mid; `left=mid` → must use upper-mid
- **Four binary search templates:**
  - Template 1: standard, both boundaries move past mid, mid choice irrelevant
  - Template 2: find leftmost, `right=mid`, must use lower-mid
  - Template 3: find rightmost, `left=mid`, must use upper-mid
  - Template 4: open interval sentinels (`left=-1, right=size`), neither boundary=mid, mid choice irrelevant — natural for continuous/floating point domains
- **Recursive binary search:** pass left/right indices, not sub-vectors. Sub-vector copy is O(n) total — defeats the purpose. Index recursion is tail-recursive → TCO applies at -O2/-O3
- **#34 Find First and Last Position:** combines Template 2 and Template 3. Early exit after findLeft returns -1 avoids unnecessary findRight call. Alternative: lowerBound trick — call lowerBound(target) and lowerBound(target+1)-1

---

### Problem #226 — Invert Binary Tree
- **Difficulty:** Easy
- **Topic:** Binary Tree, DFS, BFS

#### Solution
```cpp
TreeNode* invertTree(TreeNode* root) {
    if (root == nullptr) return root;
    auto left  = invertTree(root->left);
    auto right = invertTree(root->right);
    root->left  = right;
    root->right = left;
    return root;
}
```

#### Complexity
- **Time:** O(n) — **Space:** O(h), h=tree height

#### Concepts Discussed
- **Post-order vs pre-order:** both correct for this problem — inversion is symmetric, order of swap relative to recursion does not matter
- **std::swap:** `std::swap(root->left, root->right)` more expressive than manual temporaries for pre-order variant
- **TCO does not apply to two recursive calls:** TCO requires a single tail call with no pending work. Any function with two recursive calls is inherently O(h) stack space
- **Iterative BFS:** uses `std::queue`, processes level by level, swaps each node. O(w) space where w=max width
- **Iterative pre-order with stack:** push right before left so left is processed first (LIFO). Correctly labelled pre-order, NOT classic DFS inorder — inorder iterative would corrupt traversal after swap
- **Inorder iterative broken here:** swapping at a node corrupts `cur->right` pointer used for traversal continuation
- **⚠️ TODO — REVISIT:** DFS and BFS traversal orders (pre-order, in-order, post-order) iteratively and recursively — #94, #144, #145 are the dedicated problems for this

---

### Problem #104 — Maximum Depth of Binary Tree
- **Difficulty:** Easy
- **Topic:** Binary Tree, DFS, BFS

#### Recursive Solution
```cpp
int maxDepth(TreeNode* root) {
    if (root == nullptr) return 0;
    return 1 + std::max(maxDepth(root->left), maxDepth(root->right));
}
```

#### Iterative DFS Solution
```cpp
int maxDepth(TreeNode* root) {
    if (root == nullptr) return 0;
    std::stack<std::pair<TreeNode*, int>> stk;
    stk.push({root, 1});
    int maxDep = 0;
    while (!stk.empty()) {
        auto [node, depth] = stk.top(); stk.pop();
        maxDep = std::max(maxDep, depth);
        if (node->right) stk.push({node->right, depth + 1});
        if (node->left)  stk.push({node->left,  depth + 1});
    }
    return maxDep;
}
```

#### Iterative BFS Solution
```cpp
int maxDepth(TreeNode* root) {
    if (root == nullptr) return 0;
    std::queue<TreeNode*> q;
    q.push(root);
    int depth = 0;
    while (!q.empty()) {
        depth++;
        int levelSize = static_cast<int>(q.size());  // snapshot before inner loop
        for (int i = 0; i < levelSize; i++) {
            TreeNode* node = q.front(); q.pop();
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
    }
    return depth;
}
```

#### Complexity
- **Time:** O(n) — **Space:** O(h) DFS stack, O(w) BFS queue

#### Concepts Discussed
- **std::max over ternary:** `std::max(a, b)` clearer than `a > b ? a : b` — identical machine code
- **Pair with depth:** `std::pair<TreeNode*, int>` carries depth alongside node — avoids reconstructing depth from traversal
- **Brace initialisation over make_pair:** `stk.push({node, depth+1})` cleaner than `stk.push(std::make_pair(node, depth+1))` — compiler infers type from stack template parameter
- **C++17 structured bindings:** `auto [node, depth] = stk.top()` — idiomatic, scoped cleanly
- **Push right before left:** in LIFO stack, right pushed first means left processed first — gives left-to-right DFS order
- **stk.top() then stk.pop() — why split:** `pop()` returns void deliberately — returning by value would be exception-unsafe if copy constructor throws. Always copy via `top()` before calling `pop()`
- **BFS levelSize snapshot:** `int levelSize = q.size()` captured before inner loop — freezes node count at current level before children are enqueued. Core pattern for level-order traversal
- **BFS level-order template:** used in #102 Level Order Traversal, #111 Minimum Depth, #199 Right Side View
- **⚠️ TODO — REVISIT:** DFS and BFS traversal orders (pre-order, in-order, post-order) iteratively and recursively — #94, #144, #145

---

## 2026-03-09

---

### Problem #100 — Same Tree
- **Difficulty:** Easy
- **Topic:** Binary Tree, DFS, BFS

#### Recursive Solution
```cpp
bool isSameTree(const TreeNode* p, const TreeNode* q) {
    if (p == nullptr || q == nullptr) return p == q;
    if (p->val != q->val) return false;
    return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
}
```

#### Iterative Solution
```cpp
bool isSameTree(TreeNode* p, TreeNode* q) {
    if (p == nullptr || q == nullptr) return p == q;
    std::stack<std::pair<TreeNode*, TreeNode*>> stk;
    stk.push({p, q});
    while (!stk.empty()) {
        auto [l, r] = stk.top(); stk.pop();
        if (l->val != r->val) return false;
        if (l->right != nullptr && r->right != nullptr) stk.push({l->right, r->right});
        else if (l->right != r->right) return false;
        if (l->left != nullptr && r->left != nullptr)  stk.push({l->left, r->left});
        else if (l->left != r->left)   return false;
    }
    return true;
}
```

#### Complexity
- **Time:** O(n) — **Space:** O(h)

#### Concepts Discussed
- **Collapsed null check:** `if (p == nullptr || q == nullptr) return p == q` handles both-null and one-null cases in one line — cleaner than two separate checks
- **Stack invariant — only push non-null pairs:** avoids null dereference inside loop without extra null check per iteration. Cleaner than pushing null pairs and using continue
- **Short-circuit &&:** right subtree only checked if left matches — correct instinct
- **Lambda for repeated logic:** repeated null-check pattern can be extracted to a local lambda in production code to reduce duplication
- **Top-level null guard is necessary:** even in iterative version — stack pushes {p,q} unconditionally, loop dereferences immediately. Guard cannot be removed
- **Foundation problem:** #100 is a direct building block for #101 Symmetric Tree, #572 Subtree of Another Tree

---

### Problem #572 — Subtree of Another Tree
- **Difficulty:** Easy
- **Topic:** Binary Tree, DFS

#### Solution
```cpp
bool isSubtree(const TreeNode* root, const TreeNode* subRoot) {
    if (subRoot == nullptr) return true;
    if (root == nullptr)    return false;
    if (root->val == subRoot->val && isSameTree(root, subRoot)) return true;
    return isSubtree(root->left, subRoot) || isSubtree(root->right, subRoot);
}

private:
bool isSameTree(const TreeNode* p, const TreeNode* q) {
    if (p == nullptr || q == nullptr) return p == q;
    if (p->val != q->val) return false;
    return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
}
```

#### Complexity
- **Time:** O(n × m) — isSameTree costs O(m) at each of n nodes in worst case
- **Space:** O(h) — recursion depth bounded by height of root tree

#### Concepts Discussed
- **Null check order matters:** `subRoot == nullptr` must be checked before `root == nullptr` — empty tree is subtree of anything. Reversing order causes incorrect false return when subRoot is null
- **Early val check before isSameTree:** `root->val == subRoot->val &&` short-circuits full comparison — avoids O(m) work unless root values match
- **Private helper method:** `isSameTree` is implementation detail, not part of public interface — correct encapsulation
- **Composing from previous solutions:** direct reuse of #100 — recognising and reusing structure is a strong interview signal
- **Adversarial worst case:** `root = aaaaab`, `subRoot = aaab` — isSameTree runs nearly to completion at every node before failing → true O(n × m) behaviour
- **Tree hashing optimisation:** serialise each subtree to a hash, store in unordered_set, check membership — reduces to O(n + m). Requires careful serialisation with null markers and delimiters to avoid false matches e.g. "1,2,#,#,3,#,#" not "123"
- **KMP / suffix tree:** theoretically optimal O(n + m) — treat post-order serialisation as string matching problem. Academic but worth knowing exists
- **⚠️ TODO — REVISIT:** DFS and BFS traversal orders (pre-order, in-order, post-order) iteratively and recursively — #94, #144, #145

---



## 2026-03-10

---

### Problem #509 — Fibonacci Number
- **Difficulty:** Easy
- **Topic:** Dynamic Programming, Math

#### Solution
```cpp
int fib(int n) {
    if (n <= 1) return n;
    int first = 0, second = 1;
    for (int i = 2; i <= n; ++i)
        std::tie(first, second) = std::make_pair(second, first + second);
    return second;
}
```

#### Complexity
- **Time:** O(n) — **Space:** O(1)

#### Concepts Discussed
- **`n <= 1` over `n == 0 || n == 1`:** cleaner, also handles negative input gracefully
- **Eliminate temporary variable trick:** `second = first + second; first = second - first` — works because addition is reversible. Same principle as XOR swap. Clever but sacrifices readability — prefer `std::tie` or explicit temp in production
- **`std::tie` + `std::make_pair`:** simultaneous assignment — both values evaluated before any assignment happens, no stale value risk. Zero overhead at -O2/-O3 due to inlining and copy elision
- **Compiler optimisation flags:**
  - `-O0` — no optimisation, default, easy to debug
  - `-O1` — basic optimisations
  - `-O2` — standard production choice: inlining, copy elision, loop optimisation
  - `-O3` — aggressive: loop unrolling, auto-vectorisation (SIMD), can sometimes be slower due to instruction cache pressure
  - `-Os` — optimise for binary size
- **Copy elision / RVO:** at -O2, temporaries like `std::make_pair` are never materialised in memory — compiler emits identical assembly to hand-written temp variable version
- **Godbolt (godbolt.org):** paste code, select compiler, add `-O2` flag — compare assembly of different implementations to verify identical output
- **Naive recursive fib:** O(2^n) — catastrophic. Not tail recursive (two calls), overlapping subproblems recomputed repeatedly
- **Memoised recursive fib:** O(n) time, O(n) space — correct but iterative is superior at O(1) space
- **Matrix exponentiation:** raises 2×2 matrix to nth power via fast exponentiation — O(log n) time. Theoretically optimal, rarely asked
- **First DP problem:** space optimisation of keeping only last two values instead of full array is the core DP space optimisation — appears in #70 Climbing Stairs, #198 House Robber

---

### Concepts Discussed — Tree Hashing and Merkle Trees
*(Referenced from #572 discussion)*

#### Tree Hashing For Subtree Matching
- **Naive string serialisation cost:** string concatenation at each node costs O(subtree size) — total still O(n×m), not O(n+m)
- **Correct serialisation format:** must include null markers and delimiters e.g. `"1,2,#,#,3,#,#"` not `"123"` — prevents false matches from different tree shapes with same values
- **Post-order serialisation:** children serialised before parent — every substring uniquely identifies a subtree

#### Merkle Tree — Incremental Hashing
```cpp
size_t hashNode(TreeNode* node,
                std::unordered_map<TreeNode*, size_t>& cache) {
    if (node == nullptr) return 0;
    size_t leftHash  = hashNode(node->left,  cache);
    size_t rightHash = hashNode(node->right, cache);
    size_t h = leftHash  * 1000000007ULL
             + rightHash * 998244353ULL
             + std::hash<int>{}(node->val);
    cache[node] = h;
    return h;
}
```
- **Core idea:** every node's hash derived from children's hashes — any change propagates to root. Root hash is fingerprint of entire tree
- **O(1) per node:** children hashes already computed — true O(n+m) overall
- **Large prime multipliers:** minimise collisions, ensure left/right child order matters — swapping children produces different hash
- **Collision risk:** single 64-bit hash ~1/2^64 per comparison. Double hash ~1/2^128 for production. Cryptographic hash (SHA-256) for financial audit trails
- **Merkle proof:** verify one element is in tree using only O(log n) hashes — path from element to root. Used in Bitcoin transaction verification

#### Real World Applications
- **Git:** every commit = hash(tree + parent + metadata). `git status` detects changes by comparing hashes — O(1) per file
- **Blockchain:** transactions in Merkle tree — O(log n) proof of inclusion without downloading entire chain
- **Distributed databases (Cassandra, DynamoDB):** detect out-of-sync data ranges during replication by bisecting Merkle tree — O(log n) messages instead of full data transfer
- **HTTPS certificate transparency:** SSL certificates in Merkle tree — O(log n) membership proof

#### Approach Comparison For #572
| Approach | Time | Space | Use when |
|---|---|---|---|
| Naive recursive | O(n×m) | O(h) | Small n,m — LeetCode |
| String hashing | O(n×m) string build | O(n×m) | Rarely better than naive |
| Merkle incremental hash | O(n+m) | O(n) | Large trees, repeated queries |
| KMP on serialisation | O(n+m) | O(n+m) | Theoretical optimum only |

- **⚠️ TODO — REVISIT:** DFS and BFS traversal orders (pre-order, in-order, post-order) iteratively and recursively — #94, #144, #145

---

## 2026-03-11

---

### Problem #509 — Fibonacci Number
- **Difficulty:** Easy
- **Topic:** Dynamic Programming, Math

#### Solution
```cpp
int fib(int n) {
    if (n <= 1) return n;
    int first = 0, second = 1;
    for (int i = 2; i <= n; ++i)
        std::tie(first, second) = std::make_pair(second, first + second);
    return second;
}
```

#### Complexity
- **Time:** O(n) — **Space:** O(1)

#### Concepts Discussed
- **`n <= 1` over `n == 0 || n == 1`:** cleaner, also handles negative input gracefully
- **Eliminate temporary variable trick:** `second = first + second; first = second - first` — works because addition is reversible. Same principle as XOR swap. Clever but sacrifices readability — prefer `std::tie` or explicit temp in production
- **`std::tie` + `std::make_pair`:** simultaneous assignment — both values evaluated before any assignment happens, no stale value risk. Zero overhead at -O2/-O3 due to inlining and copy elision
- **Compiler optimisation flags:**
  - `-O0` — no optimisation, default, easy to debug
  - `-O1` — basic optimisations
  - `-O2` — standard production choice: inlining, copy elision, loop optimisation
  - `-O3` — aggressive: loop unrolling, auto-vectorisation (SIMD), can sometimes be slower due to instruction cache pressure
  - `-Os` — optimise for binary size
- **Copy elision / RVO:** at -O2, temporaries like `std::make_pair` are never materialised in memory — compiler emits identical assembly to hand-written temp variable version
- **Godbolt (godbolt.org):** paste code, select compiler, add `-O2` flag — compare assembly of different implementations to verify identical output
- **Naive recursive fib:** O(2^n) — catastrophic. Not tail recursive (two calls), overlapping subproblems recomputed repeatedly
- **Memoised recursive fib:** O(n) time, O(n) space — correct but iterative is superior at O(1) space
- **Matrix exponentiation:** raises 2×2 matrix to nth power via fast exponentiation — O(log n) time. Theoretically optimal, rarely asked
- **First DP problem:** space optimisation of keeping only last two values instead of full array is the core DP space optimisation — appears in #70 Climbing Stairs, #198 House Robber

---

### Concepts Discussed — Tree Hashing and Merkle Trees
*(Referenced from #572 discussion)*

#### Tree Hashing For Subtree Matching
- **Naive string serialisation cost:** string concatenation at each node costs O(subtree size) — total still O(n×m), not O(n+m)
- **Correct serialisation format:** must include null markers and delimiters e.g. `"1,2,#,#,3,#,#"` not `"123"` — prevents false matches from different tree shapes with same values
- **Post-order serialisation:** children serialised before parent — every substring uniquely identifies a subtree

#### Merkle Tree — Incremental Hashing
```cpp
size_t hashNode(TreeNode* node,
                std::unordered_map<TreeNode*, size_t>& cache) {
    if (node == nullptr) return 0;
    size_t leftHash  = hashNode(node->left,  cache);
    size_t rightHash = hashNode(node->right, cache);
    size_t h = leftHash  * 1000000007ULL
             + rightHash * 998244353ULL
             + std::hash<int>{}(node->val);
    cache[node] = h;
    return h;
}
```
- **Core idea:** every node's hash derived from children's hashes — any change propagates to root. Root hash is fingerprint of entire tree
- **O(1) per node:** children hashes already computed — true O(n+m) overall
- **Large prime multipliers:** minimise collisions, ensure left/right child order matters — swapping children produces different hash
- **Collision risk:** single 64-bit hash ~1/2^64 per comparison. Double hash ~1/2^128 for production. Cryptographic hash (SHA-256) for financial audit trails
- **Merkle proof:** verify one element is in tree using only O(log n) hashes — path from element to root. Used in Bitcoin transaction verification

#### Real World Applications
- **Git:** every commit = hash(tree + parent + metadata). `git status` detects changes by comparing hashes — O(1) per file
- **Blockchain:** transactions in Merkle tree — O(log n) proof of inclusion without downloading entire chain
- **Distributed databases (Cassandra, DynamoDB):** detect out-of-sync data ranges during replication by bisecting Merkle tree — O(log n) messages instead of full data transfer
- **HTTPS certificate transparency:** SSL certificates in Merkle tree — O(log n) membership proof

#### Approach Comparison For #572
| Approach | Time | Space | Use when |
|---|---|---|---|
| Naive recursive | O(n×m) | O(h) | Small n,m — LeetCode |
| String hashing | O(n×m) string build | O(n×m) | Rarely better than naive |
| Merkle incremental hash | O(n+m) | O(n) | Large trees, repeated queries |
| KMP on serialisation | O(n+m) | O(n+m) | Theoretical optimum only |

- **⚠️ TODO — REVISIT:** DFS and BFS traversal orders (pre-order, in-order, post-order) iteratively and recursively — #94, #144, #145

---

### Problem #70 — Climbing Stairs
- **Difficulty:** Easy
- **Topic:** Dynamic Programming, Math

#### Solution
```cpp
int climbStairs(int n) {
    if (n < 3) return n;
    int first = 1, second = 2;
    for (int i = 3; i <= n; ++i)
        std::tie(first, second) = std::make_pair(second, first + second);
    return second;
}
```

#### Complexity
- **Time:** O(n) — **Space:** O(1)

#### Concepts Discussed
- **Fibonacci in disguise:** `climbStairs(n) = climbStairs(n-1) + climbStairs(n-2)` — each step reachable from one or two steps below
- **Base case differs from #509:** stair climbing starts `1, 2, 3, 5, 8...` not `0, 1, 1, 2...` — `first=1, second=2` captures this. `n < 3` returns `n` correctly by coincidence of problem values — be explicit about why in interviews
- **DP table → space optimisation argument:** full O(n) DP is `dp[i] = dp[i-1] + dp[i-2]`. Since each value depends only on previous two, array collapses to two variables — always state this reasoning explicitly in interviews
- **Generalisation to k steps:** `dp[i] = dp[i-1] + ... + dp[i-k]` — O(1) space no longer applies, needs sliding window of size k. Fibonacci shortcut disappears
- **Pattern recognition:** directly reusing `std::tie` pattern from #509 — recognising DP recurrence structure across problems is key interview skill

---

## 2026-03-13

---

### Problem #252 — Meeting Rooms
- **Difficulty:** Easy
- **Topic:** Sorting, Intervals

#### Solution 1 — O(n²) const-compatible
```cpp
bool canAttendMeetings(const vector<vector<int>>& intervals) {
    for (size_t i = 0; i < intervals.size(); ++i) {
        for (size_t j = i + 1; j < intervals.size(); ++j) {
            if (intervals[j][0] >= intervals[i][1] ||
                intervals[j][1] <= intervals[i][0]) continue;
            return false;
        }
    }
    return true;
}
```

#### Solution 2 — O(n log n) sort-based
```cpp
bool canAttendMeetings(vector<vector<int>>& intervals) {
    std::sort(intervals.begin(), intervals.end());
    for (size_t i = 1; i < intervals.size(); ++i) {
        if (intervals[i-1][1] > intervals[i][0]) return false;
    }
    return true;
}
```

#### Complexity
- **Solution 1 — Time:** O(n²) — **Space:** O(1)
- **Solution 2 — Time:** O(n log n) — **Space:** O(1)

#### Concepts Discussed
- **Tradeoff between solutions:** O(n²) preserves `const` correctness and avoids copying — O(n log n) requires either removing `const` or passing by value. State this tradeoff explicitly in interviews
- **Default `std::sort` comparator:** uses `std::less<T>{}` which calls `operator<` on element type. For `std::vector`, `operator<` is lexicographic — compares element by element. `{1,2}` vs `{1,3}` → first elements equal → compare second → `{1,2}` comes first. Deterministic, not undefined
- **Custom lambda comparator — empty capture `[]`:** lambda parameters are inner scope, same as function parameters — they are never captured. `[&]` when nothing from outer scope is used is misleading — signals hidden state that doesn't exist. Always use `[]` for self-contained comparators
- **`[&]` capture generates member variable in functor:** compiler transforms lambda into a struct. Captured variables become member variables. Parameters are just function parameters — no capture needed
- **Lambda cannot convert to raw function pointer when capturing:** raw function pointer is just a memory address — no storage for captured state. Captureless `[]` lambda can convert to function pointer. Capturing lambda cannot
- **`std::function` as general solution:** wraps any callable including capturing lambdas via type erasure. Cost: heap allocation + virtual dispatch ~10-20ns per call. Avoid in hot paths — use template parameters instead
- **Small Buffer Optimisation (SBO):** `std::function` avoids heap allocation if captured state fits in internal buffer (~16-32 bytes)
- **Template parameter over `std::function` in HFT:** `template<typename Handler> void onData(Handler&& h)` — inlined by compiler, zero overhead vs `std::function` virtual dispatch
- **`std::sort` default — `std::less<T>{}`:** calls `operator<`. Built-ins: natural ordering. `std::string`: lexicographic. `std::vector`: lexicographic. `std::pair`: by `.first` then `.second`
- **Custom structs have no default `operator<`:** must define `operator<`, provide lambda comparator, or use C++20 spaceship `operator<=>` with `= default`
- **C++20 spaceship operator `<=>`:** `auto operator<=>(const T&) const = default` generates all comparisons lexicographically across members in declaration order
- **Descending sort:** `std::greater<T>{}`, reverse iterators `v.rbegin()/v.rend()`, or `>` in lambda
- **`std::sort` vs `std::stable_sort`:**
  - `std::sort`: introsort (quicksort + heapsort + insertion sort), O(n log n), O(log n) stack, unstable, ~2-3x faster
  - `std::stable_sort`: merge sort, O(n log n) with O(n) heap allocation, O(n log² n) if allocation fails, stable
  - Stability matters only when comparator treats elements as equal — equal elements may appear in any order after `std::sort`
- **Multi-key sort pattern:** sort secondary key first, `std::stable_sort` by primary — OR use single comparator with explicit tiebreaker and `std::sort`. The latter is preferred: faster, explicit, not dependent on input order
- **Strict weak ordering requirement:** comparator must satisfy irreflexivity, asymmetry, transitivity. Using `<=` instead of `<` violates irreflexivity — undefined behaviour, may crash or loop infinitely
- **Buy-side preference:** `std::sort` with complete comparator including all tiebreaker fields (e.g. timestamp for time priority). `std::stable_sort` only when no tiebreaker field exists — rare in financial systems where every event is sequenced

---

### Problem #338 — Counting Bits
- **Difficulty:** Easy
- **Topic:** Dynamic Programming, Bit Manipulation

#### Solution
```cpp
vector<int> countBits(int n) {
    vector<int> counts(n + 1, 0);
    for (int i = 1; i <= n; ++i)
        counts[i] = counts[i >> 1] + (i & 1);
    return counts;
}
```

#### Alternative — Drop Lowest Set Bit
```cpp
vector<int> countBits(int n) {
    vector<int> counts(n + 1, 0);
    for (int i = 1; i <= n; ++i)
        counts[i] = counts[i & (i - 1)] + 1;
    return counts;
}
```

#### Complexity
- **Time:** O(n) — **Space:** O(n)

#### Concepts Discussed
- **`i >> 1` approach:** `counts[i] = counts[i/2] + (i is odd ? 1 : 0)`. Even numbers have same popcount as half (right shift removes trailing zero). Odd = previous even + 1
- **`i & (i-1)` — clear lowest set bit:** subtracting 1 flips lowest set bit to 0 and all trailing zeros to 1. AND-ing keeps identical upper bits, zeros everything from lowest set bit down. Result: `i` with exactly one fewer bit
- **Recurrence:** `popcount(i) = popcount(i & (i-1)) + 1` — always valid, no branching, works uniformly for odd and even
- **Bit operations over arithmetic:** `i >> 1` faster than `i / 2`, `i & 1` faster than `i % 2` at O0. At O2 compiler handles it — but signals low-level fluency in interviews
- **Redundant initialisation:** `counts[0] = 0` unnecessary — vector already initialised to 0 via constructor

#### Bit Manipulation Toolkit
| Expression | Effect | Common Use |
|---|---|---|
| `i & (i-1)` | Clear lowest set bit | Count bits, check power of 2 |
| `i & (-i)` | Isolate lowest set bit | Fenwick tree, find lowest bit |
| `i \| (i+1)` | Set lowest clear bit | Bit tricks |
| `i & (i+1)` | Clear trailing ones | Bit tricks |
| `n & (n-1) == 0` | Check power of 2 | Very common interview question |
| `n ^ (n-1)` | Mask from lowest set bit downward | Bit tricks |
| `a ^ b` | XOR — find differing bits | Find unique element, swap |
| `~i & (i+1)` | Isolate lowest clear bit | Bit tricks |

#### Key Bit Manipulation Patterns
```cpp
// Check power of 2
bool isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}

// Brian Kernighan — count set bits, O(number of set bits)
int popcount(int n) {
    int count = 0;
    while (n) { n = n & (n - 1); ++count; }
    return count;
}

// Check exactly one bit differs
bool oneBitDifference(int a, int b) {
    int diff = a ^ b;
    return diff > 0 && (diff & (diff - 1)) == 0;
}

// Isolate lowest set bit
int lowest = n & (-n);

// Clear lowest set bit
n = n & (n - 1);
```

- **⚠️ TODO — REVISIT:** DFS and BFS traversal orders (pre-order, in-order, post-order) iteratively and recursively — #94, #144, #145

---

## 2026-03-14

---

### Problem #268 — Missing Number
- **Difficulty:** Easy
- **Topic:** Math, Bit Manipulation

#### Solution — Gauss Sum
```cpp
int missingNumber(vector<int>& nums) {
    const int n = static_cast<int>(nums.size());
    const int expected = n * (n + 1) / 2;
    return expected - std::accumulate(nums.begin(), nums.end(), 0);
}
```

#### Solution — XOR
```cpp
int missingNumber(vector<int>& nums) {
    int result = static_cast<int>(nums.size());
    for (int i = 0; i < static_cast<int>(nums.size()); ++i)
        result ^= i ^ nums[i];
    return result;
}
```

#### Complexity
- **Time:** O(n) — **Space:** O(1)

#### Concepts Discussed
- **Gauss sum:** expected = n*(n+1)/2 — missing = expected - actual sum
- **Single pass trick:** seed `expected_total = size`, add i and nums[i] simultaneously — avoids separate loops
- **`std::accumulate`:** idiomatic STL reduction — signals fluency, prefer over manual sum loop
- **XOR approach:** `a ^ a = 0` — paired indices and values cancel, leaving only missing number. Zero overflow risk — advantage over sum when n is large
- **Overflow risk in sum approach:** sum of 0..n overflows int when n > ~65,000. XOR has no overflow risk — always prefer for large n

---

### Problem #191 — Number of 1 Bits
- **Difficulty:** Easy
- **Topic:** Bit Manipulation

#### Solution — Logical Right Shift
```cpp
int hammingWeight(uint32_t n) {
    int count = 0;
    while (n) {
        count += n & 1;
        n >>= 1;
    }
    return count;
}
```

#### Solution — Brian Kernighan
```cpp
int hammingWeight(uint32_t n) {
    int count = 0;
    while (n) {
        n = n & (n - 1);  // clear lowest set bit
        ++count;
    }
    return count;
}
```

#### Solution — C++20
```cpp
int hammingWeight(uint32_t n) {
    return std::popcount(n);  // maps to POPCNT CPU instruction — single cycle
}
```

#### Complexity
- **Time:** O(log n) logical shift, O(popcount(n)) Brian Kernighan, O(1) std::popcount
- **Space:** O(1)

#### Concepts Discussed
- **Must use `uint32_t` not `int`:** signed right shift is implementation-defined in C++17 (arithmetic shift fills with sign bit — infinite loop for negative n). Left shift into sign bit is UB. `uint32_t` guarantees logical shift
- **C++20 signed right shift:** defined as arithmetic shift — still causes infinite loop for negative input. `uint32_t` is correct fix in any standard
- **Logical shift iterations:** `floor(log2(n)) + 1` — terminates at highest set bit, not always 32
- **Brian Kernighan iterations:** exactly `popcount(n)` — number of set bits
- **Brian Kernighan is always equal or better:** `popcount(n) <= floor(log2(n)) + 1` always. Equal only when n = 2^k - 1 (all ones: 1,3,7,15...) — every bit up to highest is set. Strictly better for all other numbers
- **`std::popcount` (C++20):** maps to x86 POPCNT instruction — single clock cycle regardless of input. Production answer for hot paths

---

### Problem #190 — Reverse Bits
- **Difficulty:** Easy
- **Topic:** Bit Manipulation

#### Solution — Loop
```cpp
uint32_t reverseBits(uint32_t n) {
    uint32_t result = 0;
    for (int i = 0; i < 32; ++i) {
        result <<= 1;
        result |= (n & 1);
        n >>= 1;
    }
    return result;
}
```

#### Solution — Divide and Conquer O(1)
```cpp
uint32_t reverseBits(uint32_t n) {
    n = (n >> 16) | (n << 16);
    n = ((n >> 8)  & 0x00FF00FF) | ((n & 0x00FF00FF) << 8);
    n = ((n >> 4)  & 0x0F0F0F0F) | ((n & 0x0F0F0F0F) << 4);
    n = ((n >> 2)  & 0x33333333) | ((n & 0x33333333) << 2);
    n = ((n >> 1)  & 0x55555555) | ((n & 0x55555555) << 1);
    return n;
}
```

#### Complexity
- **Loop — Time:** O(32) = O(1) — **Space:** O(1)
- **Divide and conquer — Time:** O(1), 5 operations — **Space:** O(1)

#### Concepts Discussed
- **`|=` over `^=`:** after `result <<= 1`, LSB is always 0 — XOR and OR behave identically. But `|=` expresses "set this bit", `^=` expresses "toggle" — `|=` is clearer
- **32 iterations required:** unlike #191, bit reversal must process all 32 positions — leading zeros of n become trailing zeros of result. Early termination would lose this information
- **Divide and conquer — swap increasingly smaller groups:** halves → bytes → nibbles → pairs → individual bits. 5 steps total
- **Mask pattern:**
  - `0x00FF00FF` = alternating bytes
  - `0x0F0F0F0F` = alternating nibbles
  - `0x33333333` = alternating pairs
  - `0x55555555` = alternating individual bits
  - Each mask selects lower half of each group at that level
- **Why divide and conquer works here but not for strings:** bitmasks operate on ALL bits in parallel — O(1) per step. String characters are arbitrary — cannot use bitmasks, no parallel operation possible
- **`std::byteswap` (C++23):** reverses bytes not bits — related but different. No standard bit-reversal function exists

---

### Concept — `i & (-i)` Isolate Lowest Set Bit

#### Two's Complement
```
-i = ~i + 1  (flip all bits, add 1)
i  =  0000 1100
-i =  1111 0100
i & -i = 0000 0100  — only lowest set bit survives
```
Adding 1 to `~i` carries through trailing ones — flipping them back to 0 and setting lowest set bit position to 1. Result: `i` and `-i` agree only at lowest set bit position.

#### Contrast With `i & (i-1)`
```
i & (i-1)  → REMOVES  lowest set bit
i & (-i)   → ISOLATES lowest set bit
i == (i & (i-1)) + (i & (-i))  — always true
```

#### Uses
```cpp
// Isolate lowest set bit
int lowest = n & (-n);

// Check power of 2 (alternative)
bool isPowerOfTwo(int n) { return n > 0 && (n & -n) == n; }

// Fenwick Tree — jump to next responsible index
void update(int i, int delta) {
    for (; i <= n; i += i & (-i)) tree[i] += delta;
}
int query(int i) {
    int sum = 0;
    for (; i > 0; i -= i & (-i)) sum += tree[i];
    return sum;
}
```
- **Fenwick tree:** each index responsible for range of length equal to its lowest set bit. Built entirely around `i & (-i)` — appears in order book implementations for efficient prefix sum queries
- **`__builtin_ctz(n)`:** count trailing zeros = position of lowest set bit (GCC/Clang)
- **`std::countr_zero(n)` (C++20):** portable equivalent of `__builtin_ctz`

#### C++20 `<bit>` Header — Full Toolkit
```cpp
std::countr_zero(n)    // position of lowest set bit
std::countl_zero(n)    // count leading zeros
std::popcount(n)       // count set bits
std::has_single_bit(n) // check power of 2
std::byteswap(n)       // reverse bytes (C++23)
```

---

### Concept — Three-Reversal Trick For Substring Swap

#### Swapping Two Adjacent Substrings In Place
```
s = [A][B]  →  [B][A]

Step 1: reverse A  →  [A'][B]
Step 2: reverse B  →  [A'][B']
Step 3: reverse all → [B][A]  ✅
```

```cpp
void reverseRange(std::string& s, int left, int right) {
    while (left < right) std::swap(s[left++], s[right--]);
}

void swapSubstrings(std::string& s, int leftStart, int leftEnd,
                                    int rightStart, int rightEnd) {
    reverseRange(s, leftStart, leftEnd);
    reverseRange(s, rightStart, rightEnd);
    reverseRange(s, leftStart, rightEnd);
}
```

#### Why It Works — Algebra
```
reverse(reverse(A) + reverse(B)) = B + A  ✅
```
reversal is its own inverse — `reverse(reverse(X)) = X`

#### Non-Adjacent Substrings — `std::rotate`
```cpp
std::rotate(s.begin() + leftStart,
            s.begin() + rightStart,
            s.begin() + rightEnd + 1);
```
`std::rotate` uses three-reversal trick internally — O(n), in-place.

#### Real World Uses
- **Array left rotation by k:** reverse first k, reverse rest, reverse all
- **Reverse words in sentence:** reverse each word, reverse whole string
  ```
  "hello world" → "olleh dlrow" → "world hello" ✅
  ```
- **`std::rotate` implementation**
- **Divide and conquer string reversal:** structural analogy to bit reversal but O(n log n) due to rotate at each level — worse than two-pointer O(n). Analogy is structural only, performance benefit does not transfer

#### Complexity
- **Time:** O(n) — each character touched constant number of times
- **Space:** O(1) — purely in-place

- **⚠️ TODO — REVISIT:** DFS and BFS traversal orders (pre-order, in-order, post-order) iteratively and recursively — #94, #144, #145

---

## 2026-03-16

---

### Problem #49 — Group Anagrams
- **Difficulty:** Medium
- **Topic:** Hash Map, String, Sorting

#### Solution — Sort Key
```cpp
vector<vector<string>> groupAnagrams(const vector<string>& strs) {
    unordered_map<string, vector<string>> groups;
    for (const auto& s : strs) {
        string key = s;
        std::sort(key.begin(), key.end());
        groups[key].push_back(s);
    }
    vector<vector<string>> result;
    result.reserve(groups.size());
    for (auto& [key, group] : groups)
        result.push_back(std::move(group));
    return result;
}
```

#### Solution — Frequency Key O(N×M)
```cpp
vector<vector<string>> groupAnagrams(const vector<string>& strs) {
    unordered_map<array<int,26>, vector<string>, ArrayHash> groups;
    for (const auto& s : strs) {
        array<int,26> freq{};
        for (char c : s) ++freq[c - 'a'];
        groups[freq].push_back(s);
    }
    vector<vector<string>> result;
    for (auto& [key, group] : groups)
        result.push_back(std::move(group));
    return result;
}
```

#### Complexity
- **Sort key — Time:** O(N × M log M) — **Space:** O(N × M)
- **Frequency key — Time:** O(N × M) — **Space:** O(N × M)

#### Concepts Discussed
- **`operator[]` default initialisation:** `++counts[nums[i]]` inserts 0 if key absent then increments — eliminates find/emplace branch. Standard idiom for frequency counting
- **`emplace` over `insert`:** avoids unnecessary copies — consistent habit
- **`find` then check iterator:** single lookup vs `count()` then `[]` which is two lookups
- **`std::move(group)`:** moves vector out of map instead of copying — avoids O(N) copy per group
- **`result.reserve(groups.size())`:** pre-allocates result vector — avoids reallocations
- **Index-based approach:** store group index in separate map instead of group itself — avoids duplicate key storage. Valid optimisation but less readable than direct map to vector
- **C++17 structured binding in range-for:** `for (auto& [key, group] : groups)` — clean, avoids `.first`/`.second`
- **`array<int,26>` not hashable by default:** requires custom hasher — `std::hash` only specialised for primitives, string, string_view, pointers, and a few standard library types
- **Not hashable by default:** `std::vector<T>`, `std::array<T,N>`, `std::pair`, `std::tuple`, custom structs

#### boost::hash_combine
```cpp
seed ^= std::hash<T>{}(value) + 0x9e3779b9 + (seed << 6) + (seed >> 2);
```
- **Why not XOR alone:** XOR is commutative — `hash(A)^hash(B) == hash(B)^hash(A)`. Order lost — anagrams would collide
- **`0x9e3779b9`:** golden ratio constant = `floor(2^32 / φ)`. Most "irrational" number — distributes bits uniformly across 32-bit space, breaks up clustering
- **`(seed << 6) + (seed >> 2)`:** pseudo-rotation of seed — mixes current seed back into itself before combining. Ensures order sensitivity: `hash({1,2}) ≠ hash({2,1})`
- **Universal composite key pattern:** use for `pair<int,int>`, `tuple`, `array`, custom structs
- **C++26:** proposes standard hash for ranges and tuples — eliminates need for custom hashers

#### if-with-initialiser (C++17) — Semicolon Not &&
```cpp
// CORRECT — semicolon separates init from condition
if (auto it = map.find(key); it != map.end()) { }

// WRONG — does not compile
if (auto it = map.find(key) && it != map.end()) { }
// parsed as: auto it = (map.find(key) && it != map.end())
// it used before declared — compile error
```
- **Scoping benefit:** `it` confined to if/else block — cannot be accidentally used after

---

### Problem #347 — Top K Frequent Elements
- **Difficulty:** Medium
- **Topic:** Hash Map, Heap, Bucket Sort

#### Solution — Min-Heap O(N + M log k)
```cpp
vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int,int> counts;
    for (int n : nums) ++counts[n];
    using P = pair<int,int>;
    priority_queue<P, vector<P>, greater<P>> minHeap;
    for (auto& [val, freq] : counts) {
        minHeap.push({freq, val});
        if (static_cast<int>(minHeap.size()) > k) minHeap.pop();
    }
    vector<int> result;
    while (!minHeap.empty()) {
        result.push_back(minHeap.top().second);
        minHeap.pop();
    }
    return result;
}
```

#### Solution — Bucket Sort O(N)
```cpp
vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int,int> counts;
    for (int n : nums) ++counts[n];
    vector<vector<int>> bucket(nums.size() + 1);
    for (auto& [val, freq] : counts)
        bucket[freq].push_back(val);
    vector<int> result;
    for (int i = static_cast<int>(bucket.size()) - 1;
         i >= 0 && static_cast<int>(result.size()) < k; --i)
        for (int val : bucket[i])
            result.push_back(val);
    return result;
}
```

#### Complexity
| | multimap | Min-heap | Bucket sort |
|---|---|---|---|
| Time | O(N + M log M) | O(N + M log k) | O(N) |
| Space | O(N + M) | O(M + k) | O(N) |
| When best | Simple | k << M | Always fastest |

#### Concepts Discussed
- **multimap vs priority_queue:** multimap sorts ALL M unique elements O(M log M) even though only k needed. Min-heap of size k evicts small elements immediately — O(M log k). Significant for k=1 on large M
- **Bucket sort correctness:** index IS the frequency — exact mapping, not approximate. Valid because frequency is always a bounded non-negative integer in [1,n]. Not a general-purpose sort — only works when key is bounded non-negative integer
- **O(N log N) comparison sort lower bound:** applies only to comparison-based sorting. Bucket/counting/radix sort bypass it by exploiting integer key structure
- **Inner break unnecessary given LeetCode guarantee:** outer loop condition `result.size() < k` handles termination. Inner break only needed for general case where ties at boundary can overshoot k
- **Range-based for preferred over index loop:** when index unused, `for (int n : nums)` is cleaner and less error-prone than `for (size_t i = 0; i < nums.size(); ++i)`

---

### Problem #167 — Two Sum II
- **Difficulty:** Medium
- **Topic:** Two Pointers, Binary Search

#### Solution — Two Pointer O(N)
```cpp
vector<int> twoSum(const vector<int>& numbers, int target) {
    int left = 0, right = static_cast<int>(numbers.size()) - 1;
    while (left < right) {
        const int sum = numbers[left] + numbers[right];
        if (sum == target) return {left + 1, right + 1};
        if (sum > target)  --right;
        else               ++left;
    }
    return {};
}
```

#### Complexity
- **Time:** O(N) — **Space:** O(1)

#### Concepts Discussed
- **Why `--left` is never needed after `++left`:** we only do `++left` when `sum < target` — `numbers[left]` is too small for current `right`. Since `right` only decreases from here, `numbers[left]` is too small for ALL future right values too. Permanently ruled out
- **Why `++right` is never needed after `--right`:** we only do `--right` when `sum > target` — `numbers[right]` is too large for current `left`. Since `left` only increases from here, `numbers[right]` is too large for ALL future left values too. Permanently ruled out
- **Both pointers are one-way:** `left` is "increase sum" lever, `right` is "decrease sum" lever. Never need to pull either lever in reverse
- **2D search space intuition:** all pairs form a grid. Two-pointer traces the staircase boundary — each `++left` eliminates an entire row, each `--right` eliminates an entire column. Going back revisits already-eliminated rows/columns — provably useless
- **Uniqueness guarantee not needed for correctness:** sorted property alone guarantees each pointer move is correct. Uniqueness only guarantees answer is found before `left >= right`
- **Binary search variant O(N log N):** lower_bound per left step — right shrinks correctly as left advances (complement decreases as left increases). Correct but O(N log N) vs O(N)
- **Why binary search is not faster despite skipping right elements:** left still advances one by one — O(N) left steps × O(log N) per binary search = O(N log N). Two-pointer is O(N) because BOTH pointers move — total moves = N. Binary search only moves one pointer linearly while other does repeated O(log N) work
- **When binary search wins in practice:** when answer found in very few left steps — early termination saves remaining N steps. But worst and average case favour two-pointer
- **General principle:** when structural properties allow both boundaries to move simultaneously, prefer it over repeatedly searching within a shrinking range

- **⚠️ TODO — REVISIT:** DFS and BFS traversal orders (pre-order, in-order, post-order) iteratively and recursively — #94, #144, #145

---

# C++ LeetCode Study Log
**Wednesday, March 18, 2026**

---

## Problem Overview

| Field | Detail |
|---|---|
| Problem | #424 – Longest Repeating Character Replacement |
| Difficulty | Medium |
| Topic Tags | Sliding Window, String, Hash Map |
| Target Role | Buy-Side C++ Developer |
| Status | Solved (multiple iterations) |

---

## Session Summary

Started with an ad-hoc index-tracking solution and iteratively improved it toward the canonical O(n) sliding window approach through three distinct versions.

---

## Version 1 — Original Ad-Hoc Solution

*Approach: Track occurrence indices per character in an `unordered_map`. For each new occurrence, scan backwards through stored indices to find the leftmost anchor where the remaining budget connects the window.*

**Bugs identified:**
- `longest` initialised to `k+1` without bounding by string size
- Extension formula `i - v[j] + 1 + (k - fillings)` could exceed string length, inflating window size and causing a premature `break`
- First-occurrence initialisation used magic number `k+1` without the same size guard

**Complexity:** O(n²) time (inner loop scans all prior indices), O(n) space

---

## Version 2 — Patched Solution

**Fixes applied:**
- `min(k+1, size)` — init guard
- `min(length, size)` — extension guard

**Mathematical proof of correctness for the extension guard:**

Given window `[v[j], i]` with remaining budget `b = k - fillings`:

```
actual_length = window + min(b, left_available + right_available)
              = window + min(b, size - window)
              = min(window + b, size)
```

This is exactly what `min(length, size)` computes — it is not merely a guard, it is mathematically exact. The extension budget can be split arbitrarily left or right; total achievable length is direction-independent.

**Status:** Logically correct. O(n²) worst case persists (e.g. `k=0`, alternating chars — inner loop never breaks early).

---

## Version 3 — Binary Search Elimination of Inner Loop

*Key insight: prove `fillings(j)` is monotonically non-increasing in `j`, enabling binary search for the leftmost valid anchor.*

**Monotonicity proof:**
```
fillings(j)  = i - v[j] + j - v_size
fillings(j+1) - fillings(j) = -(v[j+1] - v[j]) + 1 ≤ 0
```
Since `v` contains strictly increasing indices, `v[j+1] - v[j] ≥ 1`, so the difference is always `≤ 0`. Once the condition is satisfied, all subsequent `j` also satisfy it — classic `lower_bound` scenario.

**Binary search target:**
```
fillings(j) ≤ k  ⟺  j - v[j] ≤ k - i + v_size
```
Search for leftmost `j` satisfying `j - v[j] ≤ rhs`, where `rhs` is constant per outer iteration.

**Complexity:** O(n log n) time, O(n) space

*Why O(n) is not reachable this way: storing all historical positions and paying O(log n) per element is a hard floor. To reach O(n), you must discard stale left positions as `i` advances — which is exactly the sliding window left pointer.*

---

## Complexity Comparison

| Version | Time | Space | Correctness |
|---|---|---|---|
| v1 — original | O(n²) worst | O(n) | Buggy (init + extension) |
| v2 — patched | O(n²) worst | O(n) | Correct |
| v3 — binary search | O(n log n) | O(n) | Correct |
| **Sliding Window** | **O(n)** | **O(1)** | **Correct** |

---

## Key Takeaways

- The sliding window pattern relies on the observation that only `max_freq` within the current window matters — not which specific positions the target character occupies.
- `maxFreq` intentionally never decreases in the canonical solution. A stale high value causes the window to slide without growing — never a false answer, never needs recomputation.
- Monotonicity proofs are the bridge between brute-force inner loops and binary search. Always check if your scan variable is monotone before accepting O(n²).
- `min(length, size)` in v2 is not a patch — it is the mathematically exact bound, proven by the budget-splitting argument.
- Related problems to consolidate the sliding window pattern: #3, #76, #340, #992.

---

## C++ Notes

- `if (auto it = map.find(key); it != map.end())` — C++17 init-statement in `if`, idiomatic for map lookup without double search.
- `static_cast<int>(v.size())` — avoids signed/unsigned comparison warnings when indexing with `int` loop variables.
- `std::array<int, 26> freq{}` — prefer over `unordered_map` for fixed alphabets; O(1) guaranteed, cache-friendly, zero-initialised by value-init.
- `lo + (hi - lo) / 2` — canonical overflow-safe midpoint, mandatory in competitive and production code.

---

*Logged from Claude study session · March 18, 2026*

# C++ LeetCode Study Log
**Thursday, March 19, 2026**

---

## Problem Overview

| Field | Detail |
|---|---|
| Problem | #142 – Linked List Cycle II |
| Difficulty | Medium |
| Topic Tags | Linked List, Two Pointers, Hash Set |
| Target Role | Buy-Side C++ Developer |
| Status | Solved (two approaches) |

---

## Session Summary

Solved with a hash set approach first, then analysed Floyd's cycle detection algorithm in depth — including exact step counts and space accounting, not just asymptotic bounds.

---

## Version 1 — Hash Set (Initial Attempt)

```cpp
class Solution {
public:
    ListNode* detectCycle(ListNode* head) {
        unordered_set<ListNode*> visited;
        while (head) {
            auto [_, inserted] = visited.insert(head);
            if (!inserted) break;
            head = head->next;
        }
        return head;
    }
};
```

**Notes:**
- Structured binding on `insert()` returns `pair<iterator, bool>` — only `inserted` is needed, discard the iterator with `_`
- First version used `while(head = head->next)` — assignment in condition, draws `-Wparentheses` warnings; refactored to explicit `head = head->next` inside the loop body
- No-cycle path returns `nullptr` implicitly when `head` exhausts the list — correct and now explicit

**Complexity:** O(N) time, O(N) space (exact — see analysis below)

---

## Version 2 — Floyd's Cycle Detection (Canonical O(1) Space)

```cpp
class Solution {
public:
    ListNode* detectCycle(ListNode* head) {
        ListNode* slow = head;
        ListNode* fast = head;

        // Phase 1: find meeting point inside cycle
        while (fast != nullptr && fast->next != nullptr) {
            slow = slow->next;
            fast = fast->next->next;
            if (slow == fast) {
                // Phase 2: find cycle entry
                ListNode* entry = head;
                while (entry != fast) {
                    entry = entry->next;
                    fast  = fast->next;
                }
                return entry;
            }
        }
        return nullptr;
    }
};
```

---

## Mathematical Analysis

Define:
```
F = nodes before cycle entry
C = cycle length
h = meeting point offset from cycle entry (Phase 1)
N = F + C = total nodes
```

### Does slow always meet fast before completing a full cycle?

When slow enters the cycle after F steps, fast has traveled 2F steps and is already `2F mod C` steps into the cycle. The gap fast must still close is:

```
gap = C - (2F mod C)   ← at most C-1
```

Fast gains exactly 1 step on slow per iteration, so they meet in exactly `gap` more iterations. Since `gap ≤ C-1`, slow travels at most **C-1 steps inside the cycle** before meeting — strictly less than one full loop.

The gain of exactly 1 per iteration means fast can never skip over slow and miss it. It is a modular countdown that always hits zero.

### Why does Phase 2 find the cycle entry?

At the meeting point:
```
slow traveled:  F + h
fast traveled:  F + h + n·C   (lapped slow n times)

fast = 2·slow  →  n·C = F + h  →  F = n·C - h
```

From the meeting point, taking `F` more steps completes n full loops minus the head-start h, landing exactly at the cycle entry. A pointer reset to `head` walks F steps in sync — both arrive at the entry simultaneously.

---

## Exact Complexity Accounting

### Hash Set — exact space

| Case | Insertions |
|---|---|
| No cycle | N (all nodes until nullptr) |
| Cycle exists | F + C = N (until first repeat) |

O(N) space is tight in both cases — no slack hidden in the asymptotic notation.

### Floyd's — exact step count

**Phase 1:**
```
slow steps = F + gap = F + C - (2F mod C)
           ≤ F + C = N
           ≥ F + 1
```

**Phase 2:**
```
Both entry and fast walk exactly F more steps to meet at cycle entry.
Total slow steps = (F + gap) + F ≤ 2N
```

**Space:** exactly 2 pointers regardless of input — true O(1) with no hidden growth.

### Punchline

```
Hash set:  time = N,    space = N    (both tight)
Floyd's:   time ≤ 2N,   space = 2    (time trades a constant factor for eliminating space entirely)
```

Floyd's does not improve the time constant — it potentially doubles the step count. The win is entirely in space. The tradeoff is worth articulating precisely in an interview.

---

## Key Takeaways

- The structured binding discard idiom `auto [_, inserted] = ...` is idiomatic C++17 for single-field use of pair returns.
- Assignment in condition (`while(head = head->next)`) is legal but always draws warnings — avoid in production code.
- Floyd's correctness rests on two independent proofs: (1) slow can never complete a full cycle before meeting fast, and (2) the reset trick follows from `F = nC - h`. Be able to derive both from scratch.
- Asymptotic notation hides meaningful constant factors. Floyd's is strictly worse on the time constant (2N vs N) — the tradeoff is purely about space.
- Asking "does slow always meet fast before a full cycle?" and then proving it is the kind of first-principles thinking quant interviewers look for.

---

## C++ Notes

- `unordered_set<T*>::insert()` returns `pair<iterator, bool>` — use structured bindings to extract cleanly without `.second`.
- `auto [_, inserted] = set.insert(val)` — `_` as a discard name is idiomatic; preferred over `.second` for readability.
- `while (fast != nullptr && fast->next != nullptr)` — always check both before dereferencing `fast->next->next`; order matters since `&&` short-circuits left to right.

---

*Logged from Claude study session · March 19, 2026*

# C++ LeetCode Study Log
**Friday, March 20, 2026**

---

## Problem Overview

| Field | Detail |
|---|---|
| Problem | #739 – Daily Temperatures |
| Difficulty | Medium |
| Topic Tags | Monotonic Stack, Array |
| Target Role | Buy-Side C++ Developer |
| Status | Solved |

---

## Session Summary

Solved with a monotonic stack. Identified two code issues — a redundant early-exit guard caused by `do-while` structure, and a misleading complexity comment. Extended the session into a deep dive on amortized O(1) analysis.

---

## Solution

```cpp
class Solution {
public:
    vector<int> dailyTemperatures(const vector<int>& temperatures) {
        int size = static_cast<int>(temperatures.size());
        vector<int> results(size, 0);
        stack<int> stk;

        for (int i = 0; i < size; ++i) {
            while (!stk.empty() && temperatures[i] > temperatures[stk.top()]) {
                results[stk.top()] = i - stk.top();
                stk.pop();
            }
            stk.push(i);
        }
        return results;
    }
};
```

---

## Issues in Initial Version

**1. Unnecessary early-exit + `do-while`**

Initial version guarded with `if (stk.empty()) { stk.push(i); continue; }` before a `do-while` loop. The guard only existed because `do-while` unconditionally calls `stk.top()` before checking emptiness. Replacing with a `while` condition eliminates both the guard and the `continue`:

```cpp
// Before
if (stk.empty()) { stk.push(i); continue; }
do {
    auto top = stk.top();
    if (temperatures[i] > temperatures[top]) { stk.pop(); results[top] = i - top; }
    else break;
} while (!stk.empty());
stk.push(i);

// After
while (!stk.empty() && temperatures[i] > temperatures[stk.top()]) {
    results[stk.top()] = i - stk.top();
    stk.pop();
}
stk.push(i);
```

**2. Misleading complexity comment**

```cpp
} while(!stk.empty());   // O(N)
```

Labelling the inner loop O(N) implies O(N²) overall. The correct framing is **amortized O(1)** per outer iteration — each index is pushed once and popped at most once across the entire run, so the inner loop contributes O(N) total work, not O(N) per call.

---

## Complexity

| | Complexity | Reasoning |
|---|---|---|
| Time | O(N) | Each index pushed once, popped at most once |
| Space | O(N) | Stack holds at most N indices |

**Worst case for space:** a non-increasing sequence like `[100, 90, 80, ...]` — every index is pushed and never popped, so the stack grows to size N.

**Stack invariant:** the stack always holds indices of a non-increasing temperature sequence. Maintaining this invariant is what makes the algorithm correct.

---

## Amortized O(1) — Deep Dive

### Core Idea

Regular O(1) means every individual operation is constant time. Amortized O(1) means the **average cost per operation over N operations** is constant — even if individual operations occasionally cost more. The cost is redistributed, not eliminated.

### The Accounting Method

Assign each element a token at creation. Tokens are spent when work is done.

```
push(i)  →  earns 1 token
pop()    →  spends 1 token

Each index pushed exactly once   →  N tokens created
Each index popped at most once   →  at most N tokens spent
Total inner loop work across entire run = O(N)
Amortized cost per outer iteration = O(1)
```

A batch-pop of 1000 elements in one iteration looks expensive, but those 1000 pushes already paid for it.

### The Potential Method (Rigorous)

Define a potential function Φ measuring stored-up work (e.g. current stack size):

```
amortized_cost = actual_cost + ΔΦ
```

For the monotonic stack:
- `push`: actual cost 1, Φ increases by 1 → amortized cost = 2 = O(1)
- `pop k elements`: actual cost k, Φ decreases by k → amortized cost = 0, plus the final push = O(1)

As long as Φ starts at 0 and never goes negative, the sum of amortized costs upper-bounds the total actual cost.

### Common Occurrences

**Monotonic stack / queue**
Each element enters and exits exactly once. Any "next greater/smaller element" problem. Batch pops are covered by prior pushes.

**`std::vector::push_back`**
Doubling on resize copies all N elements — O(N) for that one call. But:
```
total copies to reach size N: 1 + 2 + 4 + ... + N/2 = N - 1
```
N insertions cause at most N-1 copies. Amortized O(1) per `push_back`. This is why `push_back` is amortized O(1) but `insert(begin())` is not.

**`unordered_map` insert**
Rehashing doubles the table and reinserts everything — same doubling argument as `std::vector`. Amortized O(1) per insert.

**Union-Find with path compression**
```cpp
int find(int x) {
    if (parent[x] != x)
        parent[x] = find(parent[x]);  // flattens chain for future calls
    return parent[x];
}
```
One `find()` can traverse a long chain, but flattening makes subsequent calls O(1). Over M operations on N elements: amortized O(α(N)) per call — inverse Ackermann, effectively constant.

**Splay trees**
Each access rotates the node to root — expensive for deep nodes, but the rotation improves the structure for future accesses. Amortized O(log N) per operation even though individual ops can be O(N).

### Summary Table

| Structure / Operation | Worst Single Op | Amortized | Why |
|---|---|---|---|
| Monotonic stack pop | O(N) | O(1) | Each element popped at most once |
| `vector::push_back` | O(N) | O(1) | Doubling amortises copy cost |
| `unordered_map` insert | O(N) | O(1) | Rehash cost spread over insertions |
| Union-Find `find` | O(N) | O(α(N)) ≈ O(1) | Path compression flattens future calls |
| Splay tree access | O(N) | O(log N) | Rotation improves future structure |

---

## Key Takeaways

- `while (!stk.empty() && condition)` is always cleaner than a `do-while` with a guarded `continue` — the guard is a symptom of the wrong loop construct.
- Labelling an inner loop O(N) without the word "amortized" implies O(N²) total — be precise in comments and in interviews.
- The stack invariant (non-increasing sequence) is the correctness argument; the token argument is the complexity argument. Know both separately.
- Amortized O(1) is the right mental model for throughput-bound systems — HFT order books, ring buffers, memory pool allocators. Expensive slab allocations upfront, true O(1) individual allocations thereafter.
- The potential method (amortized = actual + ΔΦ) is the rigorous framework expected in a quant interview when asked to prove amortized bounds formally.

---

## Related Problems

| Problem | Pattern |
|---|---|
| #496 – Next Greater Element I | Monotonic stack, basic |
| #503 – Next Greater Element II | Monotonic stack, circular array |
| #84 – Largest Rectangle in Histogram | Monotonic stack, harder variant |
| #239 – Sliding Window Maximum | Monotonic deque |

---

## C++ Notes

- `stk.top()` followed by `stk.pop()` is the correct `std::stack` idiom — `top()` returns a reference, `pop()` discards; there is no combined `pop()` returning a value (intentional — exception safety).
- `const vector<int>&` in the function signature is correct; the stack stores indices (`int`), not copied values.
- `vector<int> results(size, 0)` — value-initialised to 0, which is the correct default (no warmer day found).

---

*Logged from Claude study session · March 20, 2026*

# C++ LeetCode Study Log
**Sunday, March 22, 2026**

---

## Problems Covered

| # | Problem | Difficulty | Status |
|---|---|---|---|
| #77 | Combinations | Medium | Solved + optimised |
| #1 | Two Sum | Easy | Solved in C++ and Python |

---

## #77 — Combinations

### Solution with Pruning

```cpp
class Solution {
public:
    vector<vector<int>> combine(int n, int k) {
        vector<vector<int>> results;
        vector<int> comb;
        backtrack(1, n, k, comb, results);
        return results;
    }

private:
    void backtrack(int start, int range, int count,
                   vector<int>& comb, vector<vector<int>>& results) {
        if (static_cast<int>(comb.size()) == count) {
            results.push_back(comb);
            return;
        }
        const int remaining = count - static_cast<int>(comb.size());
        for (int i = start; i <= range - remaining + 1; ++i) {
            comb.push_back(i);
            backtrack(i + 1, range, count, comb, results);
            comb.pop_back();
        }
    }
};
```

### Pruning Derivation

Without pruning, the loop runs to `range` even when there aren't enough elements left to complete the combination. The tighter bound comes from one constraint: the last element picked must not exceed `range`.

If you start at `i` and still need `remaining` elements, you will pick `i, i+1, ..., i + remaining - 1`. The last element must be `<= range`:

```
i + remaining - 1 <= range
i <= range - remaining + 1
```

`remaining = count - comb.size()` — the number of elements still needed at the current depth, which relaxes as the combination fills up:

```
At root:          comb=[], remaining=k   → upper = range - k + 1
After one push:   comb=[1], remaining=k-1 → upper = range - k + 2
After two pushes: comb=[1,2], remaining=k-2 → upper = range - k + 3
```

The bound relaxes at each depth because each element placed reduces how many more are needed.

### Edge Cases

```
count == 1     →  upper = range, no pruning — every single element is valid
count == range →  upper = 1 at root, maximum pruning — only one valid path
count == 0     →  base case triggers immediately, returns {{}}
```

### Pruning for the 0-Indexed Array Version

The same algebra with 0-indexed arrays:

```
i + remaining - 1 <= nums.size() - 1
i <= nums.size() - remaining
```

```cpp
// 1-indexed range (#77)
for (int i = start; i <= range - remaining + 1; ++i)

// 0-indexed array
const int n = static_cast<int>(nums.size());
for (int i = start; i <= n - remaining; ++i)
```

The `+1` difference is exactly the offset between 1-indexed and 0-indexed. Always hoist `nums.size()` cast outside the loop — not because the cast is expensive, but because it builds the correct habit for cases where the bound expression is genuinely costly.

---

## #1 — Two Sum

### C++ Solution

```cpp
class Solution {
public:
    vector<int> twoSum(const vector<int>& nums, int target) {
        unordered_map<int, int> m;
        const int n = static_cast<int>(nums.size());
        for (int i = 0; i < n; ++i) {
            if (auto it = m.find(target - nums[i]); it != m.end())
                return {it->second, i};
            m[nums[i]] = i;
        }
        return {};  // guaranteed unreachable per problem constraints
    }
};
```

**Key points:**
- Use `find` to avoid double lookup — `count` followed by `[]` hits the map twice for the same key
- C++17 init-statement in `if` gives the iterator directly, no second lookup needed
- `return {it->second, i}` — stored (smaller) index first, current (larger) index second
- `return {}` at the end is unreachable by problem constraints but required for compilation

### Python Solution

```python
class Solution:
    def twoSum(self, nums: list[int], target: int) -> list[int]:
        hashmap: dict[int, int] = {}
        for i, value in enumerate(nums):
            complement = target - value
            if complement in hashmap:
                return [hashmap[complement], i]
            hashmap[value] = i
        return []
```

### Complexity

| | Complexity | Reasoning |
|---|---|---|
| Time | O(N) | Single pass, O(1) amortised per lookup |
| Space | O(N) | Map holds at most N-1 entries before hit |

---

## Python Fundamentals

### `enumerate` Instead of `range(len(...))`

The most important habit shift coming from C++. Python iterates over values by default — indexing is the exception.

```python
# C++ habit — works but un-Pythonic
for i in range(len(nums)):
    value = nums[i]

# Pythonic
for i, value in enumerate(nums):
    pass
```

`enumerate` unpacks into `(index, value)` pairs. Eliminates all `nums[i]` accesses in the loop body.

### Type Hints

```python
# old style — requires: from typing import List
def twoSum(self, nums: List[int], target: int) -> List[int]:

# modern style (Python 3.9+) — no import needed
def twoSum(self, nums: list[int], target: int) -> list[int]:

# annotating a local variable
hashmap: dict[int, int] = {}
```

Type hints are never enforced at runtime but aid readability and are checked by tools like `mypy`.

### List Indexing — 0-Indexed, Same as C++

```python
nums = [10, 20, 30]
nums[0]   # 10 — same as C++
nums[-1]  # 30 — last element (no C++ equivalent natively)
nums[-2]  # 20 — second to last
```

### Slicing — Half-Open Range, Same as C++ `[begin, end)`

```python
nums = [10, 20, 30, 40, 50]
nums[1:4]   # [20, 30, 40] — indices 1,2,3 (4 excluded)
nums[1:]    # [20, 30, 40, 50] — from index 1 to end
nums[:3]    # [10, 20, 30] — from start to index 2
nums[:]     # [10, 20, 30, 40, 50] — full copy
```

Identical convention to C++ half-open ranges `[start, end)`. Length-based slicing:

```python
nums[start : start + length]   # same arithmetic as C++ iterator arithmetic
```

**Important:** slicing creates a **copy** — O(N) time and space. Prefer passing indices for performance-sensitive code.

### `set` vs `dict` — The Empty Case

```python
d = {"a": 1}   # dict
d = {}          # empty dict

s = {1, 2, 3}  # set
s = set()       # empty set — {} would be an empty dict, NOT a set
```

Once there's at least one element the type is unambiguous — sets have no colons, dicts do. The empty case is the only trap.

### C++ to Python Cheat Sheet

| C++ | Python |
|---|---|
| `for (int i = 0; i < n; ++i)` | `for i, v in enumerate(collection)` |
| `vector.push_back(x)` | `list.append(x)` |
| `unordered_map<K,V>` | `dict` |
| `unordered_set<T>` | `set` |
| `s.count(x)` (set) | `x in s` |
| `s.insert(x)` (set) | `s.add(x)` |
| `s.erase(x)` | `s.discard(x)` (silent) / `s.remove(x)` (raises KeyError) |
| `auto it = m.find(k)` | `if k in d` / `d.get(k)` |
| `static_cast<int>(v.size())` | `len(v)` — always returns `int` |

---

## Key Takeaways

- The pruning bound `range - remaining + 1` falls from one line of algebra (`i + remaining - 1 <= range`), not memorisation. Derive it on paper rather than pattern-matching the formula.
- `remaining = count - comb.size()` is index-agnostic. `count - start` would only work if the range started at 0 — the 1-indexed offset breaks it.
- Always hoist loop bounds out of the loop condition into a named `const` variable — cheap now, critical habit for when the expression is expensive.
- In Python, reach for `enumerate` before `range(len(...))`. Indexing is the exception, not the rule.
- Python's slice `[start:end]` and C++'s `[begin, end)` are the same half-open range convention — the mental model transfers directly.
- Empty `set()` vs `{}` is one of the most common Python traps for C++ developers. `{}` is always a dict.

---

*Logged from Claude study session · March 22, 2026*

# C++ LeetCode Study Log
**Thursday, March 26, 2026**

---

## Problems Covered

| # | Problem | Difficulty | Status |
|---|---|---|---|
| #200 | Number of Islands | Medium | In-place restoration strategies |
| #3 | Longest Substring Without Repeating Characters | Medium | C++ array optimisation + Python |
| #208 | Implement Trie | Medium | Solved + smart pointer deep dive |

---

## #200 — Number of Islands: Restoring In-Place Mutation

Two strategies to restore `'1'` after using in-place `'0'` marking.

### Approach 1: Collect Mutated Cells and Restore Per Island

```cpp
vector<pair<int,int>> mutated;
grid[i][j] = '0';
mutated.push_back({i, j});
stk.push({i, j});

while (!stk.empty()) {
    auto [r, c] = stk.top(); stk.pop();
    for (auto [dr, dc] : dirs) {
        int nr = r + dr, nc = c + dc;
        if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] == '1') {
            grid[nr][nc] = '0';
            mutated.push_back({nr, nc});
            stk.push({nr, nc});
        }
    }
}
for (auto [r, c] : mutated) grid[r][c] = '1';
```

Defeats the purpose — still O(M·N) storage for `mutated`, same as the visited set.

### Approach 2: Temp Marker `'2'` with Single Final Pass (Best)

```cpp
class Solution {
public:
    int numIslands(vector<vector<char>>& grid) {
        const int rows = static_cast<int>(grid.size());
        const int cols = static_cast<int>(grid[0].size());
        int num = 0;

        for (int i = 0; i < rows; ++i) {
            for (int j = 0; j < cols; ++j) {
                if (grid[i][j] != '1') continue;
                ++num;
                stack<pair<int,int>> stk;
                stk.push({i, j});
                grid[i][j] = '2';
                while (!stk.empty()) {
                    auto [r, c] = stk.top(); stk.pop();
                    for (auto [dr, dc] : dirs) {
                        int nr = r + dr, nc = c + dc;
                        if (nr >= 0 && nr < rows && nc >= 0 && nc < cols
                            && grid[nr][nc] == '1') {
                            grid[nr][nc] = '2';
                            stk.push({nr, nc});
                        }
                    }
                }
            }
        }
        for (int i = 0; i < rows; ++i)
            for (int j = 0; j < cols; ++j)
                if (grid[i][j] == '2') grid[i][j] = '1';
        return num;
    }
private:
    const vector<pair<int,int>> dirs = {{1,0},{-1,0},{0,1},{0,-1}};
};
```

Three distinct states: `'0'` water, `'1'` unvisited land, `'2'` visited land. Single O(M·N) restoration pass at the end adds no asymptotic cost.

### Restoration Strategy Comparison

| Approach | Extra Space | Mutates Input | Restoration |
|---|---|---|---|
| In-place `'0'` (no restore) | O(1) | Yes — permanent | None |
| Collect mutated cells | O(M·N) | Restored per island | Per island |
| Temp marker `'2'` | O(1) | Restored at end | One final pass |
| Original visited set | O(M·N) | No | None |

---

## #3 — Longest Substring: `array<int, 128>` Optimisation

### Motivation

`unordered_map<char, int>` is O(1) amortised but carries hash overhead and heap allocation. Since the alphabet is fixed at 128 ASCII characters, a stack-allocated array gives true O(1) with no hashing:

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        array<int, 128> visited;
        visited.fill(-1);
        const int size = static_cast<int>(s.size());
        int start = 0, longest = 0;

        for (int i = 0; i < size; ++i) {
            const int idx = s[i];          // char implicitly converts to ASCII index
            if (visited[idx] >= start)
                start = visited[idx] + 1;
            longest = max(longest, i - start + 1);
            visited[idx] = i;
        }
        return longest;
    }
};
```

`-1` as sentinel: `-1 >= start` is always false since `start >= 0` — uninitialised entries naturally behave as "not visited", eliminating a separate existence check.

### Why Not `array<char, 128>`?

`char` stores characters (max value 127), not positions. Any string longer than 127 characters would overflow:

```
visited['a'] = 200   // doesn't fit in char — overflow
```

The array values represent last-seen positions (integers), not characters. Use `int`.

### Performance Comparison

| | `unordered_map` | `array<int, 128>` |
|---|---|---|
| Lookup | O(1) amortised + hash | O(1) exact, one instruction |
| Memory | Heap allocated | 512 bytes on stack |
| Cache | Pointer chasing | Sequential, cache-friendly |

### `map<char, int>` Is Also O(1) for This Problem

`map` lookup is O(log N) where N is map size. Since size is bounded by 128: `log(128) = 7` — a fixed constant regardless of input string length. All three structures are O(1) with respect to input size, just different constant factors:

```
map<char,int>        O(log 128) = 7 comparisons
unordered_map        O(1) amortised + hash overhead
array<int, 128>      O(1) exact — fastest
```

---

## #3 — Python Solutions

### Dict + `defaultdict` Version

```python
from collections import defaultdict

class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        mapping = defaultdict(lambda: -1)
        start = 0
        longest = 0
        for i, c in enumerate(s):
            start = max(start, mapping[c] + 1)
            longest = max(longest, i - start + 1)
            mapping[c] = i
        return longest
```

`defaultdict(lambda: -1)` returns `-1` for unseen characters — same sentinel trick as `array<int,128>`. `max(start, mapping[c] + 1)` replaces the explicit `if mapping[c] >= start` guard entirely — if stale, `max` keeps `start` unchanged.

### Alternative: String-as-Window Approach

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        window = ""
        longest = 0
        for c in s:
            idx = window.find(c)
            if idx != -1:
                window = window[idx+1:]
            window += c
            longest = max(longest, len(window))
        return longest
```

Intuitive — `window` literally contains the current substring. But O(N²) due to:
- `find` — O(N) linear scan
- `window[idx+1:]` — O(N) slice creates new string
- `window += c` — O(N) string concatenation (strings are immutable in Python)

### Version Lineage — Who Loses the Last Window

```
Original Python (measures only on collision)  →  needs final max(longest, len(s)-start) patch
String window (measures every iteration)      →  correct, but O(N²)
defaultdict version (measures every iteration) →  correct and O(N)
```

---

## #208 — Implement Trie

### Bug: `return false` Inside Loop Body

```cpp
for (char c : word) {
    if (auto it = children.find(c); it != children.end()) {
        current = it->second;
    }
    return false;  // ← bare statement, always returns after first character
}
```

Must be in the `else` branch:

```cpp
} else {
    return false;
}
```

### Canonical Solution

```cpp
class Trie {
public:
    Trie() : root(make_unique<Node>()) {}

    void insert(const string& word) {
        Node* current = root.get();
        for (char c : word) {
            int idx = c - 'a';
            if (!current->children[idx])
                current->children[idx] = make_unique<Node>();
            current = current->children[idx].get();
        }
        current->is_word_end = true;
    }

    bool search(const string& word) {
        const Node* current = traverse(word);
        return current && current->is_word_end;
    }

    bool startsWith(const string& prefix) {
        return traverse(prefix) != nullptr;
    }

private:
    struct Node {
        array<unique_ptr<Node>, 26> children;
        bool is_word_end = false;
    };

    unique_ptr<Node> root;

    const Node* traverse(const string& s) const {
        const Node* current = root.get();
        for (char c : s) {
            int idx = c - 'a';
            if (!current->children[idx]) return nullptr;
            current = current->children[idx].get();
        }
        return current;
    }
};
```

### Design Improvements Over Initial Version

**`unique_ptr` over `shared_ptr`**
Each node has exactly one parent — no shared ownership exists. `shared_ptr` carries atomic reference counting overhead for no benefit. `unique_ptr` is zero-overhead.

**`array<unique_ptr<Node>, 26>` over `map<char, shared_ptr<Node>>`**
Fixed alphabet → fixed array. O(1) exact lookup vs O(log 26) tree traversal. Better cache locality.

**`char c` removed from Node**
The character is already the key in the parent's structure. Storing it in the node is redundant.

**`traverse` helper eliminates duplicated logic**
`search` and `startsWith` share identical traversal — extract into private `traverse` returning `nullptr` on miss. Both functions become one-liners.

### Complexity

| Operation | Time | Space |
|---|---|---|
| `insert` | O(L) | O(L) new nodes worst case |
| `search` | O(L) | O(1) |
| `startsWith` | O(L) | O(1) |
| Total space | — | O(N·L·26) |

---

## Smart Pointer Deep Dive

### `unique_ptr` + Raw Pointer Pattern

Ownership and access are separate concerns:

```
unique_ptr  — owns, manages lifetime, cannot be copied
raw pointer — non-owning handle, freely copyable, read OR write
```

Raw pointers obtained from `unique_ptr::get()` can freely modify the object — the only restriction is they cannot affect ownership:

```cpp
Node* current = root.get();
current->is_word_end = true;                    // modify ✓
current->children[idx] = make_unique<Node>();   // modify ✓
delete current;                                 // NEVER — unique_ptr owns the memory ✗
```

### When `shared_ptr` Is Actually Needed

`shared_ptr` is needed when **multiple owners exist** — the object must survive until the last owner releases it.

**Graph with shared nodes (diamond pattern):**
```cpp
//     A
//    / \
//   B   C
//    \ /
//     D   ← owned by both B and C — unique_ptr impossible
auto D = make_shared<GraphNode>(4);
B->neighbours.push_back(D);   // ref count = 2
C->neighbours.push_back(D);   // ref count = 2
// D destroyed only when both B and C release it
```

**Shared cache entry:**
```cpp
shared_ptr<CacheEntry> get(const string& key) {
    return store[key];  // caller and cache co-own the entry
}
// cache can evict — caller's copy keeps entry alive until done
```

### `weak_ptr` — Observe Without Extending Lifetime

Solves circular reference problem. `shared_ptr` A → B and B → A creates a cycle — neither ever destroyed. Making one direction `weak_ptr` breaks the cycle:

```cpp
weak_ptr<MarketData> data;   // doesn't increment ref count

if (auto d = data.lock()) {  // lock() returns shared_ptr if still alive
    cout << d->price;
} else {
    cout << "data expired";  // original was destroyed
}
```

### External Raw Pointer Access

Raw pointers for external read/write access are fine when the **lifetime contract is clear**:

```cpp
// Safe — short-lived, used within known scope
const Order* bid = book.getBestBid();
if (bid && bid->price > threshold) execute();
// bid goes out of scope here, never stored

// Dangerous — stored raw pointer, lifetime unclear
class Strategy {
    const Order* cached_bid;   // who guarantees OrderBook lives?
};
```

### Smart Pointer Decision Table

| Situation | Use |
|---|---|
| Single clear owner | `unique_ptr` |
| Multiple owners, last one cleans up | `shared_ptr` |
| Observe without owning, detect destruction | `weak_ptr` |
| Short-lived traversal or modification | raw pointer from `get()` |
| Long-lived external storage | `weak_ptr` or `shared_ptr` |

In hot-path trading code — market data feeds, order book snapshots — raw pointer access from `unique_ptr` is standard for performance. `shared_ptr` is used for longer-lived shared state like configuration or cached reference data. The discipline is enforcing the lifetime contract clearly.

---

## Key Takeaways

- In-place grid mutation during BFS: always mark on **push**, not on pop — marking on pop allows the same cell to be pushed multiple times before processing.
- The `array<int, 128>` optimisation for fixed-alphabet problems eliminates hash overhead entirely — 512 bytes on the stack vs heap-allocated map nodes.
- `map<char,int>` is O(1) for this problem because its size is bounded by 128 — `log(128) = 7` is a constant. All three structures are O(1) w.r.t. input size, just different constant factors.
- `unique_ptr` + raw pointer is the correct pattern for tree traversal and modification — `shared_ptr` solves shared ownership, not the need to copy a pointer.
- Raw pointers from `unique_ptr` can freely read and write — the only restriction is they cannot `delete` or otherwise affect ownership.
- `weak_ptr` is the correct tool for external long-lived observation — it detects destruction without preventing it, and breaks `shared_ptr` cycles.
- `defaultdict(lambda: -1)` in Python mirrors the `array.fill(-1)` sentinel pattern in C++ — eliminates explicit existence checks with a guaranteed safe default.

---

*Logged from Claude study session · March 26, 2026*



# C++ LeetCode Study Log

**Sunday, March 29, 2026**

-----

## Problems Covered

|# |Problem |Difficulty|Status |
|----|---------------------------|----------|---------------------------------------------------------|
|#322|Coin Change |Medium |Solved — full evolution from backtracking to bottom-up DP|
|#4 |Median of Two Sorted Arrays|Hard |Solved — canonical partition binary search |

-----

## #322 — Coin Change: Full Evolution

### Stage 1 — Backtracking + Container (Wrong Algorithm)

```cpp
void recursive(const vector<int>& coins, size_t begin, int amount,
int count, vector<int>& counts) {
if (amount == 0) { counts.push_back(count); return; }
if (begin == coins.size()) return;
for (size_t i = begin; i < coins.size(); ++i) {
if (amount < coins[i]) continue;
amount -= coins[i]; // Bug: should be coins[i] not coins[begin]
recursive(coins, i, amount, count + 1, counts);
amount += coins[i];
}
}
```

**Bug:** `coins[begin]` used instead of `coins[i]` in original — always subtracts the wrong coin when `i != begin`.

**Fundamental problem:** collecting all paths into a container then scanning for minimum is exponential. The container is a signal that the wrong algorithm is being used — coin change only needs a single integer answer.

-----

### Stage 2 — Backtracking, Value Return

```cpp
int recursive(const vector<int>& coins, int amount) {
if (amount == 0) return 0;
int count = -1;
for (int coin : coins) {
if (amount < coin) continue;
int sub = recursive(coins, amount - coin);
if (sub != -1) {
int candidate = sub + 1;
count = (count == -1) ? candidate : min(candidate, count);
}
}
return count;
}
```

**Improvements:**

- Container eliminated — `-1` propagates as “impossible” sentinel
- `begin` parameter removed — coin change has no ordering constraint, every coin is valid at every step
- `amount - coin` passed directly — no mutate/restore needed since `amount` is a local value, not shared mutable state

**Why mutate/restore is unnecessary here:**

In backtracking, `amount` is shared across siblings — undo is essential. Here each call gets its own copy. After `amount += coin` restores the local value, the next iteration does `amount -= next_coin` fresh from the original anyway. Passing `amount - coin` directly makes the independence explicit.

**Still exponential** — same subproblems recomputed many times.

-----

### Stage 3 — Top-Down DP (Memoization)

```cpp
class Solution {
public:
int coinChange(vector<int>& coins, int amount) {
vector<int> memo(amount + 1, -2); // -2 = unvisited
return recursive(coins, amount, memo);
}
private:
int recursive(const vector<int>& coins, int amount, vector<int>& memo) {
if (amount == 0) return 0;
if (memo[amount] != -2) return memo[amount]; // reuse subproblem

int count = -1;
for (int coin : coins) {
if (amount < coin) continue;
int sub = recursive(coins, amount - coin, memo);
if (sub != -1) {
int candidate = sub + 1;
count = (count == -1) ? candidate : min(candidate, count);
}
}
return memo[amount] = count;
}
};
```

**Why `vector<int>` not `unordered_map`:**
Keys are integers in `[0, amount]` — a perfect fit for a vector. Direct index lookup vs hash overhead. Same reason `array<int,128>` beats `unordered_map<char,int>` for fixed domains.

**Sentinel values:**

- `-2` = unvisited (not yet computed)
- `-1` = impossible (no valid combination)
- `≥ 0` = valid answer

-----

### Stage 4 — Bottom-Up DP (Iterative)

```cpp
class Solution {
public:
int coinChange(vector<int>& coins, int amount) {
vector<int> dp(amount + 1, -1);
dp[0] = 0;
const int n = static_cast<int>(dp.size());

for (int i = 1; i < n; ++i) {
int count = -1;
for (int coin : coins) {
if (i - coin >= 0 && dp[i - coin] != -1) {
int candidate = dp[i - coin] + 1;
count = (count == -1) ? candidate : min(count, candidate);
}
}
dp[i] = count;
}
return dp[amount];
}
};
```

**Direction:** starts at base case `dp[0] = 0` and builds upward to `dp[amount]` — opposite of top-down which starts at `amount` and recurses down to `0`.

**Order guarantee:** when computing `dp[i]`, every `dp[i - coin]` is already computed since `i - coin < i` always.

**`dp` naming:** conventional shorthand for “dynamic programming table” — same as how `i,j,k` are conventional loop variables. Signals to any reader that this array stores subproblem answers.

-----

### Complexity Progression

|Stage |Time |Space |Correct|
|--------------------------|-------------|----------------------|-------|
|Backtracking + container |O(S^n) |O(n) stack + counts |Buggy |
|Backtracking, value return|O(S^n) |O(n) stack |Correct|
|Top-down DP |O(amount × n)|O(amount) memo + stack|Correct|
|Bottom-up DP |O(amount × n)|O(amount) |Correct|

-----

### The Four-Stage Mental Model for DP Problems

```
1. Backtracking — recursion, no reuse → not DP
2. Value-returning recursion — recursion, no reuse → not DP, cleaner
3. Top-down DP — recursion + memo → DP
4. Bottom-up DP — iteration + table → DP
```

Adding the memo table is what makes it DP — not the switch to iteration.

-----

## Greedy Algorithms

### Core Idea

At each step, make the **locally optimal choice** and never reconsider it. No backtracking, no exploring alternatives.

```
Greedy mantra: "Take the best option available right now and move on."
```

### When Greedy Works — Activity Selection

Always pick the meeting that ends earliest. By leaving maximum remaining time, no future choice can make you regret this selection. The **exchange argument** holds — swapping the greedy choice for any other never improves the result.

### When Greedy Fails — Coin Change

```
coins = [1, 3, 4], amount = 6

Greedy (take largest first):
take 4 → 2 remaining → take 1 → take 1 → 3 coins

Optimal:
take 3 → take 3 → 2 coins
```

Choosing `4` seemed best locally but forced two `1`s. The exchange argument fails — no consistent rule guarantees the greedy choice leads to the global optimum for arbitrary denominations.

**Note:** US coins `[25,10,5,1]` are specifically structured so greedy always works. Arbitrary denominations require DP.

### Greedy vs DP Decision

```
"If I make this choice now, could I ever regret it later?"

No → Greedy sufficient
Yes → Need DP
```

|Property |Greedy |DP |
|-----------------------|------------------|-----------------------|
|Optimal substructure |Yes |Yes |
|Greedy choice property |Yes |No |
|Reconsider past choices|Never |Via memoization |
|Time complexity |Usually O(N log N)|Usually O(N²) or O(N·M)|

### Common Greedy Problems

|Problem |Greedy Choice |Why It Works |
|------------------------------|--------------------------|--------------------------|
|Activity selection |Earliest end time |Maximises remaining time |
|#455 Assign Cookies |Smallest sufficient cookie|Wastes least capacity |
|#435 Non-overlapping Intervals|Earliest end time |Same as activity selection|
|#763 Partition Labels |Extend to last occurrence |Ensures self-containment |
|Dijkstra’s shortest path |Nearest unvisited node |Optimal substructure holds|

-----

## Dynamic Programming — Full Picture

### Three Categories

**1. Optimisation** — find minimum, maximum, or best value:

```
#322 Coin Change dp[i] = min(dp[i-coin] + 1)
#300 Longest Increasing Subsequence dp[i] = max(dp[j] + 1)
#72 Edit Distance dp[i][j] = min(insert, delete, replace)
#53 Maximum Subarray dp[i] = max(dp[i-1] + nums[i], nums[i])
```

**2. Counting** — count the number of ways:

```
#70 Climbing Stairs dp[i] = dp[i-1] + dp[i-2]
#518 Coin Change II dp[i] += dp[i-coin]
#62 Unique Paths dp[i][j] = dp[i-1][j] + dp[i][j-1]
```

**3. Existence / Decision** — does a solution exist:

```
#139 Word Break dp[i] = any(dp[i-len] && word matches)
#416 Partition Equal Subset Sum dp[i] = dp[i] || dp[i-num]
```

### The Unifying Recurrence

All three categories share the same structure — only the combination operator changes:

```
Optimisation: dp[i] = min/max(dp[j] + cost)
Counting: dp[i] = sum(dp[j])
Existence: dp[i] = any(dp[j] && condition)
```

### Return Type as Algorithm Signal

|Goal |Return type|Container|Undo step |
|---------------|-----------|---------|-----------------|
|All solutions |`void` |Yes |Yes |
|First solution |`bool` |No |Yes — until found|
|Count solutions|`int` |No |Yes |
|Optimum value |`int` |No |Sometimes (memo) |

**The rule:** if your backtracking container is only ever used to compute a single aggregate value (min, max, count), that’s a signal DP or value-returning recursion would be cleaner and faster.

### Naturally DP vs Evolved From Backtracking

|Problem type |Evolved from backtracking?|Example |
|---------------------------------|--------------------------|---------------------------|
|Choose from options, find optimum|Yes |Coin change, word break |
|Array/sequence recurrence |No |Max subarray, LIS |
|Grid traversal |No |Unique paths, edit distance|

### Top-Down vs Bottom-Up

| |Top-down |Bottom-up |
|------------------|-------------------------|---------------------------------|
|Implementation |Recursive + memo |Iterative + table |
|Direction |Target → base case |Base case → target |
|Subproblems solved|Only reachable ones |All of them |
|Call stack |O(N) deep |None |
|Code derivation |Natural from backtracking|Requires knowing dependency order|

-----

## #4 — Median of Two Sorted Arrays

### Why Median Comparison/Elimination Fails

The median elimination strategy is **fundamentally unworkable**, not just incorrectly implemented. The invariant “the combined median lies within the remaining windows” cannot be maintained because the two arrays have different sizes. Eliminating equal amounts from both sides changes the combined median unpredictably:

- **Aggressive elimination** — discards the actual median
- **Conservative elimination** — windows don’t shrink, infinite loop

There is no consistent elimination rule that works for all size combinations. This is the wrong mental model entirely.

### Correct Approach: Binary Search on Partition

The right question is not “which half can I discard?” but “where does the partition that splits the combined array in half sit?”

**Invariant:**

```
p1 + p2 == half (left halves have correct total count)
maxLeft1 <= minRight2 && maxLeft2 <= minRight1 (cross-partition order holds)
```

**Binary search direction is always unambiguous:**

```
maxLeft1 > minRight2 → p1 too large, move left
maxLeft2 > minRight1 → p1 too small, move right
both hold → valid partition found
```

```cpp
class Solution {
public:
double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
if (nums1.size() > nums2.size())
return findMedianSortedArrays(nums2, nums1); // always search smaller

const int m = static_cast<int>(nums1.size());
const int n = static_cast<int>(nums2.size());
const int half = (m + n + 1) / 2;

int lo = 0, hi = m;
while (lo <= hi) {
int p1 = lo + (hi - lo) / 2;
int p2 = half - p1;

int maxLeft1 = (p1 == 0) ? INT_MIN : nums1[p1 - 1];
int minRight1 = (p1 == m) ? INT_MAX : nums1[p1];
int maxLeft2 = (p2 == 0) ? INT_MIN : nums2[p2 - 1];
int minRight2 = (p2 == n) ? INT_MAX : nums2[p2];

if (maxLeft1 <= minRight2 && maxLeft2 <= minRight1) {
if ((m + n) % 2 == 1)
return max(maxLeft1, maxLeft2);
return (max(maxLeft1, maxLeft2) + min(minRight1, minRight2)) / 2.0;
} else if (maxLeft1 > minRight2) {
hi = p1 - 1;
} else {
lo = p1 + 1;
}
}
return 0.0;
}
};
```

### Why `INT_MIN` / `INT_MAX` as Sentinels

```
p1 == 0 → nums1 contributes nothing to left half → maxLeft1 = INT_MIN
ensures it never fails the maxLeft1 <= minRight2 check

p1 == m → nums1 contributes nothing to right half → minRight1 = INT_MAX
ensures it never passes as a minimum incorrectly
```

### Trace: `nums1 = [1,4,5]`, `nums2 = [2,3]`

```
m=3, n=2, half=3, lo=0, hi=3

p1=1, p2=2:
maxLeft1=1, minRight1=4, maxLeft2=3, minRight2=INT_MAX
1 <= INT_MAX ✓, 3 <= 4 ✓ → valid partition
total=5 odd → return max(1,3) = 3 ✓
```

### Complexity

| |Median Elimination |Partition Binary Search|
|-----------|--------------------|-----------------------|
|Time |Unbounded — can loop|O(log(min(m,n))) |
|Space |O(1) |O(1) |
|Correctness|Fundamentally broken|Provably correct |

-----

## Key Takeaways

- If a backtracking container is only used to compute a single aggregate (min, max, count), that’s a signal to use DP or value-returning recursion instead.
- The mutate/restore pattern from backtracking (`amount -= coin ... amount += coin`) is unnecessary when `amount` is passed by value — passing `amount - coin` directly makes the independence of each loop iteration explicit.
- DP is a paradigm, not an implementation style. Top-down (recursion + memo) and bottom-up (iteration + table) are both DP. The memoization is what makes it DP, not the switch to iteration.
- Greedy fails for coin change because the exchange argument doesn’t hold for arbitrary denominations — US coins are specifically structured to make greedy work.
- The coin change problem is the textbook example for why greedy fails and DP is necessary — know the `[1,3,4], amount=6` counterexample.
- For #4, the median elimination strategy is fundamentally unworkable — not just buggy. The partition approach asks the right question and yields a clean binary search with unambiguous direction at every step.
- Recognising when an intuitive approach is the wrong mental model entirely — and pivoting quickly — is the actual skill being tested in hard binary search problems.

-----

*Logged from Claude study session · March 29, 2026*

# C++ LeetCode Study Log
**Wednesday, April 1, 2026**

---

## Problems Covered

| # | Problem | Difficulty | Status |
|---|---|---|---|
| #881 | Rescue Boats | Medium | Two-pointer + greedy proof |
| #56 | Merge Intervals | Medium | In-place marking → result vector |
| #48 | Rotate Image | Medium | Cyclic swap vs transpose+reverse |
| #371 | Sum of Two Integers | Medium | Bit manipulation, C++17 UB discussion |
| #76 | Minimum Window Substring | Hard | Sliding window, hidden O(N²) bug |
| #128 | Longest Consecutive Sequence | Medium | Hash set, sequence-start optimisation |
| #133 | Clone Graph | Medium | DFS + visited map, smart pointer pitfalls |

---

## #881 — Rescue Boats: Greedy + Two-Pointer

### Progression

| Version | Time | Space | Key Change |
|---|---|---|---|
| Initial (nested loop) | O(N²) | O(N) | onboard[] visited array |
| Intermediate | O(N log N) | O(N) | lightest pointer, O(1) inner |
| Final | O(N log N) | O(1) | Two-pointer, no auxiliary array |

### Final Solution

```cpp
sort(people.begin(), people.end());
while (lo <= hi) {
    if (people[lo] + people[hi] <= limit) ++lo;
    --hi;
    ++boats;
}
```

`weighiest--` post-decrements — the value used in the comparison is `people[weighiest]` before the decrement. The decrement happens regardless of the condition, meaning the heaviest person always consumes a boat.

### Greedy vs Two-Pointer

These are independent concepts applied simultaneously:

- **Greedy** — the *strategy*: always send the heaviest person, pair with the lightest if possible.
- **Two-pointer** — the *implementation technique*: two indices move toward each other to avoid redundant scanning.

| | Single pointer | Two-pointer |
|---|---|---|
| **Greedy** | Intermediate version | Final version |
| **Not greedy** | Linear scan | Pair-sum search |

### Exchange Argument (Correctness Proof)

Suppose optimal solution S pairs `(Heavy, Mid)` leaving `Light` unpaired. Swap Mid and Light:

- `(Heavy, Light)` — valid, since `Light ≤ Mid`, so `Heavy + Light ≤ Heavy + Mid ≤ limit` ✓
- `Mid` takes Light's old slot — worst case Mid goes alone, no extra boat ✓

The swap never increases boat count. Any solution can be iteratively transformed into the greedy solution without cost increase — therefore greedy is optimal.

### Algorithm Strategy Spectrum

| | Greedy | Dynamic Programming | Backtracking |
|---|---|---|---|
| **Past decisions** | Never revisited | Stored and reused | Actively undone |
| **Correctness** | Exchange argument required | Always optimal | Always complete |
| **Typical complexity** | O(N log N) | O(N²) or O(N³) | Exponential |

Backtracking is a depth-first search over a decision tree — it needs full path history to backtrack up the tree. Greedy never builds that tree at all.

### Two-Person Constraint

The constraint is explicit in the problem, not an assumption. Without it the problem becomes NP-hard Bin Packing — no polynomial exact solution exists. The two-person constraint is precisely what makes the exchange argument clean.

```
[60, 60, 60, 100, 100, 100], limit = 180

"Pack densely": (60,60,60), (100), (100), (100) = 4 boats
"Greedy pair":  (60,100),   (60,100), (60,100)  = 3 boats
```

---

## #56 — Merge Intervals: In-place Marking → Result Vector

### Progression

| Version | Space | Notes |
|---|---|---|
| In-place mark & erase | O(1) | Clever but mutates input, non-idiomatic |
| Result vector | O(N) | Does not mutate input, immediately readable |

### Final Solution

```cpp
sort(intervals.begin(), intervals.end());
for (auto& interval : intervals) {
    if (result.empty() || result.back()[1] < interval[0])
        result.push_back(interval);
    else
        result.back()[1] = max(result.back()[1], interval[1]);
}
```

`||` short-circuits — `result.back()` is never called when `result.empty()` is true. Default lexicographic sort on `vector<vector<int>>` handles the multi-key sort without a custom comparator.

Not mutating the input is preferred in production — callers rarely expect their data modified.

---

## #48 — Rotate Image: Cyclic Swap vs Transpose+Reverse

### Two Approaches

```cpp
// Approach 1: Cyclic 4-way swap
int tmp = matrix[i][j];
matrix[i][j]         = matrix[n-1-j][i];
matrix[n-1-j][i]     = matrix[n-1-i][n-1-j];
matrix[n-1-i][n-1-j] = matrix[j][n-1-i];
matrix[j][n-1-i]     = tmp;

// Approach 2: Transpose + reverse rows (interview preferred)
for (int i = 0; i < n; ++i)
    for (int j = i+1; j < n; ++j) swap(matrix[i][j], matrix[j][i]);
for (auto& row : matrix) reverse(row.begin(), row.end());
```

| | Cyclic swap | Transpose + reverse |
|---|---|---|
| Swaps | n²/4 (each element once) | n²/2 + n²/2 (each element twice) |
| Readability | Requires deriving the cycle formula | Two obvious geometric steps |
| Interview | Hard to justify under pressure | Easy to state intuitively |

`col = (n+1)/2` is cleaner ceiling division than the ternary `n%2 ? n/2+1 : n/2`.

---

## #371 — Sum of Two Integers: Bit Manipulation

### Final Solution

```cpp
while (b) {
    int carry = (a & b) << 1;
    a ^= b;
    b = carry;
}
return a;
```

XOR computes sum without carry. AND then left-shift computes carry bits. The temporary variable for carry is essential — updating `a` before computing carry corrupts the result.

### C++17 Undefined Behaviour

Left-shifting into or past the sign bit is UB in C++17 and earlier. Safe here only due to problem constraints (`|a|, |b| ≤ 1000`). In C++20, two's complement is mandated by the standard and this concern disappears.

**Termination guarantee:** carry bits shift left one position per iteration and eventually fall off the MSB — the loop always terminates.

---

## #76 — Minimum Window Substring: Sliding Window

### Progression

| Version | Time | Key Issue |
|---|---|---|
| countIsEnough O(58) scan | O(58N) | Full array scan per shrink step |
| ss_effective_size counter | O(N) | O(1) counter replaces scan |
| substr() in hot path | O(N²) string cost | Hidden allocation per minimum update |
| Index tracking + single substr | O(N) | Production quality |

### The Hidden O(N²) Bug

```cpp
// WRONG — O(N) string allocation inside the shrink loop
ss = s.substr(start, window_size);

// CORRECT — track indices, single allocation at return
if (window_size < min_size) { min_size = window_size; result_start = start; }
...
return s.substr(result_start, min_size);  // one allocation
```

### Final Solution

```cpp
for (int i = 0; i < s_size; ++i) {
    // expand right
    if (t_count[ci] && ss_count[ci]++ < t_count[ci]) ++ss_effective_size;
    // shrink left
    while (ss_effective_size == t_size) {
        if (i - start + 1 < min_size) { min_size = i - start + 1; result_start = start; }
        if (ss_count[si] && --ss_count[si] < t_count[si]) --ss_effective_size;
        while (++start <= i && t_count[s[start] - 'A'] == 0);
    }
}
return s.substr(result_start, min_size);
```

Expand-then-shrink ordering eliminates the awkward `i <= s_size` flush iteration and off-by-one boundary conditions.

### ss_effective_size vs formed/required

Both are O(1) per update and equivalent in result:

| | formed/required | ss_effective_size |
|---|---|---|
| Tracks | Distinct chars fully satisfied | Total character satisfactions |
| Variables | Two (formed, required) | One |

---

## #128 — Longest Consecutive Sequence: Hash Set

### Final Solution

```cpp
unordered_set<int> s(nums.begin(), nums.end());
for (int n : s) {
    if (s.count(n - 1)) continue;   // only start from sequence beginnings
    int length = 1;
    while (s.count(n + length)) ++length;
    longest = max(longest, length);
}
```

Only expand rightward from sequence starts (where `n-1` not in set). No visited tracking needed — each number is touched at most once across all inner while iterations, giving O(N) amortised.

`unordered_map<int, bool>` merges two concerns (existence + visited marking) awkwardly. `unordered_set<int>` is the correct container. Iterating over the set rather than `nums` avoids redundant processing of duplicate values.

---

## #133 — Clone Graph: DFS + Visited Map

### Bugs Fixed

| Bug | Symptom | Fix |
|---|---|---|
| `static` map | Persists across LeetCode test cases | Remove static |
| `unique_ptr` ownership | Double-free with judge's cleanup | Raw pointer, judge owns memory |
| `int val` as map key | Fails if values not unique | `Node*` as key |
| No early return on visited | Duplicate neighbors added on revisit | Return immediately if in map |

### Final Solution

```cpp
Node* dfs(Node* node, unordered_map<Node*, Node*>& visited) {
    if (auto it = visited.find(node); it != visited.end())
        return it->second;           // early return before processing neighbors

    Node* clone = new Node(node->val);
    visited[node] = clone;           // mark BEFORE recursing — prevents cycles

    for (Node* n : node->neighbors)
        clone->neighbors.push_back(dfs(n, visited));

    return clone;
}
```

Mark as visited before recursing into neighbors — prevents infinite loops on cycles. Key on `Node*` not `int val` — the problem does not guarantee unique values.

### emplace() to Avoid Double Lookup

```cpp
// Two lookups — redundant
nodes.emplace(node->val, new Node(node->val));
c_node = nodes[node->val];

// One lookup — use emplace return value directly
auto [it, _] = nodes.emplace(node->val, new Node(node->val));
c_node = it->second;
```

Even at amortised O(1), double lookup doubles the constant factor, doubles worst-case exposure, and is non-idiomatic. In hot-path trading code where hash maps run millions of times per second, constants matter.

---

## Key Takeaways

- **Greedy and two-pointer are independent axes** — greedy is the strategy (what to choose), two-pointer is the implementation technique (how to traverse). The same greedy strategy can be implemented with or without two-pointer.
- **Exchange argument template:** assume optimal S differs from greedy → identify first difference → swap to look more like greedy → show swap cannot worsen result → conclude greedy is optimal.
- **Two-person constraint in #881 is load-bearing** — removing it turns the problem into NP-hard Bin Packing. Recognising which constraint makes an algorithm tractable is what interviewers probe for.
- **Expand-then-shrink is the canonical sliding window pattern** — avoids flush iterations, off-by-one conditions, and `i <= size` loop bounds.
- **substr() in a hot path is O(N²)** — string allocation inside a loop that runs O(N) times produces quadratic string cost despite linear loop structure. Track indices; allocate once at return.
- **static local variables persist across LeetCode test cases** — never use for per-call state.
- **Mark visited before recursing, not after** — marking on pop allows the same node to be pushed multiple times before processing, producing duplicates.
- **emplace() returns an iterator** — `auto [it, _] = map.emplace(k, v)` avoids a second lookup; idiomatic C++17.
- **Left-shifting into the sign bit is UB in C++17** for signed integers — resolved in C++20 where two's complement is mandated.
- **O(58N) is technically O(N)** since 58 is a constant — but eliminating it removes a real constant factor that matters in high-frequency trading hot paths.

---

*Logged from Claude study session · April 1, 2026*

# C++ LeetCode Study Log
**Friday, April 3, 2026**

---

## Problems Covered

| # | Problem | Difficulty | Status |
|---|---|---|---|
| #647 | Palindromic Substrings | Medium | Center expansion O(N²) + Manacher's O(N) overview |
| #11 | Container With Most Water | Medium | O(N²) pruning → O(N) two-pointer |
| #139 | Word Break | Medium | Backtracking → memoized top-down DP |

---

## #647 — Palindromic Substrings: Center Expansion

### Progression

| Version | Time | Space | Key Issue |
|---|---|---|---|
| Nested loop + ranges cache | O(N³) worst case | O(N) | ranges prunes palindrome-dense input only |
| Center expansion (two loops) | O(N²) | O(1) | Canonical solution |
| Manacher's algorithm | O(N) | O(N) | Rarely expected in interviews |

### Final Solution

```cpp
int count = 0;
for (int i = 1; i < size; ++i) {
    int left = i - 1, right = i;           // even-length center
    while (left >= 0 && right < size) {
        if (s[left--] != s[right++]) break;
        ++count;
    }
    left = i - 1; right = i + 1;           // odd-length, i is center
    while (left >= 0 && right < size) {
        if (s[left--] != s[right++]) break;
        ++count;
    }
}
return count + size;    // +size accounts for all single-character palindromes
```

`return count + size` cleanly avoids a separate loop for single characters — idiomatic once understood.

Post-decrement/increment inside the condition is correct here (indices move regardless, but mismatch triggers `break` immediately so moved indices are discarded) — but explicit movement after the check is clearer:

```cpp
if (s[left] != s[right]) break;
++count; --left; ++right;
```

### Why 2N-1 Centers

For a string of length N there are two types of centers:

```
"abcba"  (N=5)

Odd  centers (single char):    a  b  c  b  a   → 5 centers
Even centers (between chars):  a|b b|c c|b b|a  → 4 centers

Total: 5 + 4 = 9 = 2N - 1
```

The single-loop formulation `center / 2` and `center % 2` is a clever encoding but the variable name `center` is misleading — it is an index into an interleaved sequence of characters and gaps, not an actual center position. The two-loop version is more readable because the distinction between even and odd cases is stated explicitly.

### Manacher's Algorithm — Conceptual Overview

Manacher's reuses mirror symmetry to avoid redundant expansion. If a palindrome P is centered at C and reaches right boundary R, any center i inside `[L..R]` has a mirror `i' = 2C - i` on the left. The palindrome radius at i is at least `min(radius[i'], R - i)` — expansion is only needed when a palindrome might reach beyond R.

The string is transformed `"abc" → "#a#b#c#"` to unify odd and even palindromes into a single pass. The `right` boundary only ever moves rightward — total expansions across the entire loop are O(N), giving linear time.

**Honest assessment:** Manacher's is almost never expected in interviews. O(N²) center expansion is the correct answer for #647. Manacher's is worth knowing exists conceptually but memorising the implementation has low return on investment.

### Key Takeaways

- Two-loop center expansion is preferred over single-loop 2N-1 encoding — explicit odd/even split is immediately readable without decoding arithmetic.
- `return count + size` for single-character palindromes is an idiomatic shortcut worth keeping.
- Post-decrement/increment inside the condition is correct but subtle — prefer explicit movement after the check in production code.
- Manacher's achieves O(N) by tracking the rightmost-reaching palindrome and reusing mirror symmetry — total expansions bounded by N since the right boundary only ever moves forward.

---

## #11 — Container With Most Water: Two-Pointer

### Progression

| Version | Time | Space | Notes |
|---|---|---|---|
| Nested loop + pruning | O(N²) worst case | O(1) | Prunings correct but don't improve worst case |
| Two-pointer | O(N) | O(1) | Canonical solution |

### Pruning Analysis (Initial Version)

Both prunings in the O(N²) version were sound:

**Inner skip** — if `height[j+1] > height[j]`, then for fixed `i`, `area(i, j+1)` is strictly better: wider width and taller right boundary simultaneously. Safe to skip `j`.

**Outer skip** — if `height[i+1] < height[i]`, then `i+1` is dominated by `i` for all future `j`: both height and width factors are worse. Safe to skip `i+1`.

Neither fires on a monotonically increasing array — worst case remains O(N²).

### Final Solution

```cpp
int left = 0, right = size - 1;
while (left < right) {
    int area = 0;
    if (height[left] < height[right]) {
        area = height[left] * (right - left);
        ++left;
    } else {
        area = height[right] * (right - left);
        --right;
    }
    max_area = max(max_area, area);
}
```

Branching on `height[left] < height[right]` before multiplying avoids `min()` — the shorter side is already known so the multiplication is direct.

**Equal height case:** when `height[left] == height[right]`, the `else` branch moves `right` inward. This is correct — both sides are equally limiting so either can be moved. Moving both simultaneously would also be valid.

### Why Moving the Shorter Side Is Safe (Exchange Argument)

Suppose `height[left] ≤ height[right]`. For any `j` where `left < j < right`:

```
area(left, j) = min(h[left], h[j]) × (j - left)
              ≤ h[left] × (j - left)       ← height capped by h[left]
              < h[left] × (right - left)    ← width strictly smaller
              ≤ area(left, right)            ← current area
```

`left` is exhausted — no future pairing with `left` can beat the current area. Safe to discard.

### Pattern Connections

| Pattern | Problems |
|---|---|
| Two-pointer from both ends | #881, #11 |
| Exchange argument for greedy proof | #881, #11 |
| Discard the weaker side | #881 (lightest person), #11 (shorter wall) |

The structural similarity between #881 and #11 is direct — both reduce to "always eliminate the side that cannot possibly contribute to a better answer", proven by the same exchange argument template.

### Key Takeaways

- Two-pointer from both ends is the correct structure for optimisation over pairs in a sorted or bounded array.
- Moving the shorter side is proven safe by the exchange argument — not a heuristic.
- Branching before multiplying avoids `min()` when you already know which side is shorter.
- Equal-height case: either side can be moved; the `else` branch choosing `right` is correct.

---

## #139 — Word Break: Backtracking → Memoized Top-Down DP

### Progression

| Version | Time | Issue |
|---|---|---|
| Pure backtracking | O(2^N) | Overlapping subproblems recomputed |
| Memoized backtracking | O(N × W) | Top-down DP — each start position computed once |
| Bottom-up DP | O(N × W) | Iterative, but anti-intuitive to derive |

### Why Pure Backtracking Fails

On `s = "aaaaaab"`, `wordDict = ["a", "aa"]`:

```
backtrack(0) → tries "a"  → backtrack(1)
                          → tries "a" → backtrack(2) ...
             → tries "aa" → backtrack(2)  ← recomputed
```

`backtrack(2)` is reached via different prefixes and recomputed each time. With overlapping words this becomes O(2^N). Pure backtracking is correct for problems where subproblems are genuinely distinct (permutations, combinations). When the same subproblem recurs, memoization is required.

### Final Solution

```cpp
bool backtrack(const string& s, size_t start,
               const vector<string>& wordDict,
               unordered_map<size_t, bool>& memo) {
    if (start == s.size()) return true;    // base case ALWAYS first
    if (auto it = memo.find(start); it != memo.end()) return it->second;

    for (const auto& word : wordDict) {
        if (auto pos = s.find(word, start);
            pos != string::npos
            && pos == start
            && backtrack(s, start + word.size(), wordDict, memo)) {
            return memo[start] = true;
        }
    }
    return memo[start] = false;
}
```

**Base case before memo check** — `start == s.size()` is never in the cache and is cheap to check. The terminal state must come first: a general rule for all backtracking problems.

**Cache both directions** — `memo[start] = true` on success, `memo[start] = false` on failure. Caching only failure (the initial bug) leaves successful paths recomputed on second arrival.

**`memo[start] =` over `emplace`** — `emplace` won't overwrite an existing key. Both are equivalent here since the memo check at the top guarantees the key is absent, but `memo[start] =` is more idiomatic for "store this result" and pairs symmetrically in both branches.

### Bottom-Up DP Alternative

```cpp
vector<bool> dp(n + 1, false);
dp[0] = true;   // empty string always reachable
for (int i = 1; i <= n; ++i) {
    for (const auto& word : wordDict) {
        const int len = static_cast<int>(word.size());
        if (i >= len && dp[i - len] && s.substr(i - len, len) == word)
            dp[i] = true;
    }
}
return dp[n];
```

`dp[i]` means "the first i characters of s can be segmented." Builds forward from the base case rather than recursing backward.

### Top-Down vs Bottom-Up — General Relationship

Bottom-up is anti-intuitive for most DP problems, not just the ones encountered so far. The degree varies by problem type:

| Problem type | More intuitive direction |
|---|---|
| Fibonacci, Climbing Stairs, Coin Change | Bottom-up — linear forward dependency is obvious |
| Word Break, LCS, interval DP | Top-down — recursive structure mirrors the problem statement |

Bottom-up is almost always *derived from* the top-down solution by: identifying what the memo table represents, determining fill order (what must be computed before what), and replacing recursive calls with table lookups. Step 2 — fill order — is where intuition breaks down. The recursion handles ordering implicitly via the call stack; bottom-up must make it explicit.

**Practical advice for interviews:** start top-down, convert to bottom-up only if asked. Memoized backtracking is easier to derive correctly under pressure and is a fully valid DP solution.

### Backtracking Decision Table

| Situation | Approach |
|---|---|
| No overlapping subproblems | Pure backtracking |
| Overlapping subproblems, need all solutions | Backtracking + pruning |
| Overlapping subproblems, need existence/optimum | Memoized backtracking = top-down DP |
| Overlapping subproblems, iterative preferred | Bottom-up DP |

### Key Takeaways

- Pure backtracking is O(2^N) when subproblems overlap — the same start position is recomputed via different prefixes.
- Adding memoization is a cheap transformation (one cache lookup, two cache writes) that changes the complexity class from exponential to polynomial.
- Base case always before memo check — the terminal state is never in the cache and must be handled first. A general rule for all memoized backtracking.
- Cache both success and failure — caching only one direction leaves the other recomputed on repeated arrival.
- `memo[start] = value` is preferred over `emplace` for "store this result" — idiomatic and symmetric across both branches.
- Bottom-up anti-intuition is universal, not a personal gap — it reflects that bottom-up is a secondary derivation from recursive thinking, not a natural first instinct.

---

## Key Takeaways

- **Center expansion over single-loop encoding** — two explicit loops for odd and even cases are more readable than the 2N-1 encoded center variable.
- **Exchange argument applies beyond greedy** — both #881 and #11 use the same template: assume a deviation, show a swap cannot worsen the result, conclude the greedy choice is safe. Recognising this common structure across problems is what interviewers test for.
- **Memoized backtracking is top-down DP** — the transition from pure backtracking to memoized backtracking is the key skill: recognise overlapping subproblems, add a cache, store results in both success and failure directions.
- **Base case always first in backtracking** — before any memo lookup, pruning, or loop. The terminal state is never in the cache and must short-circuit immediately.
- **Bottom-up DP is a derived form** — it is almost always obtained by inverting the recursion, not derived independently. Top-down is the natural starting point under interview pressure.
- **Manacher's is O(N) but rarely expected** — conceptual understanding (mirror symmetry, rightmost boundary) is sufficient; implementation memorisation has low return on investment.

---

*Logged from Claude study session · April 3, 2026*

# C++ LeetCode Study Log
**Saturday, April 4, 2026**

---

## Problems Covered

| # | Problem | Difficulty | Status |
|---|---|---|---|
| #15 | 3Sum | Medium | Backtracking → sort + two-pointer |
| #143 | Reorder List | Medium | Deque O(N) space → find middle + reverse + merge O(1) |

---

## #15 — 3Sum: Backtracking → Sort + Two-Pointer

### Progression

| Version | Time | Space | Deduplication |
|---|---|---|---|
| Backtracking + sort per triplet + comparison loop | O(N³) + O(M×3) | O(N) stack + O(M) results | Explicit comparison against all results |
| Backtracking + sort upfront + unordered_set per level | O(N³) + O(N) per step | O(N) stack + O(N) set | Hash set tracking values used at current level |
| Backtracking + sort upfront + `i > start` skip | O(N³) worst case | O(N) stack | One-line structural skip |
| Sort + two-pointer | O(N²) | O(1) auxiliary | Skip equal neighbors in-place |

### Deduplication Strategies

**Strategy 1 — Sort + `i > start && nums[i] == nums[i-1]`**

The standard backtracking deduplication idiom on sorted arrays. Skips `nums[i]` only when `nums[i-1]` was already tried as the **same combination slot** in this call — meaning `i-1 >= start`:

```cpp
sort(nums.begin(), nums.end());
for (int i = start; i < n; ++i) {
    if (i > start && nums[i] == nums[i-1]) continue;  // skip duplicate at this level
    ...
}
```

Critical distinction — the boundary is `start`, not `0`. The same value can appear in different slots at different recursion levels:

```
nums = [-4, 2, 2] — both 2s correctly included:

slot 1 call (start=0): i=0 (-4), i>0? No → proceed
  slot 2 call (start=1): i=1 (2), i>1? No → proceed
    slot 3 call (start=2): i=2 (2), i>2? No → proceed → comb=[-4,2,2] ✓

Each 2 is picked at a different recursion level so i > start never fires for either.
```

**Strategy 2 — Unsorted + `unordered_set` per level**

```cpp
unordered_set<int> used;   // declared INSIDE backtrack — fresh per call
for (int i = start; i < n; ++i) {
    if (used.count(nums[i])) continue;
    used.insert(nums[i]);
    comb.push_back(nums[i]);
    backtrack(nums, i + 1, sum + nums[i], result, comb);
    comb.pop_back();
}
```

`used` must be local to each call — if shared across recursion levels it incorrectly blocks the same value from appearing in different slots. Tracks values not indices, so it handles consecutive duplicates of any length correctly.

| | Sort + `i > start` | Unsorted + hash set |
|---|---|---|
| Sort required | Yes | No |
| Space per level | O(1) | O(N) |
| Additional pruning | Enabled (`nums[i] > 0 → break`) | Not possible |
| Correctness risk | Clear — `start` boundary | Subtle — `used` must be local |

Sort + `i > start` is almost always preferred — simpler, no extra space, enables magnitude-based pruning.

### Final Solution — Sort + Two-Pointer

```cpp
vector<vector<int>> threeSum(vector<int> nums) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> result;
    const int n = static_cast<int>(nums.size());

    for (int i = 0; i < n - 2; ++i) {
        if (i > 0 && nums[i] == nums[i-1]) continue;   // skip duplicate first elements
        if (nums[i] > 0) break;                         // sorted — remaining sums all positive

        int left = i + 1, right = n - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                result.push_back({nums[i], nums[left], nums[right]});
                while (left < right && nums[left]  == nums[left+1])  ++left;
                while (left < right && nums[right] == nums[right-1]) --right;
                ++left; --right;
            } else if (sum < 0) ++left;
            else                --right;
        }
    }
    return result;
}
```

### The Mental Leap to Two-Pointer

"Find all triplets" pattern-matches immediately to backtracking. The unlock is a reframing:

```
Instead of: "find all three elements that sum to 0"
Think as:   "fix one element, then find a pair that sums to its negative"
```

Once `nums[i]` is fixed, the problem reduces to **two-sum on a sorted subarray** — the classic two-pointer problem. This reduction generalises:

| Problem | Reduction |
|---|---|
| 3Sum | Fix 1 → two-pointer 2Sum |
| 4Sum | Fix 2 → two-pointer 2Sum |
| #11 Container With Most Water | Fix both ends → move shorter side |
| #881 Rescue Boats | Fix heaviest → find best partner |

When a problem involves k elements summing to a target, fix k-2 of them and reduce to two-pointer on the remainder. Backtracking is the fallback when no such structure exists.

In interviews it is acceptable — and often expected — to start with backtracking and say: "this works but it's O(N³); if I fix the first element, the remaining two-element search becomes a two-pointer sweep reducing it to O(N²)." Showing the progression demonstrates deeper understanding than jumping to the optimal solution.

### Backtracking vs Two-Pointer Decision

| Problem | Backtracking appropriate? | Reason |
|---|---|---|
| #39 Combination Sum | Yes | Need all combinations, no exploitable structure |
| #46 Permutations | Yes | Genuinely distinct paths |
| #139 Word Break | No — use memo | Overlapping subproblems |
| #15 3Sum | No — use two-pointer | Sorted structure eliminates exhaustive search |

### Key Takeaways

- `i > start && nums[i] == nums[i-1]` skips duplicates **at the same recursion level** only — the `start` boundary is what makes it correct, not `i > 0`.
- The same value appearing in different slots at different recursion levels is not a duplicate — this is why `[-4, 2, 2]` correctly produces one triplet.
- Sort upfront enables both structural deduplication and magnitude-based pruning (`nums[i] > 0 → break`) — two benefits from one transformation.
- Unsorted + hash set works but `used` must be local per call, not shared — a subtle correctness requirement.
- 3Sum → fix one element → 2Sum on sorted subarray → two-pointer: internalise this reduction as a general pattern.

---

## #143 — Reorder List: Three Linked List Techniques

### Progression

| Version | Time | Space | Notes |
|---|---|---|---|
| Deque | O(N) | O(N) | Correct, intuitive, but unnecessary allocation |
| Find middle + reverse + merge | O(N) | O(1) | Canonical — composition of three fundamental techniques |

### Deque Solution

```cpp
deque<ListNode*> dq;
ListNode* current = head->next;
while (current) { dq.push_back(current); current = current->next; }

current = head;
while (!dq.empty()) {
    auto node = dq.back(); dq.pop_back();
    current->next = node; current = current->next;
    if (!dq.empty()) {
        node = dq.front(); dq.pop_front();
        current->next = node; current = current->next;
    }
}
current->next = nullptr;   // critical — last node still points to old neighbor without this
```

`current->next = nullptr` is non-obvious but mandatory — without it the last node retains its original `next` pointer causing a cycle or dangling chain.

### Three Sub-Patterns

This problem decomposes into three fundamental linked list techniques that appear independently across many problems:

| Technique | Problems |
|---|---|
| Slow/fast pointer to find middle | #876, #143, cycle detection |
| Reverse a linked list in place | #206, #143, #25 |
| Merge two linked lists alternately | #21, #143 |

Recognising the decomposition is the insight interviewers look for — not the deque shortcut.

### Final Solution

```cpp
void reorderList(ListNode* head) {
    // Step 1: find middle
    ListNode* slow = head, *fast = head, *median = head;
    while (fast && fast->next) {
        fast = fast->next->next;
        median = slow = slow->next;
    }

    // Step 2: cut and reverse second half
    auto current = median->next;
    median->next = nullptr;         // cut explicitly — clear intent
    if (!current) return;           // single element or empty — nothing to merge
    ListNode* previous = nullptr;
    while (current) {
        auto next = current->next;
        current->next = previous;
        previous = current;
        current = next;
    }

    // Step 3: merge two halves
    current = head;
    ListNode* second = previous;    // head of reversed second half
    while (second) {
        auto tmp1 = current->next;
        auto tmp2 = second->next;
        current->next = second;
        second->next  = tmp1;
        current = tmp1;
        second  = tmp2;
    }
}
```

### The Repurposed Variable Bug

The initial O(1) solution repurposed `fast` across all three steps — fast pointer in step 1, then reassigned to the reversed head in step 2 via:

```cpp
if(next == nullptr || fast == nullptr) fast = previous;
```

This caused `heap-use-after-free` on `[1]` (single element):

```
Step 1: fast = node(1), median = node(1)
Step 2: median->next = null → reverse loop never executes → fast never reassigned
Step 3: current = head = node(1), fast = node(1)
        fast->next = current->next = null
        current->next = fast → node(1)->next = node(1)  ← self-cycle
Judge traverses cycle → accesses freed memory → AddressSanitizer fires
```

**Fix attempted:** `while(fast && fast != current)` — guards the self-cycle case but for the wrong reason. Works incidentally because `fast == head` in the single-element case.

**Correct fix:** `if (!median->next) return;` — states the intent directly. If there is nothing after the median, there is no second half to merge. No reasoning about what `fast` might equal required.

### Code Review Checklist for Linked List Problems

| Risk | Guard |
|---|---|
| Repurposed variable across phases | Use dedicated named variables per phase |
| Dense multi-assignment initialisation | Split into separate statements |
| Last node retains old `next` | Explicitly set `current->next = nullptr` after merge |
| Empty or single-element input | Guard before each phase, not just at entry |
| Cycle from self-assignment | Explicit cut (`median->next = nullptr`) before reverse |

### Key Takeaways

- #143 decomposes into three independently useful patterns: find middle (slow/fast), reverse in place, merge alternately. Recognising the decomposition is the interview insight.
- Repurposing a variable across multiple phases of an algorithm makes correctness dependent on non-obvious invariants — dedicate one variable per role.
- `heap-use-after-free` from AddressSanitizer on a linked list problem almost always means a cycle was created — trace the single-element and two-element cases first.
- `if (!median->next) return` is the semantically correct early exit — prefer guards that state intent over guards that happen to work.
- `current->next = nullptr` after the merge loop is mandatory — the last node of the first half still holds its original pointer without it.

---

## Concepts: Backtracking Deduplication Patterns

### When to Sort First

Sorting upfront is almost always the right first step when the problem involves:
- Finding combinations summing to a target
- Deduplicating results
- Any pruning based on element magnitude

It is one of the first transformations to consider before writing the backtracking logic.

### Deduplication Rule Summary

```
Sorted array:
  if (i > start && nums[i] == nums[i-1]) continue;
  → skips duplicate values at the same combination slot
  → allows the same value in different slots at different levels

Unsorted array:
  unordered_set<int> used;  // LOCAL to this call
  if (used.count(nums[i])) continue;
  → tracks values tried at this slot only
  → must be re-declared each call, never shared
```

---

*Logged from Claude study session · April 4, 2026*

# C++ LeetCode Study Log
**Sunday, April 5, 2026**

---

## Problems Covered

| # | Problem | Difficulty | Status |
|---|---|---|---|
| #19 | Remove Nth Node From End of List | Medium | Two-pointer gap + dummy node pattern |
| #23 | Merge K Sorted Lists | Hard | Priority queue O(N log k) + complexity deep dive |

---

## #19 — Remove Nth Node From End: Dummy Node Pattern

### Correctness — Initial Version

Trace on `[1,2,3,4,5]`, n=2:
```
Initial: tail=1, nth=1, pre=null, count=0

iter 1: count(0) >= n-1(1)? No.  tail=2, count=1
iter 2: count(1) >= 1?     Yes.  pre=1, nth=2. tail=3, count=2
iter 3: count(2) >= 1?     Yes.  pre=2, nth=3. tail=4, count=3
iter 4: count(3) >= 1?     Yes.  pre=3, nth=4. tail=5, count=4
tail->next=null → exit

nth=4, pre=3 → pre->next=5 ✓
```

The `nth == head` branch exists solely to handle head removal — a signal that a dummy node would eliminate the special case entirely.

### Dummy Node — Eliminates the Head Special Case

```cpp
ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode dummy(0, head);
    ListNode* fast = &dummy, *slow = &dummy;

    // advance fast n+1 steps ahead of slow
    for (int i = 0; i <= n; ++i) fast = fast->next;

    // move both until fast falls off
    while (fast) {
        fast = fast->next;
        slow = slow->next;
    }

    // slow->next is the node to remove
    ListNode* del = slow->next;
    slow->next = del->next;
    del->next = nullptr;
    return dummy.next;
}
```

Trace on `[1,2,3,4,5]`, n=5 (remove head):
```
fast advances 6 steps from dummy: fast=null (falls off immediately)
while fast: never executes, slow=dummy

slow->next=1 (delete), slow->next=2
return dummy.next=2 ✓   — no special case needed
```

The dummy node makes `slow` always point to the **predecessor** of the target node — including when the target is head — absorbing the special case uniformly.

### Comparison

| | Initial version | Dummy node |
|---|---|---|
| Time | O(N) | O(N) |
| Space | O(1) | O(1) |
| Head removal | Special case branch | Handled uniformly |
| Pointers needed | 3 (tail, nth, pre) | 2 (fast, slow) |
| Gap mechanism | count >= n-1 | Fixed n+1 steps upfront |

### Key Takeaways

- The dummy node is the idiomatic pattern for any linked list problem where the head might be removed — it eliminates the `if (nth == head)` branch by making the predecessor always valid.
- Fixed n+1 gap upfront is cleaner than a counter condition — the gap is established once and both pointers move in lockstep thereafter.
- `del->next = nullptr` after removal — same discipline as #143: explicitly cut the removed node's pointer to avoid dangling references.

---

## #23 — Merge K Sorted Lists: Priority Queue + Complexity Deep Dive

### Complexity Notation

Using standard convention throughout: **k = number of lists, N = total nodes across all lists**.

### Priority Queue Solution

```cpp
ListNode* mergeKLists(vector<ListNode*>& lists) {
    auto dummy = ListNode();
    auto current = &dummy;
    auto comp = [](const ListNode* left, const ListNode* right) {
        return left->val > right->val;
    };
    priority_queue<ListNode*, vector<ListNode*>, decltype(comp)> pq(comp);

    for (auto& list : lists)
        if (list) pq.push(list);          // O(k log k) total

    while (!pq.empty()) {                 // N iterations
        current->next = pq.top(); pq.pop();  // O(log k)
        current = current->next;
        if (current->next)
            pq.push(current->next);          // O(log k)
    }
    return dummy.next;
}
```

The heap never exceeds k elements — at most one node per list is in the heap simultaneously. Each push/pop is therefore O(log k), not O(log N):

```
Initial pushes: O(k log k)
N nodes × O(log k) per push/pop = O(N log k)
─────────────────────────────────────────────
Total: O(N log k)
```

### `multiset` as Alternative

`multiset` can replace `priority_queue` with identical O(log k) insertion. The key difference is erase:

```cpp
multiset<ListNode*, decltype(comp)> ms(comp);
auto it = ms.begin();   // smallest — O(1)
ms.erase(it);           // O(log k) — red-black tree rebalance
```

| | `priority_queue` | `multiset` |
|---|---|---|
| Underlying structure | Binary heap (contiguous array) | Red-black tree (pointer-chasing) |
| Access | Top only | Any iterator |
| Erase arbitrary element | Not supported | O(log k) |
| Cache performance | Better | Worse |
| Duplicate handling | Natural | Requires `multiset` not `set` |

`multiset` wins when arbitrary element removal mid-stream is needed — `priority_queue` cannot do it without a full rebuild. This is the container trade-off interviewers probe: not just complexity but what each container actually supports.

### `const ListNode*` — Non-Modifying Version

When the input lists must not be modified:

```cpp
priority_queue<const ListNode*, vector<const ListNode*>, decltype(comp)> pq(comp);

while (!pq.empty()) {
    auto top = pq.top(); pq.pop();
    current->next = new ListNode(top->val, top->next);  // new node, original untouched
    current = current->next;
    if (current->next) pq.push(current->next);
}
```

`top->next` in the constructor is **essential** — it temporarily carries the remainder of the original chain so that `current->next != nullptr` correctly triggers the next push. Without it, only the first node of each list ever enters the heap and all subsequent nodes are silently lost.

`const ListNode*` enforces the non-modification contract at the type level — the compiler catches any accidental write to original nodes. In a trading system where order book data is shared across multiple consumers, this is the correct design.

**Memory note:** `new ListNode` leaks in production — the caller receives ownership but has no way to know the nodes were heap-allocated. Options: document the contract, return `unique_ptr`, or use an arena allocator (common in low-latency trading code).

### Sequential Merge — Simple But O(Nk)

```cpp
ListNode* mergeKLists(vector<ListNode*>& lists) {
    ListNode* head = nullptr;
    for (auto& list : lists)
        head = mergeTwoLists(head, list);
    return head;
}
```

Intuition says this should be O(N) since `mergeTwoLists` processes only the current pair. The trap: the merged list grows each iteration, so early lists are re-traversed in every subsequent merge:

```
merge(null,   list1): touches n1 nodes
merge(result, list2): touches n1+n2 nodes
merge(result, list3): touches n1+n2+n3 nodes
...
merge(result, listk): touches all N nodes
─────────────────────────────────────────
Total = N/k × (1+2+...+k) = O(Nk)
```

### Divide and Conquer — O(N log k), No Heap

```cpp
ListNode* mergeKLists(vector<ListNode*>& lists) {
    if (lists.empty()) return nullptr;
    int size = static_cast<int>(lists.size());
    while (size > 1) {
        for (int i = 0; i < size / 2; ++i)
            lists[i] = mergeTwoLists(lists[i], lists[size - 1 - i]);
        size = (size + 1) / 2;
    }
    return lists[0];
}
```

Merges in pairs like merge sort — each round touches all N nodes, but there are only log k rounds:

```
Round 1: [1,2] [3,4] ... → k/2 lists, N nodes touched
Round 2: [1,2,3,4] ...   → k/4 lists, N nodes touched
...
log k rounds × N nodes = O(N log k)
```

### Why Divide and Conquer Beats Sequential — The Core Insight

```
Total work = (elements per round) × (number of rounds)
```

**Sequential** minimises elements per round but pays with linearly growing rounds:

```
Elements per round: n1, n1+n2, n1+n2+n3, ...   ← growing
Rounds:             k                            ← linear
Product:            O(Nk)
```

**Divide and conquer** accepts full N elements every round but crushes the round count:

```
Elements per round: N     ← always full
Rounds:             log k ← logarithmic
Product:            O(N log k)
```

The counterintuitive conclusion: **processing fewer elements per round is not always cheaper** if it forces more rounds. Even though sequential merge always involves fewer than N elements per round, the additional rounds required outweigh the per-round savings.

Visualised:

```
Sequential:                    Divide and conquer:

L1 ──────────────────────►    L1 ──┐
L2 ──────────────────►    │    L2 ──┘merge──┐
L3 ──────────────►        │    L3 ──┐       │merge
L4 ────────────►          │    L4 ──┘merge──┘
     ↑ L1 re-touched k times    ↑ every list touched log k times
```

### The Same Pattern Everywhere

| Algorithm | "Fewer per round" trap | Balanced approach |
|---|---|---|
| K-way merge | Sequential — O(k) rounds, growing size | Divide & conquer — O(log k) rounds × O(N) |
| Sorting | Insertion sort — O(N) rounds × O(N) work | Merge sort — O(log N) rounds × O(N) |
| Search | Linear scan — O(N) rounds × O(1) | Binary search — O(log N) rounds × O(1) |
| BST operations | Linked list — O(N) levels | Balanced BST — O(log N) levels |

Logarithmic round count almost always beats minimising per-round work — log k grows so slowly that paying full N per round is a bargain.

### Full Comparison

| | Sequential merge | Priority queue | Divide and conquer |
|---|---|---|---|
| Time | O(Nk) | O(N log k) | O(N log k) |
| Space | O(1) | O(k) heap | O(log k) stack |
| Heap allocation | No | Yes | No |
| Implementation complexity | Lowest | Medium | Medium |
| Latency-sensitive use | Acceptable for small k | Standard | Preferred — no heap |

### Key Takeaways

- Heap size in priority queue is bounded by k, not N — push/pop is O(log k) not O(log N). Stating this distinction precisely in an interview signals depth.
- `multiset` vs `priority_queue`: identical asymptotic complexity, but `multiset` supports arbitrary erasure at O(log k). Use `multiset` when mid-stream removal of non-minimum elements is required.
- `top->next` in `new ListNode(top->val, top->next)` is essential for the non-modifying version — without it, only the first node of each list enters the heap and all subsequent nodes are silently dropped.
- Sequential merge is O(Nk) not O(N) — the merged accumulator grows each round, re-traversing early list nodes in every subsequent merge.
- Total work = elements per round × number of rounds. Fewer elements per round does not always win — more rounds can outweigh the per-round saving. This is the same reason merge sort beats insertion sort.
- Divide and conquer achieves O(N log k) without heap allocation — preferred in latency-sensitive trading code where heap overhead matters.

---

*Logged from Claude study session · April 5, 2026*

# C++ LeetCode Study Log
**Monday, April 6, 2026**

---

## Problems Covered

| # | Problem | Difficulty | Status |
|---|---|---|---|
| #152 | Maximum Product Subarray | Medium | O(N²) memo → O(N) max/min tracking + prefix/suffix proof |
| #33 | Search in Rotated Sorted Array | Medium | Binary search with pre-checked boundary equality |
| #417 | Pacific Atlantic Water Flow | Medium | BFS/DFS traversal + constexpr deep dive |
| #295 | Find Median from Data Stream | Hard | Sorted list O(N) → two heaps O(log N) |

---

## #152 — Maximum Product Subarray: Max/Min Tracking

### Progression

| Version | Time | Space | Notes |
|---|---|---|---|
| Memoized nested loop | O(N²) | O(N²) + hash overhead | Correctly diagnosed as ceiling for this approach |
| Max/min tracking | O(N) | O(1) | Canonical solution |
| Prefix/suffix scan | O(N) | O(1) | Alternative — proven correct via boundary argument |

### Why Pure Memoization Cannot Improve Beyond O(N²)

Each subarray product `[i..j]` depends on `product[i+1..j]` — no way to collapse the two-level dependency into a single pass. The overlapping subproblems are unavoidable with this formulation.

### The O(N) Insight — Dual Nature of Negatives

A large negative product can become the largest positive when multiplied by another negative. Tracking only max loses this information. Both must be tracked:

```cpp
int maxProduct(const vector<int>& nums) {
    int max_prod = nums[0], min_prod = nums[0], result = nums[0];

    for (int i = 1; i < static_cast<int>(nums.size()); ++i) {
        // compute both before updating — same discipline as #371
        int candidate_max = max({nums[i], max_prod * nums[i], min_prod * nums[i]});
        int candidate_min = min({nums[i], max_prod * nums[i], min_prod * nums[i]});
        max_prod = candidate_max;
        min_prod = candidate_min;
        result = max(result, max_prod);
    }
    return result;
}
```

Trace on `[2, -3, -4]`:
```
i=0: max=2,   min=2,   result=2
i=1: max=-3,  min=-6,  result=2
i=2: max=24,  min=-4,  result=24 ✓  (-6 × -4 = 24)
```

### The Three Candidates — What Each Means

```
nums[i]              → start a brand new subarray here
max_prod × nums[i]   → extend the best subarray ending at i-1
min_prod × nums[i]   → extend the worst subarray ending at i-1
                        (only useful when nums[i] is negative)
```

### The Implicit Boundary Insight

The mental block: "how does max_prod capture a subarray starting in the middle?" The resolution: `max_prod` at position i means "the best product of any subarray ending exactly at i." The starting index is never needed — the accumulated value IS the proof that some subarray achieves it. Explicit boundaries are almost never needed in subarray DP problems.

### Prefix/Suffix Method

Your original intuition independently discovered a known approach:

```
Total product positive → whole array is the answer ✓
Zero → splits array into independent subproblems ✓
Total product negative → must remove either leftmost or rightmost negative
```

The prefix scan covers all subarrays touching the left end. The suffix scan covers all subarrays touching the right end. Together they cover all optimal subarrays — proven by the boundary theorem below.

```cpp
int prefix = 1, suffix = 1;
for (int i = 0; i < n; ++i) {
    prefix = (prefix == 0 ? 1 : prefix) * nums[i];
    suffix = (suffix == 0 ? 1 : suffix) * nums[n-1-i];
    result = max({result, prefix, suffix});
}
```

### Boundary Theorem — Why Middle Subarrays Are Never Optimal

**Claim:** In a non-zero integer array, the optimal subarray always touches at least one end.

**Proof:** Suppose optimal subarray `nums[i..j]` is a pure middle subarray (i>0, j<n-1) with product P>0. Let L = product of `nums[0..i-1]`, R = product of `nums[j+1..n-1]`.

- If `L×R > 0`: entire array product `L×P×R > P`. Contradiction.
- If `L×R < 0`: exactly one of L, R is negative.
  - If `L<0`: `R>0`, so `nums[i..n-1]` has product `P×R > P` and touches the right end. Contradiction.
  - If `R<0`: `L>0`, so `nums[0..j]` has product `L×P > P` and touches the left end. Contradiction.

In all cases a boundary-touching subarray beats the middle. QED.

Zero breaks this proof because `L×R=0` escapes both sign cases — hence zeros require resetting the scan, splitting the array into independent non-zero segments.

### Diagnostic Questions for Subarray Problems

```
1. Additive operation?       → Single Kadane's (track max only)
2. Multiplicative operation? → Dual tracking (max and min)
3. Overlapping subproblems?  → Memoization or DP table
4. Sortable structure?       → Two-pointer
```

### Key Takeaways

- Tracking only max loses the "large negative in reserve" information — multiplicative problems require tracking both max and min.
- Temporary variables are essential when two values depend on each other's previous state — same discipline as #371 bit manipulation.
- `max_prod` implicitly encodes the optimal subarray's starting position — explicit boundaries are almost never needed in subarray DP.
- The optimal subarray in a non-zero array always touches at least one end — this is provable via sign analysis, not just empirical observation.
- Zeros break the boundary theorem — reset the scan at zeros, treating each non-zero segment independently.

---

## #33 — Search in Rotated Sorted Array: Binary Search

### Design Rationale — Pre-checked Boundary Equality

```cpp
if (nums[mid] == target) return mid;
if (nums[left] == target) return left;   // rule out boundary equality upfront
if (nums[right] == target) return right;
// all subsequent comparisons can be strictly < and >
```

This is a deliberate tradeoff — three explicit boundary checks at the top in exchange for strictly clean `<` and `>` comparisons in the logic below. After ruling out `nums[mid]`, `nums[left]`, `nums[right]` equalling target, the branch conditions are provably strict. The design is internally consistent.

### Standard Approach — One Branch Condition with `<=`

```cpp
while (left <= right) {
    int mid = left + (right - left) / 2;
    if (nums[mid] == target) return mid;

    if (nums[left] <= nums[mid]) {         // left half is sorted
        if (target >= nums[left] && target < nums[mid])
            right = mid - 1;
        else
            left = mid + 1;
    } else {                               // right half is sorted
        if (target > nums[mid] && target <= nums[right])
            left = mid + 1;
        else
            right = mid - 1;
    }
}
```

`nums[left] <= nums[mid]` uses `<=` to handle the two-element case `[3,1]` where `left==mid` — without it, the two-element case misclassifies which half is sorted.

### Core Invariant

After finding mid, exactly one of the two halves is always sorted. Identify which half is sorted, then decide which half the target falls into. Two decisions, two branches — clean separation of concerns.

### Key Takeaways

- Pre-checking boundary equality trades three extra comparisons for strict inequality in all subsequent logic — a valid deliberate design.
- `left <= right` in the while condition is necessary — without `=`, a single-element array where `left==mid==right` would exit without checking.
- `nums[left] <= nums[mid]` requires `<=` — the two-element case where `left==mid` must be classified as "sorted left half."
- One half is always sorted after finding mid — this is the invariant that makes O(log N) binary search possible on a rotated array.

---

## #417 — Pacific Atlantic Water Flow: BFS/DFS + constexpr

### Progression

| Version | Issue | Fix |
|---|---|---|
| Two-corner seeding | Corners `{0,col-1}` and `{row-1,0}` missed most valid cells | Seed entire boundary |
| Off-by-one boundary loops | `j < col-1` missed `j=col-1` corner | `j < col` |
| unordered_set + PairHash | O(1) amortised, pointer chasing, custom hash required | `vector<vector<bool>>` |
| Neighbors vector inside loop | O(M×N) heap allocations | `static constexpr` at class scope |
| Mark at pop (BFS) | Same cell pushed multiple times | Mark at push |

### Final Solution Structure

```
Seed Pacific:  entire top row (i=0) + entire left column (j=0)
Seed Atlantic: entire bottom row (i=row-1) + entire right column (j=col-1)
Traverse:      uphill BFS/DFS from each boundary
Result:        cells reachable by both traversals
```

### Reverse Flow Insight

Water flows downhill. To find which cells can reach the ocean, seed from the ocean boundary and expand **uphill** (`heights[nr][nc] >= heights[r][c]`). A cell reachable from the boundary in this uphill traversal can flow downhill back to the ocean. "Reachable from boundary" and "can reach ocean" are the same thing — this is why `pacific_ocean[i][j] = true` is semantically more honest than `visited[i][j] = true`.

### BFS vs DFS for This Problem

All three implementations produce identical results — traversal order does not affect which cells get marked reachable:

| | Recursive DFS | Iterative DFS (stack) | BFS (queue) |
|---|---|---|---|
| Stack overflow risk | Yes — 90K frames on 300×300 | No | No |
| Mark timing | At entry — naturally safe | At push | At push |
| Order | Depth-first | Depth-first | Level-by-level |
| Result | Identical | Identical | Identical |

### Mark at Push — Universal Rule

```
Mark at push/recurse → each cell enters container exactly once  ✓
Mark at pop          → cell can be pushed multiple times before processing ✗
```

Recursive DFS is naturally safe — `reachable[i][j] = true` as the first line marks before any recursion. Iterative stack and queue both require conscious discipline to mark at push.

### `static constexpr` Deep Dive

```cpp
class Solution {
    static constexpr int neighbors[4][2]{{1,0},{-1,0},{0,1},{0,-1}};
    // ...
};
```

| | `static int` | `static constexpr int` |
|---|---|---|
| Initialization | Runtime — first call | Compile time — baked into binary |
| Location | `.data` segment (writable) | `.rodata` segment (read-only) |
| Mutability | Mutable — no protection | Immutable — compiler + hardware |
| Hidden guard check | Yes — thread-safe init check every call | No — zero runtime overhead |
| Intent | Persists | Persists + immutable + compile-time |

`.rodata` is memory-mapped with write-protection at the hardware level — any modification causes a segfault, not just a compile error. Two layers of protection: compiler and OS.

**Scope vs lifetime — independent concepts:**
```
static   → controls lifetime  (persists for program duration)
scope    → controls visibility (where the name is accessible)

Function scope: long lifetime, visible only in that function
Class scope:    long lifetime, visible to all member functions ← correct for neighbors
```

`static constexpr` at function scope technically valid in C++17 for arrays, but class scope is universally supported and semantically correct — the direction table belongs to the class, not to any one function.

**General rule:**
```
Known at compile time + never changes → constexpr
Known at compile time + might change  → const
Not known at compile time             → static (with const if immutable)
```

### Key Takeaways

- Seed the entire boundary, not corners — the two-corner approach catches only a small subset of valid cells.
- `pacific_ocean` and `atlantic_ocean` are semantically more honest names than `visited` — they express reachability, not just traversal state.
- `vector<vector<bool>>` over `unordered_set<pair>`: O(1) direct index vs O(1) amortised hash, cache-friendly, no custom hash needed.
- `static constexpr` embeds values in `.rodata` at compile time — zero runtime initialization cost, hardware-enforced immutability.
- Mark at push is the universal BFS/DFS discipline — prevents the same cell from entering the container multiple times.
- Iterative DFS with explicit stack is production-safe; recursive DFS risks stack overflow at 90K frames on a 300×300 grid.

---

## #295 — Find Median from Data Stream: Two Heaps

### Progression

| Version | `addNum` | `findMedian` | Notes |
|---|---|---|---|
| Sorted list + iterator tracking | O(N) | O(1) | Correct but complex iterator logic |
| Two heaps | O(log N) | O(1) | Canonical solution |

### Two-Heap Invariant

```
lo (max-heap): lower half, size >= hi size
hi (min-heap): upper half

Always: every element in lo <= every element in hi
Median: lo.top()                          if sizes differ (odd count)
        (lo.top() + hi.top()) / 2.0       if sizes equal (even count)
```

### Final Solution

```cpp
class MedianFinder {
public:
    void addNum(int num) {
        lo.push(num);                      // always push to max-heap first
        hi.push(lo.top()); lo.pop();       // move max of lo to hi — maintains ordering

        if (lo.size() < hi.size()) {       // keep lo >= hi in size
            lo.push(hi.top()); hi.pop();
        }
    }

    double findMedian() {
        return lo.size() > hi.size()
            ? lo.top()
            : (lo.top() + hi.top()) / 2.0;
    }

private:
    priority_queue<int> lo;                             // max-heap — lower half
    priority_queue<int, vector<int>, greater<int>> hi;  // min-heap — upper half
};
```

### Why the Cross-Push Pattern Works

Step 1 — `lo.push(num)` then `hi.push(lo.top()); lo.pop()`:
- Guarantees `hi.top() >= lo.top()` — ordering invariant maintained
- Even if `num` is very small, it enters `lo`, and `lo`'s max moves to `hi`

Step 2 — `if (lo.size() < hi.size()) lo.push(hi.top()); hi.pop()`:
- Rebalances size — ensures `lo` is always >= `hi` in size
- Median is always accessible at `lo.top()` for odd counts

### Comparison

| | Sorted list | Two heaps |
|---|---|---|
| `addNum` | O(N) — linear scan + insert | O(log N) |
| `findMedian` | O(1) | O(1) |
| Iterator management | 5 cases — complex | None |
| Correctness verification | Hard | Easy — two invariants |

### Key Takeaways

- Two heaps split the problem into "lower half" and "upper half" — median is always at the boundary between them.
- The cross-push pattern (`lo→hi, rebalance lo`) maintains both invariants simultaneously: ordering and size balance.
- `priority_queue<int>` is max-heap by default. Min-heap requires `priority_queue<int, vector<int>, greater<int>>`.
- O(log N) per insertion is the correct complexity for this problem — any O(N) approach (sorted list, linear scan) will TLE on large streams.

---

## Concepts: Static, Constexpr, and Binary Layout

### Binary Section Summary

```
.text   — executable instructions
.rodata — read-only data (constexpr, string literals)
.data   — initialized mutable globals and statics
.bss    — zero-initialized mutable globals and statics
```

`constexpr` values live in `.rodata` — write-protected at OS level. Modification causes hardware segfault. `static` non-const values live in `.data` — writable, runtime initialized with a hidden thread-safe guard check on every call in C++11+.

### Static Local Variable Guard — Hidden Cost

```cpp
static int x = compute();  // C++11: compiler emits mutex-like guard
                            // checked on every call even after init
static constexpr int x = 42; // no guard — value known at compile time
                              // zero runtime overhead
```

In hot-path trading code executing millions of times per second, eliminating the guard check via `constexpr` is standard discipline.

---

*Logged from Claude study session · April 6, 2026*
