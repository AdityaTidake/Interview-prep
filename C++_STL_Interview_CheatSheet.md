# C++ STL + Interview Essentials Cheatsheet

> **Purpose:** Fast revision for DSA interviews.  
> Focus: containers, iterators, algorithms, strings, templates, lambdas, comparators, custom sorting, heaps, and the STL patterns used most often in coding rounds.

---

# 0. STL Mental Model

The C++ Standard Template Library can be thought of as:

| Part | What it gives you |
|---|---|
| **Containers** | Data structures to store data |
| **Iterators** | Objects used to traverse ranges |
| **Algorithms** | Reusable operations such as `sort`, `find`, `reverse` |
| **Function objects / Lambdas** | Custom behavior for algorithms |
| **Utilities** | `pair`, `tuple`, `swap`, etc. |

### Interview rule

Before coding, ask:

1. **Do I need ordering?** → `set` / `map`
2. **Do I only need fast average lookup?** → `unordered_set` / `unordered_map`
3. **Do I need index access?** → `vector`
4. **Do I need both-end insertion/removal?** → `deque`
5. **LIFO?** → `stack`
6. **FIFO?** → `queue`
7. **Repeatedly need smallest/largest?** → `priority_queue`
8. **Need sorting/searching on a range?** → STL algorithms

<img width="172" height="42" alt="image" src="https://github.com/user-attachments/assets/de948b0a-8883-46a9-956e-3a0815c79a4f" />

<img width="1172" height="613" alt="image" src="https://github.com/user-attachments/assets/75776796-41ef-488f-bbc0-95113e3e3ab4" />

<img width="783" height="495" alt="image" src="https://github.com/user-attachments/assets/40d6b678-7261-4d8e-939b-c28259608054" />

<img width="694" height="717" alt="image" src="https://github.com/user-attachments/assets/b3de5fed-aa4c-47fb-8048-c4f7216955b1" />

---

# 1. Complexity Cheat Table

| Container | Access | Search | Insert | Erase | Ordered? |
|---|---:|---:|---:|---:|---|
| `vector` | `O(1)` | `O(n)` | End: amortized `O(1)`, middle: `O(n)` | End: `O(1)`, middle: `O(n)` | No |
| `deque` | `O(1)` | `O(n)` | Front/back: `O(1)` | Front/back: `O(1)` | No |
| `list` | `O(n)` | `O(n)` | Given iterator: `O(1)` | Given iterator: `O(1)` | No |
| `forward_list` | `O(n)` | `O(n)` | Given position: `O(1)` | Given position: `O(1)` | No |
| `set` | No index | `O(log n)` | `O(log n)` | `O(log n)` | Yes |
| `multiset` | No index | `O(log n)` | `O(log n)` | `O(log n)` | Yes, duplicates |
| `map` | By key `O(log n)` | `O(log n)` | `O(log n)` | `O(log n)` | Yes |
| `multimap` | By key `O(log n)` | `O(log n)` | `O(log n)` | `O(log n)` | Yes, duplicate keys |
| `unordered_set` | No index | Avg `O(1)` | Avg `O(1)` | Avg `O(1)` | No |
| `unordered_map` | By key Avg `O(1)` | Avg `O(1)` | Avg `O(1)` | Avg `O(1)` | No |
| `stack` | Top only `O(1)` | — | `O(1)` | `O(1)` | No |
| `queue` | Front/back `O(1)` | — | `O(1)` | `O(1)` | No |
| `priority_queue` | Top `O(1)` | — | `O(log n)` | `O(log n)` | Heap order only |

> **Important:** Hash-table operations are **average-case `O(1)`**, not guaranteed `O(1)`. Worst case can be `O(n)`.
>
> 




---

# 2. `pair`

Useful for storing two related values.

```cpp
pair<int, int> p = {10, 20};

cout << p.first;
cout << p.second;
```

### Common uses

```cpp
vector<pair<int,int>> v;

map<int, int> mp;

priority_queue<pair<int,int>> pq;
```

### Nested pair

```cpp
pair<int, pair<int,int>> p = {1, {2, 3}};

cout << p.first;          // 1
cout << p.second.first;   // 2
cout << p.second.second;  // 3
```

### `make_pair`

```cpp
auto p = make_pair(10, 20);
```

### Default pair comparison

Pairs are compared **lexicographically**:

```text
first
  ↓
if equal
  ↓
second
```

Example:

```cpp
{2, 8} < {10, 1}   // true
{10, 2} < {10, 5}  // true
```

---

# 3. `vector`

A dynamic array with contiguous storage.

```cpp
vector<int> v;
vector<int> v = {1, 2, 3};
vector<int> v(5, 10);     // {10,10,10,10,10}
vector<int> v2(v);        // copy
```

## Most important functions

| Function | Description | Complexity | Example |
|---|---|---:|---|
| `push_back(x)` | Add element to end | Amortized `O(1)` | `v.push_back(10);` |
| `pop_back()` | Remove last element | `O(1)` | `v.pop_back();` |
| `insert(it, x)` | Insert at position | `O(n)` | `v.insert(v.begin()+1, 5);` |
| `erase(it)` | Remove at position | `O(n)` | `v.erase(v.begin()+1);` |
| `front()` | First element | `O(1)` | `v.front();` |
| `back()` | Last element | `O(1)` | `v.back();` |
| `at(i)` / `v[i]` | Access element | `O(1)` | `v.at(2);` / `v[2];` |
| `size()` | Number of elements | `O(1)` | `v.size();` |
| `empty()` | Check if empty | `O(1)` | `v.empty();` |
| `clear()` | Remove all elements | `O(n)` | `v.clear();` |
| `reserve(n)` | Increase capacity | Up to `O(n)` | `v.reserve(100);` |
| `resize(n)` | Change number of elements | Up to `O(n)` | `v.resize(10);` |

