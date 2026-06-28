# DSA Solutions — Linked Lists

> Full worked solutions in Java 17. Each problem gives a self-contained statement, the approach, a complete compilable solution, complexity, and a key insight or follow-up. Recurring techniques: the **dummy head node** (uniform handling of the front), **two pointers** (fast/slow for cycles and midpoints), and careful **pointer rewiring** in O(1) extra space.

All solutions assume the standard singly-linked list node defined once below.

```java
// Shared definition used by every problem in this file.
class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}
```

---

### LC206 — Reverse Linked List *(easy)*
**Problem:** Given the `head` of a singly linked list, reverse the list and return the new head. Example: `1 → 2 → 3 → 4 → 5` becomes `5 → 4 → 3 → 2 → 1`.

**Approach:** Iterate once, reversing each `next` pointer to point at the previous node. Keep three references: `prev` (already-reversed prefix), `cur` (node being processed), and a saved `next` so the rest of the list isn't lost when the pointer is flipped.
```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode cur = head;
        while (cur != null) {
            ListNode next = cur.next; // save before overwriting
            cur.next = prev;          // reverse the link
            prev = cur;               // advance prev
            cur = next;               // advance cur
        }
        return prev; // prev is the new head
    }
}
```
**Complexity:** O(n) time, O(1) space.

**Key insight / follow-up:** Saving `cur.next` *before* rewiring is the whole trick — overwrite it first and you sever the rest of the list. The recursive version is elegant but uses O(n) stack space: `reverseList(head.next)` then `head.next.next = head; head.next = null;`. Iterative is preferred for long lists to avoid stack overflow.

---

### LC21 — Merge Two Sorted Lists *(easy)*
**Problem:** You are given the heads of two sorted singly linked lists `list1` and `list2`. Merge them into one sorted list by splicing together the existing nodes (no new node values), and return the head of the merged list. Example: `1→2→4` and `1→3→4` → `1→1→2→3→4→4`.

**Approach:** Use a dummy head to avoid special-casing the first node. Walk both lists with a `tail` pointer, always appending the smaller current node, then attach whatever remains of the non-exhausted list.
```java
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode();
        ListNode tail = dummy;
        while (list1 != null && list2 != null) {
            if (list1.val <= list2.val) {
                tail.next = list1;
                list1 = list1.next;
            } else {
                tail.next = list2;
                list2 = list2.next;
            }
            tail = tail.next;
        }
        tail.next = (list1 != null) ? list1 : list2; // append the remainder
        return dummy.next;
    }
}
```
**Complexity:** O(m + n) time, O(1) extra space (nodes are spliced, not copied).

**Key insight / follow-up:** The dummy node turns "set the head" into "advance the tail," eliminating a separate branch for the first element. Because one list is appended wholesale at the end, there is no need to loop through it node by node. This is the merge step of merge sort on lists (LC148) and the pairwise merge inside LC23 (Merge k Sorted Lists).

---

### LC141 — Linked List Cycle *(easy)*
**Problem:** Given the `head` of a linked list, determine whether the list contains a cycle. A cycle exists if some node can be reached again by continuously following `next` pointers. Return `true` if there is a cycle, otherwise `false`.

**Approach:** Floyd's tortoise-and-hare. Advance `slow` by one node and `fast` by two each step. If there is a cycle the fast pointer eventually laps and meets the slow one; if `fast` reaches the end (`null`), there is no cycle.
```java
class Solution {
    public boolean hasCycle(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) {
                return true;
            }
        }
        return false;
    }
}
```
**Complexity:** O(n) time, O(1) space.

**Key insight / follow-up:** Inside a cycle the gap between fast and slow closes by one each step, so they must collide — no infinite loop is possible. This beats the O(n) space hash-set approach (store visited nodes) by using only two pointers. The null checks `fast != null && fast.next != null` guard the two-step jump on even- and odd-length acyclic lists.

---

### LC142 — Linked List Cycle II *(medium)*
**Problem:** Given the `head` of a linked list, return the node where the cycle begins. If there is no cycle, return `null`. Do not modify the list.

**Approach:** Phase 1 — Floyd's algorithm to find a meeting point inside the cycle. Phase 2 — reset one pointer to the head and advance both one step at a time; they meet exactly at the cycle's entry. This works because the distance from head to entry equals the distance from the meeting point to entry (mod cycle length).
```java
class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) {            // cycle confirmed
                ListNode ptr = head;
                while (ptr != slow) {
                    ptr = ptr.next;
                    slow = slow.next;
                }
                return ptr;                // cycle entry
            }
        }
        return null;
    }
}
```
**Complexity:** O(n) time, O(1) space.

**Key insight / follow-up:** Let `L` be the head-to-entry distance, `C` the cycle length, and `k` the entry-to-meeting distance. The math `2(L+k) = L+k+nC` reduces to `L = nC - k`, so walking `L` steps from the head and `L` steps from the meeting point both land on the entry. This pointer-distance identity is the heart of the proof — memorizing it makes the second phase obvious instead of magic.

---

