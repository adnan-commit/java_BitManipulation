# Bit Manipulation — Interview Cheat Sheet (Java)

A practical revision guide for **easy-to-medium LeetCode bit-manipulation problems**. It covers the core ideas, formulas, reusable Java snippets, common patterns, and mistakes that frequently cause bugs.

> **How to use this file:** First learn the operator meanings, then memorize the core formulas, and finally practice the pattern table near the end. In Java, use `1L << bit` when a mask may involve bit 31 or any bit in a `long`.

---

## 1. What is a bit?

A bit is a binary digit: `0` or `1`. Integer values are represented using binary place values.

For example, `13` is `1101₂`:

| Bit position (from right, starting at 0) | 3 | 2 | 1 | 0 |
|---|---:|---:|---:|---:|
| Bit value | 1 | 1 | 0 | 1 |
| Place value | 8 | 4 | 2 | 1 |

So `13 = 8 + 4 + 1 = 2³ + 2² + 2⁰`.

- **Least significant bit (LSB):** rightmost bit, position `0`.
- **Most significant bit (MSB):** leftmost bit in the representation.
- **Set bit:** a bit equal to `1`.
- **Unset/clear bit:** a bit equal to `0`.
- Bit positions are normally **zero-indexed from the right** in programming problems.

### Powers of two

`1, 2, 4, 8, 16, 32, ...` have exactly one set bit:

```text
1  = 0001
2  = 0010
4  = 0100
8  = 1000
```

For a non-negative integer, a power of two has the form `2^k` and its binary representation contains one `1`.

---

## 2. Bitwise operators

These operators work on the individual bits of integer values.

| Operator | Name | Rule | Example |
|---|---|---|---|
| `&` | AND | Result bit is `1` only if both bits are `1` | `1100 & 1010 = 1000` |
| `\|` | OR | Result bit is `1` if either bit is `1` | `1100 \| 1010 = 1110` |
| `^` | XOR | Result bit is `1` if bits differ | `1100 ^ 1010 = 0110` |
| `~` | NOT / complement | Flips every bit in the fixed-width integer representation | `~0000...0000 = 1111...1111` |
| `<<` | Left shift | Shifts bits left; zeros enter on the right | `0011 << 1 = 0110` |
| `>>` | Signed right shift | Shifts right and copies the sign bit on the left | depends on sign |
| `>>>` | Unsigned right shift (Java) | Shifts right and fills the left with zeros | depends on value |

### Truth table

| `a` | `b` | `a & b` | `a \| b` | `a ^ b` |
|---:|---:|---:|---:|---:|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 1 | 1 |
| 1 | 0 | 0 | 1 | 1 |
| 1 | 1 | 1 | 1 | 0 |

### The most important XOR rules

```text
a ^ a = 0        // equal values cancel
 a ^ 0 = a       // zero changes nothing
 a ^ b = b ^ a   // commutative
(a ^ b) ^ c = a ^ (b ^ c) // associative
```

Consequences:

- XOR of a value with itself is `0`.
- XOR of any number with `0` is the original number.
- Order does not matter when XORing a collection of numbers.
- If all values occur an even number of times, their total XOR is `0`.
- XOR can recover a value when all other values cancel in pairs.

### Quick examples in Java

```java
int a = 12; // 1100
int b = 10; // 1010

System.out.println(a & b);  // 8  (1000)
System.out.println(a | b);  // 14 (1110)
System.out.println(a ^ b);  // 6  (0110)
System.out.println(~a);     // -13 for a Java int
```

`~12` is `-13`, not `3`, because Java's `int` is a signed 32-bit two's-complement value. `~x == -x - 1` for Java `int`/`long` values.

---

## 3. Shifts and powers of two

For ordinary values where no relevant bits are discarded:

```java
n << k   // shifts left by k positions; often equivalent to n * 2^k
n >> k   // signed right shift; for non-negative n, equivalent to floor(n / 2^k)
n >>> k  // logical right shift, fills with zeros
```

**Cautions:**