### `size()` vs `capacity()`

```cpp
v.size();       // number of actual elements
v.capacity();   // allocated storage capacity
```

When capacity is insufficient, vector allocates a larger block and moves/copies elements.

### `reserve()`

```cpp
v.reserve(1000);
```

Pre-allocates capacity.

Useful when you know approximately how many elements will be inserted.

### `resize()`

```cpp
v.resize(10);
```

Changes the number of elements.

> `reserve()` changes **capacity**, while `resize()` changes **size**.

### Iteration

```cpp
for(int x : v)
    cout << x << " ";
```

Iterator:

```cpp
for(auto it = v.begin(); it != v.end(); ++it)
    cout << *it;
```

---

# 4. `deque`

Double-ended queue.

```cpp
deque<int> dq = {1, 2, 3};
```

### Most important functions

| Function | Description | Complexity | Example |
|---|---|---:|---|
| `push_front(x)` | Add to front | `O(1)` | `dq.push_front(10);` |
| `push_back(x)` | Add to back | `O(1)` | `dq.push_back(20);` |
| `pop_front()` | Remove first | `O(1)` | `dq.pop_front();` |
| `pop_back()` | Remove last | `O(1)` | `dq.pop_back();` |
| `front()` | First element | `O(1)` | `dq.front();` |
| `back()` | Last element | `O(1)` | `dq.back();` |
| `operator[i]` / `at(i)` | Index access | `O(1)` | `dq[2];` / `dq.at(2);` |
| `insert(it, x)` | Insert at position | `O(n)` | `dq.insert(dq.begin()+1, 5);` |
| `erase(it)` | Remove at position | `O(n)` | `dq.erase(dq.begin()+1);` |
| `size()` | Number of elements | `O(1)` | `dq.size();` |
| `empty()` | Check empty | `O(1)` | `dq.empty();` |

### Interview use

Use `deque` when you need:

```text
push_front + push_back
pop_front  + pop_back
```

without requiring contiguous storage.

---

# 5. `list`

Doubly linked list.

```cpp
list<int> l = {1, 2, 3};
```

### Most important functions

| Function | Description | Complexity | Example |
|---|---|---:|---|
| `push_front(x)` | Add at front | `O(1)` | `l.push_front(10);` |
| `push_back(x)` | Add at back | `O(1)` | `l.push_back(20);` |
| `pop_front()` | Remove first | `O(1)` | `l.pop_front();` |
| `pop_back()` | Remove last | `O(1)` | `l.pop_back();` |
| `insert(it, x)` | Insert before iterator | `O(1)` | `l.insert(it, 5);` |
| `erase(it)` | Erase at iterator | `O(1)` | `l.erase(it);` |
| `remove(x)` | Remove matching values | `O(n)` | `l.remove(5);` |
| `sort()` | Sort list | `O(n log n)` | `l.sort();` |
| `reverse()` | Reverse list | `O(n)` | `l.reverse();` |
| `unique()` | Remove consecutive duplicates | `O(n)` | `l.unique();` |
| `size()` | Number of elements | `O(1)` | `l.size();` |

### Important

`list` does **not** support:

```cpp
l[3];        // invalid
```

To reach an index, you must traverse.

```cpp
auto it = next(l.begin(), 3);
```

### Special operations

```cpp
l.sort();
l.reverse();
l.unique();
l.remove(value);
l.merge(l2);
l.splice(pos, l2);
```

### Interview insight

A linked list's insertion/erase is `O(1)` **once you already have the correct iterator**.

Finding the position may still cost `O(n)`.

---

# 6. `forward_list`

Singly linked list.

```cpp
forward_list<int> fl = {1, 2, 3};
```

Only forward traversal.

Useful when you specifically want singly-linked-list semantics and lower node overhead.

Common operations:

```cpp
fl.push_front(10);
fl.pop_front();

fl.insert_after(it, 20);
fl.erase_after(it);
```

---

# 7. `stack`

LIFO — Last In, First Out.

```cpp
stack<int> st;
```

### Most important functions

| Function | Description | Complexity | Example |
|---|---|---:|---|
| `push(x)` | Add to top | `O(1)` | `st.push(10);` |
| `pop()` | Remove top | `O(1)` | `st.pop();` |
| `top()` | Read top element | `O(1)` | `st.top();` |
| `empty()` | Check empty | `O(1)` | `st.empty();` |
| `size()` | Number of elements | `O(1)` | `st.size();` |

All core operations are `O(1)`.


### Important trap

`pop()` does **not** return the removed element.

Wrong:

```cpp
int x = st.pop();
```

Correct:

```cpp
int x = st.top();
st.pop();
```

### Common interview uses

- Balanced parentheses
- Monotonic stack
- Next greater element
- Expression evaluation
- Undo-like behavior
- DFS implementation

---

# 8. `queue`

FIFO — First In, First Out.

```cpp
queue<int> q;
```

### Most important functions

| Function | Description | Complexity | Example |
|---|---|---:|---|
| `push(x)` | Add to back | `O(1)` | `q.push(10);` |
| `pop()` | Remove front | `O(1)` | `q.pop();` |
| `front()` | Read front element | `O(1)` | `q.front();` |
| `back()` | Read back element | `O(1)` | `q.back();` |
| `empty()` | Check empty | `O(1)` | `q.empty();` |
| `size()` | Number of elements | `O(1)` | `q.size();` |

All core operations are `O(1)`.

### Common interview uses

