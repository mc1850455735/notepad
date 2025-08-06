**主要用于规则排列相关的题目**

- 深度优先搜索会优先向下走
- 无路可走时会进行回溯, 然后继续进行
- 直到彻底无路可走, 其搜索顺序可以看作是一个树, 每次只存一个路径
- 其形式上可以看作一个递归
- 回溯时要注意恢复现场

### 排列数字
给定数字n, 得出1-n的全部排列方法

```cpp
const int N = 10;
int path[N];
int n;
bool status[N];
void dfs(int u){
    
    if(u == n){
        // 在一条道路上到头了
        for(int i = 0; i < n; i++){
            cout << path[i] << " ";
        }
        puts("");
        return;
    }
    
    // 如果还没到头
    // 进行的三次循环对应3个开头
    for(int i = 1; i <= n; i++){
        
        if(!status[i]){
            path[u] = i;
            status[i] = true;
            dfs(u + 1);
            status[i] = false;
        }
        
    }
    
}
```

### n皇后问题

1. 像全排列问题一样搜索
2. 时间复杂度为`n·n!`
```cpp
#include <iostream>
using namespace std;
const int N = 10;
int n;
int chest[N];
bool col[N];
bool l[2*N - 1];
bool r[2*N - 1];
void dfs(int u){
	
	// 结束条件: 达到末尾
	if(u == n){
		// 输出每行位置
		for(int i = 0; i < n; i++){
			for(int j = 0; j < chest[i]; j++){
				printf(".");
			}
			printf("Q");
			for(int j = 0; j < n - chest[i] - 1; j++){
				printf(".");
			}
			puts("");
		} 
		puts("");
		return;
	} 
	
	// 开始循环
	// 一共有n个不同的开始位置
	// u代表行号, i代表列号 
	for(int i = 0; i < n; i++){
		
		// 判断这个位置是否可以放东西 
		if(!col[i]&&!l[2*n-1+i-u-1]&&!r[i+u]){
			col[i] = true;
			l[2*n-1 + i - u -1] = true;
			r[i+u] = true;
			chest[u] = i;
			dfs(u + 1);
			col[i] = false;
			l[2*n-1 + i - u -1] = false;
			r[i+u] = false;
		}
	} 
}
```

2. 左右对角线的计算
- 分别是y = x + b和y = b - x
- 不需要row的标志, 因为每一次遍历都是一个唯一的row
- 计算完成后要记得恢复现场

**第二种搜索顺序**
- 一种更加原始的搜索方法
- 从左上角开始搜索, 每个格子搜索一次
- 时间复杂度为`2^n^2`
```cpp
#include <iostream>
using namespace std;

const int N = 20;

int n;
char g[N][N];

// 该算法不是每次递归按行来, 所以需要判断row 
bool row[N], col[N];
bool dg[N], ndg[N];

void dfs(int x, int y, int s){
	// 枚举到了最后一列, 将其转移回来 
	if(y == n){
		y = 0; x++;
	}
	
	// 如果x到了最后一列且已越界 
	if(x == n){
		// 如果皇后数量达标 
		if(s == n){
			for(int i = 0; i < n; i++){
				puts(g[i]);
			}
			puts("");
		}
		return;
	}
	
	// 不放皇后
	dfs(x, y+1, s);
	
	// 放皇后
	if(!row[x] && !col[y] && !dg[x + y] && !ndg[x - y + n]){
		g[x][y] = 'Q';
		row[x] = col[y] = dg[x + y] = ndg[x - y + n] = true;
		dfs(x, y + 1, s + 1);
		row[x] = col[y] = dg[x + y] = ndg[x - y + n] = false;
		g[x][y] = '.';
	} 
}
```