- Left shifts can discard high bits and overflow the fixed-width type.
- For negative values, `>>` rounds toward negative infinity in terms of the shifted two's-complement representation; do not treat it as ordinary division in every case.
- `>>>` differs from `>>` when the sign bit is `1`.
- In Java, `1 << k` is an `int` expression. For an `int`, Java uses only the low 5 bits of the shift distance (`k & 31`); for a `long`, it uses the low 6 bits (`k & 63`). Thus `1 << 32` behaves like `1 << 0`.
- Use `1L << k` for long masks. A shift does not automatically promote an `int` literal to `long`.

### Create a bit mask

```java
int mask = 1 << i;     // bit i for positions 0..31; bit 31 gives a negative int mask
long mask64 = 1L << i; // bit i in a 64-bit long mask
```

To check whether `n` is divisible by `2^k` for non-negative `n`, its lowest `k` bits must be zero:

```java
(n & ((1 << k) - 1)) == 0
```

Use a `long` mask when the shift may reach the type's width, and avoid shifting by the full width.

---

## 4. The four must-know bit-position operations

Let bit position `i` be zero-indexed from the right. Use `int mask = 1 << i` for ordinary `int` positions, or `long mask = 1L << i` for `long` values / wider positions.

| Task | Formula | Meaning |
|---|---|---|
| **Check/test** bit `i` | `(n & mask) != 0` | Is bit `i` set? |
| **Set** bit `i` | `n \| mask` | Force bit `i` to `1` |
| **Clear** bit `i` | `n & ~mask` | Force bit `i` to `0` |
| **Toggle/flip** bit `i` | `n ^ mask` | Change `0→1` and `1→0` |

Example:

```java
int n = 10; // 1010
int i = 2;
int mask = 1 << i; // 0100

boolean isSet = (n & mask) != 0; // false
int set = n | mask;             // 1110 = 14
int clear = n & ~mask;          // 1010 = 10 (bit 2 was already 0)
int toggle = n ^ mask;          // 1110 = 14
```

### Test a bit without an `if`

```java
int bit = (n >> i) & 1; // gives 0 or 1 for the selected bit
```

For bit extraction from signed values, `((n >>> i) & 1)` makes the intent especially clear.

---

## 5. Mask formulas worth memorizing

Assume `i >= 0`; use `long` forms when a shift could overflow or reach high positions.

### Keep or clear groups of bits

```java
long lowIBitsMask = (1L << i) - 1; // i low bits are 1: ...000111...111
long keepLowI = n & lowIBitsMask;  // keep only the lowest i bits
long clearLowI = n & ~lowIBitsMask; // clear the lowest i bits
```

Examples for `i = 4`:

```text
lowIBitsMask = 0000...1111
keepLowI     = n & 0000...1111
clearLowI    = n & 1111...0000
```

### Keep bits from position `l` through `r` (inclusive)

For a valid range within the width of the integer type:

```java
int width = r - l + 1;
long rangeMask = ((1L << width) - 1) << l;
long selectedBits = n & rangeMask;
```

The expression above needs special handling if `width == 64`; Java masks shift distances, so do not use `1L << 64` to construct a 64-bit-wide mask. For a 32-bit range with width `32`, use a carefully typed full-width mask or special-case it as well.

### Build a mask with the lowest `k` bits set

```java
long mask = (1L << k) - 1; // valid for 0 <= k < 64
```

### Is a bit set?

```java
boolean set = ((n >>> i) & 1) == 1;
```

### Extract a bit field

To read `width` bits starting at `l`:

```java
long field = (n >>> l) & ((1L << width) - 1);
```

Handle `width == 64` separately; the expression shown assumes `width < 64`.

---

## 6. The three most famous formulas

### A. Check whether `n` is a power of two

```java
boolean isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}
```

Why it works: a positive power of two has exactly one set bit. Subtracting one turns that bit off and turns all lower bits on, so ANDing the two values gives zero.

```text
n       = 010000
n - 1   = 001111
AND     = 000000
```