- BFS
- Level-order traversal
- Scheduling
- Sliding-window structures

---

# 9. `priority_queue`

A heap-based container adaptor.

## Max heap — default

```cpp
priority_queue<int> pq;

pq.push(10);
pq.push(30);
pq.push(20);

cout << pq.top();    // 30
```

## Min heap

```cpp
priority_queue<int, vector<int>, greater<int>> pq;
```

Now:

```text
top() = smallest element
```

### Most important functions

| Function | Description | Complexity | Example |
|---|---|---:|---|
| `push(x)` | Insert element | `O(log n)` | `pq.push(10);` |
| `pop()` | Remove top | `O(log n)` | `pq.pop();` |
| `top()` | Read top | `O(1)` | `pq.top();` |
| `empty()` | Check empty | `O(1)` | `pq.empty();` |
| `size()` | Number of elements | `O(1)` | `pq.size();` |

### Common interview uses

- Top K elements
- Kth largest/smallest
- Merge K sorted lists
- Scheduling
- Dijkstra
- Median of data stream
- Greedy problems

---

# 10. Priority Queue with `pair`

### Max heap

```cpp
priority_queue<pair<int,int>> pq;
```

Largest pair is on top using lexicographic comparison.

### Min heap

```cpp
priority_queue<
    pair<int,int>,
    vector<pair<int,int>>,
    greater<pair<int,int>>
> pq;
```

---

# 11. `set`

Stores **unique sorted values**.

```cpp
set<int> s;

s.insert(5);
s.insert(2);
s.insert(5);
```

Contents:

```text
2 5
```

### Complexity

```text
insert  O(log n)
find    O(log n)
erase   O(log n)
count   O(log n)
```

### Most important functions

| Function | Description | Complexity | Example |
|---|---|---:|---|
| `insert(x)` | Insert unique value | `O(log n)` | `s.insert(10);` |
| `erase(x)` | Remove value | `O(log n)` | `s.erase(10);` |
| `find(x)` | Find value | `O(log n)` | `s.find(10);` |
| `count(x)` | Check existence | `O(log n)` | `s.count(10);` |
| `lower_bound(x)` | First element `>= x` | `O(log n)` | `s.lower_bound(x);` |
| `upper_bound(x)` | First element `> x` | `O(log n)` | `s.upper_bound(x);` |
| `size()` | Number of elements | `O(1)` | `s.size();` |
| `empty()` | Check empty | `O(1)` | `s.empty();` |

These are extremely important in interviews.

---

# 12. `multiset`

Sorted container that allows duplicates.

```cpp
multiset<int> ms = {1, 2, 2, 3};
```

### Most important functions

| Function | Description | Complexity | Example |
|---|---|---:|---|
| `insert(x)` | Insert value, duplicates allowed | `O(log n)` | `ms.insert(2);` |
| `find(x)` | Find an occurrence | `O(log n)` | `ms.find(2);` |
| `count(x)` | Count occurrences | `O(log n + k)` | `ms.count(2);` |
| `erase(x)` | Remove all occurrences | `O(log n + k)` | `ms.erase(2);` |
| `erase(it)` | Remove one occurrence | `O(log n)` | `ms.erase(it);` |
| `lower_bound(x)` | First `>= x` | `O(log n)` | `ms.lower_bound(x);` |
| `upper_bound(x)` | First `> x` | `O(log n)` | `ms.upper_bound(x);` |

### Important distinction

```cpp
ms.erase(2);
```

removes **all occurrences** of `2`.

To remove only one:

```cpp
auto it = ms.find(2);
if(it != ms.end())
    ms.erase(it);
```

### Complexity

Core operations: `O(log n)`.

---

# 13. `map`

Stores key-value pairs sorted by key.

```cpp
map<string, int> mp;

mp["Alice"] = 10;
mp["Bob"] = 20;
```

Keys are unique.

### Complexity

```text
insert      O(log n)
find        O(log n)
erase       O(log n)
operator[]  O(log n)
```

### Access

```cpp
cout << mp["Alice"];
```

### Important trap

```cpp
mp["missing"]
```

creates the key if it does not exist, with a default value.

If you only want to check existence:

```cpp
if(mp.find("missing") != mp.end())
```

or:

```cpp
if(mp.count("missing"))
```

### Iterate

```cpp
for(auto [key, value] : mp)
    cout << key << " " << value;
```

Keys appear in sorted order.

---

# 14. `unordered_map`

Hash table.

```cpp
unordered_map<string, int> mp;
```

Average:

```text
insert  O(1)
find    O(1)
erase   O(1)
```

Worst case can be `O(n)`.

### Use when

You need fast lookup and **do not need sorted keys**.

### Classic interview use

Frequency counting:

```cpp
unordered_map<int,int> freq;

for(int x : nums)
    freq[x]++;
```

---

# 15. `set` vs `unordered_set`

| Feature | `set` | `unordered_set` |
|---|---|---|
| Duplicate values | No | No |
| Ordering | Sorted | No |
| Average lookup | `O(log n)` | `O(1)` |
| Worst-case lookup | `O(log n)` | `O(n)` |
| Lower/upper bound | Yes | No |

### Decision

Need order → `set`

Need only existence → usually `unordered_set`

---

# 16. `map` vs `unordered_map`

| Feature | `map` | `unordered_map` |
|---|---|---|
| Ordering | Sorted by key | No |
| Lookup | `O(log n)` | Avg `O(1)` |
| Lower/upper bound | Yes | No |
| Worst-case lookup | `O(log n)` | `O(n)` |
| Typical use | Ordered data | Frequency/counting |

---

# 17. Iterators

An iterator behaves like a generalized pointer used to traverse a range.

