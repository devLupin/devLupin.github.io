---
layout: post
title: "Codetree. 승자독식 모노폴리"
date: 2024-02-14
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/odd-monopoly) <br/><br/>


# 1. 문제설명
<hr>

- `NxN` 크기의 격자칸에서 모노폴리 게임을 진행한다.
- `M`개의 플레이어로 구성되어 있고, 각각 격자칸에서 한 자리를 차지한다.
- 게임의 규칙
  - 턴이 한번 진행될 때 각 플레이어들이 한칸씩 동시에 이동한다.
    - 이동한 칸을 `K`턴동안 독점 계약한다.
	- 초기 상태에 플레이어가 위치한 땅 역시 해당 플레이어가 독점 계약한 칸이다.
  - `K`턴이 지나면 다시 주인이 없는 땅으로 돌아간다.
  - 플레이어의 이동
    - 각 플레이어는 이동 우선순위를 갖는다.
	- 인접한 상하좌우 4칸 중 아무도 독점계약을 맺지 않은 칸으로 이동
	- 만약 이러한 칸이 없는 경우 본인이 독점계약한 땅으로 이동
	- 이러한 칸이 여러 칸 이라면 이동 우선순위에 따라 칸을 결정
  - 이동 후 하나의 땅에 두명 이상의 플레이어가 있는 경우, 번호가 가장 작은 플레이어만 살아남는다.
- 위 과정이 계속 반복될 때 1번 플레이어만 살아남기까지 걸린 턴의 수를 계산하라


<br/>


# 2. 입/출력
<hr>

- 입력
  - 격자의 크기 `N`, 플레이어의 수 `M`, 계약 턴 수 `K`
  - `2 ~ N+1`번째 줄까지 격자의 정보
    - `0`은 빈칸, 자연수는 `p`번 플레이어의 위치
  - `N+2`번째 줄에는 각 플레이어의 초기 방향 `d`가 주어진다.
    - 1, 2, 3, 4 각각은 위, 아래, 왼쪽, 오른쪽
  - `N+3 ~ M*4`번째 줄에는 각 플레이어의 방향에 따른 이동 우선순위
    - 첫번째 줄은 위를 향했을 때의 이동 우선순위, 두번째 줄은 아래를 향했을 때의 우선순위, ...
- 출력 : 1번 플레이어만 남게 되기까지 걸린 턴의 수

<br/>


# 3. 문제 분류
- Simulation

<br/>


# 4. 알고리즘 설계
<hr>

- 자료형 및 변수 선언
  - 플레이어의 정보를 담기 위한 구조체 `info`
    - `alive` : 플레이어가 죽었다면 `false`
	- `dir` : 최초 방향
	- `dirs` : 입력받은 우선순위 방향 (4x4)
  - 플레이어의 위치 `(x, y)`를 찾기 위한 배열 `pos`
  - 맵의 정보를 담은 배열 `board`
    - `first` : 플레이어 번호
	- `second` : 해당 칸의 남은 턴 수
  - 플레이어의 정보를 담은 배열 `players`
  - 플레이어가 이동한 후 좌표 정보 `nxt_board`

	```cpp
	struct info {
    	bool alive;
    	int dir;
    	vector<vector<int>> dirs;
    };
    
    pii pos[SZ * SZ];
    pii board[SZ][SZ];
    vector<info> players;
    vector<pii> nxt_board[SZ][SZ];
	```

- 프로세스 총 1000번 실행
  - `if(1번 플레이어가 죽었다면) break;`
  - `if(1번을 제외한 모든 플레이어가 죽었다면) 정답값 = 실행 횟수; break;`
  - 플레이어 이동
  - 턴 감소
  - 플레이어 킬

1. 플레이어의 죽음 여부 

  ```cpp
  // alive는 플레이어의 생존 여부를 담고있는 boolean 값
  for (int i = 1; i <= M; i++)
      if (players[i].alive) return false;
      return true;
  ```

2. 플레이어 이동
  - 핵심 부분을 살펴보면
    - 처음에는 주변에 0인 칸이 있는지 찾고
	- 없다면 -1이 리턴되어 본인이 방문했던 칸을 찾는다.
    - 참고로 본인이 방문했던 칸이 없을 수는 없다.
	  - 처음에 모든 플레이어의 위치가 독립적이므로

  ```cpp
  for(cur : players) {
  	if(cur.alive) continue // 죽었으면 탐색하지 않음.
  
    /**이 부분이 핵심!**/
  	nd = next_direction(i, 0);  // 주변에 아무도 들르지 않은 칸(0)이 있다면
  	if(nd == -1) nd = next_direction(i, i);  // 아무도 들르지 않은 칸이 없다면 본인이 방문했던 칸
  
  	nx = x + dx[nd];
  	ny = y + dy[nd];
  
  	nxt_board[nx][ny].push_back({i, k});
  	cur.dir = nd;
  }
  ```

  ```cpp
  int next_direction(int idx, int target) {
	int x = pos[idx].x;
	int y = pos[idx].Y;
	int nd = players[idx].dir;

	for(int dir : 1~4) {
		int ndir = cur.dir[nd][dir];
		int nx = x + dx[ndir];
		int ny = y + dy[ndir];

		if(out of memory) continue;
		if(board[nx][ny].first == target) return ndir;
	}

	return -1;
  }
  ```

