# Java String Concepts: Detailed Revision Guide

## 1. The String Class

### 1.1. What is a String?
- A String in Java is an object that represents a sequence of characters.
- Defined in `java.lang.String`.
- Strings are **immutable** (cannot be changed once created).

### 1.2. String Creation

```java
// Using string literal (added to string pool)
String s1 = "Hello";

// Using new keyword (creates a new object)
String s2 = new String("Hello");
```
- String literals are stored in the **String Constant Pool**.
- The `new` keyword always creates a new object in the heap, not in the pool.

### 1.3. Immutability of String
- Once a String object is created, its value cannot be changed.
- Any operation that seems to modify a string actually creates a new object.
- Example:
  ```java
  String s = "Java";
  s.concat("Programming");
  // s is still "Java". The result "JavaProgramming" is a new object.
  ```

### 1.4. String Pool (String Constant Pool)
- Special memory region inside the JVM.
- When a string literal is created, JVM checks the pool first.
- If already present, reference is returned; else, a new object is created in the pool.
- Benefits: saves memory, increases performance.

## 2. String Comparison

### 2.1. `==` vs `.equals()`
- `==` compares object references.
- `.equals()` compares the content.
- Example:
  ```java
  String s1 = "test";
  String s2 = "test";
  String s3 = new String("test");
  s1 == s2 // true (same pool object)
  s1 == s3 // false (different object)
  s1.equals(s3) // true (same content)
  ```

### 2.2. `.compareTo()`
- Lexicographical comparison. Returns:
  - `0` if equal
  - `<0` if first string is lexicographically less
  - `>0` if greater

## 3. String Manipulation Methods

- `.length()`
- `.charAt(int index)`
- `.substring(int start, [int end])`
- `.toUpperCase()`, `.toLowerCase()`
- `.trim()`
- `.replace(old, new)`
- `.replaceAll(regex, replacement)`
- `.split(regex)`
- `.indexOf(str)`, `.lastIndexOf(str)`
- `.startsWith(prefix)`, `.endsWith(suffix)`
- `.contains(seq)`
- `.valueOf(primitive)`

## 4. StringBuilder and StringBuffer

### 4.1. Why not String for heavy modifications?
- Strings are immutable; repeated modifications create many intermediate objects, causing memory and performance issues.

### 4.2. StringBuilder (Java 1.5+)
- Mutable sequence of characters.
- Not synchronized (not thread-safe). Faster than StringBuffer.

### 4.3. StringBuffer
- Mutable and thread-safe (synchronized methods).
- Slightly slower than StringBuilder due to synchronization.

### 4.4. Basic Usage
```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World"); // "Hello World"
sb.insert(5, " Java"); // "Hello Java World"
sb.reverse(); // reverses string
sb.delete(0, 6); // deletes from index 0 to 5
```

## 5. String Interning

- `String.intern()` puts the string in the pool if not already present.
- Useful for memory optimization when many duplicate strings are created.

```java
String s1 = new String("abc");
String s2 = s1.intern();
String s3 = "abc";
System.out.println(s2 == s3); // true
```

## 6. Common String Interview Questions

- **Reverse a string**
- **Check palindrome**
- **Remove duplicate characters**
- **Count vowels/consonants**
- **Find all permutations**
- **Anagram check**
- **Longest substring without repeating characters**
- **String compression**

## 7. Regular Expressions with String

- `String.matches(regex)` — checks if string matches the regex.
- `String.replaceAll(regex, replacement)` — replaces all substrings matching regex.
- `String.split(regex)` — splits string using regex.

## 8. String Encoding and Decoding

- Converting between `String` and `byte[]` using encodings:
  ```java
  String s = "hello";
  byte[] bytes = s.getBytes(StandardCharsets.UTF_8);
  String s2 = new String(bytes, StandardCharsets.UTF_8);
  ```

## 9. String vs. char[]

- `String` is immutable, `char[]` is mutable.
- For sensitive data (like passwords), use `char[]` so you can clear it after use.

## 10. String Join, Repeat, and Other Utilities

- `String.join(delimiter, elements...)`
- `String.repeat(n)` (Java 11+)
- `String.isBlank()` (Java 11+)
- `String.strip()`, `stripLeading()`, `stripTrailing()` (Java 11+)

## 11. String Formatting

- `String.format()`
  ```java
  String formatted = String.format("Hello, %s!", "World");
  ```
- Formatting numbers, dates, etc.

## 12. Unicode, Escape Sequences, and Special Characters

- Strings in Java are Unicode.
- Escape sequences: `\n`, `\t`, `\\`, `\"`, etc.
- Unicode escape: `\uXXXX`

## 13. Performance Tips

- Use `StringBuilder` for concatenation inside loops.
- Use string pool and interning for repeated values.
- Avoid `+` for concatenation in loops.

## 14. String-Related Classes

- `StringTokenizer` (legacy, replaced by `split`)
- `Pattern` and `Matcher` for advanced regex use
- `CharSequence` interface (parent of String, StringBuilder, StringBuffer)

## 15. Best Practices

- Prefer `equals()` or `equalsIgnoreCase()` for comparison.
- Use `.isEmpty()` or `.isBlank()` for checking empty strings.
- Use `StringBuilder` for building dynamic strings.
- Be aware of Unicode normalization for internationalization.

---

## Example Questions & Practice

**1. Reverse a String:**
```java
String original = "Java";
String reversed = new StringBuilder(original).reverse().toString();
```

**2. Check Palindrome:**
```java
public boolean isPalindrome(String s) {
    int i = 0, j = s.length() - 1;
    while(i < j) {
        if(s.charAt(i++) != s.charAt(j--)) return false;
    }
    return true;
}
```

**3. Remove Duplicates:**
```java
public String removeDuplicates(String s) {
    StringBuilder sb = new StringBuilder();
    Set<Character> seen = new HashSet<>();
    for(char c : s.toCharArray()) {
        if(seen.add(c)) sb.append(c);
    }
    return sb.toString();
}
```

---

# Quick Reference Table

| Concept                | String        | StringBuilder   | StringBuffer    |
|------------------------|---------------|-----------------|-----------------|
| Mutability             | Immutable     | Mutable         | Mutable         |
| Thread-safe            | Yes           | No              | Yes             |
| Performance            | Slow (modif.) | Fast            | Slower than SB  |
| Use case               | Fixed data    | Single-threaded | Multi-threaded  |

---

# Recommended Further Resources

- [Official Java String Documentation](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html)
- [Baeldung Java Strings Guide](https://www.baeldung.com/java-strings)
- [Java String Pool Explained](https://www.geeksforgeeks.org/string-pool-java/)
- [Effective Java, Item 6: Avoid Creating Unnecessary Objects](https://books.google.com/books/about/Effective_Java.html?id=ka2VUBqHiWkC)

---

This guide covers all core concepts and common interview questions for Java Strings. Practice the examples and ensure you understand the memory and performance implications of each concept, as these are common focus areas in backend interviews.