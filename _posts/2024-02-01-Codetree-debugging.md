---
layout: post
title: "Codetree. 디버깅"
date: 2024-02-01
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/debugging) <br/><br/>


# 1. 문제설명
<hr>

- 문제가 길지만 요약해서 적어보겠다.
- 사다리 타기 게임이 주어진다.
  - 일부 경로는 다리가 놓아졌을 수도 있고 아닐 수도 있다.
- 다리를 최대 3개까지 놓았을 때 모든 `i`번에서 출발해 `i`에 도착하게 만들면 된다.

- 문제 분류
  - Backtracking, Simulation


<br/>

# 2. 알고리즘 설계
<hr>

- 입력
  - `a`, `b`에 다리가 놓여있다는 의미는 `[b][a] -> [b][a+1]` 이라는 의미이다.
  - 본인은 `[b][a] = true`로 설정
- 다리를 놓는 DFS
  - `if (adj[j][i] || adj[j - 1][i] || adj[j + 1][i])`
  - 위 조건 중 하나라도 만족하면 다리를 놓을 수 없다.
- 다리를 놓은 후 탐색
  - 갱신된 맵에서 `i`에서 `i`로 도착하는지 매번 탐색


<br/>

# 3. 전체 코드
<hr>

{% highlight cpp %}
#define _CRT_SECURE_NO_WARNINGS
#include <bits/stdc++.h>
#define X first
#define Y second
using namespace std;
using pii = pair<int, int>;

const int INF = 0x3f3f3f;
int N, M, H;
bool adj[15][35];
int ans = INF;

bool check() {
	for (int i = 1; i <= N; i++) {
		int cur = i;
		for (int j = 1; j <= H; j++) {
			if (adj[cur][j]) cur++;
			else if (adj[cur - 1][j]) cur--;
		}
		if (cur != i) return false;
	}

	return true;
}

void solve(int cnt, int x) {
	if (cnt > 3 || cnt > ans) return;
	if (check()) {
		ans = min(ans, cnt);
		return;
	}

	for (int i = x; i <= H; i++) {
		for (int j = 1; j < N; j++) {
			if (adj[j][i] || adj[j - 1][i] || adj[j + 1][i]) continue;

			adj[j][i] = true;
			solve(cnt + 1, i);
			adj[j][i] = false;
		}
	}
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	// freopen("input.txt", "r", stdin);

	cin >> N >> M >> H;
	for (int a, b, i = 0; i < M; i++) {
		cin >> a >> b;
		adj[b][a] = true;
	}

	solve(0, 1);

	if (ans == INF) cout << -1;
	else cout << ans;

	return 0;
}
{% endhighlight %}

<br/>

# 4. 소감
<hr>

- 간단한 구현 문제였다.