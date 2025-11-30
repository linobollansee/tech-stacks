# Code-Challenges - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung in Code-Challenges](#einführung-in-code-challenges)
2. [Problem-Solving Strategien](#problem-solving-strategien)
3. [Arrays & Strings](#arrays--strings)
4. [Hash Maps & Sets](#hash-maps--sets)
5. [Linked Lists](#linked-lists)
6. [Stacks & Queues](#stacks--queues)
7. [Trees & Graphs](#trees--graphs)
8. [Recursion & Backtracking](#recursion--backtracking)
9. [Dynamic Programming](#dynamic-programming)
10. [Algorithmen & Komplexität](#algorithmen--komplexität)
11. [Praktische Challenges](#praktische-challenges)

---

## Einführung in Code-Challenges

### Warum Code-Challenges?

**Problemlösung** 🧩
- Analytisches Denken
- Algorithmen verstehen
- Pattern erkennen

**Interview-Vorbereitung** 💼
- Technische Interviews
- Live-Coding
- Time Management

**Skills verbessern** 📈
- Code-Qualität
- Optimierung
- Best Practices

### Challenge-Plattformen

```
LeetCode        → 2000+ Problems, Community
HackerRank      → Strukturiertes Learning
Codewars        → Kata-System, Rankings
CodeSignal      → Interview Practice
Exercism        → Mentoring, Feedback
```

### Herangehensweise

```
1. Problem verstehen
   - Requirements klären
   - Edge Cases identifizieren
   - Beispiele durchgehen

2. Plan entwickeln
   - Datenstrukturen wählen
   - Algorithmus skizzieren
   - Komplexität einschätzen

3. Implementieren
   - Code schreiben
   - Tests durchführen
   - Refactoring

4. Optimieren
   - Zeit-Komplexität
   - Raum-Komplexität
   - Code-Qualität
```

---

## Problem-Solving Strategien

### UMPIRE-Methode

```typescript
// U - Understand (Verstehen)
function understand(problem: string) {
  // - Was ist der Input?
  // - Was ist der Output?
  // - Was sind Constraints?
  // - Was sind Edge Cases?
}

// M - Match (Abgleichen)
function match() {
  // - Ähnliche Probleme?
  // - Bekannte Pattern?
  // - Datenstrukturen?
}

// P - Plan (Planen)
function plan() {
  // - Schritt für Schritt
  // - Pseudocode
  // - Komplexität
}

// I - Implement (Implementieren)
function implement() {
  // - Code schreiben
  // - Kommentieren
  // - Clean Code
}

// R - Review (Überprüfen)
function review() {
  // - Test Cases
  // - Edge Cases
  // - Bugs fixen
}

// E - Evaluate (Evaluieren)
function evaluate() {
  // - Time Complexity
  // - Space Complexity
  // - Optimierungen
}
```

### Problem-Patterns

```typescript
// 1. Two Pointers
function twoPointers(arr: number[]) {
  let left = 0;
  let right = arr.length - 1;
  
  while (left < right) {
    // Process
    left++;
    right--;
  }
}

// 2. Sliding Window
function slidingWindow(arr: number[], k: number) {
  let windowSum = 0;
  
  // Initial window
  for (let i = 0; i < k; i++) {
    windowSum += arr[i];
  }
  
  // Slide window
  for (let i = k; i < arr.length; i++) {
    windowSum = windowSum - arr[i - k] + arr[i];
  }
}

// 3. Fast & Slow Pointers
function fastSlow(head: ListNode | null) {
  let slow = head;
  let fast = head;
  
  while (fast && fast.next) {
    slow = slow!.next;
    fast = fast.next.next;
  }
  
  return slow;
}

// 4. Divide & Conquer
function divideConquer(arr: number[]): number[] {
  if (arr.length <= 1) return arr;
  
  const mid = Math.floor(arr.length / 2);
  const left = divideConquer(arr.slice(0, mid));
  const right = divideConquer(arr.slice(mid));
  
  return merge(left, right);
}
```

---

## Arrays & Strings

### Challenge 1: Two Sum

```typescript
// Problem: Finde zwei Zahlen, die target ergeben
// Input: nums = [2,7,11,15], target = 9
// Output: [0,1]

// Brute Force: O(n²)
function twoSumBrute(nums: number[], target: number): number[] {
  for (let i = 0; i < nums.length; i++) {
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[i] + nums[j] === target) {
        return [i, j];
      }
    }
  }
  return [];
}

// Optimiert: O(n) mit Hash Map
function twoSum(nums: number[], target: number): number[] {
  const map = new Map<number, number>();
  
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    
    if (map.has(complement)) {
      return [map.get(complement)!, i];
    }
    
    map.set(nums[i], i);
  }
  
  return [];
}

// Tests
console.log(twoSum([2, 7, 11, 15], 9));  // [0, 1]
console.log(twoSum([3, 2, 4], 6));       // [1, 2]
```

### Challenge 2: Reverse String

```typescript
// Problem: String umkehren
// Input: "hello"
// Output: "olleh"

// Methode 1: Built-in
function reverseString1(s: string): string {
  return s.split('').reverse().join('');
}

// Methode 2: Two Pointers (in-place für Array)
function reverseString2(s: string[]): void {
  let left = 0;
  let right = s.length - 1;
  
  while (left < right) {
    [s[left], s[right]] = [s[right], s[left]];
    left++;
    right--;
  }
}

// Methode 3: Rekursiv
function reverseString3(s: string): string {
  if (s.length <= 1) return s;
  return reverseString3(s.slice(1)) + s[0];
}
```

### Challenge 3: Valid Anagram

```typescript
// Problem: Sind zwei Strings Anagramme?
// Input: s = "anagram", t = "nagaram"
// Output: true

// Methode 1: Sortieren O(n log n)
function isAnagram1(s: string, t: string): boolean {
  if (s.length !== t.length) return false;
  
  const sortedS = s.split('').sort().join('');
  const sortedT = t.split('').sort().join('');
  
  return sortedS === sortedT;
}

// Methode 2: Character Count O(n)
function isAnagram(s: string, t: string): boolean {
  if (s.length !== t.length) return false;
  
  const count = new Map<string, number>();
  
  // Count characters in s
  for (const char of s) {
    count.set(char, (count.get(char) || 0) + 1);
  }
  
  // Decrement for characters in t
  for (const char of t) {
    if (!count.has(char)) return false;
    count.set(char, count.get(char)! - 1);
    if (count.get(char)! < 0) return false;
  }
  
  return true;
}
```

### Challenge 4: Longest Substring

```typescript
// Problem: Längster Substring ohne wiederholte Zeichen
// Input: "abcabcbb"
// Output: 3 ("abc")

function lengthOfLongestSubstring(s: string): number {
  const seen = new Map<string, number>();
  let maxLen = 0;
  let start = 0;
  
  for (let end = 0; end < s.length; end++) {
    const char = s[end];
    
    if (seen.has(char) && seen.get(char)! >= start) {
      start = seen.get(char)! + 1;
    }
    
    seen.set(char, end);
    maxLen = Math.max(maxLen, end - start + 1);
  }
  
  return maxLen;
}

// Tests
console.log(lengthOfLongestSubstring("abcabcbb"));  // 3
console.log(lengthOfLongestSubstring("bbbbb"));     // 1
console.log(lengthOfLongestSubstring("pwwkew"));    // 3
```

---

## Hash Maps & Sets

### Challenge 5: Group Anagrams

```typescript
// Problem: Gruppiere Anagramme
// Input: ["eat","tea","tan","ate","nat","bat"]
// Output: [["bat"],["nat","tan"],["ate","eat","tea"]]

function groupAnagrams(strs: string[]): string[][] {
  const map = new Map<string, string[]>();
  
  for (const str of strs) {
    // Sortierter String als Key
    const key = str.split('').sort().join('');
    
    if (!map.has(key)) {
      map.set(key, []);
    }
    
    map.get(key)!.push(str);
  }
  
  return Array.from(map.values());
}

// Optimiert: Character Count als Key
function groupAnagramsOptimized(strs: string[]): string[][] {
  const map = new Map<string, string[]>();
  
  for (const str of strs) {
    const count = new Array(26).fill(0);
    
    for (const char of str) {
      count[char.charCodeAt(0) - 'a'.charCodeAt(0)]++;
    }
    
    const key = count.join(',');
    
    if (!map.has(key)) {
      map.set(key, []);
    }
    
    map.get(key)!.push(str);
  }
  
  return Array.from(map.values());
}
```

### Challenge 6: First Unique Character

```typescript
// Problem: Ersten einzigartigen Character finden
// Input: "leetcode"
// Output: 0

function firstUniqChar(s: string): number {
  const count = new Map<string, number>();
  
  // Count occurrences
  for (const char of s) {
    count.set(char, (count.get(char) || 0) + 1);
  }
  
  // Find first unique
  for (let i = 0; i < s.length; i++) {
    if (count.get(s[i]) === 1) {
      return i;
    }
  }
  
  return -1;
}
```

---

## Linked Lists

### Challenge 7: Reverse Linked List

```typescript
class ListNode {
  val: number;
  next: ListNode | null;
  
  constructor(val?: number, next?: ListNode | null) {
    this.val = val === undefined ? 0 : val;
    this.next = next === undefined ? null : next;
  }
}

// Iterativ
function reverseList(head: ListNode | null): ListNode | null {
  let prev: ListNode | null = null;
  let curr = head;
  
  while (curr) {
    const nextTemp = curr.next;
    curr.next = prev;
    prev = curr;
    curr = nextTemp;
  }
  
  return prev;
}

// Rekursiv
function reverseListRecursive(head: ListNode | null): ListNode | null {
  if (!head || !head.next) return head;
  
  const newHead = reverseListRecursive(head.next);
  head.next.next = head;
  head.next = null;
  
  return newHead;
}
```

### Challenge 8: Detect Cycle

```typescript
// Problem: Hat Linked List einen Cycle?
function hasCycle(head: ListNode | null): boolean {
  if (!head) return false;
  
  let slow = head;
  let fast = head;
  
  while (fast && fast.next) {
    slow = slow.next!;
    fast = fast.next.next;
    
    if (slow === fast) {
      return true;
    }
  }
  
  return false;
}
```

### Challenge 9: Merge Two Sorted Lists

```typescript
// Problem: Zwei sortierte Listen mergen
function mergeTwoLists(
  l1: ListNode | null,
  l2: ListNode | null
): ListNode | null {
  const dummy = new ListNode(0);
  let current = dummy;
  
  while (l1 && l2) {
    if (l1.val < l2.val) {
      current.next = l1;
      l1 = l1.next;
    } else {
      current.next = l2;
      l2 = l2.next;
    }
    current = current.next;
  }
  
  current.next = l1 || l2;
  
  return dummy.next;
}
```

---

## Stacks & Queues

### Challenge 10: Valid Parentheses

```typescript
// Problem: Sind Klammern korrekt?
// Input: "()[]{}"
// Output: true

function isValid(s: string): boolean {
  const stack: string[] = [];
  const pairs: Record<string, string> = {
    ')': '(',
    '}': '{',
    ']': '['
  };
  
  for (const char of s) {
    if (char in pairs) {
      // Closing bracket
      if (stack.length === 0 || stack.pop() !== pairs[char]) {
        return false;
      }
    } else {
      // Opening bracket
      stack.push(char);
    }
  }
  
  return stack.length === 0;
}

// Tests
console.log(isValid("()"));        // true
console.log(isValid("()[]{}"));    // true
console.log(isValid("(]"));        // false
```

### Challenge 11: Queue mit Stacks

```typescript
// Problem: Queue mit zwei Stacks implementieren
class MyQueue {
  private inStack: number[] = [];
  private outStack: number[] = [];
  
  push(x: number): void {
    this.inStack.push(x);
  }
  
  pop(): number {
    this.moveIfNeeded();
    return this.outStack.pop()!;
  }
  
  peek(): number {
    this.moveIfNeeded();
    return this.outStack[this.outStack.length - 1];
  }
  
  empty(): boolean {
    return this.inStack.length === 0 && this.outStack.length === 0;
  }
  
  private moveIfNeeded(): void {
    if (this.outStack.length === 0) {
      while (this.inStack.length > 0) {
        this.outStack.push(this.inStack.pop()!);
      }
    }
  }
}
```

---

## Trees & Graphs

### Challenge 12: Binary Tree Level Order

```typescript
class TreeNode {
  val: number;
  left: TreeNode | null;
  right: TreeNode | null;
  
  constructor(val?: number, left?: TreeNode | null, right?: TreeNode | null) {
    this.val = val === undefined ? 0 : val;
    this.left = left === undefined ? null : left;
    this.right = right === undefined ? null : right;
  }
}

// Problem: Level Order Traversal
function levelOrder(root: TreeNode | null): number[][] {
  if (!root) return [];
  
  const result: number[][] = [];
  const queue: TreeNode[] = [root];
  
  while (queue.length > 0) {
    const levelSize = queue.length;
    const currentLevel: number[] = [];
    
    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift()!;
      currentLevel.push(node.val);
      
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    
    result.push(currentLevel);
  }
  
  return result;
}
```

### Challenge 13: Maximum Depth

```typescript
// Problem: Maximale Tiefe eines Binary Tree
function maxDepth(root: TreeNode | null): number {
  if (!root) return 0;
  
  const leftDepth = maxDepth(root.left);
  const rightDepth = maxDepth(root.right);
  
  return Math.max(leftDepth, rightDepth) + 1;
}

// Iterativ mit BFS
function maxDepthIterative(root: TreeNode | null): number {
  if (!root) return 0;
  
  const queue: TreeNode[] = [root];
  let depth = 0;
  
  while (queue.length > 0) {
    const levelSize = queue.length;
    
    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift()!;
      
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    
    depth++;
  }
  
  return depth;
}
```

### Challenge 14: Graph DFS

```typescript
// Problem: Depth-First Search in Graph
function dfs(graph: Map<number, number[]>, start: number): number[] {
  const visited = new Set<number>();
  const result: number[] = [];
  
  function traverse(node: number) {
    if (visited.has(node)) return;
    
    visited.add(node);
    result.push(node);
    
    const neighbors = graph.get(node) || [];
    for (const neighbor of neighbors) {
      traverse(neighbor);
    }
  }
  
  traverse(start);
  return result;
}

// BFS
function bfs(graph: Map<number, number[]>, start: number): number[] {
  const visited = new Set<number>();
  const result: number[] = [];
  const queue: number[] = [start];
  
  visited.add(start);
  
  while (queue.length > 0) {
    const node = queue.shift()!;
    result.push(node);
    
    const neighbors = graph.get(node) || [];
    for (const neighbor of neighbors) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push(neighbor);
      }
    }
  }
  
  return result;
}
```

---

## Recursion & Backtracking

### Challenge 15: Fibonacci

```typescript
// Rekursiv (ineffizient)
function fib(n: number): number {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}

// Mit Memoization
function fibMemo(n: number, memo: Map<number, number> = new Map()): number {
  if (n <= 1) return n;
  if (memo.has(n)) return memo.get(n)!;
  
  const result = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
  memo.set(n, result);
  return result;
}

// Iterativ (optimal)
function fibIterative(n: number): number {
  if (n <= 1) return n;
  
  let prev = 0, curr = 1;
  
  for (let i = 2; i <= n; i++) {
    [prev, curr] = [curr, prev + curr];
  }
  
  return curr;
}
```

### Challenge 16: Permutations

```typescript
// Problem: Alle Permutationen generieren
function permute(nums: number[]): number[][] {
  const result: number[][] = [];
  
  function backtrack(current: number[], remaining: number[]) {
    if (remaining.length === 0) {
      result.push([...current]);
      return;
    }
    
    for (let i = 0; i < remaining.length; i++) {
      current.push(remaining[i]);
      const next = [...remaining.slice(0, i), ...remaining.slice(i + 1)];
      backtrack(current, next);
      current.pop();
    }
  }
  
  backtrack([], nums);
  return result;
}

// Test
console.log(permute([1, 2, 3]));
// [[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]]
```

### Challenge 17: Subsets

```typescript
// Problem: Alle Subsets generieren
function subsets(nums: number[]): number[][] {
  const result: number[][] = [];
  
  function backtrack(start: number, current: number[]) {
    result.push([...current]);
    
    for (let i = start; i < nums.length; i++) {
      current.push(nums[i]);
      backtrack(i + 1, current);
      current.pop();
    }
  }
  
  backtrack(0, []);
  return result;
}

// Test
console.log(subsets([1, 2, 3]));
// [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]
```

---

## Dynamic Programming

### Challenge 18: Climbing Stairs

```typescript
// Problem: Wie viele Wege gibt es n Stufen zu steigen?
// (1 oder 2 Stufen pro Schritt)

// Rekursiv mit Memo
function climbStairs(n: number): number {
  const memo = new Map<number, number>();
  
  function climb(i: number): number {
    if (i <= 2) return i;
    if (memo.has(i)) return memo.get(i)!;
    
    const result = climb(i - 1) + climb(i - 2);
    memo.set(i, result);
    return result;
  }
  
  return climb(n);
}

// DP Tabulation
function climbStairsDP(n: number): number {
  if (n <= 2) return n;
  
  const dp = new Array(n + 1);
  dp[1] = 1;
  dp[2] = 2;
  
  for (let i = 3; i <= n; i++) {
    dp[i] = dp[i - 1] + dp[i - 2];
  }
  
  return dp[n];
}

// Space Optimized
function climbStairsOptimized(n: number): number {
  if (n <= 2) return n;
  
  let prev2 = 1, prev1 = 2;
  
  for (let i = 3; i <= n; i++) {
    [prev2, prev1] = [prev1, prev2 + prev1];
  }
  
  return prev1;
}
```

### Challenge 19: Coin Change

```typescript
// Problem: Minimale Anzahl Münzen für Betrag
// Input: coins = [1,2,5], amount = 11
// Output: 3 (5+5+1)

function coinChange(coins: number[], amount: number): number {
  const dp = new Array(amount + 1).fill(Infinity);
  dp[0] = 0;
  
  for (let i = 1; i <= amount; i++) {
    for (const coin of coins) {
      if (i >= coin) {
        dp[i] = Math.min(dp[i], dp[i - coin] + 1);
      }
    }
  }
  
  return dp[amount] === Infinity ? -1 : dp[amount];
}

// Test
console.log(coinChange([1, 2, 5], 11));  // 3
console.log(coinChange([2], 3));         // -1
```

### Challenge 20: Longest Increasing Subsequence

```typescript
// Problem: Längste aufsteigende Subsequence
// Input: [10,9,2,5,3,7,101,18]
// Output: 4 ([2,3,7,101])

function lengthOfLIS(nums: number[]): number {
  if (nums.length === 0) return 0;
  
  const dp = new Array(nums.length).fill(1);
  
  for (let i = 1; i < nums.length; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[i] > nums[j]) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
      }
    }
  }
  
  return Math.max(...dp);
}
```

---

## Algorithmen & Komplexität

### Big O Notation

```typescript
// O(1) - Constant
function getFirst(arr: number[]): number {
  return arr[0];
}

// O(n) - Linear
function sum(arr: number[]): number {
  return arr.reduce((a, b) => a + b, 0);
}

// O(n²) - Quadratic
function bubbleSort(arr: number[]): number[] {
  for (let i = 0; i < arr.length; i++) {
    for (let j = 0; j < arr.length - i - 1; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
      }
    }
  }
  return arr;
}

// O(log n) - Logarithmic
function binarySearch(arr: number[], target: number): number {
  let left = 0, right = arr.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) left = mid + 1;
    else right = mid - 1;
  }
  
  return -1;
}

// O(n log n) - Linearithmic
function mergeSort(arr: number[]): number[] {
  if (arr.length <= 1) return arr;
  
  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));
  
  return merge(left, right);
}

function merge(left: number[], right: number[]): number[] {
  const result: number[] = [];
  let i = 0, j = 0;
  
  while (i < left.length && j < right.length) {
    if (left[i] < right[j]) {
      result.push(left[i++]);
    } else {
      result.push(right[j++]);
    }
  }
  
  return result.concat(left.slice(i)).concat(right.slice(j));
}
```

### Sorting Algorithms

```typescript
// Quick Sort - O(n log n) average
function quickSort(arr: number[]): number[] {
  if (arr.length <= 1) return arr;
  
  const pivot = arr[Math.floor(arr.length / 2)];
  const left = arr.filter(x => x < pivot);
  const middle = arr.filter(x => x === pivot);
  const right = arr.filter(x => x > pivot);
  
  return [...quickSort(left), ...middle, ...quickSort(right)];
}

// Insertion Sort - O(n²)
function insertionSort(arr: number[]): number[] {
  for (let i = 1; i < arr.length; i++) {
    const key = arr[i];
    let j = i - 1;
    
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    
    arr[j + 1] = key;
  }
  
  return arr;
}
```

---

## Praktische Challenges

### Challenge 21: Shopping Cart System

```typescript
interface Product {
  id: string;
  name: string;
  price: number;
}

interface CartItem extends Product {
  quantity: number;
}

class ShoppingCart {
  private items: Map<string, CartItem> = new Map();
  
  addItem(product: Product, quantity: number = 1): void {
    if (this.items.has(product.id)) {
      const item = this.items.get(product.id)!;
      item.quantity += quantity;
    } else {
      this.items.set(product.id, { ...product, quantity });
    }
  }
  
  removeItem(productId: string): void {
    this.items.delete(productId);
  }
  
  updateQuantity(productId: string, quantity: number): void {
    if (this.items.has(productId)) {
      if (quantity <= 0) {
        this.removeItem(productId);
      } else {
        this.items.get(productId)!.quantity = quantity;
      }
    }
  }
  
  getTotal(): number {
    return Array.from(this.items.values())
      .reduce((sum, item) => sum + item.price * item.quantity, 0);
  }
  
  getItems(): CartItem[] {
    return Array.from(this.items.values());
  }
  
  clear(): void {
    this.items.clear();
  }
}
```

### Challenge 22: LRU Cache

```typescript
// Problem: Least Recently Used Cache implementieren
class LRUCache {
  private capacity: number;
  private cache: Map<number, number>;
  
  constructor(capacity: number) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  
  get(key: number): number {
    if (!this.cache.has(key)) return -1;
    
    // Move to end (most recent)
    const value = this.cache.get(key)!;
    this.cache.delete(key);
    this.cache.set(key, value);
    
    return value;
  }
  
  put(key: number, value: number): void {
    // Delete if exists
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }
    
    // Add to end
    this.cache.set(key, value);
    
    // Remove oldest if over capacity
    if (this.cache.size > this.capacity) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
  }
}

// Test
const cache = new LRUCache(2);
cache.put(1, 1);
cache.put(2, 2);
console.log(cache.get(1));  // 1
cache.put(3, 3);            // evicts key 2
console.log(cache.get(2));  // -1
```

### Challenge 23: Rate Limiter

```typescript
class RateLimiter {
  private requests: Map<string, number[]> = new Map();
  private limit: number;
  private windowMs: number;
  
  constructor(limit: number, windowMs: number) {
    this.limit = limit;
    this.windowMs = windowMs;
  }
  
  isAllowed(userId: string): boolean {
    const now = Date.now();
    
    if (!this.requests.has(userId)) {
      this.requests.set(userId, []);
    }
    
    const userRequests = this.requests.get(userId)!;
    
    // Remove old requests
    const validRequests = userRequests.filter(
      time => now - time < this.windowMs
    );
    
    if (validRequests.length < this.limit) {
      validRequests.push(now);
      this.requests.set(userId, validRequests);
      return true;
    }
    
    return false;
  }
  
  getRemainingRequests(userId: string): number {
    const now = Date.now();
    const userRequests = this.requests.get(userId) || [];
    
    const validRequests = userRequests.filter(
      time => now - time < this.windowMs
    );
    
    return Math.max(0, this.limit - validRequests.length);
  }
}

// Test
const limiter = new RateLimiter(3, 60000); // 3 requests per minute

console.log(limiter.isAllowed('user1'));  // true
console.log(limiter.isAllowed('user1'));  // true
console.log(limiter.isAllowed('user1'));  // true
console.log(limiter.isAllowed('user1'));  // false
```

---

## Zusammenfassung

Du hast gelernt:

✅ **Problem-Solving**: UMPIRE-Methode, Pattern Recognition
✅ **Arrays & Strings**: Two Sum, Anagrams, Substrings
✅ **Hash Maps**: Grouping, Counting, Fast Lookup
✅ **Linked Lists**: Reversal, Cycle Detection, Merging
✅ **Stacks & Queues**: Parentheses, Queue Implementation
✅ **Trees & Graphs**: Traversal, DFS, BFS
✅ **Recursion**: Fibonacci, Permutations, Subsets
✅ **Dynamic Programming**: Memoization, Tabulation
✅ **Algorithms**: Sorting, Searching, Complexity
✅ **Praktische Projekte**: Shopping Cart, LRU Cache, Rate Limiter

### Best Practices

- [ ] Problem vollständig verstehen
- [ ] Edge Cases identifizieren
- [ ] Mehrere Lösungen erwägen
- [ ] Komplexität analysieren
- [ ] Code testen (Unit Tests)
- [ ] Clean Code schreiben
- [ ] Refactoring durchführen
- [ ] Täglich üben

### Hilfreiche Ressourcen

- [LeetCode](https://leetcode.com/) - 2000+ Problems
- [HackerRank](https://www.hackerrank.com/) - Structured Learning
- [Codewars](https://www.codewars.com/) - Kata System
- [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)

Viel Erfolg mit Code-Challenges! 🚀
