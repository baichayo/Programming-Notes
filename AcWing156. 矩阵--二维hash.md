### 好艰难地理解二维hash

- 思路
	1. 因为要在大矩形（m * n）中找到是否出现过 给出的小矩形(a * b)，那么对于大矩形中的任意一个小矩形，不管位置如何，**在小矩形内部，每个单元的编号从左上角到右下角都该是[1 ~ a * b]**，按照编号可以将二维小矩形变为一维。
	2. 大矩形的单元格并不存在编号（也就是说是无序的），按行处理大矩形只是为了方便处理小矩形。
	3. 枚举小矩形的方式：
		3.1.  先枚举小矩形的右边界
		3.2.  对于每个右边界，枚举小矩形的下边界（因为需要求hash值，所以只能从第一行开始枚举，当所在行是小矩形的下边界时，将矩形的hash值放进集合中。
	4. 对于每个询问，求出小矩形的hash值（将小矩形看成一维），在集合中寻找是否有相同的hash值。

- 一些细节
	1. 小矩形的前缀和：`s = s * p[b] + get(h[i], l, r);`，因为一次处理一行（大矩形的预处理重要性显现出来了），行内的字符之间进制还是 P，行间的进制 则变成了p[b]（一行有b个字符）。 
	2. 小矩形的hash值：`s -= get(h[i - a], l, r) * p[b * a];`， 首先我们要明白，当 `i > a`后，`s`是 `(a + 1) 行 b 列 的小矩形` 的哈希值。此时该减去多余的一行（第 i - a 行），那一行原本的hash值是 `get(h[i - a], l, r)`，每次移动一行都会乘以p[b]，共移动了a次，所以是 p[b]^a^ = p[b * a]。
		
		
		

####将大矩形包含的所有小矩形放进集合中可能会爆内存，因此可以另一种做法：
	
1. 将所有的询问的hash值放进集合中。  
2. 处理大矩形所包含的每个小矩形，在集合中擦除这个hash值。  
3. 对于每个询问，如果能在集合中找到，则说明没被擦除（也就是说不包含这个小矩形），没找到说明被擦除了（包含此小矩形）。  



```
#include<iostream>
#include<unordered_set>

using namespace std;

typedef unsigned long long ULL;

const int N = 1005, mod = 131;

ULL h[N][N], p[N * N];
char str[N];

ULL get(ULL *f, int l, int r)
{
    return f[r] - f[l - 1] * p[r - l + 1];
}

int main()
{
    int m, n, a, b;
    cin >> m >> n >> a >> b;
    
    p[0] = 1;
    for(int i = 1; i <= m * n; i++)
        p[i] = p[i - 1] * mod;
    
    for(int i = 1; i <= m; i++)
    {
        cin >> str + 1;
        for(int j = 1; j <= n; j++)
            h[i][j] = h[i][j - 1] * mod + str[j];
    }
    
    unordered_set<ULL> Set;
    for(int j = b; j <= n; j++)
    {
        ULL s = 0;
        int l = j - b + 1, r = j;
        for(int i = 1; i <= m; i++)
        {
            s = s * p[b] + get(h[i], l, r);
            if(i > a) s -= get(h[i - a], l, r) * p[b * a];
            if(i >= a) Set.insert(s);
        }
    }
    
    int q; cin >> q;
    while(q--)
    {
        ULL s = 0;
        for(int i = 1; i <= a; i++)
        {
            cin >> str + 1;
            for(int j = 1; j <= b; j++)
                s = s * mod + str[j];
        }
        
        if(Set.find(s) != Set.end()) cout << 1 << endl;
        else cout << 0 << endl;
    }
    
    return 0;
}
```