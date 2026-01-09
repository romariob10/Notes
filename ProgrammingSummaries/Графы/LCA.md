*Деревом* называется связный граф из $N$ вершин и $N-1$ ребер. Из любой вершины до любой другой существует ровно один путь.  
Корнем дерева будет называться такая вершина, от которой задано направление движения по дереву при его обходе.  

*Наименьшим общим предком* двух вершин $u$ и $v$ будет называться такая вершина $p$, которая лежит на пути из корня и до вершины $v$, и до вершины $u$, а также максимально удаленная от корня.



``` C++
#include <bits/stdc++.h>
using namespace std;

vector<int> used;
vector<vector<int>> g;
vector<pair<int, int>> a;
vector<int> d, tin, tout;
int ind = 0;

struct SparceTable {
    vector<vector<pair<int, int>>> table;
    
    pair<int, int> op(pair<int, int> a, pair<int, int> b) {
        if (a.second < b.second) return a;
        return b;
    }

    void build(int n, vector<pair<int, int>> a) {
        int lg = __lg(n) + 1;
        table.resize(lg, vector<pair<int, int>>(n));
        table[0] = a;
        int pw = 1;

        for (int i = 1; i < lg; i++) {
            pw <<= 1;
            
            for (int j = 0; j <= n - pw; j++) {
                table[i][j] = op(table[i - 1][j], table[i - 1][j + pw / 2]);
            }
        }
    }

    pair<int, int> get(int L, int R) {
        int lg = __lg(R - L + 1);
        auto mx = op(table[lg][L], table[lg][R - (1 << lg) + 1]);
        return mx;
    }
};

SparceTable st;

int lca(int u, int v) {
    if (tin[u] <= tin[v] && tout[u] <= tout[v]) return u;
    if (tin[v] <= tin[u] && tout[v] <= tout[u]) return v;
    int l = min(tin[u], tin[v]);
    int r = max(tout[u], tout[v]);

    return st.get(l, r).first;
}

void dfs(int v) {
    if (used[v]) return;

    a[ind] = {v, d[v]};
    tin[v] = ind;
    ind++;

    cout << v << " ";

    for (int u : g[v]) {
        d[u] = d[v] + 1;
        dfs(u);
        tout[v] = ind;
        a[ind] = {v, d[v]};
        ind++;
        cout << v << " ";
    }
}

int main() {
    int n, q;
    cin >> n >> q;

    g.resize(n);
    d.resize(n);
    tin.resize(n);
    tout.resize(n);
    used.resize(n);

    for (int i = 1; i < n; i++) {
        int par;
        cin >> par;
        par--;
        g[par].push_back(i);
    }
    dfs(0);

    st.build(a.size(), a);
    
    long long a1, a2, x, y, z;
    cin >> a1 >> a2 >> x >> y >> z;
    
    vector<long long> c(2 * q + 2);
    c[1] = a1;
    c[2] = a2;
    for (int i = 3; i < 2 * q + 1; i++) {
        c[i] = (x * c[i - 2] + y * c[i - 1] + z) % n;
    }

    int sm = 0;
    int b = lca(c[1], c[2]);

    for (int i = 1; i < q; i++) {
        int u = (c[2 * i + 1] + b) % n;
        int v = c[2 * i + 2];
        int b = lca(u, v);
        sm += b;
    }

    cout << sm;

}

```