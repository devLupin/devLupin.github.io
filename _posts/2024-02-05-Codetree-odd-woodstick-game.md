---
layout: post
title: "Codetree. 이상한 윷놀이"
date: 2024-02-05
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/odd-woodstick-game) <br/><br/>


# 1. 문제설명
<hr>

- `n * n` 격자판에서 새로운 규칙을 가진 윷놀이를 진행한다.
  - 격자의 각 칸에는 흰색, 빨간색, 파란색 중 하나의 색을 갖는다.
- 말은 총 `k`개가 주어지며, 격자판의 한 지점에 놓여있다.
  - 1번부터 k번까지 번호가 지정되어 있으며 이동 방향 또한 미리 정해져있다. 
  - 상하좌우의 4가지 방향으로 움직일 수 있다.
- 윷놀이 규칙
  - 이동하려는 칸이 흰색인 경우
    - 해당 칸으로 이동
	- 이동하려는 칸에 말이 이미 있는 경우에는 해당 말 위에 이동하려던 말을 올린다.
	- 이미 말이 올려져 있는 상태에도 말을 올릴 수 있다.
  - 이동하려는 칸이 빨간색인 경우
    - 이동하려는 칸이 빨간색인 경우에는 해당 칸으로 이동하기 전 순서를 뒤집는다.
	- 이후 해당 칸에 말이 있는 경우에는 흰색 칸과 같이 처리한다.
  - 이동하려는 칸이 파란색인 경우
    - 이동하지 않고 방향을 반대로 전환한 뒤 이동
	- 만일 반대 방향으로 전환한 뒤 이동하려는 칸도 파란색이라면 방향만 반대로 전환한 뒤 이동하지 않고 가만히 있는다.
	- 이동하려는 말에 다른 말들이 쌓여있을 경우에 이동하려는 말만 방향을 반대로 바꾼다.
  - 격자판의 범위를 벗어나는 경우
    - 파란색과 동일하게 처리한다.

<br/>


# 2. 입/출력
<hr>

- 윷놀이 판의 크기 `n`
  - 0은 흰색 판, 1은 빨간색 판, 2는 파란색 판을 의미
- 말의 개수 `k`
  - 위치 x, y와 방향 d
  - d는 1일 경우 오른쪽, 2일 경우 왼쪽, 3일 경우 윗쪽, 4일 경우 아랫쪽
- 게임이 진행되는 동안 아직 한 턴이 다 끝나지 않은 경우더라도 말이 4개 이상 겹쳐지는 경우가 생긴다면 그 즉시 게임을 종료
  - 게임이 종료되는 턴의 번호를 출력합니다. 답이 1000보다 크거나 불가능한 경우에는 -1을 출력

<br/>


# 3. 문제 분류
- Simulation

<br/>


# 4. 알고리즘 설계
<hr>

- 게임 횟수를 최대 1000번으로 잡고 시뮬레이션 진행
- 각 시뮬레이션마다 말을 1번부터 순차적으로 움직인다.
- 말을 움직이는 방법
  - 파란색 또는 격자 범위 밖
    - 방향을 전환한다.
	- 전환 후에도 파란색 또는 격자 범위 밖이면 무시된다.
  - 현재 위치의 말들의 번호 순서 벡터 `cur`, 다음 위치의 말들의 번호 순서 벡터 `next`
    - 벡터의 순서가 뒤에 있을수록 위에 있음을 나타낸다.
    - `find` 함수를 통해 현재 탐색중인 `i`번째 말의 벡터 인덱스를 탐색
	- 인덱스부터 `cur.end()`까지가 우리가 뒤짚어야할 대상이다.
	  - 자신의 차례에 자신 밑에 다른 말들이 있을 수 있다.
	  - 이때는 다음 방향으로 자신과 자신 위의 것들만 이동하기 때문이다.
  - 인덱스부터 `cur.end()`까지 `next`에 `push_back()` 한다.
  - 인덱스부터 `cur.end()`까지의 정보를 삭제한다.
    - 이동한 효과


<br/>

# 5. 전체 코드
<hr>

{% highlight cpp %}
#define _CRT_SECURE_NO_WARNINGS
#include <bits/stdc++.h>
using namespace std;

struct horse { int x, y, d; };
horse h[10];

int N, K;
int dx[] = { 0, 0, 0, -1, 1 };
int dy[] = { 0, 1, -1, 0, 0 };
int turn[] = { 0, 2, 1, 4, 3 };
int color[13][13];
vector<int> info[13][13];

bool oom(int x, int y) { return x <= 0 || y <= 0 || x > N || y > N; }

int move(int i) {
	auto& now = h[i];
	int nx = now.x + dx[now.d];
	int ny = now.y + dy[now.d];

	if (oom(nx, ny) || color[nx][ny] == 2) {
		now.d = turn[now.d];

		nx = now.x + dx[now.d];
		ny = now.y + dy[now.d];

		if (oom(nx, ny) || color[nx][ny] == 2)
			return 0;
	}
	vector<int>& cur = info[now.x][now.y];
	vector<int>& next = info[nx][ny];
	auto s = find(cur.begin(), cur.end(), i);

	if (color[nx][ny] == 1)
		reverse(s, cur.end());

	for (auto it = s; it != cur.end(); ++it) {
		h[*it].x = nx, h[*it].y = ny;
		next.push_back(*it);
	}

	cur.erase(s, cur.end());
	return next.size();
}

int simulation() {
	for (int t = 1; t <= 1000; t++)	{
		for (int i = 0; i < K; ++i) {
			int cnt = move(i);
			if (cnt >= 4) return t;
		}
	}

	return -1;
}

int main() {
	ios::sync_with_stdio(0); 
	cin.tie(0);

	// freopen("input.txt", "r", stdin);

	cin >> N >> K;
	for (int i = 1; i <= N; ++i)
		for (int j = 1; j <= N; ++j)
			cin >> color[i][j];

	for (int i = 0; i < K; ++i)	{
		horse& ho = h[i];
		cin >> ho.x >> ho.y >> ho.d;
		info[ho.x][ho.y].push_back(i);
	}
	cout << simulation();
	return 0;
}
{% endhighlight %}

<br/>

# 4. 소감
<hr>

- 벡터와 iterator를 잘 응용한 풀이라고 생각이 되었다.
- 아마 일일히 구현했다면 코드도 지저분해지고 실수도 많아졌을 것이다.