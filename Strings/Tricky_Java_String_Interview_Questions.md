# Tricky Java String Interview Questions for Backend Developers (with Explanations)

---

## 1. String Pool and Memory

**Q1:**  
```java
String a = "hello";
String b = new String("hello");
String c = b.intern();
System.out.println(a == b);
System.out.println(a == c);
```
**What will be the output and why?**  
**Explanation:**  
- `a == b` is `false` because `a` refers to the string pool object, while `b` is a new object in the heap.
- `a == c` is `true` because `c` (after `intern()`) refers to the string pool instance, which is also `a`.

---

**Q2:**  
**How does the String constant pool improve memory efficiency in Java applications? Can it cause any problems in large-scale systems?**  
**Explanation:**  
- The pool ensures only one instance of each literal is stored, reducing memory usage for repeated strings.
- In large-scale systems, excessive interning can fill up heap memory (Java 7+) or PermGen (Java 6), possibly causing memory leaks if many unique strings are interned.

---

## 2. String Immutability

**Q3:**  
**Why are strings immutable in Java? What are the advantages and disadvantages?**  
**Explanation:**  
- **Advantages:**  
  - Security (e.g., in class loading, file paths, network connections).
  - Thread-safety (safe sharing between threads).
  - Hashcode caching (for fast lookups in collections).
- **Disadvantages:**  
  - Inefficient for repeated modifications (many intermediate objects).

---

**Q4:**  
**If you want to store a password in memory securely, would you use a `String` or a `char[]`? Why?**  
**Explanation:**  
- Use `char[]`. Strings are immutable and reside in the pool until GC'd, risking exposure. `char[]` can be explicitly wiped out after use.

---

## 3. String Manipulation and Performance

**Q5:**  
```java
String s = "abc";
for (int i = 0; i < 3; i++) {
    s += i;
}
System.out.println(s);
```
**What is the output and how could this be improved for performance?**  
**Explanation:**  
- Output: `abc012`
- Each `+=` creates a new String. For performance, use `StringBuilder` for repeated modifications.

---

**Q6:**  
```java
String s1 = "foo";
String s2 = "f" + "oo";
String s3 = "f";
String s4 = s3 + "oo";
System.out.println(s1 == s2);
System.out.println(s1 == s4);
```
**What will be printed and why?**  
**Explanation:**  
- `s1 == s2` is `true` (compile-time concatenation, both point to pool).
- `s1 == s4` is `false` (`s4` is runtime concatenation, creates a new object).

---

## 4. String Interning

**Q7:**  
**What happens when you call `.intern()` on a String object created with `new String("xyz")`? When might interning be beneficial or harmful?**  
**Explanation:**  
- If `"xyz"` is in the pool, returns the pool reference; else, adds it to the pool.
- Beneficial for saving memory with repeated strings (e.g., keys). Harmful if many unique strings (memory bloat).

---

## 5. Regex and Edge Cases

**Q8:**  
```java
String input = "a,,b,c,,,";
String[] parts = input.split(",");
System.out.println(parts.length);
```
**What is the output, and why? How can you split to include trailing empty strings?**  
**Explanation:**  
- Output: `4`  
  - `split()` by default omits trailing empty strings.
- To include them: `input.split(",", -1);` Output: `7`

---

**Q9:**  
**What is the difference between `replace()`, `replaceAll()`, and `replaceFirst()` in String?**  
**Explanation:**  
- `replace()` - replaces all occurrences of a literal.
- `replaceAll()` - replaces all matches of a regex.
- `replaceFirst()` - replaces the first match of a regex.

---

## 6. Substring and Memory Leaks

**Q10:**  
**Prior to Java 7, what memory issue could occur when using `substring()` on large strings? How was this fixed in later versions?**  
**Explanation:**  
- Before Java 7, substring created a view of the original char array (not a copy), causing the whole large array to remain in memory.
- Java 7+ copies the relevant characters, so the original array can be GC'd.

---

## 7. Advanced Scenarios

**Q11:**  
**How would you efficiently find the first non-repeated character in a String? What is the time complexity?**  
**Explanation:**  
- Use a `LinkedHashMap<Character, Integer>` to count chars and preserve order, then find the first with count 1.
- Time: `O(n)`

---

**Q12:**  
**How would you check if two strings are anagrams, considering Unicode normalization and case folding?**  
**Explanation:**  
- Normalize both strings (NFC/NFD), convert to lower case, sort chars and compare, or use frequency maps.

---

## 8. StringBuilder vs StringBuffer

**Q13:**  
**What is the difference between `StringBuilder` and `StringBuffer`? When would you use each?**  
**Explanation:**  
- `StringBuffer` is synchronized (thread-safe), but slower.
- `StringBuilder` is not synchronized (not thread-safe), but faster.
- Use `StringBuilder` in single-threaded, `StringBuffer` in multi-threaded contexts.

---

**Q14:**  
**Is `StringBuilder` always more efficient than concatenating strings with `+`? Explain.**  
**Explanation:**  
- In loops or repeated modifications, `StringBuilder` is more efficient.
- For a small number of concatenations, `+` may be compiled to use `StringBuilder` by the compiler (for single statements).

---

## 9. Encoding and Decoding

**Q15:**  
**How would you convert a String to a byte array and back using UTF-8 in Java? Why specify charset?**  
**Explanation:**  
```java
byte[] bytes = str.getBytes(StandardCharsets.UTF_8);
String restored = new String(bytes, StandardCharsets.UTF_8);
```
- Specifying charset avoids platform-dependent encoding issues.

---

## 10. Miscellaneous

**Q16:**  
```java
String s1 = null;
String s2 = "null";
System.out.println(s1 + s2);
```
**What happens?**  
**Explanation:**  
- Prints `nullnull` (null reference is converted to "null" in string concatenation).

---

**Q17:**  
**How would you reverse only the words in a sentence, not the whole string?**  
**Explanation:**  
- Split by spaces, reverse array, join back.

---

**Q18:**  
**Java one-liner to count vowels in a String using streams?**  
**Explanation:**  
```java
long count = str.chars().filter(ch -> "aeiouAEIOU".indexOf(ch) >= 0).count();
```

---

**Q19:**  
```java
String str = "hello";
str.toUpperCase();
System.out.println(str);
```
**Explain the output.**  
**Explanation:**  
- Prints `hello` (toUpperCase() returns a new string, does not modify original).

---

**Q20:**  
```java
String s = "abc";
s.concat("def");
System.out.println(s);
```
**Why does this not print "abcdef"?**  
**Explanation:**  
- String is immutable. `concat` returns a new string, but the result is not assigned; `s` remains unchanged.

---

> **Tip:** In interviews, always justify answers with details about memory, performance, and Java's design principles.