The `n > 0` guard is essential: `0` and negative values should not count as powers of two.

### B. Remove the lowest set bit

```java
n = n & (n - 1);
```

Example:

```text
n       = 101100
n - 1   = 101011
AND     = 101000
```

The lowest `1` disappears. This is useful for counting set bits and checking power-of-two patterns.

### C. Isolate the lowest set bit

```java
int lowest = n & -n;
```

Example:

```text
n       = 101100
-n      = two's-complement negation
n & -n  = 000100
```

This returns a value with only the lowest set bit retained. In Java, `Integer.MIN_VALUE` is a special overflow case for negation, but `n & -n` still produces that same sign-bit value.

### Related identities

```text
n & (n - 1)   // clear lowest set bit
n & -n        // isolate lowest set bit
n | (n - 1)   // set all bits at and below the lowest set bit to 1 (interpret with care)
~n            // flip every bit
-n            // two's-complement negation; same bit pattern as ~n + 1
```

---

## 7. Count set bits (population count)

A set bit is a bit equal to `1`. The number of set bits is also called **popcount** or **Hamming weight**.

### Best built-in Java method

```java
int count = Integer.bitCount(n);
long countLong = Long.bitCount(value);
```

### Brian Kernighan's algorithm

```java
int countSetBits(int n) {
    int count = 0;
    while (n != 0) {
        n &= (n - 1); // remove one set bit
        count++;
    }
    return count;
}
```

Time: `O(number of set bits)` iterations; for a fixed 32-bit `int`, it is bounded by 32 iterations. The loop also counts all set bits in a negative `int`'s 32-bit representation, but many problems only ask about non-negative inputs.

### Check odd or even

The final binary bit decides parity:

```java
boolean isOdd = (n & 1) == 1;
boolean isEven = (n & 1) == 0;
```

For negative Java integers, the same low-bit parity test works.

### Count bits for every value from `0` to `n` (LeetCode 338)

**DP formula 1 — shift recurrence:**

```java
int[] countBits(int n) {
    int[] dp = new int[n + 1];
    for (int i = 1; i <= n; i++) {
        dp[i] = dp[i >> 1] + (i & 1);
    }
    return dp;
}
```

Because right shift removes the last bit, and `i & 1` tells whether that removed bit was `1`.

**DP formula 2 — clear lowest bit:**

```java
dp[i] = dp[i & (i - 1)] + 1;
```

This is valid because `i & (i - 1)` removes one set bit.

**DP formula 3 — highest power of two:**

If `offset` is the greatest power of two not greater than `i`, then:

```java
dp[i] = dp[i - offset] + 1;
```

When `i` reaches a new power of two, update `offset = i`.

---

## 8. XOR patterns used in LeetCode

### A. Find the missing number (values `0..n`)

XOR all indices and all values. Matching values cancel, leaving the missing number.

```java
int missingNumber(int[] nums) {
    int ans = nums.length;
    for (int i = 0; i < nums.length; i++) {
        ans ^= i;
        ans ^= nums[i];
    }
    return ans;
}
```

Equivalent alternative: XOR every value from `0` through `n`, then XOR each array value.

### B. Single Number — every other value appears twice (LeetCode 136)

```java
int singleNumber(int[] nums) {
    int ans = 0;
    for (int x : nums) ans ^= x;
    return ans;
}
```

All pairs cancel. Time `O(n)`, extra space `O(1)`.

### C. Two values appear once; all others appear twice (LeetCode 260)

First XOR of all values is `x ^ y`, where `x` and `y` are the two unique values. Any set bit in that XOR means their bits differ. Split the values by one such bit.

```java
int[] singleNumber(int[] nums) {
    int xor = 0;
    for (int x : nums) xor ^= x;

    int distinguishingBit = xor & -xor; // isolate one set bit in x ^ y
    int a = 0, b = 0;

    for (int x : nums) {
        if ((x & distinguishingBit) == 0) a ^= x;
        else b ^= x;
    }
    return new int[]{a, b};
}
```

### D. Single value appears once; every other value appears three times (LeetCode 137)

