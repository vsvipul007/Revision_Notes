# Java Revision Notes — C++ to Java Transition (NeetCode)

Collected from: Contains Duplicate, Valid Anagram, Two Sum, Group Anagrams

---

## 1. String vs char[] — no implicit conversion

Java's `String` and `char[]` are separate, unrelated types. Unlike C++ (where a `string` and char buffers are conceptually close), there's no shortcut between them — you always go through a method.

```java
char[] chars = str.toCharArray();   // String -> char[]
String s = new String(chars);       // char[] -> String
```

`new char[]{str}` is wrong — you can't put a whole `String` into a single `char` slot. A `char` holds exactly one character.

To sort a string's characters (no direct `String.sort()` exists):
```java
char[] chars = str.toCharArray();
Arrays.sort(chars);
String sorted = new String(chars);
```

---

## 2. Declaration vs. Assignment vs. Object Creation (the big one)

Three distinct things that look similar:

| Code | What it actually does |
|---|---|
| `List<String> slist;` | **Declaration** — creates an empty variable "box" that can hold a `List<String>` reference. Does NOT create a list. |
| `slist = mp.get(sorted);` | **Lookup/assignment** — box now points to whatever the map returned (a real object, or `null`) |
| `slist = new ArrayList<>();` | **Object creation** — actually builds a new object in memory, box points to it |

Key insight: a variable is like a labeled parking spot / a sticky note with an address — not a container. `mp.get(key)` can only **hand you a reference to something that already exists**; if the key isn't in the map, there is no object anywhere, so you get `null` back (not an empty list).

Once a variable is declared, reassigning it never needs the type again:
```java
List<String> slist = mp.get(sorted);
if (slist == null) {
    slist = new ArrayList<>();   // reassignment, NOT redeclaration
}
```
Writing `List<String> slist = new ArrayList<>();` again in the same scope is a compile error — "variable already defined."

---

## 3. Java's Map has no C++-style auto-vivify

C++: `mp[key].push_back(x);` auto-creates a default value if `key` is missing.

Java: `Map.get(key)` returns `null` if missing — it will NOT create anything for you. Calling a method on that `null` throws `NullPointerException`.

Manual pattern (mirrors C++ auto-vivify):
```java
List<String> list = mp.get(key);
if (list == null) {
    list = new ArrayList<>();
    mp.put(key, list);        // put only needed when NEWLY created
}
list.add(str);
```
Note: `put()` is only necessary in the "just created it" branch. If the list already existed in the map, `.get()` gave you a reference to the *actual object living in the map* — mutating it via `.add()` automatically reflects in the map. No re-`put()` needed.

**Idiomatic one-liner** for this exact pattern:
```java
mp.computeIfAbsent(sorted, k -> new ArrayList<>()).add(str);
```
- `computeIfAbsent(key, fn)`: if key exists, returns its value as-is. If not, calls `fn(key)`, stores the result under `key`, and returns it.
- Either way you get back a reference to the list *currently in the map* — chaining `.add(str)` on it mutates the map's own copy in place.
- The second argument is a **function** (lambda), not a value — `computeIfAbsent(key, new ArrayList<>())` is wrong (wrong type, and would allocate unconditionally).

`merge(key, value, biFunction)` is the sibling method — better suited to combining scalar values (e.g. summing counts), whereas `computeIfAbsent` fits "always append into a collection."

---

## 4. Common naming mistakes (Java is strict, no STL guessing)

| Wrong (C++-ish guess) | Correct Java |
|---|---|
| `mp.getValues()` | `mp.values()` |
| `ans.put(item)` | `ans.add(item)` — `put()` is a `Map` method, `add()` is `List`/`Collection` |
| `sort(str)` as a free function | `Arrays.sort(charArray)` (in-place, only works on arrays, not directly on `String`) |

---

## 5. Generics: raw types vs. diamond operator

