---
title: AT2141 AtCoDeerくんと変なじゃんけん / AtCoDeer and Rock-Paper 题解
comments: true
date: 2019-08-07 13:59:36
tags: 
 - c++
 - OI
 - 解题报告
 - 贪心
categories: 贪心
---

## [洛谷翻译网页](https://www.luogu.org/problem/UVA437)


冒昧地问一句，这题是被恶意评分的吗？

显然你能选帕子就选帕子。

假设第一个人全出石头。

考虑把一些石头修改成帕子。

这样贡献只增不减，加起来就是答案。

```cpp
#include<cstdio>
#include<cstring>
#define re register
using namespace std;
template<typename T>
inline void read(T&x)
{
	x=0;
	char s=getchar();
	bool f=false;
	while(!(s>='0'&&s<='9'))
	{
		if(s=='-')
			f=true;
		s=getchar();
	}
	while(s>='0'&&s<='9')
	{
		x=(x<<1)+(x<<3)+s-'0';
		s=getchar();
	}
	if(f)
		x=(~x)+1;
}
char s[1000010],len;
long long ans,sum_1,sum_2;
bool col[1000010];
int main()
{
	scanf("%s",s+1);
	len=strlen(s+1);
	for(int i=1; i<=len; ++i)
	{
		if(s[i]=='p')
		{
			if(sum_1<sum_2)
				sum_1++;
			else
			{
				sum_2++;
				ans--;
			}
		}
		else
		{
			if(sum_1<sum_2)
			{
				sum_1++;
				ans++;
			}
			else
				sum_2++;
		}
	}
	printf("%lld\n",ans);
}
```