**Approach 1: count each of the 32 bit positions modulo 3.** This is usually the clearest interview solution.

```java
int singleNumber(int[] nums) {
    int answer = 0;
    for (int bit = 0; bit < 32; bit++) {
        int count = 0;
        for (int x : nums) {
            count += (x >>> bit) & 1;
        }
        if (count % 3 != 0) {
            answer |= (1 << bit);
        }
    }
    return answer;
}
```

This handles negative answers too, because bit 31 is included. Time `O(32n)`, extra space `O(1)`.

**Approach 2: state-machine trick** (more advanced): maintain masks `ones` and `twos` representing bits seen once and twice; repeated-thrice bits are cleared. Learn the counting method first because it is easier to explain and verify.

### E. XOR of a range / prefix XOR

Define `prefix[i] = nums[0] ^ nums[1] ^ ... ^ nums[i]`.

```text
XOR(l..r) = prefix[r] ^ prefix[l - 1]  when l > 0
XOR(0..r) = prefix[r]
```

Why? XORing the prefix twice cancels the shared earlier values.

### F. XOR from `0` to `n` in `O(1)`

```java
int xor0ToN(int n) {
    switch (n & 3) {
        case 0: return n;
        case 1: return 1;
        case 2: return n + 1;
        default: return 0;
    }
}
```

Pattern based on `n % 4`:

| `n % 4` | `0 ^ 1 ^ ... ^ n` |
|---:|---|
| 0 | `n` |
| 1 | `1` |
| 2 | `n + 1` |
| 3 | `0` |

For LeetCode Java, a direct `if/else` version is equally good.

---

## 9. Two's complement and negative numbers

Java's `byte`, `short`, `int`, and `long` use two's-complement signed representations; `int` is 32 bits and `long` is 64 bits.

Two's-complement negation follows:

```text
-x = ~x + 1
~x = -x - 1
```

Example with 8-bit illustration only:

```text
 5  = 00000101
~5  = 11111010   // -6 in 8-bit two's complement
-5  = 11111011   // ~5 + 1
```

Java `int` range is `-2^31` through `2^31 - 1`. `Integer.MIN_VALUE` has no positive counterpart in `int`, so `-Integer.MIN_VALUE == Integer.MIN_VALUE` due to overflow. Analogously, `Long.MIN_VALUE` cannot be negated into a positive `long`.

### Signed versus unsigned right shift

```java
int n = -8;
System.out.println(n >> 1);  // -4 (sign bit copied)
System.out.println(n >>> 1); // 2147483644 (zeros shifted in)
```

### Binary display helpers

```java
System.out.println(Integer.toBinaryString(13));  // "1101"
System.out.println(Integer.toBinaryString(-1));  // 32 ones
System.out.println(Long.toBinaryString(value));
```

`Integer.toBinaryString()` does not pad positive values to 32 characters; it shows a compact representation for non-negative values and the full two's-complement representation for negative values.

---

## 10. Reverse bits and bit-position helpers

### Reverse all 32 bits (LeetCode 190)

A reliable bit-by-bit solution:

```java
int reverseBits(int n) {
    int answer = 0;
    for (int i = 0; i < 32; i++) {
        answer <<= 1;
        answer |= (n & 1);
        n >>>= 1;
    }
    return answer;
}
```

Key idea: take the current lowest bit of `n`, append it to `answer`, then shift `n` right logically. Use exactly 32 iterations because the problem reverses a fixed 32-bit pattern, including leading zeros.

### Check whether a number is a power of four

A positive power of four is a power of two with its set bit at an even zero-indexed position (`0, 2, 4, ...`).

```java
boolean isPowerOfFour(int n) {
    return n > 0
        && (n & (n - 1)) == 0
        && (n & 0x55555555) != 0;
}
```

`0x55555555` has `1` bits in even positions: `0101...0101`. The first condition excludes zero/negative numbers, and the second requires exactly one set bit.

### Trailing zeros and leading zeros

