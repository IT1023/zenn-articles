---
title: "LeetCode 55 を貪欲法なしで O(n) で解く方法"
emoji: "👨‍💻"
type: "tech"
topics:
  - "typescript"
  - "leetcode"
  - "jumpgame"
published: true
published_at: "2026-08-28 03:47"
---

# Introduction

LeetCode 55, **Jump Game**, is usually solved using a greedy algorithm.

The standard solution keeps track of the farthest index that can currently be reached. While scanning the array, we continuously update this maximum reachable position.

However, I found solution to the problem.

Instead of asking:

> "How far can I reach?"

we can ask:

> "What can actually prevent me from reaching the end?"

The answer is **zero**.

If we encounter a position containing `0`, we cannot make a useful jump from that position. Therefore, the only thing we need to determine is whether some previous position can **jump over that zero**.

Based on this observation, we can solve the problem in **O(n) time without using a greedy algorithm**.

# The Key Observation

Consider the following array:

```text
[2, 3, 1, 0, 2]
```

There is a `0` at index `3`.

At first, this looks like a dead end. But we don't necessarily need to land on index `3`.

From index `1`, we can jump directly to index `4`:

```text
index:  0  1  2  3  4
        2  3  1  0  2
           └─────────
```

So encountering a zero does **not** immediately mean that the answer is `false`.

The important question is:

> Can one of the previous positions jump past this zero?

If yes, we can continue.

If no, the end is unreachable.

This gives us a useful way to think about the problem:

**Zeros are barriers.**

# The Stack-Based Approach

We maintain a stack containing previous indices.

When `nums[i] !== 0`, we simply put the current index into the stack.

When we encounter a zero, we start checking the previous indices from the most recent one.

For each candidate index `last`, we check:

```typescript
last + nums[last] > i
```

If this is true, that index can jump over the current zero.

If it is false, that index cannot help us, so we remove it from the stack and check the next candidate.

If the stack becomes empty, there is no previous index that can cross the zero.

Therefore, we return `false`.

The implementation is:

```typescript
function canJump(nums: number[]): boolean {
    const stack = [];

    for (let i = 0; i < nums.length - 1; i++) {
        if (nums[i] !== 0) {
            stack.push(i);
            continue;
        }

        while (stack.length) {
            const last = stack[stack.length - 1];

            if (last + nums[last] > i) break;

            stack.pop();
        }

        if (!stack.length) return false;
    }

    return true;
}
```

# Why Do We Need `>` Instead of `>=`?

This is an important detail.

Suppose we have:

```text
[2, 1, 0]
```

From index `0`, we can reach index `2`:

```text
0 + nums[0] = 2
```

But index `2` is exactly the zero.

Reaching the zero is not enough. We need to **pass over** it.

Therefore, we need:

```typescript
last + nums[last] > i
```

not:

```typescript
last + nums[last] >= i
```

The distinction is:

```text
>= i  → can reach the zero
>  i  → can pass the zero
```

For this approach, only the second case is useful.

# Example: `[3, 2, 1, 0, 4]`

Let's walk through the algorithm.

### Index 0

`nums[0]` is not zero, so we push it.

```text
stack = [0]
```

### Index 1

Again, it is not zero.

```text
stack = [0, 1]
```

### Index 2

Again:

```text
stack = [0, 1, 2]
```

### Index 3

Now we encounter a zero.

We start with the most recent candidate:

```text
last = 2
```

Can index `2` jump over index `3`?

```text
2 + nums[2]
= 2 + 1
= 3
```

No. It only reaches the zero.

So we pop index `2`.

Next:

```text
last = 1
```

```text
1 + nums[1]
= 1 + 2
= 3
```

Again, it cannot pass the zero.

Pop it.

Finally:

```text
last = 0
```

```text
0 + nums[0]
= 0 + 3
= 3
```

Still not enough.

We pop it as well.

Now:

```text
stack = []
```

There is no previous position that can jump over the zero.

Therefore:

```typescript
return false;
```

And the correct answer is indeed `false`.

# Example: `[2, 3, 1, 0, 2]`

Now consider:

```text
[2, 3, 1, 0, 2]
```

When we reach index `3`, the stack contains:

```text
[0, 1, 2]
```

We first check index `2`:

```text
2 + nums[2]
= 2 + 1
= 3
```

It cannot cross the zero, so we remove it.

Then we check index `1`:

```text
1 + nums[1]
= 1 + 3
= 4
```

Now:

```text
4 > 3
```

