# Java Interview Programs — Cognizant

> Format: **Input → Program → Output**

Add at top of every file:
```java
import java.util.*;
import java.util.stream.*;
```

---

# 🟢 EASY

---

## 1. Character Frequency in String ⭐

**Input:** `"Nishanth"`

```java
String s = "Nishanth";
Map<Character, Integer> map = new LinkedHashMap<>();

for (char ch : s.toCharArray()) {
    map.put(ch, map.getOrDefault(ch, 0) + 1);
}

for (Map.Entry<Character, Integer> e : map.entrySet()) {
    System.out.println(e.getKey() + " - " + e.getValue());
}
```

**Output:**
```
N - 1
i - 1
s - 1
h - 2
a - 1
n - 1
t - 1
```

---

## 2. List Frequency using HashMap ⭐

**Input:** `["red", "red", "blue"]`

```java
List<String> colors = Arrays.asList("red", "red", "blue");
Map<String, Integer> map = new HashMap<>();

for (String c : colors) {
    map.put(c, map.getOrDefault(c, 0) + 1);
}

for (Map.Entry<String, Integer> e : map.entrySet()) {
    System.out.println(e.getKey() + " - " + e.getValue());
}
```

**Output:**
```
red - 2
blue - 1
```

---

## 3. Print ArrayList

**Input:** `5, 6, 1, 4, 2`

```java
List<Integer> a = new ArrayList<>();
a.add(5); a.add(6); a.add(1); a.add(4); a.add(2);
System.out.println(a);
```

**Output:**
```
[5, 6, 1, 4, 2]
```

---

## 4. Maximum in List

**Input:** `[3, 7, 2, 9, 5]`

```java
List<Integer> a = Arrays.asList(3, 7, 2, 9, 5);
int max = a.get(0);
for (int b : a) {
    if (b > max) max = b;
}
System.out.println("Max: " + max);
```

**Output:**
```
Max: 9
```

---

## 5. Reverse a String

**Input:** `"hello"`

```java
String s = "hello";
String rev = new StringBuilder(s).reverse().toString();
System.out.println(rev);
```

**Output:**
```
olleh
```

---

## 6. Palindrome Check

**Input:** `"madam"`

```java
String s = "madam";
String rev = new StringBuilder(s).reverse().toString();
System.out.println(s.equalsIgnoreCase(rev) ? "Palindrome" : "Not Palindrome");
```

**Output:**
```
Palindrome
```

---

## 7. Factorial

**Input:** `n = 5`

```java
int n = 5;
long fact = 1;
for (int i = 1; i <= n; i++) {
    fact *= i;
}
System.out.println(fact);
```

**Output:**
```
120
```

---

## 8. Fibonacci Series

**Input:** `n = 10`

```java
int n = 10;
int a = 0, b = 1;
for (int i = 0; i < n; i++) {
    System.out.print(a + " ");
    int next = a + b;
    a = b;
    b = next;
}
```

**Output:**
```
0 1 1 2 3 5 8 13 21 34
```

---

## 9. Reverse a Number

**Input:** `1234`

```java
int n = 1234, rev = 0;
while (n > 0) {
    rev = rev * 10 + n % 10;
    n /= 10;
}
System.out.println(rev);
```

**Output:**
```
4321
```

---

## 10. Sum of Digits

**Input:** `1234`

```java
int n = 1234, sum = 0;
while (n > 0) {
    sum += n % 10;
    n /= 10;
}
System.out.println(sum);
```

**Output:**
```
10
```

---

## 11. Swap Without Temp Variable

**Input:** `a = 5, b = 10`

```java
int a = 5, b = 10;
a = a + b;
b = a - b;
a = a - b;
System.out.println("a=" + a + " b=" + b);
```

**Output:**
```
a=10 b=5
```

---

## 12. Count Vowels and Consonants

**Input:** `"Hello World"`

```java
String s = "Hello World";
int vowels = 0, consonants = 0;
for (char ch : s.toLowerCase().toCharArray()) {
    if (ch >= 'a' && ch <= 'z') {
        if ("aeiou".indexOf(ch) != -1) vowels++;
        else consonants++;
    }
}
System.out.println("Vowels: " + vowels + ", Consonants: " + consonants);
```

**Output:**
```
Vowels: 3, Consonants: 7
```