### LC19 — Remove Nth Node From End of List *(medium)*
**Problem:** Given the `head` of a linked list, remove the n-th node counting from the end of the list and return the head. Example: `1→2→3→4→5`, `n=2` → `1→2→3→5`. It is guaranteed that `1 <= n <= length`.

**Approach:** Two pointers with a dummy head. Advance `fast` n nodes ahead, then move `fast` and `slow` together until `fast` reaches the last node. `slow` now sits just before the target, so it can be unlinked in one pass. The dummy head makes removing the actual head a non-special case.
```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0, head);
        ListNode fast = dummy;
        ListNode slow = dummy;
        for (int i = 0; i < n; i++) {  // create an n-node gap
            fast = fast.next;
        }
        while (fast.next != null) {    // move both until fast is last
            fast = fast.next;
            slow = slow.next;
        }
        slow.next = slow.next.next;    // unlink the target
        return dummy.next;
    }
}
```
**Complexity:** O(n) time (single pass), O(1) space.

**Key insight / follow-up:** The fixed n-node gap converts "n-th from the end" into "the node before where fast falls off," achievable in one traversal instead of two (count length, then re-walk). The dummy head is essential: removing the first node (`n == length`) leaves `slow` at the dummy, and `dummy.next` returns the correct new head.

---

### LC143 — Reorder List *(medium)*
**Problem:** Given the head of a singly linked list `L0 → L1 → … → Ln-1 → Ln`, reorder it in place to `L0 → Ln → L1 → Ln-1 → L2 → Ln-2 → …`. You may not modify node values, only rearrange the nodes. Example: `1→2→3→4` → `1→4→2→3`; `1→2→3→4→5` → `1→5→2→4→3`.

**Approach:** Three classic sub-steps: (1) find the middle with fast/slow pointers, (2) reverse the second half, (3) merge the two halves alternately.
```java
class Solution {
    public void reorderList(ListNode head) {
        if (head == null || head.next == null) return;

        // 1. find the middle (slow ends at the start of the 2nd half)
        ListNode slow = head, fast = head;
        while (fast.next != null && fast.next.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }

        // 2. reverse the second half
        ListNode second = slow.next;
        slow.next = null;          // split the list
        ListNode prev = null;
        while (second != null) {
            ListNode next = second.next;
            second.next = prev;
            prev = second;
            second = next;
        }
        second = prev;             // head of reversed second half

        // 3. merge the two halves alternately
        ListNode first = head;
        while (second != null) {
            ListNode n1 = first.next, n2 = second.next;
            first.next = second;
            second.next = n1;
            first = n1;
            second = n2;
        }
    }
}
```
**Complexity:** O(n) time, O(1) space.

**Key insight / follow-up:** Reorder list is a "combo" problem — it composes three standalone techniques (find middle, reverse, merge) you should be able to write blindfolded. Splitting with `slow.next = null` is vital: without it the merged list develops a cycle. The middle-finding condition (`fast.next != null && fast.next.next != null`) keeps the first half ≥ the second half, which makes the alternating merge terminate cleanly.

---

### LC2 — Add Two Numbers *(medium)*
**Problem:** You are given two non-empty linked lists representing two non-negative integers. The digits are stored in **reverse order** (least-significant digit first), one digit per node. Add the two numbers and return the sum as a linked list, also in reverse order. Example: `(2→4→3) + (5→6→4)` represents `342 + 465 = 807`, returned as `7→0→8`.

**Approach:** Walk both lists together (like grade-school addition), tracking a carry. Because digits are already least-significant-first, addition flows naturally front to back. Use a dummy head and continue while either list has digits or a carry remains.
```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode();
        ListNode tail = dummy;
        int carry = 0;
        while (l1 != null || l2 != null || carry != 0) {
            int sum = carry;
            if (l1 != null) { sum += l1.val; l1 = l1.next; }
            if (l2 != null) { sum += l2.val; l2 = l2.next; }
            carry = sum / 10;
            tail.next = new ListNode(sum % 10);
            tail = tail.next;
        }
        return dummy.next;
    }
}
```
**Complexity:** O(max(m, n)) time, O(max(m, n)) space for the result list.

**Key insight / follow-up:** Folding the carry into the loop condition (`|| carry != 0`) elegantly handles a final carry-out, e.g. `5 + 5 = 10` → `0→1`. Reverse-order storage is what makes single-pass addition possible. The follow-up LC445 stores digits most-significant-first; there you either reverse both lists first or push values onto two stacks and build the result back to front.

---

### LC234 — Palindrome Linked List *(easy)*
**Problem:** Given the `head` of a singly linked list, return `true` if the list is a palindrome (reads the same forwards and backwards) and `false` otherwise. Example: `1→2→2→1` → `true`; `1→2→3` → `false`. Aim for O(n) time and O(1) space.