```java
new ArrayList();     // raw type — compiles, but "unchecked" warning, considered bad practice
new ArrayList<>();   // diamond operator — type inferred from context, idiomatic
```
Always prefer `<>()` — raw types are a pre-Java-5 legacy holdover.

---

## 6. "Idiomatic" — what it means

Idiomatic = the way an experienced Java dev would naturally write it, using the standard library the way it's designed to be used — not just code that happens to compile.

Examples from this session:
- `computeIfAbsent(...).add(...)` is idiomatic; manual `get`/null-check/`put` is not (but is equally correct).
- `new ArrayList<>()` is idiomatic; `new ArrayList()` (raw type) is not.

Worth building idiomatic habits early since interviewers/reviewers often judge *how* Java is used, not just whether output is correct.

---

## Quick Reference: Final `groupAnagrams` (idiomatic)

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        HashMap<String, List<String>> mp = new HashMap<>();
        for (String str : strs) {
            char[] chars = str.toCharArray();
            Arrays.sort(chars);
            String sorted = new String(chars);
            mp.computeIfAbsent(sorted, k -> new ArrayList<>()).add(str);
        }
        return new ArrayList<>(mp.values());
    }
}
```

---

## 7. Prefer interface on the left, implementation on the right

```java
Map<Character, Integer> map = new HashMap<>();
Set<Integer> set = new HashSet<>();
List<Integer> list = new ArrayList<>();
```

- `Map`, `Set`, `List` are interfaces; `HashMap`, `HashSet`, `ArrayList` are concrete implementations.
- Prefer the interface type when you only need the common behavior. It keeps the code flexible, e.g. `Map<...> map = new TreeMap<>();` later without changing callers.
- `HashMap<...> map = new HashMap<>();` is valid, just more tightly coupled to that implementation.
- C++ STL has no exact equivalent; in C++ you usually declare the concrete container directly (`unordered_map`, `unordered_set`, `vector`).

---

## 8. HashSet — core DSA syntax

```java
Set<Integer> set = new HashSet<>();

set.add(x);       // inserts x; returns false if x was already present
set.contains(x);  // membership check
```

Useful duplicate pattern:
```java
if (!set.add(x)) return true;
```

C++ pointer:
- Java `set.contains(x)` ~ C++ `set.find(x) != set.end()`
- Java `set.add(x)` ~ C++ `set.insert(x)`

---

## 9. HashMap — core methods + frequency counting

```java
Map<Integer, Integer> map = new HashMap<>();
```

Most-used methods:
```java
map.put(key, value);
map.get(key);
map.containsKey(key);
map.getOrDefault(key, defaultValue);
map.remove(key);
map.size();
map.isEmpty();
map.keySet();
map.values();
map.entrySet();
map.putIfAbsent(key, value);
```

Frequency counter — primary pattern to remember:
```java
map.put(x, map.getOrDefault(x, 0) + 1);
map.put(x, map.getOrDefault(x, 0) - 1);
```

Optional shorthand:
```java
map.merge(x, 1, Integer::sum);
map.merge(x, -1, Integer::sum);
```
Use `getOrDefault()` as the default interview pattern for now; `merge()` is concise but less important to memorize.

C++ pointer:
- C++ `mp[x]++` is concise because `operator[]` can create a default value.
- Java `get()` does not auto-create a missing entry, so use `getOrDefault()` + `put()`.

Naming trap:
```java
map.get(key);          // correct lookup
entry.getValue();      // only when working with Map.Entry
map.getValue(key);     // wrong
```

---

## 10. Primitive vs wrapper types in generics

Java generics require reference types, not primitives.

```java
Map<Character, Integer> map = new HashMap<>();  // correct
HashMap<char, int> map;                         // wrong
```

Common pairs:
- `int` -> `Integer`
- `char` -> `Character`
- `double` -> `Double`
- `boolean` -> `Boolean`

---

## 11. Iterating through a Java String

Unlike C++, Java `String` cannot be used directly in an enhanced `for` loop.

Character-only iteration:
```java
for (char c : s.toCharArray()) {
    // use c
}
```

Indexed access:
```java
for (int i = 0; i < s.length(); i++) {
    char c = s.charAt(i);
}
```

Use `toCharArray()` when you only need each character. Use `charAt(i)` when the index matters or when processing two strings together.

C++ pointer:
```cpp
for (char c : s) { }
s[i]
```
Java equivalents:
```java
for (char c : s.toCharArray()) { }
s.charAt(i)
```

Length reminder:
- Java `String`: `s.length()`
- Java array: `arr.length`
- C++ `string`: `s.length()` or `s.size()`

---

## 12. Java arrays — declaration, defaults, fill

Preferred Java style:
```java
int[] freq = new int[26];
```

Also valid, but less idiomatic:
```java
int freq[] = new int[26];
```

`new int[26]` initializes all entries to `0` automatically.

To initialize all entries to another value:
```java
Arrays.fill(freq, -1);
```

C++ pointer:
```cpp
vector<int> freq(26, -1);
```

---

## 13. Frequency array for lowercase English letters

If the input domain is fixed to `a`-`z`, an `int[26]` is often simpler than a `HashMap`.

```java
int[] freq = new int[26];