So index `1` can jump over the zero.

We stop removing elements and continue scanning.

Eventually, we reach the end and return:

```text
true
```

# Why Is the Complexity O(n)?

At first glance, the nested `while` loop might make the algorithm appear to be `O(n²)`.

But every index has a limited number of operations.

Each index can be:

* pushed onto the stack at most once
* popped from the stack at most once

Therefore, even though we have a `while` loop inside the `for` loop, the total number of stack operations is at most `2n`.

So the overall complexity is:

```text
Time:  O(n)
Space: O(n)
```

# How Is This Different From the Greedy Solution?

The typical greedy solution asks:

> "What is the farthest position I can reach so far?"

My approach asks:

> "If I encounter a zero, can any previous position jump over it?"

These are two different perspectives on the same problem.

The greedy approach continuously maintains the maximum reachable position.

This approach doesn't care about the farthest position at every step.

Instead, it focuses only on **potential blockers**.

That leads to a simple mental model:

```text
Normal position → keep going
Zero             → check whether we can cross it
Impossible zero  → return false
End              → return true
```

# The Core Idea

The entire algorithm can be summarized as:

1. Store previous indices in a stack.
2. When we encounter a zero, check previous indices.
3. Remove indices that cannot jump over the zero.
4. If one can cross it, continue.
5. If none can cross it, return `false`.
6. If we reach the end, return `true`.

The interesting part of this solution is not really the stack itself.

The important insight is recognizing that **zero is the only position that can directly create a dead end**.

Instead of continuously asking how far we can reach, we can focus on whether every potential barrier can be crossed.

# はじめに

LeetCode 55 の **Jump Game** は、一般的には貪欲法（Greedy Algorithm）で解く問題として知られています。

通常の解法では、現在の位置から到達できる**最も遠いインデックス**を常に管理します。

しかし、この問題を別の視点から考えることもできます。

「どこまで進めるか？」ではなく、

> 「何が自分の進行を完全に止める可能性があるのか？」

と考えてみます。

その答えは **`0`** です。

`0` の位置に到達すると、そこから先には進めません。

したがって、重要なのは、

> **現在見つかった `0` を、過去のどの位置から飛び越えられるか？**

ということになります。

この考え方を使うことで、貪欲法を使わずに **O(n)** で解くことができます。

# 重要なポイント

例えば、次の配列を考えます。

```text
[2, 3, 1, 0, 2]
```

インデックス `3` に `0` があります。

一見すると、ここで詰んでしまいそうです。

しかし、必ずしも `0` に着地する必要はありません。

インデックス `1` からインデックス `4` までジャンプできます。

```text
index:  0  1  2  3  4
        2  3  1  0  2
           └───────→
```

つまり、`0` が存在するからといって、必ずしも答えが `false` になるわけではありません。

重要なのは、

> **過去のどこかのインデックスから、その `0` を飛び越えることができるか？**

ということです。

飛び越えられるなら、そのまま進むことができます。

飛び越えられないなら、そこから先に進むことはできません。

つまり、この問題では **`0` を「壁（barrier）」として考える**ことができます。

# Stack を使ったアプローチ

過去のインデックスを `stack` に保存します。

`nums[i] !== 0` の場合は、単純に現在のインデックスを stack に追加します。

```typescript
stack.push(i);
```

そして `0` に遭遇したときだけ、過去のインデックスを調べます。

一番最後に追加されたインデックスから確認して、

```typescript
last + nums[last] > i
```

が成立するかを確認します。

これが `true` なら、そのインデックスから現在の `0` を飛び越えることができます。

逆に `false` なら、そのインデックスには `0` を飛び越える能力がありません。

その場合は stack から削除して、次の候補を確認します。

もしすべての候補を削除して stack が空になった場合、

> この `0` を飛び越えられる場所が過去に存在しない

ということなので、`false` を返します。

実装は以下です。

```typescript
function canJump(nums: number[]): boolean {
    const stack = [];

    for (let i = 0; i < nums.length - 1; i++) {
        if (nums[i] !== 0) {
            stack.push(i);
            continue;
        }

        while (stack.length) {
            const last = stack[stack.length - 1];

            if (last + nums[last] > i) break;

            stack.pop();
        }

        if (!stack.length) return false;
    }

    return true;
}
```

# なぜ `>=` ではなく `>` なのか？

ここは重要なポイントです。

例えば、

```text
[2, 1, 0]
```

を考えます。

インデックス `0` からは、

```text
0 + nums[0] = 2
```