3. 턴 감소
  - `board[i][j]`에는 `{지나간 플레이어의 번호, 남은 턴 수}`가 기록되어 있다.
  - 남은 턴 수가 0보다 크면 1을 빼주고, 이 값이 0이 되었다면 플레이어의 번호를 0으로 만들어준다.

4. 플레이어 킬
  - 앞서 `2. 플레이어 이동`에서 `nxt_board`에 다음 보드판을 기록하였다.
  - `nxt_board[x][y]`의 원소의 수가 0보다 크다면 해당 위치에 플레이어가 있다는 의미
    - 1명의 플레이어만 있으면 `board`, `pos`를 갱신해주면 되고
    - 2명 이상이면 플레이어의 번호가 적은순으로 정렬해 0번째 인덱스를 제외한 나머지 플레이어의 `alive`를 `false`로 지정한다.
  - 이렇듯 `nxt_board`의 값은 매 턴마다 초기화 되어야 한다.


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

int N, M, K, p, d, d1, d2, d3, d4;

const int SZ = 25;
const int dx[] = { 0,-1,1,0,0 };
const int dy[] = { 0,0,0,-1,1 };

struct info {
	bool alive;
	int dir;
	vector<vector<int>> dirs;
};

pii pos[SZ * SZ];
pii board[SZ][SZ];
vector<info> players;
vector<pii> nxt_board[SZ][SZ];

void input() {
	cin >> N >> M >> K;
	for (int i = 1; i <= N; i++) {
		for (int j = 1; j <= N; j++) {
			cin >> p;
			if (p > 0) {
				pos[p] = { i, j };
				board[i][j] = { p, K };
			}
			else board[i][j] = { 0,0 };
		}
	}

	players.assign(M + 1, {});
	for (int i = 1; i <= M; i++) {
		cin >> d;
		players[i].alive = true;
		players[i].dir = d;
	}

	for (int i = 1; i <= M; i++) {
		players[i].dirs.assign(5, vector<int>(5));

		for (int j = 1; j <= 4; j++) {
			cin >> d1 >> d2 >> d3 >> d4;
			players[i].dirs[j] = { 0, d1, d2, d3, d4 };
		}
	}
}

bool is_alive() {
	for (int i = 2; i <= M; i++)
		if (players[i].alive) return false;
	return true;
}

bool oom(int x, int y) { return x < 1 || y < 1 || x > N || y > N; }

int next_direction(int i, int target) {
	info& cur = players[i];
	int x = pos[i].X;
	int y = pos[i].Y;
	int nd = cur.dir;

	for (int dir = 1; dir <= 4; dir++) {
		int ndir = cur.dirs[nd][dir];
		int nx = x + dx[ndir];
		int ny = y + dy[ndir];

		if (oom(nx, ny)) continue;
		if (board[nx][ny].first == target) return ndir;
	}

	return -1;
}

void move_player() {
	for (int i = 1; i <= M; i++) {
		if (!players[i].alive) continue;

		info& cur = players[i];
		int nx, ny, nd;

		nd = next_direction(i, 0);
		if (nd == -1) nd = next_direction(i, i);

		nx = pos[i].X + dx[nd];
		ny = pos[i].Y + dy[nd];

		nxt_board[nx][ny].push_back({ i, K });

		cur.dir = nd;
	}
}

void reduce_turn() {
	for (int i = 1; i <= N; i++) {
		for (int j = 1; j <= N; j++) {
			if (board[i][j].second > 0) {
				board[i][j].second--;
				if (board[i][j].second == 0)
					board[i][j] = { 0,0 };
			}
		}
	}
}

void kill_player() {
	for (int i = 1; i <= N; i++) {
		for (int j = 1; j <= N; j++) {
			auto& nxt = nxt_board[i][j];
			if (nxt.size() == 1) {
				board[i][j] = nxt[0];
				pos[nxt[0].first] = { i, j };
			}
			else if (nxt.size() > 1) {
				sort(nxt.begin(), nxt.end());
				board[i][j] = nxt[0];
				pos[nxt[0].first] = { i, j };

				for (int idx = 1; idx < nxt.size(); idx++) {
					int num = nxt[idx].first;
					players[num].alive = false;
				}
			}
		}
	}
}

void init() {
	for (int i = 1; i <= N; i++)
		for (int j = 1; j <= N; j++)
			nxt_board[i][j].clear();
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	// freopen("input.txt", "r", stdin);
	input();

	int ans = -1;
	for(int t=0; t<=1000; t++) {
		if (!players[1].alive) break;
		if (is_alive()) {
			ans = t;
			break;
		}

		init();

		move_player();
		reduce_turn();
		kill_player();
	}

	cout << ans;

	return 0;
}
{% endhighlight %}

<br/>

# 6. 소감
<hr>

- 문제를 잘못 이해해서 오랜시간 맞왜틀을 시전했다.
- **매순간마다 방향이 바뀌는 게 핵심이었다.**
  - 즉, 현재 바라보고 있는 방향인 `players[i].dir`번째의 우선순위 방향을 통해 움직일 때마다 방향이 바뀔 수 있다.