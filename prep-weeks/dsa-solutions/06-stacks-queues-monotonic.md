# DSA Solutions — Stacks, Queues & Monotonic Stack

> Full worked solutions in Java 17. Each problem gives a self-contained statement, the approach, a complete compilable solution, complexity, and a key insight or follow-up. The recurring theme here is the **monotonic stack/deque** — a structure that keeps elements in sorted order so each "next greater / previous smaller" query is amortized O(1).

---

### LC20 — Valid Parentheses *(easy)*
**Problem:** Given a string `s` containing only the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid. A string is valid when every open bracket is closed by the same type of bracket, brackets close in the correct order, and every close bracket has a matching open bracket. Example: `"()[]{}"` → `true`, `"(]"` → `false`, `"([)]"` → `false`.

**Approach:** Push each opening bracket onto a stack. On a closing bracket, the top of the stack must be its matching opener — otherwise (mismatch or empty stack) the string is invalid. At the end the stack must be empty.
```java
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.Map;

class Solution {
    public boolean isValid(String s) {
        Map<Character, Character> pairs = Map.of(')', '(', ']', '[', '}', '{');
        Deque<Character> stack = new ArrayDeque<>();
        for (char c : s.toCharArray()) {
            if (c == '(' || c == '[' || c == '{') {
                stack.push(c);
            } else {
                if (stack.isEmpty() || stack.pop() != pairs.get(c)) {
                    return false;
                }
            }
        }
        return stack.isEmpty();
    }
}
```
**Complexity:** O(n) time, O(n) space (worst case all openers on the stack).

**Key insight / follow-up:** The stack models the "most recently opened, first to be closed" (LIFO) nesting rule directly. Use `ArrayDeque`, not the legacy `Stack` class (synchronized, slower, and iterates in the wrong order). Forgetting the final `stack.isEmpty()` check is the classic bug — it wrongly accepts `"((("`.

---

### LC155 — Min Stack *(medium)*
**Problem:** Design a stack that supports `push`, `pop`, `top`, and retrieving the minimum element, all in constant time. Implement: `MinStack()` constructor, `void push(int val)`, `void pop()`, `int top()`, and `int getMin()`.

**Approach:** Keep a second "min stack" in lockstep with the main stack. Each entry records the minimum of everything at or below it, so the current minimum is always the top of the min stack. Push/pop both stacks together.
```java
import java.util.ArrayDeque;
import java.util.Deque;

class MinStack {
    private final Deque<Integer> stack = new ArrayDeque<>();
    private final Deque<Integer> mins = new ArrayDeque<>();

    public MinStack() {}

    public void push(int val) {
        stack.push(val);
        mins.push(mins.isEmpty() ? val : Math.min(val, mins.peek()));
    }

    public void pop() {
        stack.pop();
        mins.pop();
    }

    public int top() {
        return stack.peek();
    }

    public int getMin() {
        return mins.peek();
    }
}
```
**Complexity:** O(1) for every operation; O(n) extra space for the auxiliary stack.

**Key insight / follow-up:** Storing the running minimum per level avoids recomputation when the current min is popped off. A space optimization stores only *new* minimums on the min stack (push to `mins` only when `val <= mins.peek()`), but the lockstep version is simpler and harder to get wrong. An encoding trick can store deltas in a single stack to use O(1) extra space, at the cost of readability.

---

### LC739 — Daily Temperatures *(medium)*
**Problem:** Given an array `temperatures` where `temperatures[i]` is the temperature on day `i`, return an array `answer` such that `answer[i]` is the number of days you have to wait after day `i` to get a warmer temperature. If there is no future day with a warmer temperature, set `answer[i] = 0`. Example: `[73,74,75,71,69,72,76,73]` → `[1,1,4,2,1,1,0,0]`.

**Approach:** Walk left to right keeping a **monotonic decreasing stack of indices**. When the current temperature is warmer than the temperature at the stack's top index, that earlier day's wait is resolved — pop it and record the index gap. The stack always holds days still waiting for a warmer day.
```java
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        int n = temperatures.length;
        int[] answer = new int[n];
        Deque<Integer> stack = new ArrayDeque<>(); // indices, decreasing temps
        for (int i = 0; i < n; i++) {
            while (!stack.isEmpty() && temperatures[i] > temperatures[stack.peek()]) {
                int prev = stack.pop();
                answer[prev] = i - prev;
            }
            stack.push(i);
        }
        return answer;
    }
}
```
**Complexity:** O(n) time (each index pushed and popped at most once), O(n) space.

**Key insight / follow-up:** This is the canonical "next greater element" pattern: store *indices* (so you can compute the distance) in a monotonic stack. Anything left on the stack at the end has no warmer future day, and its `answer` stays 0 — which is exactly the default value of a fresh `int[]`.

---

