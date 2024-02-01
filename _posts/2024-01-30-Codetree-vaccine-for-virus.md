---
layout: post
title: "Codetree. 바이러스 백신"
date: 2024-01-30
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/vaccine-for-virus) <br/><br/>


# 1. 문제설명
<hr>

- `NxN`크기의 도시가 주어진다.
- 도시에는 바이러스, 벽, 병원이 있고, 각각 0, 1, 2로 표현된다.
- `M`개의 병원을 골라 최대한 빨리 바이러스를 없애려고 한다.
- 골라진 병원들을 시작으로 매 초마다 상하좌우로 인접한 지역 중 벽을 제외한 지역에 백신이 공급되어 바이러스가 사라진다.
- `M`개의 병원을 놓을 수 있는 조합 중 바이러스를 없애는데 걸리는 최소 시간을 구하라!

- 문제 분류
  - BFS, Backtracking


<br/>

# 2. 알고리즘 설계
<hr>

- `M`개의 조합을 dfs로 구현
  - 각 병원의 좌표를 벡터에 담아 bfs의 입력으로 사용
- `M`개의 병원이 전부 선택되면 bfs 실행
  - `{x, y, d}` 구조체를 선언하고 큐에 삽입
    - `d`는 거리
  - 한번 방문한 지점은 다시 방문하지 않아도 됨.
  - 즉, 여러 개의 길이 존재해도 가장 가까이 있는 병원에 의해 최초 1회 갱신


<br/>

# 3. 전체 코드
<hr>

{% highlight cpp %}
#define _CRT_SECURE_NO_WARNINGS
#include <bits/stdc++.h>
using namespace std;
using pii = pair<int, int>;

const int INF = 5000;
const int dx[] = { -1,1,0,0 };
const int dy[] = { 0,0,-1,1 };

int n, m, A[50][50], ans = INF;
bool vis[50][50];
vector<pii> hospital;

struct info {
	int x, y, d;
};

int bfs(vector<pii> v) {
	queue<info> q;
	int mx = 0;

	for (int i = 0; i < n; i++)
		memset(vis[i], false, sizeof(vis[i]));

	for (auto& nxt : v) {
		q.push({ nxt.first, nxt.second, 0 });
		vis[nxt.first][nxt.second] = true;
	}

	while (!q.empty()) {
		int q_sz = q.size();
		
		while (q_sz--) {
			info cur = q.front();
			q.pop();
			if(A[cur.x][cur.y] == 0) mx = max(mx, cur.d);

			for (int dir = 0; dir < 4; dir++) {
				int nx = cur.x + dx[dir];
				int ny = cur.y + dy[dir];

				if (nx < 0 || ny < 0 || nx >= n || ny >= n) continue;
				if (A[nx][ny] != 1 && !vis[nx][ny]) {
					q.push({ nx, ny, cur.d + 1 });
					vis[nx][ny] = true;
				}
			}
		}
	}

	for (int i = 0; i < n; i++)
		for (int j = 0; j < n; j++)
			if (A[i][j] == 0 && !vis[i][j]) return INF;

	return mx;
}

void solve(int s, int cnt, vector<pii> v) {
	if (cnt == m) {
		ans = min(ans, bfs(v));
		return;
	}

	for (int i = s; i < hospital.size(); i++) {
		v.push_back(hospital[i]);
		solve(i + 1, cnt + 1, v);
		v.pop_back();
	}
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	// freopen("input.txt", "r", stdin);

	cin >> n >> m;
	for (int i = 0; i < n; i++) {
		for (int j = 0; j < n; j++) {
			cin >> A[i][j];
			if (A[i][j] == 2) hospital.push_back({ i,j });
		}
	}

	solve(0, 0, {});
	if (ans == INF) cout << -1;
	else cout << ans;

	return 0;
}
{% endhighlight %}

<br/>

# 4. 소감
<hr>

- 단순한 BFS 문제였다.