RMQ за $const$


|     |     |     |  $[L=2$ |         |          |         |         |  $R=7]$ |     |      |
| --- | --: | --: | ------: | ------: | -------: | ------: | ------: | ------: | --: | ---: |
| 0   |   0 |   1 |       2 |       3 |        4 |       5 |       6 |       7 |   8 |    9 |
| 1   |   1 |  -2 | ***5*** | ***3*** | ***-3*** | ***2*** | ***2*** | ***1*** |   4 |    2 |
| 2   |  -2 |  -2 |       3 |      -3 |       -3 |       2 |       1 |       1 |   2 | None |
| 3   |     |     |         |         |          |         |         |         |     |      |



``` C++
vector<vector<int>> table;
int n;

int func(int a, int b) { return min(a, b); }

void create_table() {
  int logn = __lg(n) + 1;
  table.resize(logn, vector<int>(n));

  for (int i = 1; i < logn; i++) {
    for (int j = 0; j + (1 << (i - 1)) < n; j++) {
      table[i][j] = func(table[i - 1][j], table[i - 1][j + (1 << (i - 1))]);
    }
  }
}

int get_func(int L, int R) {
  if (L > R) swap(L, R);
  int lg = __lg(R - L + 1);
  return func(table[lg][L], table[lg][R - (1 << lg) + 1]);
}

int main() {
  cin >> n;

  int logn = __lg(n) + 1;
  table.resize(logn, vector<int>(n));
}
```