```java
int trailing = Integer.numberOfTrailingZeros(n);
int leading = Integer.numberOfLeadingZeros(n);
```

For `n == 0`, both methods return `32` for `int` (and `64` for `long`). Check whether zero is possible before using them as bit positions.

### Isolate the highest set bit (positive `n`)

```java
int highestPowerOfTwo = Integer.highestOneBit(n);
long highestPowerOfTwoLong = Long.highestOneBit(value);
```

These built-ins are often preferable to hand-written loops.

---

## 11. Hamming distance

The Hamming distance between two integers is the number of bit positions where their bits differ.

Formula:

```java
Integer.bitCount(x ^ y)
```

Why it works: XOR is `1` exactly at positions where the bits differ.

```java
int hammingDistance(int x, int y) {
    return Integer.bitCount(x ^ y);
}
```

---

## 12. Add two integers using bit operations (LeetCode 371)

For each bit:

- `a ^ b` gives the sum without carrying.
- `(a & b) << 1` gives the carry bits, shifted to the next position.
- Repeat until there is no carry.

```java
int getSum(int a, int b) {
    while (b != 0) {
        int sumWithoutCarry = a ^ b;
        int carry = (a & b) << 1;
        a = sumWithoutCarry;
        b = carry;
    }
    return a;
}
```

This uses Java's fixed-width overflow behavior. It is important to understand why the loop eventually stops for normal 32-bit Java integer arithmetic.

### Other arithmetic identities

```text
n * 2^k  ≈ n << k      // only when overflow/range effects are acceptable
n / 2^k  ≈ n >> k      // directly safe as an ordinary division shortcut for non-negative n
-n       = ~n + 1
```

For signed values, use normal arithmetic unless the bitwise behavior is specifically required; shifts and overflow can change the result.

---

## 13. Bitwise AND of a number range (LeetCode 201)

Find `left & (left + 1) & ... & right`.

**Trick:** the common leading binary prefix of `left` and `right` is the result. When right shifts make them equal, restore the shifts with zeros at the end.

```java
int rangeBitwiseAnd(int left, int right) {
    int shifts = 0;
    while (left != right) {
        left >>= 1;
        right >>= 1;
        shifts++;
    }
    return left << shifts;
}
```

This is intended for the problem's non-negative range constraints. Each increment in a range often causes lower bits to vary through both `0` and `1`, making the AND of those positions zero.

---

## 14. Bitmasking for subsets (important medium-level pattern)

For an array of `n` elements, each subset corresponds to an `n`-bit mask:

- bit `i = 1`: include `nums[i]`
- bit `i = 0`: exclude `nums[i]`

There are `2^n` subsets, with masks from `0` to `(1 << n) - 1` (for small `n`; use long masks / careful bounds if `n` approaches 31 or more).

### Generate every subset

```java
List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    int n = nums.length;

    for (int mask = 0; mask < (1 << n); mask++) {
        List<Integer> subset = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) != 0) {
                subset.add(nums[i]);
            }
        }
        result.add(subset);
    }
    return result;
}
```

Imports: `java.util.ArrayList` and `java.util.List` (or `java.util.*`). This is an iterative alternative to recursion/backtracking for subsets.

Time is `O(n · 2^n)` because each of the `2^n` masks checks `n` positions.

### Enumerate all submasks of a mask

```java
for (int sub = mask; ; sub = (sub - 1) & mask) {
    // process sub; this includes mask itself
    if (sub == 0) break; // process zero too, then stop
}
```

The shorter common form skips zero:

```java
for (int sub = mask; sub > 0; sub = (sub - 1) & mask) {
    // process each non-empty submask
}
// process submask 0 separately if needed
```

This visits every submask of `mask` in descending numeric order. Do not write the loop condition incorrectly; `(sub - 1) & mask` eventually becomes zero.

### Enumerate set-bit positions only

```java
for (int bits = mask; bits != 0; bits &= (bits - 1)) {
    int lowestBit = bits & -bits;
    int index = Integer.numberOfTrailingZeros(lowestBit);
    // process position index
}
```