For vector:

```cpp
vector<int>::iterator it;
```

For list:

```cpp
list<int>::iterator it;
```

### Basic pattern

```cpp
for(auto it = v.begin(); it != v.end(); ++it)
    cout << *it;
```

### `begin()` vs `end()`

```text
begin()
   ↓
[10][20][30]
          ↑
         end()
```

`end()` points **one position past the last element**.

Never dereference:

```cpp
*v.end();    // invalid
```

### Reverse iterators

```cpp
v.rbegin();
v.rend();
```

Example:

```cpp
for(auto it = v.rbegin(); it != v.rend(); ++it)
    cout << *it;
```

### `next` and `prev`

```cpp
auto it2 = next(it, 3);
auto it3 = prev(it, 2);
```

These are especially useful for non-random-access containers.

---

# 18. Iterator Categories — Interview Awareness

| Iterator | Can move | Random access? |
|---|---|---|
| Input | Forward | No |
| Output | Forward | No |
| Forward | Forward | No |
| Bidirectional | Both directions | No |
| Random access | Both + jumps | Yes |
| Contiguous | Random + contiguous storage | Yes |

Important examples:

```text
vector → random access / contiguous
deque → random access
list → bidirectional
forward_list → forward
```

This explains why:

```cpp
v.begin() + 5
```

works for vector, while arbitrary iterator arithmetic is not supported for `list`.

---

# 19. STL Algorithms

## STL Algorithm Quick-Reference

| Function | Description | Typical Complexity | Example |
|---|---|---:|---|
| `sort()` | Sort range | `O(n log n)` | `sort(v.begin(),v.end());` |
| `stable_sort()` | Stable sort | `O(n log n)` | `stable_sort(v.begin(),v.end());` |
| `reverse()` | Reverse range | `O(n)` | `reverse(v.begin(),v.end());` |
| `rotate()` | Rotate range | `O(n)` | `rotate(v.begin(),mid,v.end());` |
| `find()` | Find value | `O(n)` | `find(v.begin(),v.end(),x);` |
| `find_if()` | Find first matching element | `O(n)` | `find_if(v.begin(),v.end(),pred);` |
| `count()` | Count value | `O(n)` | `count(v.begin(),v.end(),x);` |
| `count_if()` | Count matching elements | `O(n)` | `count_if(v.begin(),v.end(),pred);` |
| `max_element()` | Find maximum | `O(n)` | `max_element(v.begin(),v.end());` |
| `min_element()` | Find minimum | `O(n)` | `min_element(v.begin(),v.end());` |
| `lower_bound()` | First `>= x` | `O(log n)`* | `lower_bound(v.begin(),v.end(),x);` |
| `upper_bound()` | First `> x` | `O(log n)`* | `upper_bound(v.begin(),v.end(),x);` |
| `binary_search()` | Check existence | `O(log n)`* | `binary_search(v.begin(),v.end(),x);` |
| `unique()` | Move consecutive duplicates | `O(n)` | `unique(v.begin(),v.end());` |
| `accumulate()` | Sum/reduce range | `O(n)` | `accumulate(v.begin(),v.end(),0LL);` |
| `iota()` | Fill consecutive values | `O(n)` | `iota(v.begin(),v.end(),1);` |
| `next_permutation()` | Next lexicographical permutation | `O(n)` | `next_permutation(v.begin(),v.end());` |
| `all_of()` | All satisfy predicate? | `O(n)` | `all_of(v.begin(),v.end(),pred);` |
| `any_of()` | Any satisfies predicate? | `O(n)` | `any_of(v.begin(),v.end(),pred);` |
| `none_of()` | None satisfy predicate? | `O(n)` | `none_of(v.begin(),v.end(),pred);` |
| `remove()` | Move matching values to end | `O(n)` | `remove(v.begin(),v.end(),x);` |
| `remove_if()` | Move matching elements to end | `O(n)` | `remove_if(v.begin(),v.end(),pred);` |

> \* `lower_bound`, `upper_bound`, and `binary_search` are logarithmic with random-access iterators and sorted data.


Most are in:

```cpp
#include <algorithm>
```

## `sort`

```cpp
sort(v.begin(), v.end());
```

Complexity:

```text
O(n log n)
```

Descending:

```cpp
sort(v.begin(), v.end(), greater<int>());
```

---

## `reverse`

```cpp
reverse(v.begin(), v.end());
```

`O(n)`.

---

## `find`

```cpp
auto it = find(v.begin(), v.end(), x);
```

Returns iterator to first occurrence or `end()`.

Complexity: `O(n)` for a normal vector.

---

## `count`

```cpp
int c = count(v.begin(), v.end(), x);
```

`O(n)`.

---

## `max_element`

```cpp
auto it = max_element(v.begin(), v.end());

cout << *it;
```

`O(n)`.

Similarly:

```cpp
min_element(...)
```

---

# 20. `lower_bound` and `upper_bound`

These are **must-know interview tools**.

They require a sorted range for their standard logarithmic behavior.

### `lower_bound(x)`

First position where:

```text
value >= x
```

### `upper_bound(x)`

First position where:

```text
value > x
```

Example:

```cpp
vector<int> v = {1, 2, 2, 2, 5, 7};

auto lb = lower_bound(v.begin(), v.end(), 2);
auto ub = upper_bound(v.begin(), v.end(), 2);
```

Visual:

```text
index:  0 1 2 3 4 5
value:  1 2 2 2 5 7
          ↑       ↑
         lb       ub
```

Therefore:

```cpp
lb - v.begin()   // 1
ub - v.begin()   // 4
```

### Count occurrences

For sorted data:

```cpp
int count = upper_bound(v.begin(), v.end(), x)
          - lower_bound(v.begin(), v.end(), x);
```

---

# 21. `binary_search`

```cpp
binary_search(v.begin(), v.end(), x);
```

Returns:

```text
true / false
```

Requires sorted range.

Complexity: `O(log n)` for random-access iterators such as vector.

---

# 22. `unique`

Important: `unique()` does **not actually resize the container**.

```cpp
auto it = unique(v.begin(), v.end());
v.erase(it, v.end());
```

Typical duplicate-removal pattern for a sorted vector:

```cpp
sort(v.begin(), v.end());

v.erase(unique(v.begin(), v.end()), v.end());
```

---

# 23. `accumulate`

Include:

```cpp
#include <numeric>
```

```cpp
int sum = accumulate(v.begin(), v.end(), 0);
```

For `long long`:

```cpp
long long sum =
    accumulate(v.begin(), v.end(), 0LL);
```

> **Important:** The type of the initial value affects the accumulation type.

---

# 24. `iota`

Include:

```cpp
#include <numeric>
```

```cpp
vector<int> v(5);

iota(v.begin(), v.end(), 1);
```

Result:

```text
1 2 3 4 5
```

Useful for initialization and index arrays.

---

# 25. `swap`

```cpp
swap(a, b);
```

Also:

```cpp
std::swap(a, b);
```

---

# 26. `next_permutation`

Generates the next lexicographically greater permutation.

```cpp
vector<int> v = {1, 2, 3};

next_permutation(v.begin(), v.end());
```

Result:

```text
1 3 2
```

Common interview pattern:

```cpp
sort(v.begin(), v.end());

do {
    // process permutation
} while(next_permutation(v.begin(), v.end()));
```

---

# 27. `min` / `max`

```cpp
min(a, b);
max(a, b);
```

Multiple values:

```cpp
int x = min({a, b, c});
int y = max({a, b, c});
```

---

# 28. Useful Numeric Limits

```cpp
#include <climits>

INT_MAX
INT_MIN
LLONG_MAX
LLONG_MIN
```

Prefer:

```cpp
#include <limits>

numeric_limits<int>::max();
numeric_limits<long long>::max();
```

---

# 29. String — `std::string`

`std::string` is the standard C++ string class and is heavily used in DSA.

```cpp
string s;
string s1("Hello");
string s2 = "Hello";
string s3(s1);
string s4(s1, 1, 2);       // "el"
string s5("Hello World", 5);
string s6(5, '*');         // "*****"
string s7(s1.begin(), s1.begin() + 3);
```

---

# 30. String Functions

| Function | Description | Typical Complexity | Example |
|---|---|---:|---|
| `size()` / `length()` | Number of characters | `O(1)` | `s.size();` |
| `empty()` | Check empty | `O(1)` | `s.empty();` |
| `operator[i]` / `at(i)` | Access character | `O(1)` | `s[2];` / `s.at(2);` |
| `front()` / `back()` | First / last character | `O(1)` | `s.front();` |
| `begin()` / `end()` | Character iterators | `O(1)` | `s.begin();` |
| `append()` / `+=` | Append characters | Depends on size/reallocation | `s += "abc";` |
| `insert()` | Insert characters | `O(n)` | `s.insert(2,"abc");` |
| `erase()` | Remove characters | `O(n)` | `s.erase(2,3);` |
| `replace()` | Replace substring | `O(n)` | `s.replace(2,3,"xyz");` |
| `substr(pos,len)` | Copy substring | `O(len)` | `s.substr(2,5);` |
| `find()` | Find substring/character | Depends on implementation | `s.find("abc");` |
| `compare()` | Lexicographic comparison | Up to `O(n+m)` | `s.compare(t);` |
| `clear()` | Remove contents | `O(n)` in general | `s.clear();` |
| `resize(n)` | Change string length | Up to `O(n)` | `s.resize(10);` |
| `c_str()` | Get null-terminated C string | `O(1)` | `s.c_str();` |

### Examples

```cpp
s += "world";
s.append("!");

char c = s[2];
char d = s.at(2);

s.insert(2, "ABC");
s.erase(2, 3);

string sub = s.substr(2, 5);

size_t pos = s.find("abc");
```

### `find()` check

```cpp
if(s.find("abc") != string::npos)
    cout << "Found";
```

> `string::npos` means "not found".

---

# 31. String Input

For a single word:

```cpp
cin >> s;
```

For a complete line:

```cpp
getline(cin, s);
```

### Common trap

After:

```cpp
cin >> n;
```

the newline may remain in the input buffer.

Often:

```cpp
cin.ignore();
getline(cin, s);
```

is needed.

---

# 32. String Conversions

Number → string:

```cpp
string s = to_string(123);
```

String → int:

```cpp
int x = stoi("123");
```

String → long long:

```cpp
long long x = stoll("123456");
```

Other useful conversions:

```cpp
stol()
stoul()
stoull()
stof()
stod()
stold()
```

---

# 33. C-Style String

```cpp
const char* p = s.c_str();
```

Useful when interacting with APIs expecting a null-terminated C string.

Do not modify the returned pointer's characters.

---

# 34. Templates

Templates allow functions/classes to work with generic types.

## Generic function

```cpp
template <class X>
void swapValues(X &a, X &b) {
    X temp = a;
    a = b;
    b = temp;
}
```

`typename` can be used instead of `class`:

```cpp
template <typename X>
```

### Mental model

A normal parameter passes a **value**:

```cpp
void f(int x);
```

A template parameter passes a **type**:

```cpp
template <typename T>
void f(T x);
```

The compiler generates the appropriate version based on how the template is used.