### LC496 — Next Greater Element I *(easy)*
**Problem:** You are given two distinct-integer arrays `nums1` and `nums2`, where `nums1` is a subset of `nums2`. For each element `x` in `nums1`, find the **next greater element** to its right in `nums2` (the first element greater than `x` that appears after it). Return an array of these answers; use `-1` when there is no such element. Example: `nums1=[4,1,2]`, `nums2=[1,3,4,2]` → `[-1,3,-1]`.

**Approach:** Precompute the next-greater-element for *every* value in `nums2` using a monotonic decreasing stack, storing results in a map keyed by value (safe because values are distinct). Then look up each element of `nums1`.
```java
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int[] nextGreaterElement(int[] nums1, int[] nums2) {
        Map<Integer, Integer> nextGreater = new HashMap<>();
        Deque<Integer> stack = new ArrayDeque<>(); // values, decreasing
        for (int x : nums2) {
            while (!stack.isEmpty() && x > stack.peek()) {
                nextGreater.put(stack.pop(), x);
            }
            stack.push(x);
        }
        int[] answer = new int[nums1.length];
        for (int i = 0; i < nums1.length; i++) {
            answer[i] = nextGreater.getOrDefault(nums1[i], -1);
        }
        return answer;
    }
}
```
**Complexity:** O(n + m) time where n = `nums2.length`, m = `nums1.length`; O(n) space for the map and stack.

**Key insight / follow-up:** Decouple the heavy work (scan `nums2` once) from the queries (O(1) map lookups). Because elements are distinct, the value itself is a valid map key — store indices instead if duplicates were allowed. Anything still on the stack has no greater element and is simply never added to the map, so `getOrDefault(..., -1)` handles it.

---

### LC503 — Next Greater Element II *(medium)*
**Problem:** Given a **circular** integer array `nums` (the next element of the last element is the first element), return an array where each position holds the next greater number for the corresponding element, searching circularly. If no greater number exists, use `-1`. Example: `[1,2,1]` → `[2,-1,2]` (the last `1` wraps around to find the leading `2`).

**Approach:** Same monotonic-decreasing-index stack, but iterate `2n` times using `i % n` to simulate one wrap-around. Push indices only during the first pass; the second pass just resolves elements still waiting.
```java
import java.util.ArrayDeque;
import java.util.Arrays;
import java.util.Deque;

class Solution {
    public int[] nextGreaterElements(int[] nums) {
        int n = nums.length;
        int[] answer = new int[n];
        Arrays.fill(answer, -1);
        Deque<Integer> stack = new ArrayDeque<>(); // indices, decreasing values
        for (int i = 0; i < 2 * n; i++) {
            int cur = nums[i % n];
            while (!stack.isEmpty() && cur > nums[stack.peek()]) {
                answer[stack.pop()] = cur;
            }
            if (i < n) {
                stack.push(i);
            }
        }
        return answer;
    }
}
```
**Complexity:** O(n) time (still amortized one push/pop per index despite the 2n loop), O(n) space.

**Key insight / follow-up:** The `2n` trick is the standard way to handle circular arrays without physically duplicating the array. Guarding the push with `if (i < n)` prevents re-adding indices on the second lap. Pre-filling `answer` with `-1` cleanly handles the global maximum, which never gets resolved.

---

### LC901 — Online Stock Span *(medium)*
**Problem:** Design an algorithm that collects daily stock prices and, for each day, returns the *span* of that day's price: the number of consecutive days (ending today, going backward) for which the price was less than or equal to today's price. Implement `StockSpanner()` and `int next(int price)` which is called once per day with the current price. Example: prices `100,80,60,70,60,75,85` → spans `1,1,1,2,1,4,6`.

**Approach:** Keep a monotonic decreasing stack of `(price, span)` pairs. When a new price is at least the top price, that day's run is absorbed — pop it and add its span to the current span. The accumulated span is today's answer.
```java
import java.util.ArrayDeque;
import java.util.Deque;

class StockSpanner {
    // each entry: [price, span]
    private final Deque<int[]> stack = new ArrayDeque<>();

    public StockSpanner() {}

    public int next(int price) {
        int span = 1;
        while (!stack.isEmpty() && stack.peek()[0] <= price) {
            span += stack.pop()[1];
        }
        stack.push(new int[]{price, span});
        return span;
    }
}
```
**Complexity:** O(1) amortized per `next` call (each price is pushed and popped at most once), O(n) space over n calls.

**Key insight / follow-up:** Collapsing consumed days' spans into one entry is what keeps it amortized O(1) — a naive backward scan would be O(n) per call, O(n²) overall. This is the "online" (streaming) form of a previous-greater-element problem: you compress already-dominated history into the surviving stack entry.

---

### LC84 — Largest Rectangle in Histogram *(hard)*
**Problem:** Given an array `heights` representing the heights of bars in a histogram (each bar has width 1), find the area of the largest rectangle that can be formed within the histogram. Example: `[2,1,5,6,2,3]` → `10` (the bars of height 5 and 6 form a 5×2 rectangle).

