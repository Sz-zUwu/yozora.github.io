---
title: luogu7915 回文 题解
urls: lg7915-solution
tags:
  - 贪心
  - 搜索
categories: 题解
math: true
abbrlink: afdc656b
date: 2022-06-20 17:03:46
---

## 前言

这题是真的暴力。

可是怎么就是做不出来呢？

是你会的从来都只是套路性的表面功夫，还是你从来就没有静下心来好好提高思维水平呢？

面对现实吧，二者皆有。

<!--more-->

## 分析

首先明确，字典序最小是操作序列的字典序。

只剩下最后一个数的时候，操作 1 和 2 是等价的。由于要求字典序最小，所以操作序列最后一个必定是`L`。

再考虑第一个操作。因为第一次只能操作序列 $a$ 中的第一个数或最后一个数，且最后它必将留在序列 $b$ 的第一个位置。而回文序列 $b$ 中最后一个数必然和第一个数相同。那么就能知道一定是枚举 $a_1$ 和 $a_{2n}$ 两种操作选择，再通过满足 $a_i = a_1$ 或 $a_i = a_{2n}$ 的位置 $i$ 作为一个操作的临界点。

一旦确定了 $i$，那么 $i$ 必定是最后一个操作。我们可以将顺序的操作与倒序的操作的数相对应，满足最终得到回文串。于是考虑搜索。

建议玩一下样例。
$$
[4, 1, 2, 4, 5, 3, 1 ,2, 3, 5 ]
$$

$$
[ 4, 5, 3, 1, 2, 2, 1, 3, 5, 4 ]
$$

不难发现每一次只能操作两个数，左边的位置单调增，右边的位置单调减。不妨设为 $L$ 与 $R$。一开始 $L=1$，$R=2n$。而倒序操作也只能操作两个数，左边的位置单调减，右边的位置单调增，且一开始始终是 $i-1$ 与 $i+1$。记为 $l$ 与 $r$。

这里默认一开始操作的是 4。

第一步用 1 操作把 4 放到第一位，那么 $L+1$，$R$ 不变，此时可以操作的数为靠左的 1 与靠右的 5。这个时候要选择能与 $a_l$ 与 $a_r$ 对应的数，否则绝对不是回文序列。那么显然就是 $a_r$，靠左的 5。所以要使用操作 2 加入 5，$R-1$，$L$ 不变。 由于 $r$ 在 $i$ 右边，最后一定是用 2 操作将 $a_r$ 归位的。此时 $a_r$ 已经不可使用，$r+1$，$l$ 不变。当然，要记录每个操作。

接下来是将 $a_r$ 与 $a_R$ 配对，$a_l$ 与 $a_R$ 配对。

相信大家都看出来套路了。就是择优配对，先考虑可行性，然后贪心选择。`L`必须尽可能靠前，因此优先度递减排序为 $L$ 与 $l$、$L$ 与 $r$、$R$ 与 $l$、$R$ 与 $r$。

当 $l=L$ 并且 $r=R$ 时操作结束。

但是还有一些小边界问题。这是由于边界的增减且不能越界造成的。

1. 当 $L \le l$ 时，必须满足 $L < l$ 才能让 $L$ 与 $l$ 配对，满足 $r \le R$ 时就能让 $L$ 与 $r$ 配对。
2. 当 $r \le R$ 时，满足 $L \le l$ 就能让 $R$ 与 $l$ 配对，必须满足 $r < R$ 时才能让 $R$ 与 $r$ 配对。

由于只有能够成为回文串才贪心操作，所以第一次到达 $l = L$ 且 $r = R$ 时一定是字典序最小的回文串。

复杂度 $O(Tn)$

## CODE

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N=1e6+5;
int t, flag, n, a[N];
char s[N];
void dfs(int p,int l,int r,int L,int R) {
    // l与r，L与R同上文
    // p表示序列b中有多少个数
	if(l<L&&r>R) {
		flag=1;
		for(int i=1;i<=2*n;++i) printf("%c",s[i]); puts("");
		return;
	}
	if(p>n) return;
	if(L<=l&&(a[l]==a[L]&&L<l||a[r]==a[L]&&r<=R)) {
		s[p]='L';
		if(a[l]==a[L]&&L<l) s[2*n-p+1]='L', dfs(p+1,l-1,r,L+1,R);
		else s[2*n-p+1]='R', dfs(p+1,l,r+1,L+1,R);
	} else if(r<=R&&(a[l]==a[R]&&L<=l||a[r]==a[R]&&r<R)) {
		s[p]='R';
		if(a[l]==a[R]&&L<=l) s[2*n-p+1]='L', dfs(p+1,l-1,r,L,R-1);
		else s[2*n-p+1]='R', dfs(p+1,l,r+1,L,R-1);
	}
}
void sol() {
	flag=0;
	scanf("%d",&n);
	for(int i=1;i<=2*n;++i) scanf("%d",&a[i]);
	s[2*n]='L'; // 一定的
	for(int i=2;i<=2*n;++i) if(a[i]==a[1]) {
		s[1]='L';
        // 一开始用1操作位置1，对应的最后一次操作位置是i
        // 顺序操作区间[2,2n]
		dfs(2,i-1,i+1,2,2*n);
		break;
	}
	if(flag) return;
	for(int i=1;i<=2*n-1;++i) if(a[i]==a[2*n]) {
		s[1]='R';
        // 一开始用2操作位置1，对应的最后一次操作位置是i
        // 顺序操作区间[1,2n-1]
		dfs(2,i-1,i+1,1,2*n-1);
		break;
	}
	if(flag) return;
	puts("-1");
}
int main() {
	scanf("%d",&t);
	while(t--) sol();
}
```