### Example

```cpp
template <class X>
void swapValues(X &a, X &b) {
    X temp = a;
    a = b;
    b = temp;
}

int a = 10, b = 20;
float c = 10.5, d = 20.5;

swapValues(a, b);
swapValues(c, d);
```

### STL connection

Many STL components are templates:

```cpp
vector<int>
vector<string>

pair<int,int>

map<string,int>
```

The same generic structure can work with different types.

---

# 35. Iterator

An iterator is an object that points to an element in a range and supports traversal using operations appropriate to its iterator category.

A raw pointer can act as an iterator for arrays.

Examples:

```cpp
vector<int>::iterator it;
list<int>::iterator it;
```

### Basic usage

```cpp
for(auto it = v.begin(); it != v.end(); ++it)
    cout << *it;
```

### Key operations

```text
*it       → value
++it      → next
--it      → previous, if supported
it + k    → jump, only for random-access iterators
```

---

# 36. Lambda Functions

A lambda is an anonymous function.

Basic syntax:

```cpp
[](int a, int b) {
    return a < b;
}
```

General form:

```cpp
[capture](parameters) -> return_type {
    // body
}
```

Return type can usually be inferred.

### Example

```cpp
auto add = [](int a, int b) {
    return a + b;
};

cout << add(2, 3);
```

---

# 37. Lambda Capture

### Capture by value

```cpp
int x = 10;

auto f = [x]() {
    cout << x;
};
```

Lambda receives its own captured copy.

### Capture by reference

```cpp
int x = 10;

auto f = [&x]() {
    x++;
};
```

Original `x` changes.

### Capture everything

```cpp
[=]    // capture used variables by value
[&]    // capture used variables by reference
```

### Interview caution

Capture only what you need when practical:

```cpp
[&x]
```

is clearer than unnecessarily capturing everything.

---

# 38. Comparator Writing

This is one of the most important STL interview skills.

Suppose:

```cpp
vector<int> v = {5, 1, 4, 2};
```

Ascending:

```cpp
sort(v.begin(), v.end());
```

Descending:

```cpp
sort(v.begin(), v.end(),
     [](int a, int b) {
         return a > b;
     });
```

## The meaning of comparator

```cpp
return true;
```

means:

> `a` should appear before `b`.

Think:

```text
Should a come before b?
```

not:

```text
Should I swap a and b?
```

---

# 39. Comparator Examples

## Sort by absolute value

```cpp
sort(v.begin(), v.end(),
     [](int a, int b) {
         return abs(a) < abs(b);
     });
```

## Vector of pairs — sort by second

```cpp
sort(v.begin(), v.end(),
     [](pair<int,int> a, pair<int,int> b) {
         return a.second < b.second;
     });
```

## Tie-breaking

Sort by first ascending; if equal, second descending:

```cpp
sort(v.begin(), v.end(),
     [](auto &a, auto &b) {
         if(a.first != b.first)
             return a.first < b.first;

         return a.second > b.second;
     });
```

This pattern appears constantly in interviews.

---

# 40. Comparator for Structs

```cpp
struct Student {
    int roll;
    int marks;
};

sort(v.begin(), v.end(),
     [](const Student &a, const Student &b) {
         return a.marks > b.marks;
     });
```

For multiple criteria:

```cpp
sort(v.begin(), v.end(),
     [](const Student &a, const Student &b) {

         if(a.marks != b.marks)
             return a.marks > b.marks;

         return a.roll < b.roll;
     });
```

---

# 41. Custom Priority Queue Comparator

```cpp
struct CompareMarks {
    bool operator()(const Student &a,
                    const Student &b) const {
        return a.marks < b.marks;
    }
};

priority_queue<Student,
               vector<Student>,
               CompareMarks> pq;
```

For a priority queue, the comparator's meaning can feel reversed compared with `sort`.

### Safe mental model

For `sort`:

```text
Should a come before b?
```

For `priority_queue`:

```text
Which element should have higher priority?
```

Test with a tiny example rather than relying on memorization.

---

# 42. Sorting `vector<pair<...>>`

Default pair sorting is:

```text
first ascending
then second ascending
```

Example:

```cpp
vector<pair<int,int>> v = {
    {10,5},
    {2,8},
    {10,2}
};

sort(v.begin(), v.end());
```

Result:

```text
{2,8}
{10,2}
{10,5}
```

Custom:

```cpp
sort(v.begin(), v.end(),
     [](auto &a, auto &b) {
         return a.second < b.second;
     });
```

---

# 43. Custom Sorting — Interview Template

### One criterion

```cpp
sort(v.begin(), v.end(),
     [](const auto &a, const auto &b) {
         return a.x < b.x;
     });
```

### Ascending + tie descending

```cpp
sort(v.begin(), v.end(),
     [](const auto &a, const auto &b) {

         if(a.x != b.x)
             return a.x < b.x;

         return a.y > b.y;
     });
```

### String length

```cpp
sort(v.begin(), v.end(),
     [](const string &a, const string &b) {
         return a.size() < b.size();
     });
```

---

# 44. `stable_sort`

```cpp
stable_sort(v.begin(), v.end(), comp);
```

Unlike ordinary sorting, it preserves the relative order of equivalent elements.

Use when relative ordering among equal keys matters.

---

# 45. STL Algorithms Worth Memorizing

```cpp
sort()
stable_sort()

reverse()
rotate()

find()
find_if()

count()
count_if()

max_element()
min_element()

lower_bound()
upper_bound()
binary_search()

accumulate()
iota()

unique()
next_permutation()
prev_permutation()

all_of()
any_of()
none_of()

for_each()
```

---

# 46. Predicate Algorithms

