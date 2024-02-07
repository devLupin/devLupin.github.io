---
layout: post
title: "Codetree. 윷놀이 사기단"
date: 2024-02-06
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/woodstick-fraud) <br/><br/>


# 1. 문제설명
<hr>

- 윷놀이 판이 주어진다.
  - 문제 속 그림의 윷놀이 판은 고정된 값이다.
  - 입력받는 것이 아니다!
- 던질 수 있는 횟수 10회가 주어지며 각각은 이동 칸 수가 된다.
- 이동 횟수에 나갈 말의 종류를 잘 조합하여 얻을 수 있는 점수의 최대값을 얻는다.
- 규칙
  - 최초 4개의 말이 주어진다.
  - 오직 시작점과 도착점에만 여러개의 말을 놓을 수 있고, 그 외에는 하나의 말만 존재할 수 있다.
  - 만약 파란색 칸에서 이동을 시작한다면 빨간색 화살표 방향으로 이동한다.
  - 말이 이동을 마칠 때마다 해당 칸의 숫자가 점수가 된다. 

<br/>


# 2. 입/출력
<hr>

- 이동 칸 수 10개가 입력으로 주어진다.
- 이동 횟수에 나갈 말의 종류를 잘 조합하여 얻을 수 있는 점수의 최대값 구하라!

<br/>


# 3. 문제 분류
- Simulation, Backtracking

<br/>


# 4. 알고리즘 설계
<hr>

- 윷놀이 판
  - 고정된 값이므로, 배열에 값을 저장해준다.
- 백트래킹
  - 10번의 횟수를 채울 때까지 반복해서 진행된다.
  - `n`번째 시뮬레이션 마다 0~4번 말을 선택해서 움직일 수 있기 때문에 모두 재귀를 통해 시뮬레이션한다.
  -  파란색 칸인 경우 회전된 방향으로 가야하므로, `turn[]`배열을 별도로 선언하였다.
    - 이 경우 위치를 틀어 한칸 이동한 후 이동횟수 `move`를 -1한다.
  - 도착지점이 아니면서 이미 다른 말이 놓여져있다면 놓을 수 없다.
  - 그렇지 않다면, 도착 위치를 갱신해주고, 방문했다고 표시한다.


<br/>

# 5. 전체 코드
<hr>

{% highlight cpp %}
#define _CRT_SECURE_NO_WARNINGS
#include <bits/stdc++.h>
using namespace std;

const int SZ = 35;

int ans, board[SZ], route[SZ], turn[SZ], dice[10], piece[4];
bool vis[SZ];

void init() {
	for (int i = 0; i < 21; i++) route[i] = i + 1;

	route[21] = 21;
	for (int i = 22; i < 27; i++) route[i] = i + 1;

	route[27] = 20; 
	route[28] = 29; 
	route[29] = 30;
	route[30] = 25; 
	route[31] = 32; 
	route[32] = 25;

	turn[5] = 22; 
	turn[10] = 31; 
	turn[15] = 28;

	for (int i = 0; i < 21; i++) board[i] = 2 * i;

	board[22] = 13; 
	board[23] = 16; 
	board[24] = 19;
	board[25] = 25; 
	board[26] = 30; 
	board[27] = 35;
	board[28] = 28; 
	board[29] = 27; 
	board[30] = 26;
	board[31] = 22; 
	board[32] = 24;
}

void solution(int depth, int sum) {
	if (depth == 10) {
		ans = max(ans, sum);
		return;
	}

	for (int i = 0; i < 4; i++) {
		int prev = piece[i];
		int nxt = prev;
		int move = dice[depth];

		if (turn[nxt] > 0) {
			nxt = turn[nxt];
			move--;
		}

		while (move--) nxt = route[nxt];

		if (nxt != 21 && vis[nxt]) continue;

		vis[prev] = false;
		vis[nxt] = true;
		piece[i] = nxt;

		solution(depth + 1, sum + board[nxt]);

		vis[prev] = true;
		vis[nxt] = false;
		piece[i] = prev;
	}
}

int main() {
	ios::sync_with_stdio(0);
	cin.tie(0);

	// freopen("input.txt", "r", stdin);

	for (int i = 0; i < 10; i++) cin >> dice[i];

	init();
	solution(0, 0);

	cout << ans;
	return 0;
}
{% endhighlight %}

<br/>

# 4. 소감
<hr>

- 윷놀이 판의 위치 때문에 어느정도 하드코딩이 필요한 모습을 볼 수 있다.
- 이런 문제의 경우 인덱스 하나로 정답이 나오지 않아 맞왜틀을 시전할 수 있으니,
- 입력 부분에 주의가 필요하다.