This is often faster and cleaner than checking every position if the mask has relatively few set bits.

---

## 15. Gray code

A Gray code sequence changes exactly one bit between adjacent values.

The `i`-th binary-reflected Gray code value is:

```java
int gray = i ^ (i >> 1);
```

Example for `i = 0..3`:

```text
Binary i   Gray = i ^ (i >> 1)
000        000
001        001
010        011
011        010
```

This formula appears in problems asking for Gray code sequences or bit-state ordering.

---

## 16. More useful patterns

### A. Check if two numbers have opposite signs

For two's-complement integers, their sign bits differ if `(x ^ y)` is negative. This can be useful, but use direct comparisons (`(x < 0) != (y < 0)`) when the task is simply to check signs; it is clearer.

### B. Check whether `n` is a power of two without a loop

```java
n > 0 && (n & (n - 1)) == 0
```

### C. Lowest set bit index

```java
int index = Integer.numberOfTrailingZeros(n); // only if n != 0
```

### D. Remove the lowest set bit repeatedly

```java
while (n != 0) {
    // use n's current state
    n &= n - 1;
}
```

This is the basis of Kernighan's count-bits algorithm.

### E. Check if an integer has an even number of set bits

```java
boolean evenParity = (Integer.bitCount(n) % 2 == 0);
```

A parity bit is the XOR of all bits: `1` means an odd number of set bits; `0` means even.

### F. Convert a one-bit mask to an index

```java
int index = Integer.numberOfTrailingZeros(mask); // mask must be nonzero and have one set bit
```

### G. Set the lowest `k` bits

```java
int mask = (1 << k) - 1; // for 0 <= k < 31; use long and handle width boundaries for larger k
```

### H. Check whether `n` is divisible by 2

```java
(n & 1) == 0
```

### I. Find the complement of a positive number

To flip only bits up to the highest set bit (rather than all 32 bits):

```java
int bitLength = 32 - Integer.numberOfLeadingZeros(n); // n > 0
int mask = (1 << bitLength) - 1; // assumes bitLength < 32; handle the full-width case separately
int complement = (~n) & mask;
```

Why the mask matters: `~n` flips all 32 bits of an `int`, including leading zeros. Many “number complement” problems define the complement only across the significant bits.

For `n == 0`, define the behavior according to the problem statement because there are no set bits to determine a significant width.

---

## 17. Java bit-manipulation built-ins

| Method | Purpose |
|---|---|
| `Integer.bitCount(n)` | Number of `1` bits in a 32-bit `int` |
| `Long.bitCount(n)` | Number of `1` bits in a 64-bit `long` |
| `Integer.toBinaryString(n)` | Binary representation as a string |
| `Long.toBinaryString(n)` | Binary representation of a `long` as a string |
| `Integer.numberOfTrailingZeros(n)` | Number of zero bits after the lowest `1`; returns `32` for `0` |
| `Integer.numberOfLeadingZeros(n)` | Number of leading zero bits; returns `32` for `0` |
| `Integer.highestOneBit(n)` | Value containing only the highest set bit |
| `Integer.lowestOneBit(n)` | Value containing only the lowest set bit |
| `Integer.reverse(n)` | Reverses the order of all 32 bits |
| `Integer.reverseBytes(n)` | Reverses the order of the 4 bytes, not every bit |
| `Integer.rotateLeft(n, distance)` | Rotates bits left (bits shifted out on the left re-enter on the right) |
| `Integer.rotateRight(n, distance)` | Rotates bits right |

Examples:

```java
Integer.bitCount(13);                  // 3
Integer.numberOfTrailingZeros(8);      // 3
Integer.lowestOneBit(12);              // 4
Integer.highestOneBit(12);             // 8
Integer.reverse(1);                    // 1 with the single bit moved to position 31
```

Built-ins are accepted in normal LeetCode solutions unless the problem explicitly asks you to implement the operation yourself.

---

## 18. Common LeetCode questions and their key trick