## `all_of`

```cpp
bool ok = all_of(v.begin(), v.end(),
                 [](int x) {
                     return x > 0;
                 });
```

Are all elements positive?

## `any_of`

```cpp
bool ok = any_of(v.begin(), v.end(),
                 [](int x) {
                     return x < 0;
                 });
```

Does at least one element satisfy the condition?

## `none_of`

Checks whether no element satisfies the condition.

These are useful for readable interview code.

---

# 47. `find_if`

Find first element satisfying a condition.

```cpp
auto it = find_if(v.begin(), v.end(),
                  [](int x) {
                      return x % 2 == 0;
                  });
```

---

# 48. Erase While Iterating

A common interview mistake:

```cpp
for(auto it = v.begin(); it != v.end(); ++it) {
    if(condition)
        v.erase(it);   // dangerous
}
```

For vector, `erase()` invalidates iterators at and after the erased position.

Safer pattern:

```cpp
for(auto it = v.begin(); it != v.end(); ) {

    if(condition)
        it = v.erase(it);

    else
        ++it;
}
```

For list:

```cpp
for(auto it = l.begin(); it != l.end(); ) {

    if(condition)
        it = l.erase(it);

    else
        ++it;
}
```

---

# 49. `erase-remove` Idiom

Remove all occurrences of a value from a vector:

```cpp
v.erase(
    remove(v.begin(), v.end(), x),
    v.end()
);
```

For a condition:

```cpp
v.erase(
    remove_if(v.begin(), v.end(),
        [](int x) {
            return x % 2 == 0;
        }),
    v.end()
);
```

This is a very common STL interview pattern.

---

# 50. Frequency Counting

## `unordered_map`

```cpp
unordered_map<int,int> freq;

for(int x : v)
    freq[x]++;
```

## Character frequency

```cpp
unordered_map<char,int> freq;

for(char c : s)
    freq[c]++;
```

If the alphabet is small/fixed, a frequency array is often simpler:

```cpp
vector<int> freq(26, 0);

for(char c : s)
    freq[c - 'a']++;
```

---

# 51. Fast Frequency Array Pattern

Lowercase English letters:

```cpp
int freq[26] = {};

for(char c : s)
    freq[c - 'a']++;
```

Digits:

```cpp
int freq[10] = {};

for(char c : s)
    freq[c - '0']++;
```

ASCII-sized:

```cpp
int freq[256] = {};
```

---

# 52. `map` with Frequency

Useful when sorted keys are required:

```cpp
map<int,int> freq;

for(int x : v)
    freq[x]++;
```

Then iteration is automatically sorted by key.

---

# 53. Common STL Interview Patterns

## Pattern 1 — Existence

```cpp
if(s.count(x))
```

or:

```cpp
if(mp.find(x) != mp.end())
```

---

## Pattern 2 — Frequency

```cpp
unordered_map<int,int> freq;

for(int x : nums)
    freq[x]++;
```

---

## Pattern 3 — Sorted unique values

```cpp
set<int> s(nums.begin(), nums.end());
```

---

## Pattern 4 — Sort then remove duplicates

```cpp
sort(v.begin(), v.end());

v.erase(unique(v.begin(), v.end()), v.end());
```

---

## Pattern 5 — Binary-search position

```cpp
int pos =
    lower_bound(v.begin(), v.end(), x)
    - v.begin();
```

---

## Pattern 6 — Count occurrences in sorted array

```cpp
int count =
    upper_bound(v.begin(), v.end(), x)
    - lower_bound(v.begin(), v.end(), x);
```

---

## Pattern 7 — Top K

```cpp
priority_queue<int> pq;
```

or maintain a min-heap of size `k`:

```cpp
priority_queue<int,
               vector<int>,
               greater<int>> pq;
```

---

# 54. STL Header Cheat Sheet

For competitive programming/interviews:

```cpp
#include <bits/stdc++.h>
using namespace std;
```

Common individual headers:

```cpp
#include <vector>
#include <deque>
#include <list>
#include <stack>
#include <queue>
#include <set>
#include <map>
#include <unordered_set>
#include <unordered_map>
#include <algorithm>
#include <numeric>
#include <string>
#include <climits>
#include <limits>
```

---

# 55. Common Interview Traps

### Trap 1 — `pop()` returns nothing

```cpp
st.pop();       // removes
st.top();       // reads
```

---

### Trap 2 — `end()` is not last element

```cpp
end()
```

is one past the last element.

---

### Trap 3 — `mp[key]` may insert

```cpp
mp[key]
```

can create the key.

Use `find()` when only checking existence.

---

### Trap 4 — `unordered_map` is not always O(1)

Average: `O(1)`  
Worst: `O(n)`

---

### Trap 5 — `unique()` does not resize

Use:

```cpp
v.erase(unique(v.begin(), v.end()), v.end());
```

---

### Trap 6 — `vector` insertion is not always O(1)

Only appending is amortized `O(1)`.

Middle insertion requires shifting.

---

### Trap 7 — `list` insertion is O(1) only with the position known

Finding the position may take `O(n)`.

---

### Trap 8 — `lower_bound` requires sorted data for the standard binary-search behavior

---

### Trap 9 — Comparator should express ordering

For:

```cpp
sort(v.begin(), v.end(), comp);
```

think:

> Should `a` appear before `b`?

---

# 56. One-Page Decision Tree

