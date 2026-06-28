# DSA Solutions — Greedy & Intervals

> Self-contained solutions reference for greedy decision-making and the interval family: jump games, gas-station circuits, merging/inserting/scheduling intervals, and partitioning. Every solution is Java 17, compilable, and idiomatic. Read the **Problem** first, attempt it, then check the **Approach** and code.

---

## Interval playbook

Almost every interval problem reduces to **sorting + one linear sweep**. The only real decision is *what key to sort by*, and that follows from what you are optimizing:

- **Sort by start** when you process intervals left-to-right and merge/extend them: merging, inserting, meeting-room feasibility, counting concurrent rooms. After sorting by start, two intervals overlap iff the next start is `≤` the current running end.
- **Sort by end** when you greedily *keep* or *remove* intervals to maximize a count: maximum non-overlapping set, minimum removals, minimum arrows. The greedy choice "always commit to the interval that finishes earliest" is safe by an **exchange argument**: the earliest-finishing interval leaves the most room for everything after it, so no optimal solution is ever hurt by preferring it. Any optimal set can be rewritten to start with that earliest-finishing pick without losing intervals.

Greedy is "safe" only when a locally optimal choice provably never blocks the global optimum. For jump-style problems the invariant is a *reachability frontier*; for intervals it is the earliest-finish exchange argument above. When in doubt, name the invariant before you trust the greedy.

---

### LC55 — Jump Game *(medium)*
**Problem:** Given an int array `nums` where `nums[i]` is the maximum jump length from index `i`, you start at index 0. Return `true` if you can reach the last index, otherwise `false`.
**Approach:** Track the farthest index reachable so far. Scan left to right; if the current index is ever beyond that frontier you are stuck. Otherwise extend the frontier by `i + nums[i]`.
```java
class Solution {
    public boolean canJump(int[] nums) {
        int farthest = 0;
        for (int i = 0; i < nums.length; i++) {
            if (i > farthest) {
                return false;        // current index unreachable
            }
            farthest = Math.max(farthest, i + nums[i]);
            if (farthest >= nums.length - 1) {
                return true;
            }
        }
        return true;
    }
}
```
**Complexity:** Time O(n), Space O(1).
**Key insight / follow-up:** Reachability is monotone — if you can reach `i`, you can reach everything up to `farthest`. That single scalar replaces any DP table. The greedy never needs to know *which* jumps were taken, only the frontier.

---

### LC45 — Jump Game II *(medium)*
**Problem:** Given an int array `nums` where `nums[i]` is the maximum jump length from index `i`, starting at index 0, return the **minimum number of jumps** to reach the last index. It is guaranteed the last index is reachable.
**Approach:** BFS-by-levels without a queue. Treat each "jump" as a level whose right boundary `curEnd` is the farthest reachable with the jumps taken so far. While scanning, track `farthest` reachable in the next level; when `i` hits `curEnd`, you must spend a jump and advance the boundary.
```java
class Solution {
    public int jump(int[] nums) {
        int jumps = 0, curEnd = 0, farthest = 0;
        for (int i = 0; i < nums.length - 1; i++) {  // stop before last index
            farthest = Math.max(farthest, i + nums[i]);
            if (i == curEnd) {       // exhausted current jump's range
                jumps++;
                curEnd = farthest;
            }
        }
        return jumps;
    }
}
```
**Complexity:** Time O(n), Space O(1).
**Key insight / follow-up:** Each index belongs to the smallest jump-count "level" that can reach it, exactly like BFS layers — so the first time you can cover the last index is optimal. Looping to `n-1` (not `n`) avoids an extra phantom jump when you land exactly on the end.

---

### LC134 — Gas Station *(medium)*
**Problem:** There are `n` gas stations in a circle. `gas[i]` is the fuel available at station `i` and `cost[i]` is the fuel needed to travel from station `i` to `i+1`. Starting with an empty tank, return the starting station index from which you can complete the full circuit once, or `-1` if impossible. If a solution exists it is unique.
**Approach:** If total gas ≥ total cost a solution exists. Track a running tank from a candidate start; whenever it goes negative, no station in the failed span can be the start, so the next station becomes the new candidate and the tank resets.
```java
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int total = 0, tank = 0, start = 0;
        for (int i = 0; i < gas.length; i++) {
            int diff = gas[i] - cost[i];
            total += diff;
            tank += diff;
            if (tank < 0) {          // can't reach i+1 from current start
                start = i + 1;       // skip past the whole failed segment
                tank = 0;
            }
        }
        return total >= 0 ? start : -1;
    }
}
```
**Complexity:** Time O(n), Space O(1).
**Key insight / follow-up:** If you fail to reach station `j` from start `s`, then no station between `s` and `j` works either — each had a non-negative prefix when you passed it, so starting later only makes it worse. That lets one pass replace the O(n²) try-every-start brute force.