| Problem / pattern | Key observation / formula |
|---|---|
| **136. Single Number** | XOR every element; pairs cancel |
| **190. Reverse Bits** | Process exactly 32 bits; append `n & 1`, then use `n >>>= 1` |
| **191. Number of 1 Bits** | `Integer.bitCount(n)` or repeatedly `n &= n - 1` |
| **201. Bitwise AND of Numbers Range** | Find the common leading prefix by shifting both boundaries right |
| **231. Power of Two** | `n > 0 && (n & (n - 1)) == 0` |
| **260. Single Number III** | XOR all; split into groups using `xor & -xor` |
| **268. Missing Number** | XOR indices and values, or use arithmetic sum with overflow awareness |
| **338. Counting Bits** | `dp[i] = dp[i >> 1] + (i & 1)` |
| **342. Power of Four** | Power-of-two test plus mask `0x55555555` |
| **371. Sum of Two Integers** | Sum without carry: `a ^ b`; carry: `(a & b) << 1` |
| **461. Hamming Distance** | `Integer.bitCount(x ^ y)` |
| **476. Number Complement** | Flip only the significant bits using a mask |
| **78. Subsets** | Enumerate masks from `0` to `(1 << n) - 1` |
| **89. Gray Code** | `i ^ (i >> 1)` |
| **137. Single Number II** | Count each bit modulo three, or use a bit-state machine |

Always check the exact constraints and expected handling of negative numbers in the specific problem.

---

## 19. Complexity reminders

| Technique | Time | Extra space |
|---|---:|---:|
| Test/set/clear/toggle one bit | `O(1)` | `O(1)` |
| `Integer.bitCount(n)` | Constant for fixed-width `int` | `O(1)` |
| Kernighan's set-bit count | `O(number of set bits)` | `O(1)` |
| Check all 32 bit positions | `O(32)` = `O(1)` for `int` | `O(1)` |
| Subset enumeration | `O(n · 2^n)` | Depends on whether subsets are stored |
| Enumerate submasks | `O(2^k)` for a mask with `k` set bits | `O(1)` beyond processing |
| Reverse a fixed-width integer | `O(32)` = `O(1)` for `int` | `O(1)` |

---

## 20. Common mistakes to avoid

1. **Using `^` for exponentiation.** In Java, `^` means XOR, not power. Use `1 << k` for a small integer power of two, or `Math.pow()` when a floating-point result is actually wanted.
2. **Forgetting bit positions start at zero.** The lowest bit is position `0`.
3. **Using `1 << 31` and assuming it is positive.** It is a valid sign-bit mask, but the resulting `int` is negative. Use `1L << i` for a long mask.
4. **Using `1 << 32`.** Java masks the shift count, so it behaves like `1 << 0`. Never shift by the type width to create a full-width mask.
5. **Confusing `>>` with `>>>`.** `>>` preserves the sign; `>>>` fills with zeros.
6. **Assuming `~n` is a positive complement.** It flips all 32 or 64 bits of the type. Mask it when the problem defines a smaller logical width.
7. **Forgetting to guard power-of-two checks.** Use `n > 0 && (n & (n - 1)) == 0`.
8. **Using `n & -n` on zero and expecting a bit position.** It returns `0`; trailing-zero methods return the type width for zero.
9. **Ignoring overflow.** Bitwise operations keep working on fixed-width bit patterns, but arithmetic expressions may overflow. Read the constraints.
10. **Using `int` for masks that need high positions.** `int` has only 32 bits; choose `long` for 64-bit values.
11. **Enumerating subsets with a shift that overflows.** `1 << n` is appropriate only when `n` is small. For medium subset problems, constraints are typically around `n <= 20`; inspect them.
12. **Treating leading zeros as real subset bits in a complement problem.** Match the problem's defined width.
13. **Forgetting duplicates' frequency assumptions.** XOR directly solves the “pairs plus one unique value” pattern, but not every duplicate-frequency problem.
14. **Not using parentheses.** Write `(n & mask) != 0`, not ambiguous expressions. Parentheses make intent clear and prevent precedence bugs.

---

