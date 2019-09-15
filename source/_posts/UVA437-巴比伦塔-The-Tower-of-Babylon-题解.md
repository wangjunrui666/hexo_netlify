---
title: UVA437 巴比伦塔 The Tower of Babylon 题解
comments: true
date: 2019-08-07 14:01:37
tags:
 - c++
 - 解题报告
 - OI
 - DP/拓扑/最短路
categories: DP/拓扑/最短路
---

## [原网页](https://uva.onlinejudge.org/external/4/p437.pdf)

## [洛谷翻译网页](https://www.luogu.org/problem/UVA437)

~~一个被恶意评分的题~~

这题就是让你求最长路

输入方块，有三种摆放方式，默认为x是长，y是宽，于是衍生出存边操作(n已经扩大到它的三倍)
```cpp
for(re int i=1,x,y,z; i<=n; i+=3)
{
	read(x);
	read(y);
	read(z);
	cnt[i].add(max(x,y),min(x,y),z);
	cnt[i+1].add(max(x,z),min(x,z),y);
	cnt[i+2].add(max(y,z),min(y,z),x);
}
```
使用记忆化搜索，很容易推出以下程序
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
struct Edge
{
	int next,to,dis;
} edge[14410];
int in[200],dp[200],n,answer,kase;
bool vis[200];
struct node
{
	int x,y,z;
	inline void add(const int & x,const int & y,const int & z)
	{
		this->x=x;
		this->y=y;
		this->z=z;
	}
} cnt[200];
#define max(x,y) (x>y?x:y)
#define min(x,y) (x<y?x:y)
inline int DP(int x)
{
	if(vis[x])
		return dp[x];
	vis[x]=true;
	int & ans=dp[x]=cnt[x].z;
	for(re int i=1; i<=n; i++)
		if(cnt[x].x>cnt[i].x&&cnt[x].y>cnt[i].y)
			ans=max(ans,cnt[x].z+DP(i));
	return ans;
}
int main()
{
	while(read(n),n)
	{
		answer=~0x7fffffff;
		n*=0b11;
		memset(vis,false,sizeof(vis));
		for(re int i=1,x,y,z; i<=n; i+=3)
		{
			read(x);
			read(y);
			read(z);
			cnt[i].add(max(x,y),min(x,y),z);
			cnt[i+1].add(max(x,z),min(x,z),y);
			cnt[i+2].add(max(y,z),min(y,z),x);
		}
		for(re int i=1; i<=n; i++)
			answer=max(answer,DP(i));
		printf("Case %d: maximum height = %d\n",++kase,answer);
	}
	return 0;
}

```
复杂度O(n^2)

因为这题连边后是有向无环图，所以除了记忆化搜索，还可以使用拓扑排序O(n+m)~~至少比跑Floyd强~~
连边部分：
```cpp
for(re int i=1; i<=n; i++)
	for(re int j=1; j<=n; j++)
		if(cnt[i].x>cnt[j].x&&cnt[i].y>cnt[j].y)
		{
			add_edge(i,j,cnt[j].z);
			in[j]++;
		}
```
代码：
```cpp
#include<cstdio>
#include<cstring>
#include<queue>
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
struct Edge
{
	int next,to,dis;
} edge[14410];
int in[200],dis[200],n,answer,kase,head[200],num_edge;
bool vis[200];
struct node
{
	int x,y,z;
	inline void add(const int & x,const int & y,const int & z)
	{
		this->x=x;
		this->y=y;
		this->z=z;
	}
} cnt[200];
#define max(x,y) (x>y?x:y)
inline void add_edge(const int & from,const int & to,const int & dis)
{
	edge[++num_edge].next=head[from];
	head[from]=num_edge;
	edge[num_edge].to=to;
	edge[num_edge].dis=dis;
}
int main()
{
	while(read(n),n)
	{
		answer=~0x7fffffff;
		n*=0b11;
		num_edge=0;
		memset(head,0,sizeof(head));
		memset(vis,false,sizeof(vis));
		for(re int i=1,x,y,z; i<=n; i+=3)
		{
			read(x);
			read(y);
			read(z);
			cnt[i].add(max(x,y),min(x,y),z);
			cnt[i+1].add(max(x,z),min(x,z),y);
			cnt[i+2].add(max(y,z),min(y,z),x);
		}
		for(re int i=1; i<=n; i++)
			for(re int j=1; j<=n; j++)
				if(cnt[i].x>cnt[j].x&&cnt[i].y>cnt[j].y)
				{
					add_edge(i,j,cnt[j].z);
					in[j]++;
				}
		queue<int>q;
		for(re int i=1; i<=n; i++)
			if(in[i]==0)
				q.push(i);
		for(re int i=1; i<=n; i++)
			dis[i]=cnt[i].z;
		re int u;
		while(!q.empty())
		{
			u=q.front();
			q.pop();
			answer=max(answer,dis[u]);
			for(re int i=head[u]; i; i=edge[i].next)
			{
				int &v=edge[i].to,&w=edge[i].dis;
				dis[v]=max(dis[v],dis[u]+w);
				if(--in[v]==0)
					q.push(v);
			}
		}
		printf("Case %d: maximum height = %d\n",++kase,answer);
	}
	return 0;
}

```