**Approach:** Maintain a monotonic increasing stack of indices. When the current bar is shorter than the bar at the stack top, that taller bar can no longer extend right — pop it and compute the largest rectangle with it as the limiting height. The width spans from just after the new stack top to just before the current index. Append a sentinel height of 0 to flush the stack at the end.
```java
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int largestRectangleArea(int[] heights) {
        int n = heights.length;
        Deque<Integer> stack = new ArrayDeque<>(); // indices, increasing heights
        int best = 0;
        for (int i = 0; i <= n; i++) {
            int curHeight = (i == n) ? 0 : heights[i];
            while (!stack.isEmpty() && heights[stack.peek()] > curHeight) {
                int height = heights[stack.pop()];
                int leftBoundary = stack.isEmpty() ? -1 : stack.peek();
                int width = i - leftBoundary - 1;
                best = Math.max(best, height * width);
            }
            stack.push(i);
        }
        return best;
    }
}
```
**Complexity:** O(n) time (each index pushed/popped once), O(n) space.

**Key insight / follow-up:** When a bar is popped, the new stack top is the nearest *shorter* bar to its left, and the current index is the nearest shorter bar to its right — so `width = i - leftBoundary - 1` is the exact maximal width for that height. The `i == n` sentinel of height 0 forces every remaining bar to be resolved without special end-of-loop code. This solution underpins LC85 (Maximal Rectangle), where each row of a binary matrix becomes a histogram.

---

### LC239 — Sliding Window Maximum *(hard)*
**Problem:** Given an integer array `nums` and a window size `k`, the window slides from the left to the right of the array one position at a time. Return an array of the maximum value in each window. Example: `nums=[1,3,-1,-3,5,3,6,7]`, `k=3` → `[3,3,5,5,6,7]`.

**Approach:** Use a **monotonic decreasing deque of indices**. The front always holds the index of the current window's maximum. For each new element: drop indices that have slid out of the window (front), drop from the back all indices whose values are ≤ the incoming value (they can never be the max while it is present), then add the current index. Once the first full window is formed, record the front's value.
```java
import java.util.ArrayDeque;
import java.util.Deque;

class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        int n = nums.length;
        int[] result = new int[n - k + 1];
        Deque<Integer> deque = new ArrayDeque<>(); // indices, values decreasing front->back
        for (int i = 0; i < n; i++) {
            // 1. evict indices that fell out of the window on the left
            if (!deque.isEmpty() && deque.peekFirst() <= i - k) {
                deque.pollFirst();
            }
            // 2. maintain decreasing order: pop smaller-or-equal tails
            while (!deque.isEmpty() && nums[deque.peekLast()] <= nums[i]) {
                deque.pollLast();
            }
            deque.offerLast(i);
            // 3. window is fully formed once i >= k - 1
            if (i >= k - 1) {
                result[i - k + 1] = nums[deque.peekFirst()];
            }
        }
        return result;
    }
}
```
**Complexity:** O(n) time (each index enters and leaves the deque once), O(k) space for the deque.

**Key insight / follow-up:** A deque used as a monotonic queue gives O(1) amortized max-per-window — far better than a heap's O(n log k). The front is always the window max because every smaller element to its left was already discarded. A max-heap also works but needs lazy deletion of stale indices, making it slower and bulkier.

---

### LC232 — Implement Queue using Stacks *(easy)*
**Problem:** Implement a first-in-first-out (FIFO) queue using only two stacks. The queue should support `push(int x)` (enqueue to the back), `int pop()` (dequeue from the front), `int peek()` (front element), and `boolean empty()`. All operations must use only standard stack operations (push to top, peek/pop from top, size, is-empty).

**Approach:** Use an `in` stack for pushes and an `out` stack for pops/peeks. When `out` is empty and an element is needed, pour the entire `in` stack into `out`, which reverses the order so the oldest element ends up on top — restoring FIFO.
```java
import java.util.ArrayDeque;
import java.util.Deque;

class MyQueue {
    private final Deque<Integer> in = new ArrayDeque<>();
    private final Deque<Integer> out = new ArrayDeque<>();

    public MyQueue() {}

    public void push(int x) {
        in.push(x);
    }

    public int pop() {
        shift();
        return out.pop();
    }

    public int peek() {
        shift();
        return out.peek();
    }

    public boolean empty() {
        return in.isEmpty() && out.isEmpty();
    }

    private void shift() {
        if (out.isEmpty()) {
            while (!in.isEmpty()) {
                out.push(in.pop());
            }
        }
    }
}
```
**Complexity:** `push` is O(1); `pop`/`peek` are O(1) amortized — each element is moved between stacks at most once over its lifetime. O(n) space.

**Key insight / follow-up:** Only transfer when `out` is empty — transferring eagerly on every operation would degrade to O(n) per call. Each element is pushed to `in`, moved to `out`, and popped exactly once, so the amortized cost is constant. The mirror problem (LC225, Stack using Queues) is harder to keep amortized-cheap because a single queue cannot reverse order as freely.

---

*[← DSA bank index](README.md)*
