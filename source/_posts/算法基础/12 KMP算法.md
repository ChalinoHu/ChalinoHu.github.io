---
title: 12 KMP算法
date: 2022-05-27 21:39:13
tags: 算法基础
---



# KMP

字符串匹配暴力算法当主串和模式串中的字符无法匹配时，主串和模式串的匹配指针都要后退，最终时间复杂度为O(mn)，KMP算法在进行匹配时主串的指针不需要回退，优化后时间复杂度为O(2m + n)。

### 暴力算法：

```c++
int violentMatch(const string &s, const string &p){
    for (int i = 0; i < s.size(); i++){
        int start = i, j = 0;
        while(i < s.size() && j < p.size() && s[i] == p[j]) i++,j++;
        if (j == p.size()) return start;
    }
    return -1;
}
```

暴力算法在某个位置没有匹配上模式串后移动到下一个位置进行比较，这样时间复杂度是O(mn)， m,n分别为源字符串和需要匹配的字符串的长度。

KMP的核心思想是对暴力的匹配算法进行优化，若匹配串本身满足一定的性质，可不必要每次都从下一个位置开始比较。kmp算法一般需要计算next数组，next数组是对匹配串的一个预处理，预处理后的next数组存储的是下一次比较字符串开始的位置。

```c++
next [i] = j  //p[1, j] = p[i-j+1, i], 字符串匹配下表一般从1开始，next[i] = j表示i位置的前j个字符与匹配串从1开始的后j字符的内容是相等的。
```

代码模板

```C++
// s[]是长文本，p[]是模式串，n是s的长度，m是p的长度
求模式串的Next数组：
for (int i = 2, j = 0; i <= m; i ++ )
{
    while (j && p[i] != p[j + 1]) j = ne[j];
    if (p[i] == p[j + 1]) j ++ ;
    ne[i] = j;
}

// 匹配
for (int i = 1, j = 0; i <= n; i ++ )
{
    while (j && s[i] != p[j + 1]) j = ne[j];
    if (s[i] == p[j + 1]) j ++ ;
    if (j == m)
    {
        j = ne[j];
        // 匹配成功后的逻辑
    }
}
```

## KMP字符串

给定一个模式串 S，以及一个模板串 P，所有字符串中只包含大小写英文字母以及阿拉伯数字。

模板串 P 在模式串 S 中多次作为子串出现。

求出模板串 P 在模式串 S 中所有出现的位置的起始下标。

#### 输入格式

第一行输入整数 N，表示字符串 P 的长度。

第二行输入字符串 P。

第三行输入整数 M，表示字符串 S 的长度。

第四行输入字符串 S。

#### 输出格式

共一行，输出所有出现位置的起始下标（下标从 0 开始计数），整数之间用空格隔开。

#### 数据范围

1≤N≤10^5
1≤M≤10^6

#### 输入样例：

```
3
aba
5
ababa
```

#### 输出样例：

```
0 2
```

```c++
#include <bits/stdc++.h>
using namespace std;

const int N = 100010;
const int M = 1000010;
char p[N], s[M];
int n, m, ne[N];

int main(){
    cin.tie(0);
    ios::sync_with_stdio(false);
    cin >> n >> p + 1 >> m >> s + 1;
    
    // next数组的构造
    for (int i = 2, j = 0; i <= n; ++i){	// i要从2开始，i从1开始只有一个字符，相同的前缀与后缀就是该字符本身，此时j为0
        								// i从1开始那么每个字符都会匹配成功，因为是在和自身作比较
        while(j && p[i] != p[j + 1]) j = ne[j];	// 退无可退或者退完过后的下一个字符匹配成功
        // 退回后接着匹配下一个字符
        if (p[i] == p[j + 1]) j++;
        ne[i] = j;
    }
    
    // 测试代码，查看构造的next数组
    // for_each(begin(ne), begin(ne) + n + 1, [](int i){cout << i << " ";}), cout << endl;
    
    // 字符串的构造
    // 每次匹配时p[j + 1]和s[i]进行匹配，匹配不成功再进行p[ne[j] + 1]和s[i]的匹配
    for (int i = 1, j = 0; i <= m; ++i){
        while(j && s[i] != p[j + 1]) j = ne[j];	// j == 0时表示退回到起点0，下次匹配时p[j + 1](即p[1])开始匹配
        if (s[i] == p[j + 1]) j++;
        if (j == n){
            // 匹配成功
            cout << i - n << " ";
            j = ne[j];	// 跳转到下一个位置找下一个匹配成功时的开始位置
        }
    }
 
	cout << endl;
	return 0;
}
```