## 21. Interview cheat sheet — memorize this table

| Goal | Formula |
|---|---|
| Test bit `i` | `(n & (1 << i)) != 0` |
| Get bit `i` as `0`/`1` | `(n >>> i) & 1` |
| Set bit `i` | `n \| (1 << i)` |
| Clear bit `i` | `n & ~(1 << i)` |
| Toggle bit `i` | `n ^ (1 << i)` |
| Check odd | `(n & 1) == 1` |
| Check even | `(n & 1) == 0` |
| Power of two | `n > 0 && (n & (n - 1)) == 0` |
| Clear lowest set bit | `n & (n - 1)` |
| Isolate lowest set bit | `n & -n` |
| Count set bits | `Integer.bitCount(n)` |
| Lowest set-bit index | `Integer.numberOfTrailingZeros(n)` if `n != 0` |
| Highest set bit value | `Integer.highestOneBit(n)` |
| Two values differ in bits | `x ^ y` |
| Hamming distance | `Integer.bitCount(x ^ y)` |
| Unique value among pairs | XOR all elements |
| Unique value when others appear 3 times | Count each bit modulo 3 |
| Missing value in `0..n` | XOR indices and values |
| Add without `+` | `sum = a ^ b`, `carry = (a & b) << 1`, repeat |
| Reverse 32 bits | Move low bit to answer in 32 iterations |
| `i`-th Gray code | `i ^ (i >> 1)` |
| All subsets | Iterate masks `0 .. (1 << n) - 1` |

**Safer long-mask versions:**

```java
long mask = 1L << i;
boolean set = (n & mask) != 0;
n |= mask;       // set
n &= ~mask;      // clear
n ^= mask;       // toggle
```

Make sure `n` is also a `long` if you intend to operate on all 64 bits.

---

## 22. A practical approach to solving a bit-manipulation problem

When reading a problem, ask these questions in order:

1. **Is it about parity, powers of two, or counting bits?** Try `n & 1`, `n & (n - 1)`, or `Integer.bitCount()`.
2. **Does the problem say values appear in pairs?** Consider XOR cancellation.
3. **Must one specific bit be checked or changed?** Build a mask with `1 << i`, then use `&`, `|`, `& ~`, or `^`.
4. **Are subsets involved and `n` is small?** Represent each subset with a bitmask.
5. **Are you combining ranges of numbers?** Look for common prefixes, especially in range AND problems.
6. **Is the input negative or a fixed 32-bit/64-bit pattern?** Think about two's complement, the sign bit, and `>>>`.
7. **Could shifts or masks overflow?** Confirm the type width, constraints, and shift distance.

### Suggested practice order

1. Check odd/even and power of two.
2. Count set bits and remove the lowest set bit.
3. Single Number, Missing Number, and Hamming Distance.
4. Reverse Bits and Counting Bits.
5. Number Complement and Power of Four.
6. Two Single Numbers and Single Number II.
7. Bitwise AND of Numbers Range.
8. Subsets with bitmasks and Gray Code.
9. Sum of Two Integers.

The goal is not to memorize every solution. Learn the small number of identities above, recognize when they apply, and practice explaining *why* they work.

---

## 23. One-minute final revision

```text
AND  (&)  -> keep bits that are 1 in both
OR   (|)  -> set bits present in either
XOR  (^)  -> different bits; equal values cancel
NOT  (~)  -> flip every bit in the fixed-width type

Check bit i:    (n & (1 << i)) != 0
Set bit i:      n | (1 << i)
Clear bit i:    n & ~(1 << i)
Toggle bit i:   n ^ (1 << i)

Clear lowest 1: n & (n - 1)
Isolate low 1:  n & -n
Power of 2:     n > 0 && (n & (n - 1)) == 0
Count bits:     Integer.bitCount(n)
Odd/even:       n & 1
Unique among pairs: XOR all values

Java: use 1L << i for long masks; >> keeps sign, >>> fills with 0.
```

Happy practicing — do a small binary dry run whenever a formula feels mysterious.
