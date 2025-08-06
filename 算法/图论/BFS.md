- 宽度优先搜索
- 可以借此搜到最短路(只有当所有边的边权都为1时)

## 走迷宫

给定一个`n*m`表示迷宫, 数组中只包含0和1
求从左上角1,1移动至右下角n,m, 至少需要移动多少次

**步骤**
1. 将初始状态放到queue数组中
2. while queue不空
3. t <- 对头
4. 扩展对头

**代码示例**
```cpp
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;
const int N = 110;
int n, m;
int g[N][N];
int d[N][N];

pair<int, int> queue[N * N];

int bfs(){

	// 定义队列头尾
	int hh, tt;
	hh = tt = 0;
	// 初始化队列头
	queue[0] = {0, 0};

	// 初始化距离数组, -1代表没走过
	memset(d, -1, sizeof d);
	d[0][0] = 0;

	// 初始化方向数组
	int dx[4] = {-1, 0, 1, 0};
	int dy[4] = {0, 1, 0, -1};

	while(hh <= tt){
		// 拿出队列头, 进行处理
		auto t = queue[hh++];
		for(int i = 0; i < 4; i++){
			// 尝试在原位置的基础上进行拓展
			int x = t.first + dx[i];
			int y = t.second + dy[i];

			// 尝试进行拓展
			// 要求不能越界, 不能撞墙, 不能走回头路
			if(x >= 0 && x < n && y >= 0 && y < m && g[x][y] != 1 && d[x][y] == -1){
				// 符合以下条件时, 表示可以计入数组中
				queue[++tt] = {x, y};
				d[x][y] = d[t.first][t.second] + 1;
			}
		}
	}

	// 返回距离数组中终点的距离
	return d[n - 1][m - 1];
}

int main(){
	cin >> n >> m;
	for(int i = 0; i < n; i++){
		for(int j = 0; j < m; j++){
			cin >> g[i][j];
		}
	}

	cout << bfs() << endl;

}

```

## 八数码

3x3网格中, 随机分布1-8和一个X, 每次移动X一次, 使顺序变为1-8+X
求操作最少的次数, 如果不可能达到, 输出-1

1. 状态表示比较复杂 
	- 如何记录每个状态的距离
-> 使用string进行表示 (`queue<string>`)

2. 如何判断状态的变换方法
	1. 恢复成为`3*3`的形式
	2. 枚举X移动到上下左右时的情况
	3. 将移动后的矩阵再恢复成字符串










