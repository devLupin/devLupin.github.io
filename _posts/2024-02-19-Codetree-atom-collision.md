---
layout: post
title: "Codetree. 원자 충돌"
date: 2024-02-20
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/atom-collision) <br/><br/>


# 1. 문제설명
<hr>

- `n * n` 격자판에 `m`개의 원자를 두고 충돌 실험을 진행한다.
- `m`개의 원자는 각각 질량, 방향, 속력, 초기 위치를 가지고 있으며 방향은 상하좌우, 대각선으로 주어진다.
- 격자의 모든 행,열은 각각 끝과 끝이 연결되어 있다.
- 실험 과정
  - 모든 원자는 1초가 지날 때마다 자신의 방향으로 자신의 속력만큼 이동
  - 이동이 모두 끝난 뒤에 하나의 칸에 2개 이상의 원자가 있는 경우
    - 같은 칸에 있는 원자들은 각각의 질량과 속력을 모두 합한 하나의 원자로 합쳐진다.
	- 이후 합쳐진 원자는 4개의 원자로 나눠진다.
      - 질량은 합쳐진 원자의 질량에 5를 나눈 값
	  - 속력은 합쳐진 원자의 속력에 합쳐진 원자의 개수를 나눈 값
	  - 방향은 합쳐지는 원자의 방향이 모두 상하좌우 중 하나이거나 대각선 중에 하나이면, 각각 상하좌우의 방향을 가지며 그렇지 않다면 대각선 네 방향의 값
	- 질량이 0인 원소는 소멸

<br/>


# 2. 입/출력
<hr>

- 입력
  - 격자의 크기 `n`, 원자의 개수 `m`, 실험 시간 `k`
  - 원자의 정보는 위치 `x`, `y`, 질량 `m`, 속력 `s`, 방향 `d`
    - 방향 `d`는 12시부터 시계방향으로 11시까지

- 출력
  - k초 이후 남아있는 원자들의 질량의 합

<br/>


# 3. 문제 분류
- Simulation

<br/>


# 4. 알고리즘 설계
<hr>

- 크게 원자를 움직이는 부분과 합치는 부분으로 구현하였다.
- 움직이기
  - 입력받은 `s`만큼 이동을 하게된다.
  - 맵 밖으로 벗어나면 그 반대방향의 첫 번째 칸으로 이동하게 된다.
  - `x + dx[dir]` 부분에 `맵의 크기 * 칸수`의 배수를 곱해주면 한번에 계산할 수 있다.
  - 최종적으로 `x + dx[dir] * s + n * s`가 된다.
  - 이렇게 이동한 결과를 `vector<int> board[x][y]`에 담는다.
- 합치기
  - `board[x][y]`의 크기가 2이상이면 2개 이상의 원자가 같은 자리에 있다는 뜻
  - 크기가 0이면 무시하고 1이라면 하나밖에 없으니 그대로 복사한다.
  - 2 이상인 경우,
    - boolean 값 두개를 이용해 대각선 방향인지 상하좌우 방향인지 체크하였다.
      - 둘다 `true`라면 방향이 섞여있으므로 대각선을 취하고 그렇지 않은 경우 상하좌우가 된다.
    - 방향을 상수로 전달해서 각 방향에 대한 값을 복사해주었다. 
	
	  ```cpp
	  bool chk1 = false, chk2 = false;

	  for (int nxt : board[i][j]) {
	  	if (v[nxt].d % 2 == 0) chk1 = true;
	  	else chk2 = true;
	  }

	  if (chk1 && chk2) split(i, j, { 1, 3, 5, 7 }, m, s);
	  else split(i, j, { 0, 2, 4, 6 }, m, s);
	  ```


<br/>

# 5. 전체 코드
<hr>

{% highlight cpp %}
#include <bits/stdc++.h>
using namespace std;

const int dx[] = { -1,-1,0,1,1,1,0,-1 };
const int dy[] = { 0,1,1,1,0,-1,-1,-1 };
int N, M, K;
vector<int> board[50][50];
struct info { int x, y, m, s, d; };
vector<info> v, cpy;

void move() {
	for (int i = 0; i < v.size(); i++) {
		auto& nxt = v[i];
		int x, y, s, d;
		tie(x, y, s, d) = { nxt.x, nxt.y, nxt.s, nxt.d };

		int nx = (x + dx[d] * s + N * s) % N;
		int ny = (y + dy[d] * s + N * s) % N;

		nxt.x = nx;
		nxt.y = ny;
		board[nx][ny].push_back(i);
	}
}

void split(int x, int y, vector<int> dirs, int m, int s) {
	if (m == 0) return;

	for (int d : dirs)
		cpy.push_back({ x, y, m, s, d });
}

void synthesize() {
	cpy.clear();

	for (int i = 0; i < N; i++) {
		for (int j = 0; j < N; j++) {
			int sz = board[i][j].size();

			if (sz == 0) continue;
			
			else if(sz == 1) cpy.push_back(v[board[i][j][0]]);

			else {
				bool chk1 = false, chk2 = false;
				int sumM = 0, sumS = 0;

				for (int nxt : board[i][j]) {
					sumM += v[nxt].m;
					sumS += v[nxt].s;

					if (v[nxt].d % 2 == 0) chk1 = true;
					else chk2 = true;
				}

				int m = sumM / 5;
				int s = sumS / board[i][j].size();

				if (chk1 && chk2) split(i, j, { 1, 3, 5, 7 }, m, s);
				else split(i, j, { 0, 2, 4, 6 }, m, s);
			}

			board[i][j].clear();
		}
	}

	v = cpy;
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	cin >> N >> M >> K;
	for (int x, y, m, s, d, i = 0; i < M; i++) {
		cin >> x >> y >> m >> s >> d;
		x--, y--;
		v.push_back({ x,y,m,s,d });
	}

	while (K--) {
		move();
		synthesize();
	}

	int ans = 0;
	for (auto& nxt : v)
		ans += nxt.m;
	cout << ans;

	return 0;
}
{% endhighlight %}

<br/>

# 6. 소감
<hr>

- 속력의 입력값이 최대 1000이라서 1칸씩 움직이니까 시간이 많이 소모되었다.
  - 수행 시간 `330ms`

  ```cpp
  while (s--) {
	x += dx[d];
	y += dy[d];

	if (x < 0) x = N - 1;
	if (y < 0) y = N - 1;
	if (x >= N) x = 0;
	if (y >= N) y = 0;
  }
  ```

- 5. 전체코드와 같이 한번의 계산으로 처리해 속도를 `22ms`까지 줄일 수 있었다.