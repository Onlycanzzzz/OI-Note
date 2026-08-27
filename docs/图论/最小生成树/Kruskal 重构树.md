## 引入
作者水平有限，用例题 [[NOIP 2013 提高组] 货车运输](https://www.luogu.com.cn/problem/P1967) 来引入问题。

简要题意：有一个无向连通图，每条边有限重 $z$，$q$次查询由点 $u_i$ 到点 $v_i$ 所有路径中让 *最小边权* **最大**是多少。

## 约定
边的表达格式为 $E = {u, v, w}$。

## 思路
这里只探讨关于 **Kruskal 重构树** 的做法，对于**最大生成树+倍增LCA**的做法暂不考虑。

**Kruskal 重构树**的核心思想是每次合并两个联通块时，新建一个节点代表这次合并，它的权值为当前边的权值。

我们来举个例子，例如有图 $G=(V, E)$，其中 $|V| = 4, |E| = 3$

![Example Graph 1](/图论/最小生成树/assets/example_graph_1.png)

我们建四个节点成为一个新的图，即

![Example Graph 2](/图论/最小生成树/assets/example_graph_2.png)

我们先处理 ${1, 2, 4}$ 这条边，新建节点，将节点 $1, 2$ 与节点 $5$ 连接，我们称为节点 $5$，按照刚刚的约定，$val_5 = 4$，此时并查集中，$1$ 和 $2$ 的父节点都为 $5$。

再处理第二条边 ${2, 3, 3}$，$2$ 所在联通块的根为 $5$，那我们就新建节点 $6$ 连接 节点 $5, 3$，此时的 $val_6$ 应该为 $3$。

以此类推处理第三条边，后面还有节点 $4$ 为孤立点，此时的树应该为

![Example Graph 3](/图论/最小生成树/assets/example_graph_3.png)

由 **Kruskal 重构树** 的思想可以得到，它有个美丽的性质即**任意两个叶子节点 $x$, $y$ 的 LCA 的权值，就是原图中 $x$ 到 $y$ 的最大瓶颈路径值。**

那么现在的时间复杂度为 $\boxed{O(m \log m + (n + q) \log n)}$，分别是**排序、Kruskal重新建树、DFS预处理LCA、处理询问**。

## 代码实现 by @有为骚年
```cpp
int n, m, cnt=n, val[N], vis[N];
struct edge {
  int u, v, w;
};
vector<edge> G;
vector<int> tr[N];
bool cmp(edge a, edge b) {
  return a.w > b.w;
}
struct dsu {
  int fa[N], ranking[N];
  void init(int n) {
    rep (i, 1, n) {
      fa[i] = i, ranking[i] = 1;
    }
  }
  int find(int x) {
    if (fa[x] == x) return x;
    else return fa[x] = find(fa[x]);
  }
  void merge(int x, int y) {
    int fx = find(x), fy = find(y);
    if (fx == fy) return ;
    if (ranking[fx] > ranking[fy]) fa[fy] = fa[fx];
    else fa[fx] = fa[fy];
    if (ranking[fx] == ranking[fy] && fx != fy) ranking[fx]++;
  }
};
int up[N][15], dep[N];
void dfs(int u, int fa) {
  vis[u] = 1, dep[u] = dep[fa]+1, up[u][0] = fa;
  fr (i, 1, 15) {
    up[u][i] = up[up[u][i-1]][i-1];
  }
  for (int v:tr[u]) {
    dfs(v, u);
  }
}
int lca(int u, int v) {
  if (dep[u] < dep[v]) swap(u, v);
  fr (i, 0, 15) {
    if ((dep[u]-dep[v])&(1<<i)) u = up[u][i];
  }
  if (u == v) return u;
  rep2 (i, 14, 0) {
    if (up[u][i] != up[v][i]) u=up[u][i], v=up[v][i];
  }
  return up[u][0];
}

void solve() {
  cin >> n >> m;
  // G.push_back({0, 0, 0});
  rep (i, 1, m) {
    int x, y, z;
    cin >> x >> y >> z;
    G.push_back({x, y, z});
  }

  cnt = n;
  sort(G.begin(), G.end(), cmp);
  dsu DSU;
  DSU.init(n+m+10);
  int cnte = 0;
  for (auto e:G) {
    int fx = DSU.find(e.u), fy = DSU.find(e.v);
    if (fx != fy) {
      cnt++, val[cnt] = e.w;
      tr[cnt].emplace_back(fx), tr[cnt].emplace_back(fy);
      DSU.merge(fx, cnt), DSU.merge(fy, cnt);
      cnte++;
      if (cnte == n-1) break;
    } 
  }
  
  rep2 (i, cnt, 1) {
    if (!vis[i]) {
      dep[i] = 0, up[i][0] = i;
      dfs(i, i);
    }
  }

  int q;
  cin >> q;
  while (q--) {
    int x, y;
    cin >> x >> y;
    if (DSU.find(x) != DSU.find(y)) {
      cout << -1 << endl;
    } else {
      cout << val[lca(x, y)] << endl;
    }
  }
}

int main() {
  ios::sync_with_stdio(0);
  cin.tie(0);
  cout.tie(0);
  
  int t = 1;
  // cin >> t;
  while (t--) {
    solve();
  }
  
  return 0;
}
```