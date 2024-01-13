---
layout: post
title: "Codetree. 병원거리 최소화하기"
date: 2024-01-13
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/min-of-hospital-distance/) <br/><br/>


# 1. 문제설명
<hr>

- `NxN` 크기의 맵에 빈칸, 사람, 병원이 주어진다.
  - 각각은 0, 1, 2로 표현된다.
- 어떤 두 좌표 간 거리는 $|x_2 - x_1| + |y_2 - y_1|$로 정의한다.
- `M`개의 병원을 폐업할 때 사람과 병원 간의 좌표 간 거리를 최소화 하려고 한다.
- 사람과 병원 간의 좌표 간 거리의 총합을 최소화 하라


- 문제 분류
  - Backtraking


<br/>

# 2. 알고리즘 설계
<hr>

- 사람과 병원의 좌표 값을 벡터에 저장
  - 연산의 용이
- DFS 구현을 통해 병원을 표현하는 값 2를 0으로 변환
  - 한번 거친 좌표는 다시 가지 않음!
  - 이거 안하면 시간초과 발생
- 사람의 좌표와 0이 아닌 병원의 좌표의 거리 최소값 계산
  - 모든 병원과 비교를 해줘야 한다.
- 위와 같은 과정을 전체 병원에 대해 `M`개의 병원이 남을 때까지 반복


<br/>

# 3. 전체 코드
<hr>

#include <iostream>
#include <vector>
#include <queue>
#define X first
#define Y second
using namespace std;
using pii = pair<int, int>;

const int INF = 0x3f3f3f3f;
int ans = INF;
int N, M, A[50][50];
vector<pii> person, hos;

int get_dist() {
	int ret = 0;

	for (pii p : person) {
		int cmp = INF;
		for (pii h : hos) {
			if (A[h.X][h.Y] == 2) {
				int tmp = abs(p.X - h.X) + abs(p.Y - h.Y);
				cmp = min(cmp, tmp);
			}
		}
		ret += cmp;
		if (ret > ans) return INF;
	}

	return ret;
}

void dfs(int s, int cnt) {
	if (cnt == M) {
		ans = min(ans, get_dist());
		return;
	}

	for (int i = s; i < hos.size(); i++) {
		pii nxt = hos[i];
		if (A[nxt.X][nxt.Y] == 2) {
			A[nxt.X][nxt.Y] = 0;
			dfs(i, cnt - 1);
			A[nxt.X][nxt.Y] = 2;
		}
	}
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	cin >> N >> M;
	for (int i = 0; i < N; i++) {
		for (int j = 0; j < N; j++) {
			cin >> A[i][j];
			if (A[i][j] == 1) person.push_back({ i,j });
			else if (A[i][j] == 2) hos.push_back({ i,j });
		}
	}

	dfs(0, hos.size());
	cout << ans;

	return 0;
}

<br/>

# 4. 소감
<hr>

- 학부 시절, 알고리즘 수업 때 거의 맨 앞에 나오는 알고리즘이다.
- 그 당시 반복 형태의 코드는 생각보다 엄청 복잡했었고,
- 복잡한 로직을 재귀로 간단히 풀이했던 것이 기억났다.
- '모든 반복의 형태는 재귀로 변환 가능하고, 그 역도 가능하다.'