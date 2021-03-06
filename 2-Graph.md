## 图论

### 链式前向星

```cpp
int ecnt, mp[MAXN];

struct Edge {
    int to, nxt;
    Edge(int to = 0, int nxt = 0) : to(to), nxt(nxt) {}
} es[MAXM];

void mp_init() {
    memset(mp, -1, (n + 2) * sizeof(int));
    ecnt = 0;
}

void mp_link(int u, int v) {
    es[ecnt] = Edge(v, mp[u]);
    mp[u] = ecnt++;
}

for (int i = mp[u]; i != -1; i = es[i].nxt)
```

### Dijkstra

```cpp
struct Edge {
    int to, val;
    Edge(int to = 0, int val = 0) : to(to), val(val) {}
};
vector<Edge> G[MAXN];
ll dis[MAXN];

void dijkstra(int s) {
    using pii = pair<ll, int>;
    memset(dis, 0x3f, sizeof(dis));
    priority_queue<pii, vector<pii>, greater<pii> > q;
    dis[s] = 0;
    q.push({0, s});
    while (!q.empty()) {
        pii p = q.top();
        q.pop();
        int u = p.second;
        if (dis[u] < p.first) continue;
        for (int i = 0; i < G[u].size(); i++) {
            int v = G[u][i].to;
            if (dis[v] > dis[u] + G[u][i].val) {
                dis[v] = dis[u] + G[u][i].val;
                q.push({dis[v], v});
            }
        }
    }
}
```

### 拓扑排序

```cpp
int n, deg[MAXN], dis[MAXN];
vector<int> G[MAXN];

bool topo(vector<int>& ans) {
    queue<int> q;
    for (int i = 1; i <= n; i++) {
        if (deg[i] == 0) {
            q.push(i);
            dis[i] = 1;
        }
    }
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        ans.push_back(u);
        for (int v : G[u]) {
            deg[v]--;
            dis[v] = max(dis[v], dis[u] + 1);
            if (deg[v] == 0) q.push(v);
        }
    }
    return ans.size() == n;
}
```

### 最小生成树

```cpp
// 前置：并查集
struct Edge {
    int from, to, val;
    Edge(int from = 0, int to = 0, int val = 0) : from(from), to(to), val(val) {}
};

vector<Edge> es;

ll kruskal() {
    sort(es.begin(), es.end(), [](Edge& x, Edge& y) { return x.val < y.val; });
    iota(pa, pa + n + 1, 0);
    ll ans = 0;
    for (Edge& e : es) {
        if (find(e.from) != find(e.to)) {
            merge(e.from, e.to);
            ans += e.val;
        }
    }
    return ans;
}
```

### LCA

```cpp
int dep[MAXN], up[MAXN][22]; // 22 = ((int)log2(MAXN) + 1)

void dfs(int u, int pa) {
    dep[u] = dep[pa] + 1;
    up[u][0] = pa;
    for (int i = 1; i < 22; i++) {
        up[u][i] = up[up[u][i - 1]][i - 1];
    }
    for (int i = 0; i < G[u].size(); i++) {
        if (G[u][i] != pa) {
            dfs(G[u][i], u);
        }
    }
}

int lca(int u, int v) {
    if (dep[u] > dep[v]) swap(u, v);
    int t = dep[v] - dep[u];
    for (int i = 0; i < 22; i++) {
        if ((t >> i) & 1) v = up[v][i];
    }
    if (u == v) return u;
    for (int i = 21; i >= 0; i--) {
        if (up[u][i] != up[v][i]) {
            u = up[u][i];
            v = up[v][i];
        }
    }
    return up[u][0];
}
```

### 网络流

+ 最大流

