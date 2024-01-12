## Dijsktra算法理解笔记

> 学习了柳神的笔记 [感谢柳神](https://www.liuchuo.net/archives/2357)

*Dijkstra算法是处理图问题中的最短路径的问题*
最短路径问题可以大致分为两个方向
1.  单源最短路径
2. 全局最短路径

以此为基准可以将最短路径算法这样划分：
- **单源最短路径**
1. Dijkstra ：不能求负权边
2. Bellman-Ford：可求负
3. SPFA  ：可求负。是2的优化
- **全局最短路径**
1. Floyed：可求负。

其中注意：点到点可以使用深度优先遍历

下面将着重分析Dijsktra算法

伪代码：

```cpp
Dijkstra() {
  初始化;
  for(循环n次) {
    u = 使dis[u]最小的还未被访问的顶点的编号;
    记u为确定值;
    for(从u出发能到达的所有顶点v){
      if(v未被访问 && 以u为中介点使s到顶点v的最短距离更优)
        优化dis[v];
    }
  }
}
```
**思想**：内外循环。外循环起到遍历所有点的作用。内循环1找到还未被访问的离起点最近的点，可以理解为从起点每一步都选择最短路径，目前走到了这个点。内循环2用于找到目前走到的这个点再往下走该选择的最短路径点。

```cpp
//邻接矩阵
int n, e[maxv][maxv];
int dis[maxv], pre[maxv];// pre用来标注当前结点的前一个结点
bool vis[maxv] = {false};
void Dijkstra(int s) {
  fill(dis, dis + maxv, inf);//初始化距离矩阵
  dis[s] = 0;//起点的距离置为0
  for(int i = 0; i < n; i++) pre[i] = i; //初始状态设每个点的前驱为自身
  
  //进入两层循环
  for(int i = 0; i < n; i++) {
    int u = -1, minn = inf;
    for(int j = 0; j < n; j++) {
      if(visit[j] == false && dis[j] < minn) {
        u = j;
        minn = dis[j];
      }
    }
    if(u == -1) return;
    visit[u] = true;
    for(int v = 0; v < n; v++) {
      if(visit[v] == false && e[u][v] != inf && dis[u] + e[u][v] < dis[v]) {
        dis[v] = dis[u] + e[u][v];
        pre[v] = u; // pre用来标注当前结点的前一个结点
      }
    }
  }
}
```
该部分为邻接矩阵情况的Dikjkstra算法实现。两个关键数组：dis[maxv]和vis[maxv]。初始起点距离置为0。在两层循环起始，设置u，标记现在访问至哪一点。若没有未访问的点，那么说明已经走完了。接着内循环2，判定u点到其余未访问点的距离，判优更新。
上述代码还加入了标注前一个结点的pre[maxv]数组。

```cpp
//邻接表
struct node {
  int v, dis;
}
vector<vector<node>> e[maxv];
int n;
int dis[maxv], pre[maxv];// pre用来标注当前结点的前一个结点
bool visit[maxv] = {false};
for(int i = 0; i < n; i++) pre[i] = i; //初始状态设每个点的前驱为自身
void Dijkstra(int s) {
  fill(dis, dis + maxv, inf);
  dis[s] = 0;
  for(int i = 0; i < n; i++) {
    int u = -1, minn = inf;
    for(int j = 0; j < n; j++) {
      if(visit[j] == false && dis[j] < minn) {
        u = j;
        minn = dis[j];
      }
    }
    if(u == -1) return ;
    visit[u] = true;
    //核心区别在于这里！！
    for(int j = 0; j < e[u].size(); j++) {
      int v = e[u][j].v;
      if(visit[v] == false && dis[u] + e[u][j].dis < dis[v]) {
        dis[v] = dis[u] + e[u][j].dis;
        pre[v] = u;
      }
    }
  }
}
```
该部分为邻接表情况的Dijkstra算法实现。与邻接矩阵大致相同，核心区别在于内循环2的更新是如何提取边权的而已。

输出最短路径，就要用到前面设置的pre数组了。注意要倒序输出

```cpp
void dfs(int s, int v) {
  if(v == s) {
    printf("%d\n", s);
    return ;
  }
  dfs(s, pre[v]);
  printf("%d\n", v);
}
```


> 至此，Dijkstra的基本操作和代码就差不多了。 下面是柳神给的三种附加考法。

1. 边权+边权
以一个边权为判断最短路径的标准，另一个边权为多条最短路径时的挑选准则。

```cpp
for(int v = 0; v < n; v++) { //重写v的for循环
  if(visit[v] == false && e[u][v] != inf) {
    if(dis[u] + e[u][v] < dis[v]) {
      dis[v] = dis[u] + e[u][v];
      c[v] = c[u] + cost[u][v];
    }else if(dis[u] + e[u][v] == dis[v] && c[u] + cost[u][v] < c[v]) {
      c[v] = c[u] + cost[u][v];
    }
  }
}
```
这里主要判断最短路径的边权为e[maxv][maxv]，第二个边权为cost[maxv][maxv]。

2. 边权+点权
以一个边权为判断最短路径的标准，另一个点权为多条最短路径时的挑选准则

```cpp
for(int v = 0; v < n; v++) {
  if(visit[v] == false && e[u][v] != inf) {
    if(dis[u] + e[u][v] < dis[v]) {
      dis[v] = dis[u] + e[u][v];
      w[v] = w[u] + weight[v];
    }else if(dis[u] + e[u][v] == dis[v] && w[u] + weight[v] > w[v]) {
      w[v] = w[u] + weight[v];
    }
  }
}
```
这里主要判断最短路径的边权为e[maxv][maxv]，第二个边权为weight[maxv]。

**1和2的核心都在于要更新c[maxv]和w[maxv]，尤其要在边权未改变时判断第二个指标（边权或者点权）。**

3. 问有多少条最短路径
核心在于增加一个num [ ]。起点的值置为1。其余置为0。在循环的过程中，遇到相等情况，将前一个点的路径数加到后一个点上。最后输出想要终点的值。

```cpp
for(int v = 0; v < n; v++) {
  if(visit[v] == false && e[u][v] != inf) {
    if(dis[u] + e[u][v] < dis[v]) {
      dis[v] = dis[u] + e[u][v];
      num[v] = num[u];
    }else if(dis[u] + e[u][v] == dis[v]) {
      num[v] = num[v] + num[u];
    }
  }
}
```

> 柳神博文中还介绍了一个例子，非常值得学习！再次感谢柳神
