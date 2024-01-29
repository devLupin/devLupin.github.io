---
layout: post
title: "Codetree. 생명과학부 랩 인턴"
date: 2024-01-29
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/biology-lab-intern) <br/><br/>


# 1. 문제설명
<hr>

- `n * m` 크기의 격자판에 곰팡이 정보가 주어진다.
- 곰팡이의 정보는 `x, y, s, d, b`로 표현된다.
  - 각각은 (x, y)인 위치, 1초동안 움직이는 거리, 이동 방향, 곰팡이의 크기
- 첫번째 열부터 아래로 열 개수만큼 탐색 진행
  - `i`번째 열에서 아래로 내려가며 가장 빨리 발견한 곰팡이를 채취한다.
    - 채취하고 나면 빈칸이 된다.
    - 채취할 곰팡이가 `i`번째 열에 없을 수도 있다.
  - 남은 곰팡이가 입력으로 주어진 방향과 속력으로 이동한다.
    - 만약 벽에 도달하면 방향을 반대로 바꾼다.
    - 방향을 바꿀 때는 시간이 소모되지 않는다.
  - 모든 곰팡이의 이동이 끝나고 같은 칸에 여러 개의 곰팡이가 존재할 수 있다.
    - 이 때, 곰팡이의 크기 `b`가 큰 녀석만 살아남는다.
- 각 열에서의 탐색은 1초가 걸리며, 열 개수만큼만 탐색을 진행하고 종료한다.
- 모든 열을 검사했을 때, 채취한 곰팡이 크기의 총합을 구하라!


- 문제 분류
  - Implementation, dx/dy,


<br/>

# 2. 알고리즘 설계
<hr>

- 문제의 내용을 그대로 구현해주면 되는데 헷갈릴 만한 포인트가 몇 가지 있었다.
- 가장 빨리 발견한 곰팡이를 채취할 때
  - 어떤 조건도 없이 그냥 인덱스 위치가 가장 앞에 있는 걸 잡아먹는다.
- 같은 칸에 여러 개의 곰팡이가 존재할 때
  - 크기가 같으면 어쩌나 고민했지만, 문제에 크기가 전부 다르다고 명시되어 있음.
- 벽을 만나면 방향이 정반대가 되는데,
  - 바뀐 방향이 유지되어야 한다는 것!
  - 위로 올라가다 벽을 만나면 방향이 아래로 바뀌게 되고
  - 향후 시뮬레이션에서도 이 아래 방향으로 시작해야 한다!
- 언급된 점들만 주의하면, 문제없이 구현할 수 있었다.


<br/>

# 3. 전체 코드
<hr>

{% highlight cpp %}
#define _CRT_SECURE_NO_WARNINGS
#include <bits/stdc++.h>
using namespace std;

struct info {
	int x, y, s, d, b;
};

const int dx[] = { 0,-1,1,0,0 };
const int dy[] = { 0,0,0,1,-1 };

int n, m, k, ans;
int A[105][105];
vector<info> v;

void collect(int x, int y) {
	A[x][y] = 0;

	for (int i = 0; i < v.size(); i++) {
		if (v[i].x == x && v[i].y == y) {
			v[i].x = -1;
			v[i].y = -1;
			ans += v[i].b;
			break;
		}
	}
}

bool oom(int x, int y) { return x < 1 || y < 1 || x > n || y > m; }

void move() {
	for (int i = 0; i < v.size(); i++) {
		auto& nxt = v[i];

		if (nxt.x == -1 && nxt.y == -1) continue;

		int x = nxt.x, y = nxt.y;
		int dir = nxt.d;
		int s = nxt.s;

		A[x][y]++;

		while (s--) {
			x += dx[dir];
			y += dy[dir];

			if (oom(x, y)) {
				x -= dx[dir];
				y -= dy[dir];
				s++;

				if (dx[dir] != 0)
					dir = dx[dir] > 0 ? dir - 1 : dir + 1;

				else if (dy[dir] != 0)
					dir = dy[dir] > 0 ? dir + 1 : dir - 1;
			}
		}

		v[i].x = x;
		v[i].y = y;
		v[i].d = dir;
		A[x][y]--;
	}
}

void eat() {
	vector<info> tmp;

	for (int i = 1; i <= n; i++)
		for (int j = 1; j <= m; j++)
			if (A[i][j] < -1) {
				tmp.push_back({ i,j,0,0,0 });
				A[i][j] = -1;
			}

	for (auto& nxt : tmp) {
		if (nxt.x == -1 && nxt.y == -1) continue;

		vector<int> idx;
		int mx = 0;

		for (int i = 0; i < v.size(); i++) {
			auto& cmp = v[i];

			if (nxt.x == cmp.x && nxt.y == cmp.y) {
				idx.push_back(i);
				mx = max(mx, cmp.b);
			}
		}

		for(int i : idx)
			if (v[i].b < mx) {
				v[i].x = -1;
				v[i].y = -1;
			}
	}
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	// freopen("input.txt", "r", stdin);

	cin >> n >> m >> k;

	while (k--) {
		int x, y, s, d, b;
		cin >> x >> y >> s >> d >> b;
		v.push_back({ x,y,s,d,b });
		A[x][y] = -1;
	}

	for (int j = 1; j <= m; j++) {
		for (int i = 1; i <= n; i++) {
			if (A[i][j] == -1) {
				collect(i, j);
				break;
			}
		}

		move();
		eat();
	}

	cout << ans;
	
	return 0;
}
{% endhighlight %}

<br/>

# 4. 소감
<hr>

- 잡아먹힌 곰팡이를 (-1,-1)로 처리하였다.
  - 지울 벡터의 인덱스를 기억했다가 erase로 처리해도 좋았을 거 같다.
- 거의 모든 함수에서 모든 곰팡이를 탐색한다.
  - 이거에 대한 효율성에 대해 고민한다면 시간을 조금 더 줄일 수 있을 거 같다.
  - 내 코드 수행 시간은 451ms인데, 찾아보니 82ms로 해결한 사람도 있었다.