```cpp
const int INF = 0x7fffffff;

struct Edge {
    int to, cap;
    Edge(int to, int cap) : to(to), cap(cap) {}
};

struct Dinic {
    int n, s, t;
    vector<Edge> es;
    vector<vector<int> > G;
    vector<int> dis, cur;

    Dinic(int n, int s, int t) : n(n), s(s), t(t), G(n + 1), dis(n + 1), cur(n + 1) {}

    void addEdge(int u, int v, int cap) {
        G[u].push_back(es.size());
        es.emplace_back(v, cap);
        G[v].push_back(es.size());
        es.emplace_back(u, 0);
    }

    bool bfs() {
        dis.assign(n + 1, 0);
        queue<int> q;
        q.push(s);
        dis[s] = 1;
        while (!q.empty()) {
            int u = q.front();
            q.pop();
            for (int i : G[u]) {
                Edge& e = es[i];
                if (!dis[e.to] && e.cap > 0) {
                    dis[e.to] = dis[u] + 1;
                    q.push(e.to);
                }
            }
        }
        return dis[t];
    }

    int dfs(int u, int cap) {
        if (u == t || cap == 0) return cap;
        int tmp = cap, f;
        for (int& i = cur[u]; i < G[u].size(); i++) {
            Edge& e = es[G[u][i]];
            if (dis[e.to] == dis[u] + 1) {
                f = dfs(e.to, min(cap, e.cap));
                e.cap -= f;
                es[G[u][i] ^ 1].cap += f;
                cap -= f;
                if (cap == 0) break;
            }
        }
        return tmp - cap;
    }

    ll solve() {
        ll flow = 0;
        while (bfs()) {
            cur.assign(n + 1, 0);
            flow += dfs(s, INF);
        }
        return flow;
    }
};
```

+ 最小费用流

```cpp
const int INF = 0x7fffffff;

struct Edge {
    int from, to, cap, cost;
    Edge(int from, int to, int cap, int cost) : from(from), to(to), cap(cap), cost(cost) {}
};

struct MCMF {
    int n, s, t, flow, cost;
    vector<Edge> es;
    vector<vector<int> > G;
    vector<int> d, p, a;  // dis, prev, add
    deque<bool> in;

    MCMF(int n, int s, int t) : n(n), s(s), t(t), flow(0), cost(0), G(n + 1), p(n + 1), a(n + 1) {}

    void addEdge(int u, int v, int cap, int cost) {
        G[u].push_back(es.size());
        es.emplace_back(u, v, cap, cost);
        G[v].push_back(es.size());
        es.emplace_back(v, u, 0, -cost);
    }

    bool spfa() {
        d.assign(n + 1, INF);
        in.assign(n + 1, false);
        d[s] = 0;
        in[s] = 1;
        a[s] = INF;
        queue<int> q;
        q.push(s);
        while (!q.empty()) {
            int u = q.front();
            q.pop();
            in[u] = false;
            for (int& i : G[u]) {
                Edge& e = es[i];
                if (e.cap && d[e.to] > d[u] + e.cost) {
                    d[e.to] = d[u] + e.cost;
                    p[e.to] = i;
                    a[e.to] = min(a[u], e.cap);
                    if (!in[e.to]) {
                        q.push(e.to);
                        in[e.to] = true;
                    }
                }
            }
        }
        return d[t] != INF;
    }

    void solve() {
        while (spfa()) {
            flow += a[t];
            cost += a[t] * d[t];
            int u = t;
            while (u != s) {
                es[p[u]].cap -= a[t];
                es[p[u] ^ 1].cap += a[t];
                u = es[p[u]].from;
            }
        }
    }
};
```

### 树链剖分

```cpp
// 点权
vector<int> G[MAXN];
int pa[MAXN], sz[MAXN], dep[MAXN], dfn[MAXN], maxc[MAXN], top[MAXN];

void dfs1(int u) {
    sz[u] = 1;
    maxc[u] = -1;
    int maxs = 0;
    for (int& v : G[u]) {
        if (v != pa[u]) {
            pa[v] = u;
            dep[v] = dep[u] + 1;
            dfs1(v);
            sz[u] += sz[v];
            if (updmax(maxs, sz[v])) maxc[u] = v;
        }
    }
}

void dfs2(int u, int tp) {
    static int cnt = 0;
    top[u] = tp;
    dfn[u] = ++cnt;
    if (maxc[u] != -1) dfs2(maxc[u], tp);
    for (int& v : G[u]) {
        if (v != pa[u] && v != maxc[u]) {
            dfs2(v, v);
        }
    }
}

void init() {
    dep[1] = 1;
    dfs1(1);
    dfs2(1, 1);
}

ll go(int u, int v) {
    int uu = top[u], vv = top[v];
    ll res = 0;
    while (uu != vv) {
        if (dep[uu] < dep[vv]) {
            swap(u, v);
            swap(uu, vv);
        }
        res += segt.query(dfn[uu], dfn[u]);
        u = pa[uu];
        uu = top[u];
    }
    if (dep[u] > dep[v]) swap(u, v);
    res += segt.query(dfn[u], dfn[v]);
    return res;
}
```
