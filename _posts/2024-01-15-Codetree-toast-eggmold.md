---
layout: post
title: "Codetree. 토스트 계란틀"
date: 2024-01-15
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/toast-eggmold/) <br/><br/>


# 1. 문제설명
<hr>

- `n * n`개의 격자에 `1 * 1` 크기의 계란틀이 주어진다.
- 각각의 계란틀에 담긴 계란의 양이 주어지며 계란틀은 정사각형 형태이다.
- 계란틀을 이루는 4개의 선은 분리가 가능하다.
- 규칙
  - 하나의 선을 맞대고 있는 두 계란틀의 계란의 양의 차이가 `L` 이상 `R` 이하라면 계란틀의 해당 선을 분리한다.
  - 모든 계란틀에 대해 검사를 실시하고 위의 규칙에 해당하는 모든 계란틀의 선을 분리한다.
  - 선의 분리를 통해 합쳐진 계란틀의 계란은 하나로 합치고 이후에 다시 분리한다.
  - 합쳤다 다시 분리한 이후의 각 계란틀별 계란의 양은 `(합쳐진 계란의 총 합)/(합쳐진 계란틀의 총 개수)`가 된다.
    - 소수점은 버린다.


- 문제 분류
  - Implementation, Simulation


<br/>

# 2. 알고리즘 설계
<hr>

- 크게 순회, 변경해야할 틀 탐색, 업데이트로 구현하였다.
- 배열을 전부 순회하면서 방문하지 않은 점을 찾는다.
- 방문하지 않은 점을 기준으로 BFS를 수행
- 인접한 칸의 차이가 `L`~`R`사이라면 리턴할 벡터에 저장하고 BFS 계속 진행
  - `vector<pair<int,int>>`형에 저장
  - 방문 배열에 표기
- BFS가 끝나면 리턴된 `vector<pair<int,int>>`에는 해당 점과 인접한 변경할 좌표들을 가지고 있음.
- 이를 `vector<vector<pair<int,int>>>`형에 담는다.
  - 예제로 예를 들면,
  - 25와 인접한 40
  - 30과 인접한 50, 10, 30, 45
  - 에 해당하는 좌표들이 담기게 된다.
- 이후에는 `vector<pair<int,int>>`를 하나씩 꺼내면서 `해당 좌표들의 전체 합/개수`로 값을 갱신시키면 된다.


<br/>

# 3. 전체 코드
<hr>

{% highlight cpp %}
#include <bits/stdc++.h>
#define X first
#define Y second
using namespace std;
using pii = pair<int, int>;

bool vis[50][50];
int n, L, R, A[50][50];

const int dx[] = { -1,1,0,0 };
const int dy[] = { 0,0,-1,1 };

bool oom(int x, int y) { return x < 0 || y < 0 || x >= n || y >= n; }

vector<pii> bfs(int x, int y) {
    vector<pii> ret;

    queue<pii> q;
    q.push({ x, y });
    vis[x][y] = true;
    ret.push_back({ x, y });

    while (!q.empty()) {
        pii cur = q.front();
        q.pop();

        for (int dir = 0; dir < 4; dir++) {
            int nx = cur.X + dx[dir];
            int ny = cur.Y + dy[dir];

            if (oom(nx, ny) || vis[nx][ny]) continue;

            int diff = abs(A[cur.X][cur.Y] - A[nx][ny]);
            if (diff >= L && diff <= R) {
                q.push({ nx, ny });
                vis[nx][ny] = true;
                ret.push_back({ nx, ny });
            }
        }
    }

    return ret;
}

void update(vector<vector<pii>> pos) {
    for (auto cur : pos) {
        int sum = 0;

        for (pii nxt : cur) sum += A[nxt.X][nxt.Y];
        sum /= (int)cur.size();

        for (pii nxt : cur) A[nxt.X][nxt.Y] = sum;
    }
}

bool solve() {
    vector<vector<pii>> pos;

    for (int i = 0; i < n; i++)
        memset(vis[i], false, sizeof(vis[i]));

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (!vis[i][j]) {
                vector<pii> tmp = bfs(i, j);
                if (tmp.size() >= 2)
                    pos.push_back(tmp);
            }
        }
    }

    if (pos.empty()) return false;

    update(pos);
    return true;
}

int main(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL);

    cin >> n >> L >> R;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            cin >> A[i][j];

    int ans = 0;
    while (solve()) ans++;
    cout << ans;

    return 0;
}
{% endhighlight %}

<br/>

# 4. 소감
<hr>

- 처음에는 `vis`배열을 4차원으로 생성하여 `(i,j)`~ `(k,l)`까지 방문했다고 표시할 생각이었다.
- `memset` 함수를 사용해서 배열을 초기화하려면 4차원 배열을 매 사이클마다 초기화 해줘야 했고
- 생각을 해보니, 각 영역의 좌표 값을 벡터에 담고, 그 벡터의 벡터를 가지고 놀면 되는 문제였다.