---

## 13. Prime Number Check

**Input:** `n = 7`

```java
int n = 7;
boolean isPrime = n >= 2;
for (int i = 2; i * i <= n; i++) {
    if (n % i == 0) { isPrime = false; break; }
}
System.out.println(isPrime ? "Prime" : "Not Prime");
```

**Output:**
```
Prime
```

---

## 14. Iterate HashMap

**Input:** `{101=Saikiran, 103=Siva, 102=Vamsi, 105=Madhu}`

```java
HashMap<Integer, String> map = new HashMap<>();
map.put(101, "Saikiran");
map.put(103, "Siva");
map.put(102, "Vamsi");
map.put(105, "Madhu");

for (Map.Entry<Integer, String> e : map.entrySet()) {
    System.out.println(e.getKey() + " -> " + e.getValue());
}
```

**Output:**
```
101 -> Saikiran
102 -> Vamsi
103 -> Siva
105 -> Madhu
```

---

## 15. Remove Duplicates from List

**Input:** `[1, 5, 2, 2, 7, 3, 4, 4]`

```java
List<Integer> a = Arrays.asList(1, 5, 2, 2, 7, 3, 4, 4);
Set<Integer> unique = new TreeSet<>(a);
System.out.println(unique);
```

**Output:**
```
[1, 2, 3, 4, 5, 7]
```

---

## 16. Reverse a List

**Input:** `[3, 7, 2, 9, 5]`

```java
List<Integer> a = new ArrayList<>(Arrays.asList(3, 7, 2, 9, 5));
Collections.reverse(a);
System.out.println(a);
```

**Output:**
```
[5, 9, 2, 7, 3]
```

---

# 🟡 MEDIUM

---

## 17. First Non-Repeating Character

**Input:** `"swiss"`

```java
String s = "swiss";
Map<Character, Integer> map = new LinkedHashMap<>();
for (char ch : s.toCharArray()) {
    map.put(ch, map.getOrDefault(ch, 0) + 1);
}
for (char ch : s.toCharArray()) {
    if (map.get(ch) == 1) {
        System.out.println(ch);
        break;
    }
}
```

**Output:**
```
w
```

---

## 18. Find All Duplicate Characters

**Input:** `"automation"`

```java
String s = "automation";
Map<Character, Integer> map = new HashMap<>();
for (char ch : s.toCharArray()) {
    map.put(ch, map.getOrDefault(ch, 0) + 1);
}
for (Map.Entry<Character, Integer> e : map.entrySet()) {
    if (e.getValue() > 1) System.out.println(e.getKey());
}
```

**Output:**
```
a
t
o
```

---

## 19. Find Duplicates in List

**Input:** `[2, 3, 1, 2, 3]`

```java
List<Integer> a = Arrays.asList(2, 3, 1, 2, 3);
Set<Integer> seen = new HashSet<>();
List<Integer> duplicates = new ArrayList<>();
for (int i : a) {
    if (!seen.add(i)) duplicates.add(i);
}
System.out.println(duplicates);
```

**Output:**
```
[2, 3]
```

---

## 20. Second Largest Element

**Input:** `[10, 20, 5, 8, 20]`

```java
List<Integer> a = Arrays.asList(10, 20, 5, 8, 20);
Set<Integer> set = new TreeSet<>(a);
List<Integer> list = new ArrayList<>(set);
System.out.println(list.get(list.size() - 2));
```

**Output:**
```
10
```

---

## 21. Remove Duplicates Preserving Order

**Input:** `[10, 20, 5, 8, 20]`

```java
List<Integer> a = Arrays.asList(10, 20, 5, 8, 20);
Set<Integer> seen = new HashSet<>();
List<Integer> result = new ArrayList<>();
for (int i : a) {
    if (seen.add(i)) result.add(i);
}
System.out.println(result);
```

**Output:**
```
[10, 20, 5, 8]
```

---

## 22. Anagram Check

**Input:** `"listen", "silent"`

```java
String a = "listen", b = "silent";
char[] ac = a.toCharArray();
char[] bc = b.toCharArray();
Arrays.sort(ac);
Arrays.sort(bc);
System.out.println(Arrays.equals(ac, bc) ? "Anagram" : "Not Anagram");
```

**Output:**
```
Anagram
```