---

### LC56 — Merge Intervals *(medium)*
**Problem:** Given an array `intervals` where `intervals[i] = [start, end]`, merge all overlapping intervals and return an array of the non-overlapping intervals that cover all the input ranges.
**Approach:** Sort by start. Walk the sorted list; if the next interval starts at or before the current merged end, extend the end, otherwise close the current interval and open a new one.
```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public int[][] merge(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
        List<int[]> merged = new ArrayList<>();
        int[] current = intervals[0];
        merged.add(current);
        for (int i = 1; i < intervals.length; i++) {
            int[] next = intervals[i];
            if (next[0] <= current[1]) {           // overlap
                current[1] = Math.max(current[1], next[1]);
            } else {
                current = next;                    // gap: start a new run
                merged.add(current);
            }
        }
        return merged.toArray(new int[merged.size()][]);
    }
}
```
**Complexity:** Time O(n log n) for the sort, Space O(n) for the output (O(log n) auxiliary for the sort).
**Key insight / follow-up:** Sorting by start guarantees any overlap with the running interval is contiguous — once a gap appears, nothing later can reach back. Mutating `current[1]` in place keeps the merge to a single pass. Note `current` references the same array stored in the list, so updating `current[1]` updates the stored interval.

---

### LC57 — Insert Interval *(medium)*
**Problem:** Given a list of non-overlapping `intervals` sorted by start, and a `newInterval`, insert it so the result remains sorted and non-overlapping (merging as needed). Return the result.
**Approach:** Three phases over the already-sorted list: copy intervals strictly before `newInterval`, merge every interval that overlaps it into a single widened interval, then copy the rest.
```java
import java.util.ArrayList;
import java.util.List;

class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        List<int[]> result = new ArrayList<>();
        int i = 0, n = intervals.length;
        // 1. intervals entirely before newInterval
        while (i < n && intervals[i][1] < newInterval[0]) {
            result.add(intervals[i++]);
        }
        // 2. merge all overlapping into newInterval
        while (i < n && intervals[i][0] <= newInterval[1]) {
            newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
            newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
            i++;
        }
        result.add(newInterval);
        // 3. intervals entirely after newInterval
        while (i < n) {
            result.add(intervals[i++]);
        }
        return result.toArray(new int[result.size()][]);
    }
}
```
**Complexity:** Time O(n), Space O(n) for the output.
**Key insight / follow-up:** Because the input is already sorted and disjoint, no re-sort is needed — the overlapping region is one contiguous block, so a single left-to-right pass suffices. The two boundary tests (`end < newStart`, `start <= newEnd`) precisely separate "before", "overlapping", and "after".

---

### LC435 — Non-overlapping Intervals *(medium)*
**Problem:** Given an array of `intervals`, return the minimum number of intervals you must remove so that the rest are non-overlapping. Intervals touching at endpoints (e.g. `[1,2]` and `[2,3]`) do **not** overlap.
**Approach:** Sort by end. Greedily keep an interval whenever its start is ≥ the last kept end; otherwise it overlaps and must be removed. Maximizing kept intervals minimizes removals.
```java
import java.util.Arrays;

class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1]));
        int removals = 0;
        int lastEnd = Integer.MIN_VALUE;
        for (int[] interval : intervals) {
            if (interval[0] >= lastEnd) {
                lastEnd = interval[1];   // keep it
            } else {
                removals++;              // overlaps the kept one: drop it
            }
        }
        return removals;
    }
}
```
**Complexity:** Time O(n log n), Space O(log n) auxiliary.
**Key insight / follow-up:** This is the classic activity-selection problem. Sorting by end and always taking the earliest finisher leaves maximal room for the rest — the exchange argument proves no optimal solution is ever worsened by that choice. Removals = total − maximum non-overlapping kept.

---

### LC252 — Meeting Rooms *(easy)*
**Problem:** Given an array of meeting time `intervals` where `intervals[i] = [start, end]`, determine if a single person could attend all meetings — i.e. return `true` if no two meetings overlap.
**Approach:** Sort by start. Any conflict shows up as adjacent meetings, so just check each meeting's start against the previous meeting's end.
```java
import java.util.Arrays;

class Solution {
    public boolean canAttendMeetings(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
        for (int i = 1; i < intervals.length; i++) {
            if (intervals[i][0] < intervals[i - 1][1]) {
                return false;        // overlap
            }
        }
        return true;
    }
}
```
**Complexity:** Time O(n log n), Space O(log n) auxiliary.
**Key insight / follow-up:** After sorting by start, the earliest-ending conflict (if any) is always between consecutive meetings, so a single adjacent-pair scan is sufficient — no need to compare all pairs. Touching endpoints (`prev.end == next.start`) are not a conflict here, hence strict `<`.