freq[ch - 'a']++;
freq[ch - 'a']--;
```

Prefer `ch - 'a'` over `ch - 97`; it avoids a magic number and clearly shows intent.

For two equal-length strings:
```java
for (int i = 0; i < s.length(); i++) {
    freq[s.charAt(i) - 'a']++;
    freq[t.charAt(i) - 'a']--;
}
```

Key interview lesson: choose the data structure from the constraints. Fixed alphabet -> array can be better; arbitrary characters -> `Map<Character, Integer>` is more general.

---

## 14. Valid Anagram — final versions

### General HashMap version
```java
class Solution {
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) return false;

        Map<Character, Integer> count = new HashMap<>();

        for (char c : s.toCharArray()) {
            count.put(c, count.getOrDefault(c, 0) + 1);
        }
        for (char c : t.toCharArray()) {
            count.put(c, count.getOrDefault(c, 0) - 1);
        }

        for (int value : count.values()) {
            if (value != 0) return false;
        }
        return true;
    }
}
```

### Lowercase-English optimized version
```java
class Solution {
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) return false;

        int[] freq = new int[26];

        for (int i = 0; i < s.length(); i++) {
            freq[s.charAt(i) - 'a']++;
            freq[t.charAt(i) - 'a']--;
        }

        for (int count : freq) {
            if (count != 0) return false;
        }
        return true;
    }
}
```

---

## Current C++ -> Java transition weak spots to revise

- Collection method names: `get`, `put`, `add`, `containsKey`, `values`.
- Primitive vs wrapper in generics: `char`/`Character`, `int`/`Integer`.
- String access: C++ `s[i]` vs Java `s.charAt(i)`.
- String iteration: C++ direct range-for vs Java `toCharArray()` / `charAt(i)`.
- Length syntax: `String.length()` vs array `.length`.
- Frequency maps: C++ `mp[x]++` vs Java `getOrDefault()` + `put()`.
- Java idioms: interface on left (`Map`, `Set`, `List`) and concrete implementation on right.

---

## 15. Direct array creation and return

Create a fixed-size integer array:
```java
int[] res = new int[k];
```
All elements start as `0`.

Create and initialize immediately:
```java
return new int[]{a, b};
```

Empty array:
```java
return new int[]{};
// or
return new int[0];
```

C++ pointer:
```cpp
vector<int> res(k, 0);
return vector<int>{a, b};
```

Common trap: Java object construction uses `new Class(...)`, but arrays use `new type[size]`. So `new int(k)` is invalid; use `new int[k]`.

---

## 16. Two Sum — HashMap pattern

Store the number already seen as the key and its index as the value:
```java
Map<Integer, Integer> map = new HashMap<>();

