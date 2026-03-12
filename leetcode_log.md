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