---

### LC253 — Meeting Rooms II *(medium)*
**Problem:** Given meeting time `intervals`, return the minimum number of conference rooms required so that no two overlapping meetings share a room.
**Approach:** Separate and sort the start times and end times. Sweep with two pointers: each start that occurs before the earliest free end needs a new room; otherwise reuse a freed room by advancing the end pointer. The peak number of simultaneously active meetings is the answer.
```java
import java.util.Arrays;

class Solution {
    public int minMeetingRooms(int[][] intervals) {
        int n = intervals.length;
        if (n == 0) return 0;
        int[] starts = new int[n];
        int[] ends = new int[n];
        for (int i = 0; i < n; i++) {
            starts[i] = intervals[i][0];
            ends[i] = intervals[i][1];
        }
        Arrays.sort(starts);
        Arrays.sort(ends);
        int rooms = 0, maxRooms = 0;
        int s = 0, e = 0;
        while (s < n) {
            if (starts[s] < ends[e]) {   // a meeting starts before one frees a room
                rooms++;
                maxRooms = Math.max(maxRooms, rooms);
                s++;
            } else {                     // a meeting ended: room freed
                rooms--;
                e++;
            }
        }
        return maxRooms;
    }
}
```
**Complexity:** Time O(n log n), Space O(n).
**Key insight / follow-up:** This is a sweep-line over event points: +1 at each start, −1 at each end, and the answer is the maximum running overlap. Sorting starts and ends independently lets the two pointers replay events in time order. A min-heap of end times is an equivalent O(n log n) formulation.

---

### LC452 — Minimum Number of Arrows to Burst Balloons *(medium)*
**Problem:** Balloons are given as `points[i] = [xStart, xEnd]` representing horizontal diameters. An arrow shot straight up at `x` bursts every balloon whose interval contains `x` (inclusive). Return the minimum number of arrows needed to burst all balloons.
**Approach:** Sort by end. Shoot an arrow at the end of the first balloon; it bursts every later balloon that starts at or before that x. When a balloon starts past the current arrow position, shoot a new arrow at its end.
```java
import java.util.Arrays;

class Solution {
    public int findMinArrowShots(int[][] points) {
        if (points.length == 0) return 0;
        Arrays.sort(points, (a, b) -> Integer.compare(a[1], b[1]));
        int arrows = 1;
        int arrowAt = points[0][1];          // shoot at first balloon's end
        for (int i = 1; i < points.length; i++) {
            if (points[i][0] > arrowAt) {    // current arrow can't reach it
                arrows++;
                arrowAt = points[i][1];
            }
        }
        return arrows;
    }
}
```
**Complexity:** Time O(n log n), Space O(log n) auxiliary.
**Key insight / follow-up:** This is LC435 in disguise — minimum arrows equals the number of maximal non-overlapping groups. Sorting by end and firing at the earliest end maximizes how many balloons one arrow covers. Use `Integer.compare` (not `a[1] - b[1]`) to avoid overflow on extreme coordinates. Note overlap here is inclusive, so the comparison is strict `>`.

---

### LC763 — Partition Labels *(medium)*
**Problem:** Given a string `s`, partition it into as many parts as possible so that each letter appears in at most one part. Return a list of the sizes of these parts, in order. The parts in order must concatenate back to `s`.
**Approach:** First record the last index of each character. Then sweep, extending the current partition's end to the farthest last-occurrence of any character seen; when the scan index reaches that end, the partition is closed.
```java
import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<Integer> partitionLabels(String s) {
        int[] last = new int[26];
        for (int i = 0; i < s.length(); i++) {
            last[s.charAt(i) - 'a'] = i;     // last occurrence of each letter
        }
        List<Integer> sizes = new ArrayList<>();
        int start = 0, end = 0;
        for (int i = 0; i < s.length(); i++) {
            end = Math.max(end, last[s.charAt(i) - 'a']);
            if (i == end) {                  // every letter so far ends by here
                sizes.add(end - start + 1);
                start = i + 1;
            }
        }
        return sizes;
    }
}
```
**Complexity:** Time O(n), Space O(1) (the 26-slot table is constant).
**Key insight / follow-up:** A partition can close only once the scan reaches the farthest last-occurrence of every letter encountered inside it — that frontier is the interval-merge idea applied to per-character `[firstSeen, last]` spans. The greedy "close as soon as possible" yields the maximal number of parts.

---

*[← DSA bank index](README.md)*
