
RMQ за $const$

|     |     |     |  $[L=2$ |         |          |         |         |  $R=7]$ |     |      |
| --- | --: | --: | ------: | ------: | -------: | ------: | ------: | ------: | --: | ---: |
| 0   |   0 |   1 |       2 |       3 |        4 |       5 |       6 |       7 |   8 |    9 |
| 1   |   1 |  -2 | ***5*** | ***3*** | ***-3*** | ***2*** | ***2*** | ***1*** |   4 |    2 |
| 2   |  -2 |  -2 |       3 |      -3 |       -3 |       2 |       1 |       1 |   2 | None |
| 3   |     |     |         |         |          |         |         |         |     |      |



``` C++
#include <bits/stdc++.h>

using namespace std;

  

struct SparceTable {

vector<vector<int>> table;

int op(int a, int b) {

  

}

  

void build(int n, vector<int> a) {

int lg = __lg(n);

table.resize(lg + 1, vector<int>(n));

table[0] = a;

int pw = 1;

  

for (int i = 1; i <= lg; i++) {

pw <<= 1;

for (int j = 0; j < n - pw) {

table[i][j] = op(table[i - 1][j], table[i - 1][j + pw / 2])

}

}

}

  

int get(int l, int r) {

int lg = __lg(r - l + 1);

return op(table[lg][l], table[lg][r - (1 << lg) + 1]);

}

}

  

int main() {

int n;

cin >> n;

}
```
