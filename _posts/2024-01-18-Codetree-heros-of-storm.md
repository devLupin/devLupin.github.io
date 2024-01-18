---
layout: post
title: "Codetree. 시공의 돌풍"
date: 2024-01-18
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/heros-of-storm) <br/><br/>


# 1. 문제설명
<hr>

- `n * m` 크기의 격자에 먼지와 시공의 돌풍이 존재한다.
- 시공의 돌풍은 항상 1번 열에 설치되어 있고, 두 칸을 차지한다.
- 1초 동안 다음과 같은 일이 일어난다.
  - 먼지가 인접한 4방향의 상,하,좌,우 칸으로 확산된다.
    - 방의 범위를 벗어나거나, 시공의 돌풍이 있는 곳은 확산되지 않는다.
	- 확산되는 양은 `원래 칸의 먼지 양/5`이며 소수점은 버려진다.
	- 각 칸에 확산될 때마다 원래 칸의 먼지의 양은 확산된 먼지만큼 줄어든다.
	- 확산된 먼지는 **모든 먼지가 확산을 끝낸 다음에 해당 칸에 더해진다.**
  - 시공의 돌풍이 청소를 시작한다.
    - 시공의 돌풍의 윗칸에서는 반시계 방향으로 바람을 일으키며, 
	- 아랫칸에서는 시계 방향으로 바람을 일으킨다.
	- 바람이 불면 먼지가 나온 바람의 방향대로 모두 한 칸씩 이동한다.
	- 시공의 돌풍에서 나온 바람은 먼지가 없는 청정한 바람이고 시공의 돌풍으로 들어간 먼지는 사라진다.
- `t`초후에 먼지의 양을 계산하라


- 문제 분류
  - Implementation, Simulation, dx/dy


<br/>

# 2. 알고리즘 설계
<hr>

- 크게 먼지의 확산, 청소 부분으로 구현하였다.
- 먼지의 확산
  - 주의할 점은 모든 먼지가 확산을 끝낸 다음에 더해진다는 것이다.
  - 임시 배열을 선언해 해당 배열에는 더해주고 빼줘야 할 값을 갱신해주었다.
    
	```cpp
	// cur : (원래 칸의 먼지의 양 / 5)
	B[i][j] -= cur;
	B[nx][ny] += cur;
	```
  
  - 이후 B의 값을 A에 더해주면 확산을 끝낸 다음 해당 칸에 더해지는 효과를 볼 수 있다.

    ```cpp
	for (int i = 0; i < n; i++)
		for (int j = 0; j < m; j++)
			A[i][j] += B[i][j];
	```

- 청소
  - 배열에 격자를 입력받을 때 시공의 돌풍 위쪽이 반드시 먼저 입력 된다.
    - i, j에 의해서 좌측 상단이 먼저 입력됨.
  - 격자의 값이 `-1` 일 때, `vector<pii> cleaner`에 저장해주면, 위쪽과 아래쪽이 순서대로 입력된다.
  - 문제에 되게 길게 설명되어 있지만, 한칸 씩 해당 방향으로 밀면된다.
    - 밀었을 때 가장 마지막 부분은 시공의 돌풍으로 들어가기 때문에 제외하고,
    - 밀었을 때 첫 부분은 0으로 지정한다.(문제 속 청정한 바람)

- 미는 방향이 서로 다른 시공의 돌풍
  - 시공의 돌풍은 2칸을 차지하며, 위쪽은 반시계 아래쪽은 시계 방향으로 움직인다.
  - `clean` 함수를 선언하고, 상단과 하단의 돌풍의 이동경로를 인자로 전달하였다.

    ```cpp
	while (t--) {
		spread();
		clean(cleaner[0], { 3,0,2,1 });
		clean(cleaner[1], { 3,1,2,0 });
	}
	```
  
  - 이동 경로에 따른 값을 벡터에 담는다.
    - Out of Map(OOM)이 발생할 때마다 경로가 꺾이는 형태로 구현하였다.
	- `while`문이 종료되면 벡터의 마지막 원소는 이동 경로의 마지막 값 또는 시공의 돌풍(-1)
	- 예를 들어 시공의 돌풍이 `(n-2, 0)`, `(n-1, 0)`에 위치했다고 가정해보자
	  - `n-1`의 돌풍은 가장 마지막 줄 왼쪽부터 오른쪽을 저장하고 종료된다.
	  - 이렇게 되면, 벡터의 마지막 값이 시공의 돌풍이 아닌 이동 경로의 마지막 값이 된다.
	  - 그 외의 경우 하단 시공의 돌풍의 경로 값을 저장하면 `{..., 마지막 이동경로의 값, -1}`의 형태가 된다.
	  - 그래서 아래와 같이 마지막 원소가 -1이면 벡터의 원소를 한번 더 제거하였다.

    ```cpp
	int idx = 0;
	int dir = dirs[idx];
	vector<int> v;
	int x = start.X + dx[dir];
	int y = start.Y + dy[dir];

	v.push_back(A[x][y]);

	while (A[x][y] != -1 && idx < 4) {
		x += dx[dir];
		y += dy[dir];

		if (oom(x, y)) {
			x -= dx[dir];
			y -= dy[dir];
			idx++;
			dir = dirs[idx];
			continue;
		}

		v.push_back(A[x][y]);
	}
	if (A[x][y] == -1) v.pop_back();
	v.pop_back();
	```

  - 한칸씩 밀어서 이동시킨다.
    - 위의 벡터에 담는 로직을 거의 그대로 활용하였다.
	- 시공의 돌풍 처음 이동할 칸은 무조건 0이다.
	  - 문제에서 시공의 돌풍이 1번 열에 위치한다고 명시했기 때문에
    - OOM이 발생할 때마다 경로를 꺾어주면서 벡터의 값을 저장시켜주면 된다.
	  - `for(auto nxt : v)`의 형태는 사용할 수 없다.
	  - oom이 발생하면 현재 벡터의 위치 값(`i`)을 `--`해야 정상작동하기 때문이다.

    ```cpp
	// 초기화 필요
	idx = 0;
	dir = dirs[idx];
	x = start.X + dx[dir];
	y = start.Y + dy[dir];

	A[x][y] = 0;

	for(int i=0; i<v.size(); i++) {
		x += dx[dir];
		y += dy[dir];

		if (oom(x, y)) {
			x -= dx[dir];
			y -= dy[dir];
			idx++;
			dir = dirs[idx];
			i--;
			continue;
		}
		A[x][y] = v[i];
	}
	```


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

