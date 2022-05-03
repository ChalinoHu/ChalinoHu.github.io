# Trie

一种高效的存储和查找字符串集合的数据结构

![](\image\trie.png)

## 代码模板

```c++
int son[N][26], cnt[N], idx;
// 0号点既是根节点，又是空节点
// son[][]存储树中每个节点的子节点
// cnt[]存储以每个节点结尾的单词数量

// 插入一个字符串
void insert(char *str)
{
    int p = 0;
    for (int i = 0; str[i]; i ++ )
    {
        int u = str[i] - 'a';
        if (!son[p][u]) son[p][u] = ++ idx;
        p = son[p][u];
    }
    cnt[p] ++ ;
}

// 查询字符串出现的次数
int query(char *str)
{
    int p = 0;
    for (int i = 0; str[i]; i ++ )
    {
        int u = str[i] - 'a';
        if (!son[p][u]) return 0;
        p = son[p][u];
    }
    return cnt[p];
}
```

##  Trie字符串统计

维护一个字符串集合，支持两种操作：

1. `I x` 向集合中插入一个字符串 x；
2. `Q x` 询问一个字符串在集合中出现了多少次。

共有 N 个操作，输入的字符串总长度不超过 105，字符串仅包含小写英文字母。

#### 输入格式

第一行包含整数 N，表示操作数。

接下来 N 行，每行包含一个操作指令，指令为 `I x` 或 `Q x` 中的一种。

#### 输出格式

对于每个询问指令 `Q x`，都要输出一个整数作为结果，表示 x 在集合中出现的次数。

每个结果占一行。

#### 数据范围

1≤N≤2∗10^4

#### 输入样例：

```
5
I abc
Q abc
Q ab
I ab
Q ab
```

#### 输出样例：

```
1
0
1
```

```c++
#include <bits/stdc++.h>
using namespace std;

const int N = 1e5+10;
int son[N][26], cnt[N], idx;	// idx表示当前指针节点
							//下标是0的点既是根节点，也是空节点
							//son存的是每个节点的子节点
void insert(char str[]){
    int p = 0;
    for (int i = 0; str[i]; i++){
        int u = str[i] - 'a';
        if (!son[p][u]) son[p][u] = ++idx;
        p = son[p][u];
    }
    cnt[p]++;
}

int query(char str[]){
    int p = 0;
    for (int i = 0; str[i]; ++i){
        int u = str[i] - 'a';
        if (!son[p][u]) return 0;
        p = son[p][u];
    }
    return cnt[p];
}

int main(){
    int n;
    char str[N];
    char mod;
    
    cin >> n;
    
    while(n--){
        
        cin >> mod;
        if (mod == 'I'){
            
            cin >> str;
            insert(str);
            
        }else if (mod == 'Q'){
            
            cin >> str;
            cout << query(str) << endl;
        }
    }
    
    return 0;
}
```