---

## 23. Armstrong Number

**Input:** `n = 153`

```java
int n = 153, original = n, sum = 0;
int digits = String.valueOf(n).length();
while (n > 0) {
    int d = n % 10;
    sum += Math.pow(d, digits);
    n /= 10;
}
System.out.println(sum == original ? "Armstrong" : "Not Armstrong");
```

**Output:**
```
Armstrong
```

---

## 24. GCD (Euclidean)

**Input:** `a = 48, b = 18`

```java
int a = 48, b = 18;
while (b != 0) {
    int temp = b;
    b = a % b;
    a = temp;
}
System.out.println("GCD: " + a);
```

**Output:**
```
GCD: 6
```

---

## 25. Two Sum ⭐

**Input:** `nums = [2, 7, 11, 15], target = 9`

```java
int[] nums = {2, 7, 11, 15};
int target = 9;
Map<Integer, Integer> map = new HashMap<>();
int[] result = {};

for (int i = 0; i < nums.length; i++) {
    int complement = target - nums[i];
    if (map.containsKey(complement)) {
        result = new int[]{map.get(complement), i};
        break;
    }
    map.put(nums[i], i);
}
System.out.println(Arrays.toString(result));
```

**Output:**
```
[0, 1]
```

---

## 26. Stream — Filter Even Numbers

**Input:** `[3, 6, 13, 9, 5, 4]`

```java
List<Integer> a = Arrays.asList(3, 6, 13, 9, 5, 4);
List<Integer> evens = a.stream()
                       .filter(x -> x % 2 == 0)
                       .collect(Collectors.toList());
System.out.println(evens);
```

**Output:**
```
[6, 4]
```

---

## 27. Stream — Square of Even Numbers

**Input:** `[3, 6, 13, 9, 5, 4]`

```java
List<Integer> a = Arrays.asList(3, 6, 13, 9, 5, 4);
List<Integer> result = a.stream()
                        .filter(x -> x % 2 == 0)
                        .map(n -> n * n)
                        .collect(Collectors.toList());
System.out.println(result);
```

**Output:**
```
[36, 16]
```

---

## 28. Stream — Sum, Average, Max, Count

**Input:** `[3, 6, 13, 9, 5, 4]`

```java
List<Integer> a = Arrays.asList(3, 6, 13, 9, 5, 4);
int sum = a.stream().mapToInt(Integer::intValue).sum();
double avg = a.stream().mapToInt(Integer::intValue).average().orElse(0.0);
int max = a.stream().max(Integer::compareTo).orElse(-1);
long count = a.stream().filter(x -> x > 5).count();

System.out.println("Sum: " + sum);
System.out.println("Average: " + avg);
System.out.println("Max: " + max);
System.out.println("Count > 5: " + count);
```

**Output:**
```
Sum: 40
Average: 6.666666666666667
Max: 13
Count > 5: 3
```

---

## 29. Stream — Find First Element > 10

**Input:** `[3, 6, 13, 9, 5, 4]`

```java
List<Integer> a = Arrays.asList(3, 6, 13, 9, 5, 4);
Optional<Integer> first = a.stream().filter(x -> x > 10).findFirst();
first.ifPresent(System.out::println);
```

**Output:**
```
13
```

---

## 30. Stream — Group by Frequency

**Input:** `["Banana", "Apple", "Cherry", "Apple", "Banana"]`

```java
List<String> fruits = Arrays.asList("Banana", "Apple", "Cherry", "Apple", "Banana");
Map<String, Long> freq = fruits.stream()
    .collect(Collectors.groupingBy(f -> f, Collectors.counting()));
System.out.println(freq);
```

**Output:**
```
{Apple=2, Cherry=1, Banana=2}
```

---

## 31. Sort List using Stream

**Input:** `["Banana", "Apple", "Cherry"]`

```java
List<String> fruits = Arrays.asList("Banana", "Apple", "Cherry");
fruits.stream().sorted().forEach(System.out::println);
```

**Output:**
```
Apple
Banana
Cherry
```

---

# 🔴 HARD

---

## 32. Sort HashMap by Value

**Input:** `{A=3, B=1, C=2}`

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 3); map.put("B", 1); map.put("C", 2);

