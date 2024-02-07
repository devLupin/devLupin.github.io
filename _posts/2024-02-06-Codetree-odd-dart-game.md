---
layout: post
title: "Codetree. 이상한 다트 게임"
date: 2024-02-06
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/odd-dart-game) <br/><br/>


# 1. 문제설명
<hr>

- `n`개의 원판에는 `m`개의 숫자가 적혀있다.
- 게임판의 원판을 회전시키고자 한다.
  - 회전은 독립적으로 이루어진다.
  - 회전 요청은 `{x, d, k}`이며 `x`번째 원판을 `d`방향으로 `k`번 움직인다는 의미이다.
  - 회전 이후 인접한 위치의 같은 숫자가 지워진다.
  - 원판에 지워지는 수가 없는 경우에는 원판 전체에 적힌 수의 평균을 구해서 정규화해준다.
    - 원판의 모든 값을 더해서 평균을 내고, 편의상 소수점을 버린 값을 취한다.

<br/>


# 2. 입/출력
<hr>

- 원판의 개수 `n`, 원판 내의 숫자의 개수 `m`, 회전 횟수 `q`
- `q`개의 줄에는 앞서 언급한 회전 요청이 주어진다.
- 게임판을 총 q번 회전 시킨 후에 게임판에 남아있는 수의 총합을 구하라

<br/>


# 3. 문제 분류
- Simulation

<br/>


# 4. 알고리즘 설계
<hr>

- 원판이 최대 50개가 존재할 수 있다.
- 이런 회전 문제의 경우, `deque`를 사용하면 매우 유용하다.
  - 만약 `x`번 원판을 시계방향 회전한다면 아래와 같은 방법으로 회전한 효과를 낼 수 있다.
    - `deque`의 앞단에 가장 뒤의 원소를 집어넣고
    - `deque`의 가장 뒤의 원소를 제거
- 인접한 숫자를 탐색하는 방법
  - 4방향으로 탐색
    - `x`좌표의 경우 값의 범위를 벗어나면 버리고
    - `y`좌표의 경우 값의 범위를 벗어나면 가장 마지막 부분으로 보내면 된다.
    - `x`좌표는 값의 1번 원판의 위치와 마지막 원판의 위치가 인접하지 않기 때문이다.


<br/>

# 5. 전체 코드
<hr>

{% highlight cpp %}
#define _CRT_SECURE_NO_WARNINGS
#include <bits/stdc++.h>
#define X first
#define Y second
using namespace std;
using pii = pair<int, int>;

int N, M, T, mul, d, k, num;
deque<int> board[55];
const int dx[] = { -1,1,0,0 };
const int dy[] = { 0,0,-1,1 };

void rotate() {
	for (int i = mul; i <= N; i += mul) {
		for (int j = 0; j < k; j++) {
			if (d == 0) {
				board[i].push_front(board[i].back());
				board[i].pop_back();
			}
			else {
				board[i].push_back(board[i].front());
				board[i].pop_front();
			}
		}
	}
}

bool remove_adj() {
	bool flag = false;

	vector<pii> v;

	for (int i = 1; i <= N; i++) {
		for (int j = 0; j < M; j++) {
			if (board[i][j] == 0) continue;

			bool chk = false;

			for (int dir = 0; dir < 4; dir++) {
				int x = i + dx[dir];
				int y = j + dy[dir];

				if (x < 1 || x > N) continue;

				if (y < 0) y = M - 1;
				else if (y >= M) y = 0;

				if (board[i][j] == board[x][y]) {
					chk = true;
					v.push_back({ x, y });
				}
			}

			if (chk)
				v.push_back({ i, j });
		}
	}

	for (pii nxt : v) board[nxt.X][nxt.Y] = 0;

	return (int)v.size() > 0;
}

void add() {
	int sum = 0;
	int cnt = 0;
	for (int i = 1; i <= N; i++) {
		for (int j = 0; j < M; j++) {
			if (board[i][j] == 0) continue;
			sum += board[i][j];
			cnt++;
		}
	}

	int cmp = sum / cnt;
	for (int i = 1; i <= N; i++) {
		for (int j = 0; j < M; j++) {
			if (board[i][j] == 0) continue;
			if (board[i][j] > cmp) board[i][j]--;
			else if (board[i][j] < cmp) board[i][j]++;
		}
	}
}

int main() {
	ios::sync_with_stdio(0); 
	cin.tie(0);

	// freopen("input.txt", "r", stdin);

	cin >> N >> M >> T;
	for (int i = 1; i <= N; i++) {
		for (int j = 1; j <= M; j++) {
			cin >> num;
			board[i].push_back(num);
		}
	}

	while (T--) {
		cin >> mul >> d >> k;
		rotate();
		if (!remove_adj()) add();
	}

	int sum = 0;
	for (int i = 1; i <= N; i++)
		for (int nxt : board[i])
			sum += nxt;

	cout << sum;

	return 0;
}
{% endhighlight %}

<br/>

# 4. 소감
<hr>

- 회전을 `deque`로 구현하면 굉장히 쉬운 문제였다.