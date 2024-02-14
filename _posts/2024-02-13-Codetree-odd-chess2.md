---
layout: post
title: "Codetree. 술래잡기 체스"
date: 2024-02-13
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/odd-chess2) <br/><br/>


# 1. 문제설명
<hr>

- `4x4` 격자로 이루어진 체스판에서 술래잡기를 한다.
- 말 하나를 사용하여 도둑말을 잡으며, 술래말을 잡을 때마다 도둑말의 방향을 갖게 된다.
  - 방향은 상하좌우, 대각선에 해당하는 8가지
- 말판의 위치 `x, y`를 통해 차례대로 번호를 매겨 구분
  - `1 <= 도둑말 <= 16`
- 초기 술래말은 `(0, 0)`의 도둑말을 잡아먹고 시작
  - `(0, 0)` 도둑말의 번호를 점수로 획득하고 해당 말의 방향을 갖게 됨.
- 모든 도둑말의 이동이 끝나면 술래말이 이동
- 술래말은 도둑말의 방향으로 몇 칸이든 이동 가능
- 술래말이 이동할 수 있는 곳에 도둑말이 존재하지 않으면 게임 끝

<br/>


# 2. 입/출력
<hr>

- 도둑말의 정보 순서 `p`, 방향 `d`
  - 방향은 1~8까지의 정수
- 획득할 수 있는 점수의 최대값 출력

<br/>


# 3. 문제 분류
- Simulation, Backtracking

<br/>


# 4. 알고리즘 설계
<hr>

- 말의 정보를 저장하기 위한 구조체 및 벡터, 특정 칸의 말의 번호를 탐색하기 위한 벡터 선언

    ```cpp
    struct info {
    	int x, y, d;
    	bool alive;  // 잡아먹힌 말이라면 false
    };
    vector<info> chess;
    vector<vector<int>> board;
    ```

- initialize
  - 맨 처음 도둑말이 움직이기 전
  - 술래말은 `(0,0)`에서 시작하고 해당 칸은 잡아 먹힌 상태이다.

  ```cpp
  int num = board[0][0];
	int d = chess[num].d;
	chess[num].alive = false;
	board[0][0] = -1;
  ```

- backtracking
  - 말의 정보 `chess`, 맵의 정보 `board`를 사전에 복사한다.
  - 말들을 움직인다. `move_chess()`
  - 최대 4칸 dfs
    - 현재 있는 위치의 칸을 0으로 만들고
    - 이동할 칸을 -1로 만든다.
    - `chess[board[nx][ny]].alive = false`
    - dfs()
    - 원래 상태로 되돌린다.
  - `chess`, `board`를 복사한 값으로 되돌린다.

- `move_chess`
  - 16개의 말들을 순차적으로 움직인다.
  - 만약 `i`번째 말이 죽었다면 건너뛴다.
  - 현재 방향의 값 `d`를 더해 움직일 수 있고, 해당 칸이 술래말이 있는 곳이 아니라면 움직인다.
    - 만약 보드의 값이 0이라면 잡아먹힌 자리이므로, 보드의 값만 갱신한다.
    - 보드의 값이 0보다 크다면 말이 있는 자리이므로, 보드의 값과 `chess`의 정보를 뒤바꾼다.
  - 그렇지 않다면 방향을 반시계 방향으로 45도씩 전환하면서 가장 먼저 갈 수 있는 곳으로 이동한다.

<br/>


# 5. 전체 코드
<hr>

{% highlight cpp %}
#define _CRT_SECURE_NO_WARNINGS
#include <bits/stdc++.h>
using namespace std;
using pii = pair<int, int>;

const int dx[] = { 0,-1,-1,0,1,1,1,0,-1 };
const int dy[] = { 0,0,-1,-1,-1,0,1,1,1 };

struct info {
	int x, y, d;
	bool alive;
};
vector<info> chess;
vector<vector<int>> board;
int ans;

bool oom(int x, int y) { return x < 0 || y < 0 || x >= 4 || y >= 4; }

void swap_chess(int cur, int nxt) {
	info tmp = chess[cur];
	chess[cur].x = chess[nxt].x;
	chess[cur].y = chess[nxt].y;
	chess[nxt].x = tmp.x;
	chess[nxt].y = tmp.y;
}

void move_chess() {
	for (int i = 1; i <= 16; i++) {
		if (!chess[i].alive) continue;

		auto& cur = chess[i];

		int x = cur.x;
		int y = cur.y;
		int d = cur.d;
		int nx = x + dx[d];
		int ny = y + dy[d];

		bool flag = false;

		if (!oom(nx, ny)) {
			if (board[nx][ny] == 0) {
				flag = true;
				cur.x = nx;
				cur.y = ny;
				board[nx][ny] = i;
				board[x][y] = 0;
			}
			else if (board[nx][ny] != -1) {
				flag = true;
				swap_chess(i, board[nx][ny]);
				swap(board[x][y], board[nx][ny]);
			}
		}

		if (!flag) {
			int nd = d + 1;
			if (nd == 9) nd = 1;

			nx = x + dx[nd];
			ny = y + dy[nd];

			while (nd != d) {
				if (!oom(nx, ny)) {
					if (board[nx][ny] == 0) {
						cur.x = nx;
						cur.y = ny;
						board[nx][ny] = i;
						board[x][y] = 0;
						chess[i].d = nd;
						break;
					}
					else if (board[nx][ny] != -1) {
						swap_chess(i, board[nx][ny]);
						swap(board[x][y], board[nx][ny]);
						chess[i].d = nd;
						break;
					}
				}

				nd++;
				if (nd == 9) nd = 1;

				nx = x + dx[nd];
				ny = y + dy[nd];
			}
		}
	}
}

void solve(int x, int y, int dir, int sum) {
	ans = max(ans, sum);

	vector<vector<int>> cpy_board = board;
	vector<info> cpy_chess = chess;
	move_chess();

	for (int i = 1; i < 4; i++) {
		int nx = x + dx[dir] * i;
		int ny = y + dy[dir] * i;

		if (oom(nx, ny)) break;
		if (board[nx][ny] == 0) continue;

		int num = board[nx][ny];
		int nd = chess[num].d;

		board[x][y] = 0;
		board[nx][ny] = -1;
		chess[num].alive = false;

		solve(nx, ny, nd, sum + num);

		board[x][y] = -1;
		board[nx][ny] = num;
		chess[num].alive = true;
	}

	chess = cpy_chess;
	board = cpy_board;
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	// freopen("input.txt", "r", stdin);

	chess.assign(17, {});
	board.assign(4, {});

	for (int i = 0; i < 4; i++) {
		for (int j = 0; j < 4; j++) {
			int p, d;
			cin >> p >> d;
			chess[p] = { i, j, d, true };
			board[i].push_back(p);
		}
	}

	int num = board[0][0];
	int d = chess[num].d;
	chess[num].alive = false;
	board[0][0] = -1;

	solve(0, 0, d, num);
	cout << ans;

	return 0;
}
{% endhighlight %}

<br/>

# 6. 소감
<hr>

- 체스의 정보, 보드의 정보를 가지치기 하고 되돌리는 걸 생각하는 게 까다로운 문제였다.
- 나는 벡터의 얕은 복사를 이용해 4줄로 구현했다.
  - 다른 사람 코드 보니까 2차원 배열을 2중 for문으로 복사했었다.
- 말들의 정보와 해당 칸의 말의 번호를 저장한 배열 `board`
  - 이걸 선언하지 않으면 `i`번째 말을 이동할 때 등 매번 탐색을 진행해야 한다.
  - 매우 유용한 스킬이 될 거 같다.