int n, m, t, A[1001][1001], B[1001][1001];
vector<pii> cleaner;

const int dx[] = { -1,1,0,0 };
const int dy[] = { 0,0,-1,1 };

bool oom(int x, int y) { return x < 0 || y < 0 || x >= n || y >= m; }

void spread() {
	for (int i = 0; i < n; i++)
		memset(B[i], 0, sizeof(B[i]));

	for (int i = 0; i < n; i++) {
		for (int j = 0; j < m; j++) {
			int cur = A[i][j] / 5;

			for (int dir = 0; dir < 4; dir++) {
				int nx = i + dx[dir];
				int ny = j + dy[dir];

				if (!oom(nx, ny) && A[nx][ny] != -1) {
					B[i][j] -= cur;
					B[nx][ny] += cur;
				}
			}
		}
	}

	for (int i = 0; i < n; i++)
		for (int j = 0; j < m; j++)
			A[i][j] += B[i][j];
}

void clean(pii start, vector<int> dirs) {
	int idx = 0;
	int dir = dirs[idx];
	vector<int> v;
	int x = start.X + dx[dir];
	int y = start.Y + dy[dir];

	v.push_back(A[x][y]);

	while (A[x][y] != -1 && idx < 4) {
		x += dx[dir];
		y += dy[dir];

		if (oom(x, y)) {
			x -= dx[dir];
			y -= dy[dir];
			idx++;
			dir = dirs[idx];
			continue;
		}

		v.push_back(A[x][y]);
	}
	if (A[x][y] == -1) v.pop_back();
	v.pop_back();

	idx = 0;
	dir = dirs[idx];
	x = start.X + dx[dir];
	y = start.Y + dy[dir];

	A[x][y] = 0;

	for(int i=0; i<v.size(); i++) {
		x += dx[dir];
		y += dy[dir];

		if (oom(x, y)) {
			x -= dx[dir];
			y -= dy[dir];
			idx++;
			dir = dirs[idx];
			i--;
			continue;
		}
		A[x][y] = v[i];
	}
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	// freopen("input.txt", "r", stdin);

	cin >> n >> m >> t;
	for (int i = 0; i < n; i++) {
		for (int j = 0; j < m; j++) {
			cin >> A[i][j];
			if (A[i][j] == -1) cleaner.push_back({ i,j });
		}
	}

	while (t--) {
		spread();
		clean(cleaner[0], { 3,0,2,1 });
		clean(cleaner[1], { 3,1,2,0 });
	}

	int ans = 0;
	for (int i = 0; i < n; i++)
		for (int j = 0; j < m; j++)
			ans += (A[i][j] > 0) ? A[i][j] : 0;

	cout << ans;
	
	return 0;
}
{% endhighlight %}

<br/>

# 4. 소감
<hr>

- 이런 류의 문제는 격자가 변화하는 것을 출력하면서 구현하는 게 편하다.

```cpp
void print() {
	for (int i = 0; i < n; i++) {
		for (int j = 0; j < m; j++)
			cout << A[i][j] << ' ';
		cout << '\n';
	}
	cout << '\n';
}
```

- 다음과 같이 루프 한번마다 `break`를 걸어서 값의 변화를 확인한다.

```cpp
while (t--) {
	spread();
	print();
	clean(cleaner[0], { 3,0,2,1 });
	clean(cleaner[1], { 3,1,2,0 });
	print();
	break;
}
```

- 예제2의 입력을 살펴보면 수행횟수(`t`)가 1000이다.
  - 만약, 548번째 루프에서 고려하지 않은 코너 케이스가 있다면...
  - 디버깅하는 것을 상상조차 하기 싫다..

- 시뮬레이션 문제의 경우, 문제를 대충 읽으면 놓치기 쉬운 부분을 코너 케이스로 넣는 경우가 많았다.
  - 예를 들면, 앞서 언급한 `시공의 돌풍이 `(n-2, 0)`, `(n-1, 0)`에 위치했다`의 경우이다.
  - 그 외의 경우에는 벡터의 마지막 원소 2개를 제거하면 되기 때문에 `pop_back()`을 단순히 2번 수행하면 문제가 생긴다.
- 나의 경우 이런 부분을 간과하여 디버깅의 고통을 받았다.