for (int i = 0; i < nums.length; i++) {
    int complement = target - nums[i];

    if (map.containsKey(complement)) {
        return new int[]{map.get(complement), i};
    }

    map.put(nums[i], i);
}
```

Mental model:
```text
number -> index
```

C++ pointer:
```cpp
unordered_map<int, int> mp;
```

---

## 17. Array/reference semantics

This does NOT create a copy:
```java
char[] a = s.toCharArray();
char[] b = a;
```
Both variables point to the same array. Mutating through either reference changes the same underlying object.

Example:
```java
Arrays.sort(b);  // a is now sorted too
```

Create a real copy:
```java
char[] b = a.clone();
// or
char[] b = Arrays.copyOf(a, a.length);
```

---

## 18. Map of Lists — manual and idiomatic patterns

For:
```java
Map<String, List<String>> map = new HashMap<>();
```

Manual version:
```java
if (!map.containsKey(key)) {
    map.put(key, new ArrayList<>());
}
map.get(key).add(value);
```

Why no second `put()` when the key exists? `map.get(key)` returns a reference to the actual `List` object already stored in the map, so `.add(...)` mutates that object directly.

Idiomatic version:
```java
map.computeIfAbsent(key, k -> new ArrayList<>()).add(value);
```

`computeIfAbsent` behavior:
- key exists -> return existing value
- key absent -> call the lambda, store the returned value, and return it
- `k` is simply the lambda parameter representing the key; it can be renamed

C++ pointer:
```cpp
mp[key].push_back(value);
```
C++ `operator[]` can auto-create a missing value; Java `Map.get()` does not.

---

## 19. `List.add()` return value

```java
list.add(value);
```
modifies the list and returns a `boolean` indicating whether the collection changed.

Therefore this is wrong when the map expects a `List<String>`:
```java
map.put(key, list.add(value));
```

Do this instead:
```java
list.add(value);
map.put(key, list);
```

---

## 20. `map.values()` and constructing the result

```java
map.values()
```
returns a collection view of all map values.

For Group Anagrams:
```java
return new ArrayList<>(map.values());
```
If the map is `Map<String, List<String>>`, this produces a `List<List<String>>`.

---

## 21. Group Anagrams — final pattern

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<>();

        for (String str : strs) {
            char[] chars = str.toCharArray();
            Arrays.sort(chars);
            String key = new String(chars);

            map.computeIfAbsent(key, k -> new ArrayList<>()).add(str);
        }

        return new ArrayList<>(map.values());
    }
}
```

C++ -> Java translation worth remembering:
```text
C++:  mp[key].push_back(value)
Java: map.computeIfAbsent(key, k -> new ArrayList<>()).add(value)
```

---

## 22. Iterating over a Map with `Map.Entry`

If:
```java
Map<Integer, Integer> mp = new HashMap<>();
```
then:
```java
mp.entrySet()
```
has elements of type:
```java
Map.Entry<Integer, Integer>
```

Iteration:
```java
for (Map.Entry<Integer, Integer> entry : mp.entrySet()) {
    int key = entry.getKey();
    int value = entry.getValue();
}
```

Why repeat `<Integer, Integer>`? Java's enhanced `for` loop requires the loop-variable type to be written explicitly. Java does not have C++-style `auto` here.

C++ pointer:
```cpp
for (auto entry : mp) {
    entry.first;
    entry.second;
}
```

Quick mapping:
```text
C++ entry.first   -> Java entry.getKey()
C++ entry.second  -> Java entry.getValue()
```

---

## 23. Nested Lists

Declaration:
```java
List<List<Integer>> pairs = new ArrayList<>();
```

Create one inner pair:
```java
List<Integer> pair = new ArrayList<>();
pair.add(frequency);
pair.add(number);
pairs.add(pair);
```

Important: `List` is an interface, so this is invalid:
```java
new List<>()
```
Use:
```java
new ArrayList<>()
```

Nested access:
```java
pairs.get(i).get(1);
```

C++ pointer:
```cpp
pairs[i][1];
```