なので、インデックス `2` まで到達できます。

しかし、インデックス `2` は `0` です。

つまり、`0` に**到達するだけでは不十分**です。

`0` を飛び越える必要があります。

そのため、

```typescript
last + nums[last] > i
```

を使います。

```typescript
last + nums[last] >= i
```

ではありません。

違いを簡単に書くと、

```text
>= i → 0 に到達できる
>  i → 0 を飛び越えられる
```

となります。

今回必要なのは後者です。

# `[3, 2, 1, 0, 4]` の例

実際にアルゴリズムを追ってみます。

### インデックス 0

`nums[0]` は `0` ではないので、stack に追加します。

```text
stack = [0]
```

### インデックス 1

同じように追加します。

```text
stack = [0, 1]
```

### インデックス 2

```text
stack = [0, 1, 2]
```

となります。

### インデックス 3

ここで `0` に遭遇します。

まず、一番最後の候補であるインデックス `2` を確認します。

```text
last = 2
```

インデックス `2` から `3` を飛び越せるでしょうか？

```text
2 + nums[2]
= 2 + 1
= 3
```

`3` は `0` の位置そのものなので、飛び越えることはできません。

そこで `2` を stack から削除します。

次に、

```text
last = 1
```

を確認します。

```text
1 + nums[1]
= 1 + 2
= 3
```

これも `0` を飛び越えられません。

さらに削除します。

最後に、

```text
last = 0
```

です。

```text
0 + nums[0]
= 0 + 3
= 3
```

これも飛び越えられません。

stack は空になります。

```text
stack = []
```

つまり、この `0` を飛び越えられる過去のインデックスは存在しません。

したがって、

```typescript
return false;
```

となります。

---

## `[2, 3, 1, 0, 2]` の例

次に、

```text
[2, 3, 1, 0, 2]
```

を考えます。

インデックス `3` の `0` に到達した時点で、

```text
stack = [0, 1, 2]
```

です。

まずインデックス `2` を確認します。

```text
2 + nums[2]
= 2 + 1
= 3
```

`0` を飛び越えられないので削除します。

次にインデックス `1`。

```text
1 + nums[1]
= 1 + 3
= 4
```

今回は、

```text
4 > 3
```

なので、インデックス `1` から `0` を飛び越えることができます。

そのため、これ以上 stack から削除する必要はありません。

そのまま処理を続けると、最後まで到達できるので、

```text
true
```

が返ります。

# なぜ O(n) なのか？

コードを見ると、

```typescript
for (...)
```

の中に、

```typescript
while (...)
```

があるため、一見すると `O(n²)` に見えるかもしれません。

しかし、各インデックスは stack に **最大1回しか追加されません**。

そして、stack から削除されるのも **最大1回だけ**です。

つまり、

* `push` は最大 `n` 回
* `pop` も最大 `n` 回

です。

したがって、stack に対する操作の合計は `O(n)` です。

最終的な計算量は、

```text
Time Complexity:  O(n)
Space Complexity: O(n)
```

となります。

# Greedy Algorithm との違い

一般的な Greedy 解法では、

> 「現在までに、どこまで遠くに到達できるか？」

を常に管理します。

一方、このアプローチでは、

> 「`0` が見つかったとき、それを飛び越えられる過去の位置は存在するか？」

を確認します。

つまり、同じ問題でも見ているものが違います。

Greedy では、常に**最大到達距離**を管理します。

この方法では、常に最大到達距離を管理する必要はありません。

代わりに、問題を止める可能性がある **`0` だけに注目**します。

考え方をまとめると、

```text
通常の位置 → そのまま進む
0            → 飛び越えられるか確認
飛び越せない  → false
最後まで到達   → true
```

となります。

# この解法の核心

このアルゴリズムの考え方は、次の6ステップにまとめられます。

1. 過去のインデックスを stack に保存する。
2. `0` が見つかったら、過去のインデックスを確認する。
3. `0` を飛び越えられないインデックスを stack から削除する。
4. 飛び越えられるインデックスが見つかったら、そのまま続ける。
5. stack が空になったら `false`。
6. 最後まで到達したら `true`。

この解法で最も重要なのは、stack そのものではありません。

重要なのは、

> **「この問題で本当に進行を止める可能性があるのは `0` である」**

という見方です。

「現在どこまで到達できるか」を常に計算する代わりに、

**「すべての障害物を飛び越えられるか？」**

だけを確認することで、別の `O(n)` 解法を構築できます。