Map<String, Integer> sorted = map.entrySet().stream()
    .sorted(Map.Entry.comparingByValue())
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        (e1, e2) -> e1,
        LinkedHashMap::new));
System.out.println(sorted);
```

**Output:**
```
{B=1, C=2, A=3}
```

---

## 33. Top K Frequent Elements

**Input:** `nums = [1, 1, 1, 2, 2, 3], k = 2`

```java
int[] nums = {1, 1, 1, 2, 2, 3};
int k = 2;

Map<Integer, Integer> freq = new HashMap<>();
for (int n : nums) freq.put(n, freq.getOrDefault(n, 0) + 1);

List<Integer> result = freq.entrySet().stream()
    .sorted((a, b) -> b.getValue() - a.getValue())
    .limit(k)
    .map(Map.Entry::getKey)
    .collect(Collectors.toList());
System.out.println(result);
```

**Output:**
```
[1, 2]
```

---

## 34. Longest Substring Without Repeating

**Input:** `"abcabcbb"`

```java
String s = "abcabcbb";
Set<Character> window = new HashSet<>();
int left = 0, maxLen = 0;

for (int right = 0; right < s.length(); right++) {
    while (window.contains(s.charAt(right))) {
        window.remove(s.charAt(left));
        left++;
    }
    window.add(s.charAt(right));
    maxLen = Math.max(maxLen, right - left + 1);
}
System.out.println(maxLen);
```

**Output:**
```
3
```

---

## 35. Find Missing Number in 1..N

**Input:** `[1, 2, 4, 5], n = 5`

```java
int[] arr = {1, 2, 4, 5};
int n = 5;
int expectedSum = n * (n + 1) / 2;
int actualSum = Arrays.stream(arr).sum();
System.out.println(expectedSum - actualSum);
```

**Output:**
```
3
```

---

## 36. Rotate Array Right by K

**Input:** `arr = [1, 2, 3, 4, 5], k = 2`

```java
int[] arr = {1, 2, 3, 4, 5};
int k = 2 % arr.length;

reverse(arr, 0, arr.length - 1);
reverse(arr, 0, k - 1);
reverse(arr, k, arr.length - 1);

System.out.println(Arrays.toString(arr));

static void reverse(int[] a, int l, int r) {
    while (l < r) {
        int t = a[l]; a[l] = a[r]; a[r] = t;
        l++; r--;
    }
}
```

**Output:**
```
[4, 5, 1, 2, 3]
```

---

## 37. Group Anagrams

**Input:** `["eat", "tea", "tan", "ate", "nat", "bat"]`

```java
String[] strs = {"eat", "tea", "tan", "ate", "nat", "bat"};
Map<String, List<String>> groups = new HashMap<>();

for (String s : strs) {
    char[] chars = s.toCharArray();
    Arrays.sort(chars);
    String key = new String(chars);
    groups.computeIfAbsent(key, x -> new ArrayList<>()).add(s);
}
System.out.println(groups.values());
```

**Output:**
```
[[eat, tea, ate], [tan, nat], [bat]]
```

---

## 38. Count Words in a Sentence

**Input:** `"java is fun and java is powerful"`

```java
String s = "java is fun and java is powerful";
Map<String, Integer> map = new LinkedHashMap<>();
for (String word : s.split(" ")) {
    map.put(word, map.getOrDefault(word, 0) + 1);
}
System.out.println(map);
```

**Output:**
```
{java=2, is=2, fun=1, and=1, powerful=1}
```

---

## 39. Reverse Words in a Sentence

**Input:** `"I love Java"`

```java
String s = "I love Java";
String[] words = s.split(" ");
StringBuilder sb = new StringBuilder();
for (int i = words.length - 1; i >= 0; i--) {
    sb.append(words[i]);
    if (i != 0) sb.append(" ");
}
System.out.println(sb);
```

**Output:**
```
Java love I
```

---

## 40. Find Pair with Given Sum

**Input:** `arr = [1, 4, 6, 8, 10], target = 14`

```java
int[] arr = {1, 4, 6, 8, 10};
int target = 14;
Set<Integer> seen = new HashSet<>();
for (int n : arr) {
    if (seen.contains(target - n)) {
        System.out.println("(" + (target - n) + ", " + n + ")");
    }
    seen.add(n);
}
```

**Output:**
```
(6, 8)
(4, 10)
```