---

## 24. Sorting Lists with a Comparator

For a simple list of comparable elements:
```java
Collections.sort(list);
// or
list.sort(Comparator.naturalOrder());
```

For `List<List<Integer>>`, Java does not know which inner value should determine the order, so provide a comparator.

Descending by index `0`:
```java
pairs.sort((a, b) -> Integer.compare(b.get(0), a.get(0)));
```

Ascending by index `0`:
```java
pairs.sort((a, b) -> Integer.compare(a.get(0), b.get(0)));
```

`Collections.sort` also works with a comparator:
```java
Collections.sort(pairs,
    (a, b) -> Integer.compare(b.get(0), a.get(0))
);
```

C++ pointer:
```cpp
sort(v.begin(), v.end(), greater<>());
```

---

## 25. Top K Frequent Elements — sorting approach

Your current Java translation of the C++ approach:

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        int[] res = new int[k];
        Map<Integer, Integer> mp = new HashMap<>();

        for (int num : nums) {
            mp.put(num, mp.getOrDefault(num, 0) + 1);
        }

        List<List<Integer>> freqPairs = new ArrayList<>();

        for (Map.Entry<Integer, Integer> entry : mp.entrySet()) {
            List<Integer> pair = new ArrayList<>();
            pair.add(entry.getValue()); // frequency
            pair.add(entry.getKey());   // number
            freqPairs.add(pair);
        }

        freqPairs.sort((a, b) -> Integer.compare(b.get(0), a.get(0)));

        for (int i = 0; i < k; i++) {
            res[i] = freqPairs.get(i).get(1);
        }

        return res;
    }
}
```

Mental flow:
```text
frequency map -> [frequency, number] pairs -> sort descending -> take first k
```

Complexity if there are `m` distinct values:
```text
Counting:      O(n)
Build pairs:   O(m)
Sort:          O(m log m)
Take k:        O(k)
Total:         O(n + m log m)
Space:         O(m)
```

This is a valid interview solution, but not the only useful approach. See the approach ladder below.

---

## 26. Top K Frequent — approach ladder

### A. Sort frequency pairs — current approach
- Easy to derive from C++.
- Complexity: `O(n + m log m)`.
- Good first answer if you can code it reliably.

### B. Min-heap of size `k`
- Count frequencies with a map.
- Keep only the best `k` entries in a `PriorityQueue`.
- Complexity: `O(n + m log k)`.
- Useful when `k` is much smaller than the number of distinct values.
- This is the next approach to learn after the sorting version.

### C. Bucket sort by frequency
- Frequency cannot exceed `n`.
- Create buckets indexed by frequency: bucket `f` contains numbers occurring `f` times.
- Walk buckets from `n` down to `1` until `k` numbers are collected.
- Complexity: `O(n)` time and `O(n)` extra space.
- This is the common optimal solution for the NeetCode/LeetCode version.

Prep rule: do not try to memorize all three implementations at once. Keep the sorting version reliable first, then learn the heap version, then bucket sort.

---

## Current C++ -> Java transition weak spots to revise (updated)

- Array syntax: `new int[k]`, `.length`.
- Collection declaration: `Map`/`Set`/`List` on the left; concrete implementation on the right.
- HashMap API: `get`, `put`, `containsKey`, `getOrDefault`, `entrySet`.
- `Map.Entry<K,V>` and `getKey()` / `getValue()`.
- Nested-list creation and `.get(i)` access.
- Comparator/lambda syntax for custom sorting.
- Primitive vs wrapper types in generics.
- String access/iteration: `charAt`, `toCharArray`.
- `char[]` -> `String`: use `new String(chars)`, not `chars.toString()`.
- Reference vs copy semantics for arrays.
- Java idioms: `computeIfAbsent` and `getOrDefault`.

### Reliability rule for interviews
Use the simplest Java syntax you can recall confidently. Correct, readable code is better than forcing an idiom you cannot reproduce under pressure.