```text
Need a container?
│
├── Need index access?
│      └── vector
│
├── Need front + back operations?
│      └── deque
│
├── Need linked-list insertion/erase?
│      ├── Doubly linked → list
│      └── Singly linked → forward_list
│
├── Need LIFO?
│      └── stack
│
├── Need FIFO?
│      └── queue
│
├── Need min/max repeatedly?
│      └── priority_queue
│
├── Need unique + sorted?
│      └── set
│
├── Need duplicate + sorted?
│      └── multiset
│
├── Need key-value + sorted keys?
│      └── map
│
├── Need duplicate keys + sorted?
│      └── multimap
│
├── Need unique + fast average lookup?
│      └── unordered_set
│
└── Need key-value + fast average lookup?
       └── unordered_map
```

---

# 57. Interview Quick Recall

### `vector`

```text
Dynamic array
Index → O(1)
push_back → amortized O(1)
middle insert/erase → O(n)
```

### `set`

```text
Unique + sorted
O(log n)
```

### `unordered_set`

```text
Unique + hash
Average O(1)
No ordering
```

### `map`

```text
Key-value
Unique keys
Sorted keys
O(log n)
```

### `unordered_map`

```text
Key-value
Unique keys
Hash table
Average O(1)
```

### `stack`

```text
LIFO
push/pop/top → O(1)
```

### `queue`

```text
FIFO
push/pop/front → O(1)
```

### `priority_queue`

```text
Heap
top → O(1)
push/pop → O(log n)
```

### `lower_bound`

```text
first >= x
```

### `upper_bound`

```text
first > x
```

### `unique`

```text
moves unique elements
does not resize
```

---

# 58. Must-Know STL Interview Questions

Before leaving STL, you should be able to answer these immediately:

1. Why is vector indexing `O(1)`?
2. Why is vector middle insertion `O(n)`?
3. Why is `push_back` amortized `O(1)`?
4. Difference between `size()` and `capacity()`?
5. `vector` vs `deque`?
6. `vector` vs `list`?
7. Why can't you do `list[i]`?
8. `set` vs `unordered_set`?
9. `map` vs `unordered_map`?
10. Average vs worst-case complexity of `unordered_map`?
11. Why can `mp[key]` unexpectedly insert?
12. `set` vs `multiset`?
13. `map` vs `multimap`?
14. Max heap vs min heap?
15. How do you create a min heap in C++?
16. How does `priority_queue<pair<int,int>>` order pairs?
17. What does `lower_bound` return?
18. What does `upper_bound` return?
19. How do you count occurrences using bounds?
20. What does `unique()` actually do?
21. How do you erase elements while iterating?
22. What is an iterator?
23. Difference between `begin()` and `end()`?
24. Which containers support random-access iterators?
25. How does `sort` use a comparator?
26. What does `return a < b` mean in a comparator?
27. How do you sort by one field and tie-break by another?
28. How do you sort a vector of pairs by `.second`?
29. How do you create a custom priority queue comparator?
30. When should you use a frequency array instead of `unordered_map`?
31. How do `find`, `find_if`, and `count` differ?
32. How do you remove duplicates from a vector?
33. How do you calculate the sum using STL?
34. How do you generate permutations?
35. What is a lambda?
36. Difference between capture by value and reference?
37. What is a template?
38. Why are `vector<int>` and `vector<string>` possible using the same container template?
39. Difference between `reserve()` and `resize()`?
40. What is the difference between `operator[]` and `at()` for vector/string?

---

# 59. 30-Second Revision Sheet

```text
VECTOR
index O(1), push_back amortized O(1), middle O(n)

DEQUE
O(1) front/back, O(1) index

LIST
O(1) insert/erase with iterator, O(n) traversal/access

SET
unique + sorted + O(log n)

MULTISET
duplicates + sorted + O(log n)

MAP
key/value + sorted keys + O(log n)

UNORDERED_MAP
key/value + hash + average O(1)

STACK
LIFO

QUEUE
FIFO

PRIORITY_QUEUE
heap; top O(1), push/pop O(log n)

LOWER_BOUND
first >= x

UPPER_BOUND
first > x

SORT
O(n log n)

UNIQUE
doesn't resize

ACCUMULATE
sum/reduction

IOTA
fill consecutive values

NEXT_PERMUTATION
next lexicographical permutation

LAMBDA
anonymous function

COMPARATOR
"Should a come before b?"

TEMPLATE
generic code over types
```

---

# 60. Recommended Interview Header

For most DSA problems:

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {

    // solution

    return 0;
}
```

Then use the STL component that matches the problem rather than implementing a data structure unnecessarily.

---

# Final STL Checklist

Before moving to the next Phase 0 module, you should be comfortable with:

- [ ] `pair`
- [ ] `vector`
- [ ] `deque`
- [ ] `list`
- [ ] `forward_list`
- [ ] `stack`
- [ ] `queue`
- [ ] `priority_queue`
- [ ] `set`
- [ ] `multiset`
- [ ] `map`
- [ ] `multimap`
- [ ] `unordered_set`
- [ ] `unordered_map`
- [ ] Iterators
- [ ] Iterator categories
- [ ] `sort`
- [ ] `reverse`
- [ ] `find`
- [ ] `find_if`
- [ ] `count`
- [ ] `max_element`
- [ ] `min_element`
- [ ] `lower_bound`
- [ ] `upper_bound`
- [ ] `binary_search`
- [ ] `unique`
- [ ] `accumulate`
- [ ] `iota`
- [ ] `next_permutation`
- [ ] `all_of`
- [ ] `any_of`
- [ ] `none_of`
- [ ] `erase-remove` idiom
- [ ] Strings
- [ ] String conversions
- [ ] Templates
- [ ] Lambdas
- [ ] Lambda capture
- [ ] Comparators
- [ ] Multi-criteria sorting
- [ ] Custom priority queues
- [ ] STL complexity intuition
- [ ] Common STL traps