**Approach:** Find the middle (fast/slow), reverse the second half, then compare it node by node against the first half. The values match iff the list is a palindrome. (Optionally restore the list afterward.)
```java
class Solution {
    public boolean isPalindrome(ListNode head) {
        // 1. find middle
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        // 2. reverse second half (starting at slow)
        ListNode prev = null;
        while (slow != null) {
            ListNode next = slow.next;
            slow.next = prev;
            prev = slow;
            slow = next;
        }
        // 3. compare halves
        ListNode left = head, right = prev;
        while (right != null) {        // 2nd half is shorter/equal, so guard on it
            if (left.val != right.val) return false;
            left = left.next;
            right = right.next;
        }
        return true;
    }
}
```
**Complexity:** O(n) time, O(1) space.

**Key insight / follow-up:** Iterating until the (shorter) reversed second half is exhausted neatly handles both odd and even lengths — the odd middle element is never compared and never matters. The simpler O(n)-space approach copies values into an `ArrayList` and two-pointer-compares it; mention it as a fallback, but the in-place reverse is the answer interviewers want. Politely, you can re-reverse the second half before returning to leave the list untouched.

---

### LC25 — Reverse Nodes in k-Group *(hard)*
**Problem:** Given the `head` of a linked list, reverse the nodes k at a time and return the modified list. `k` is a positive integer ≤ the list length. If the number of nodes is not a multiple of k, the leftover nodes at the end stay in their original order. You may not change node values, only the nodes themselves. Example: `1→2→3→4→5`, `k=2` → `2→1→4→3→5`; `k=3` → `3→2→1→4→5`.

**Approach:** Use a dummy head and a `groupPrev` pointer marking the node before each group. For each group, first check that k nodes exist ahead; if so, reverse exactly those k nodes and re-stitch the boundaries; if fewer than k remain, stop.
```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode dummy = new ListNode(0, head);
        ListNode groupPrev = dummy;

        while (true) {
            // verify k nodes remain after groupPrev
            ListNode kth = groupPrev;
            for (int i = 0; i < k && kth != null; i++) {
                kth = kth.next;
            }
            if (kth == null) break;        // fewer than k left -> done

            ListNode groupNext = kth.next; // node after the group
            // reverse the group [groupPrev.next .. kth]
            ListNode prev = groupNext;
            ListNode cur = groupPrev.next;
            while (cur != groupNext) {
                ListNode next = cur.next;
                cur.next = prev;
                prev = cur;
                cur = next;
            }
            // re-stitch: groupPrev.next was the old first node, now the group's tail
            ListNode newGroupPrev = groupPrev.next;
            groupPrev.next = kth;          // kth is now the group's head
            groupPrev = newGroupPrev;      // advance to the tail of this group
        }
        return dummy.next;
    }
}
```
**Complexity:** O(n) time (each node visited a constant number of times), O(1) space.

**Key insight / follow-up:** Initializing `prev = groupNext` makes the reversed group's tail point directly at the next group, so the boundary stitches itself — only the head link (`groupPrev.next = kth`) needs manual fixing. Checking for k nodes *before* reversing is what preserves the "leftover stays as-is" rule. A recursive version reads cleaner but costs O(n/k) stack frames; the iterative form is strictly O(1) space.

---

### LC146 — LRU Cache *(medium)*
**Problem:** Design a data structure for a Least Recently Used (LRU) cache with capacity `capacity`. Implement `LRUCache(int capacity)`, `int get(int key)` (return the value, or `-1` if absent, and mark the key as most recently used), and `void put(int key, int value)` (insert/update, mark as most recently used, and evict the least recently used entry if over capacity). Both operations must run in O(1) average time.

**Approach:** Combine a hash map (key → node, for O(1) lookup) with a **doubly linked list** ordered by recency (most-recent near the head, least-recent near the tail). A `get` or `put` moves the touched node to the front; eviction removes the tail. Sentinel head/tail nodes remove edge cases.
```java
import java.util.HashMap;
import java.util.Map;

class LRUCache {
    private static class Node {
        int key, value;
        Node prev, next;
        Node(int key, int value) { this.key = key; this.value = value; }
    }

    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0); // most-recently-used side
    private final Node tail = new Node(0, 0); // least-recently-used side

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        Node node = map.get(key);
        if (node == null) return -1;
        moveToFront(node);
        return node.value;
    }

    public void put(int key, int value) {
        Node node = map.get(key);
        if (node != null) {
            node.value = value;
            moveToFront(node);
            return;
        }
        if (map.size() == capacity) {
            Node lru = tail.prev;     // evict least-recently-used
            remove(lru);
            map.remove(lru.key);
        }
        Node fresh = new Node(key, value);
        map.put(key, fresh);
        addToFront(fresh);
    }

    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void addToFront(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }

    private void moveToFront(Node node) {
        remove(node);
        addToFront(node);
    }
}
```
**Complexity:** O(1) average for both `get` and `put`; O(capacity) space.

**Key insight / follow-up:** The map gives O(1) *find*; the doubly linked list gives O(1) *reorder and evict* — neither alone suffices, which is why this pairing is the canonical answer. Sentinel head/tail nodes mean `remove`/`addToFront` never touch null, eliminating boundary branches. In real Java you could subclass `LinkedHashMap` and override `removeEldestEntry`, but interviewers want the hand-built map-plus-list to prove you understand the mechanics.

---

*[← DSA bank index](README.md)*
