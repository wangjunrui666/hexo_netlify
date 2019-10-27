---
layout: w
title: CF1109E Sasha and a Very Easy Test
date: 2019-10-27 20:22:55
tags:
 - c++
 - OI
 - 解题报告
 - 数论
 - 线段树
categories: 线段树
---

>>codeforce题目链接：[
CodeForces 1109E E. Sasha and a Very Easy Test](https://codeforces.com/problemset/problem/1109/E)

>>洛谷题目链接：[CF1109E Sasha and a Very Easy Test](https://www.luogu.org/problem/CF1109E)

题解原发于[我的blog]()

这是本蒟蒻发的第二篇黑题的题解，很开心。

一道很毒瘤的线段树题

本来蒟蒻兴高采烈的刷线段树的题，看到这道黑题，~~觉得很水~~决定刚一刚

这不是一道简单的线段树题

## 考虑直接相除，
结果```WA```了，这正是此题的难处

![](https://cdn.luogu.com.cn/upload/image_hosting/utud29xg.png?x-oss-process=image/resize,m_lfit,h_170,w_225)

## 考虑使用乘法逆元
但是样例二直接否定了这种情况，除数可能不与模数互质。

## 考虑将模数分解质因数。
在懒标记中标记模数的每个质因子个数

相乘时相加质因子，相除时相减质因子

乘数中除模数质因子以外的数得累乘标记

除数除模数质因子以外的数可以直接乘法逆元计算


记住得使用欧拉定理不能使用费马小定理，因为模数不一定是质数

### 费马小定理对于乘法逆元推导(a和p互质，p为质数):
$$a^{p-2}\equiv a^{-1}(mod\ p)$$

### 欧拉定理对于乘法逆元推导(a和p互质):
$$a^{\varphi(p)-2}\equiv a^{-1}(mod\ p)$$
其中
$$\varphi(x)=\prod_{i=1}^{n}\frac{p_i-1}{p_i}$$

$p_1, p_2……p_n$为$x$的所有质因数，$x$是不为$0$的整数。

## 最终在给几个标记的性质
1. 模数不会超过有9个不同的质因子（把前九个质数累乘试一试）。
2. 模数的单个质因子不会超过31个（$2^{20}>10^5$）
3. 除模数质因子外的数可以直接取模（反正乘法逆元行得通）

~~假黑题~~

直接运算即可，一下是蒟蒻的评测记录
![](https://cdn.luogu.com.cn/upload/image_hosting/zfz5bp2b.png)


~~这么简单的代码我都调了这么久一定是我太菜了~~

最后上代码：
```cpp
#include <cstdio>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cctype>

using namespace std;

template <typename T>
inline void read(T &x)
{
	x = 0;
	char s = getchar();
	bool f = false;
	while (!(s >= '0' && s <= '9'))
	{
		if (s == '-')
			f = true;
		s = getchar();
	}
	while (s >= '0' && s <= '9')
	{
		x = (x << 1) + (x << 3) + s - '0';
		s = getchar();
	}
	if (f)
		x = (~x) + 1;
}

namespace OUT
{

	char Out[1 << 25], *fe = Out, ch[25];
	int num = 0;

	template <typename T>
	inline void write(T x)
	{
		if (!x)
			*fe++ = '0';
		if (x < 0)
		{
			*fe++ = '-';
			x = -x;
		}
		while (x)
		{
			ch[++num] = x % 10 + '0';
			x /= 10;
		}
		while (num) *fe++ = ch[num--];
		*fe++ = '\n';
	}

	inline void flush()
	{
		fwrite(Out, 1, fe - Out, stdout);
		fe = Out;
	}
}  // namespace OUT
using namespace OUT;

#define re register
#define ll long long
#define ls (k << 1)
#define rs (k << 1 | 1)
#define node vector<int>

const int N = 1e5 + 10, S = 40;

int n, q, mod, phi, pcnt, a[N];

vector<int> pm;

struct Tree
{
	int c[S], sum, mul, high;  // high是剩余的数
} tree[N << 2];

inline node dec(int n)  //质因数分解
{
	node res;
	res.clear();
	for (re int i = 2; i * i <= n; i++)
		while (n % i == 0)
		{
			n /= i;
			res.push_back(i);
		}
	if (n > 1)
		res.push_back(n);
	return res;
}

inline void Unique(node &res)  //质因数去重(血的教训)
{
	sort(res.begin(), res.end());
	int u = unique(res.begin(), res.end()) - res.begin();
	while (res.size() > u) res.pop_back();
}

inline int getphi(int n)  //欧拉函数
{
	int ans = n;
	for (re int i = 2; i * i <= n; i++)
		if (n % i == 0)
		{
			ans = ans / i * (i - 1);
			while (n % i == 0) n /= i;
		}
	if (n > 1)
		ans = ans / n * (n - 1);
	return ans;
}

inline int pow(int a, int b, int mod = ::mod)
{
	int ans = 1;
	for (; b; b >>= 1, a = (ll)a * a % mod)
		if (b & 1)
			ans = (ll)ans * a % mod;
	return ans;
}

inline int inv(int x)
{
	return pow(x, phi - 1);
}

inline void get(int res, int &ans, int *c)
{
	for (re int i = 1; i <= pcnt; i++)
		while (res % pm[i - 1] == 0)
		{
			res /= pm[i - 1];
			c[i]++;
		}
	ans = res % mod;
}

inline int calc(int v, int *dev)
{
	for (re int i = 1; i <= pcnt; i++) v = (ll)v * pow(pm[i - 1], dev[i]) % mod;
	return v;
}

inline void pushdown(int k)
{
	for (re int i = 1; i <= pcnt; i++)
	{
		tree[ls].c[i] += tree[k].c[i];
		tree[rs].c[i] += tree[k].c[i];
		tree[k].c[i] = 0;
	}
	tree[ls].sum = (ll)tree[ls].sum * tree[k].mul % mod;
	tree[rs].sum = (ll)tree[rs].sum * tree[k].mul % mod;
	tree[ls].mul = (ll)tree[ls].mul * tree[k].mul % mod;
	tree[rs].mul = (ll)tree[rs].mul * tree[k].mul % mod;
	tree[ls].high = (ll)tree[ls].high * tree[k].high % mod;
	tree[rs].high = (ll)tree[rs].high * tree[k].high % mod;
	tree[k].mul = tree[k].high = 1;
}

inline void pushup(int k)
{
	tree[k].sum = (tree[ls].sum + tree[rs].sum) % mod;
}

inline void build(int k, int l, int r)
{
	tree[k].mul = tree[k].high = 1;
	memset(tree[k].c, 0, sizeof(tree[k].c));
	if (l == r)
	{
		tree[k].sum = a[l] % mod;
		get(a[l], tree[k].high, tree[k].c);
		return;
	}
	int mid = l + r >> 1;
	build(ls, l, mid);
	build(rs, mid + 1, r);
	pushup(k);
}

inline void update1(int k, int l, int r, int x, int y, int v, int *dev, int high)
{
	if (r < x || l > y)
		return;
	if (x <= l && r <= y)
	{
		for (re int i = 1; i <= pcnt; i++) tree[k].c[i] += dev[i];
		tree[k].sum = (ll)tree[k].sum * v % mod;
		tree[k].mul = (ll)tree[k].mul * v % mod;
		tree[k].high = (ll)tree[k].high * high % mod;
		return;
	}
	pushdown(k);
	int mid = l + r >> 1;
	update1(ls, l, mid, x, y, v, dev, high);
	update1(rs, mid + 1, r, x, y, v, dev, high);
	pushup(k);
}

inline void update2(int k, int l, int r, int pos, int v, int *dev, int high)
{
	if (l == r)
	{
		tree[k].high = (ll)tree[k].high * inv(high) % mod;
		for (re int i = 1; i <= pcnt; i++) tree[k].c[i] -= dev[i];
		tree[k].sum = calc(tree[k].high, tree[k].c);
		return;
	}
	pushdown(k);
	int mid = l + r >> 1;
	if (pos <= mid)
		update2(ls, l, mid, pos, v, dev, high);
	else
		update2(rs, mid + 1, r, pos, v, dev, high);
	pushup(k);
}

inline int query(int k, int l, int r, int x, int y)
{
	if (r < x || l > y)
		return 0;
	if (x <= l && r <= y)
		return tree[k].sum;
	pushdown(k);
	int mid = l + r >> 1;
	return (query(ls, l, mid, x, y) + query(rs, mid + 1, r, x, y)) % mod;
}

int main()
{
	read(n), read(mod), phi = getphi(mod);
	for (re int i = 1; i <= n; i++) read(a[i]);
	pm = dec(mod);
	Unique(pm);
	pcnt = pm.size();
	build(1, 1, n);
	read(q);
	for (re int i = 1, c[S], opt, a, b, x, v; i <= q; i++)
	{
		read(opt), read(a), read(b);
		memset(c, 0, sizeof(c));
		if (opt == 1)
		{
			read(x);
			get(x, v, c);
			update1(1, 1, n, a, b, x, c, v);
		}
		else if (opt == 2)
		{
			get(b, v, c);
			update2(1, 1, n, a, b, c, v);
		}
		else if (opt == 3)
			write(query(1, 1, n, a, b));
	}
	flush();
	return 0;
}
```
~~马蜂差评~~
