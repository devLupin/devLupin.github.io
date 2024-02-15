---
layout: post
title: "Codetree. 자율주행 전기차"
date: 2024-02-15
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/autonomous-electric-car) <br/><br/>


# 1. 문제설명
<hr>

- 전기차와 승객이 격자칸에 위치한다.
  - 전기차는 연료 개념을 갖고 있다.
  - 승객은 출발점과 도착점이 정해져있다.
- 전기차는 맨해튼 거리가 가장 가까운 승객을 찾는다.
  - 도로의 벽으로는 이동할 수 없다.
  - 그런 승객이 여러명이라면 가장 위쪽, 그것조차 여러명이라면 가장 왼쪽에 있는 승객을 고른다.
- 승객을 찾았다면, 거리만큼 연료를 소비하며 이동한다.
  - 연료량보다 승객이 멀리 있거나 승객이 없는 경우 종료한다.
- 승객의 위치에서 승객의 도착점까지 이동한다.
  - 만약 가는 도중 연료가 다 떨어지면 그대로 종료
- 도착점까지 이동에 성공하면 소비한 연료의 두배만큼 충전된다.

<br/>


# 2. 입/출력
<hr>

- 입력
  - 격자의 크기 `N`, 승객의 수 `M`, 초기 배터리 충전량 `C`
  - `NxN` 크기의 도로 정보
    - 0은 도로, 1은 벽
  - 자율주행 전기차의 초기 위치 정보 `(x, y)`
  - 각 승객의 출발점, 도착점
- 출력 : 모든 승객을 다 태워줄 수 있는 경우 남은 연료의 양을 그렇지 않은 경우 -1을 출력

<br/>


# 3. 문제 분류
- Simulation, BFS

<br/>


# 4. 알고리즘 설계
<hr>

- 변수 및 구조체 선언
  - `status` : BFS 로직에서 `x, y, distance`를 함께 관리하기 위한 자료형
  - `info` : 승객의 입력 정보
    - `isArrived` : 승객이 도착했다면 `true`, 초기에는 `false`
  - `taxi` : 택시의 `x, y`
  - `taxiC` : 택시의 현재 남은 배터리 양

    ```cpp
    struct status { int x, y, d; };
    struct info { 
    	int sx, sy, ex, ey;
    	bool isArrived;
    };
    vector<info> passengers;
    
    pii taxi;
    int taxiC;
    ```

- 탐색 프로세스
  - 태울 승객을 찾는다.
    - 배터리 소비량 내에 승객이 있다면 `{거리, 승객 번호}`를 리턴한다.
    - 아니라면 -1을 리턴하고 종료한다.
  - 승객을 찾았다면 거리만큼 연료량을 감소시킨다.
  - 승객을 태워 목적지로 이동한다.
    - 배터리 소비량 내에 목적지가 있다면 `거리`를 리턴한다.
    - 아니라면 -1을 리턴한다.
  - 승객을 목적지로 이동하는데 성공했다면 거리만큼 연료량을 추가한다.
    - 문제에는 연료량의 두배라고 되어있지만 연료를 그때마다 소비하지 않고, 도착할 수 있는지만 확인했기 때문
- 탐색 프로세스가 종료되었다면 `isArrived`를 통해 모든 승객의 도착여부를 확인한다. 


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

int N, M, C;
int road[25][25];

struct status { int x, y, d; };
struct info { 
	int sx, sy, ex, ey;
	bool isArrived;
};
vector<info> passengers;

pii taxi;
int taxiC;

int dx[] = { -1,1,0,0 };
int dy[] = { 0,0,-1,1 };

bool oom(int x, int y) { return x < 1 || y < 1 || x > N || y > N; }

void input() {
	cin >> N >> M >> C;
	taxiC = C;

	for (int i = 1; i <= N; i++)
		for (int j = 1; j <= N; j++)
			cin >> road[i][j];

	cin >> taxi.X >> taxi.Y;

	for (int i = 0; i < M; i++) {
		int sx, sy, ex, ey;
		cin >> sx >> sy >> ex >> ey;
		passengers.push_back({ sx, sy, ex, ey, false });
	}
}

int getDistance(int sx, int sy, int ex, int ey) {
	queue<status> q;
	bool vis[25][25] = { false, };

	q.push({ sx, sy, 0 });
	vis[sx][sy] = true;

	while (!q.empty()) {
		auto cur = q.front();
		q.pop();

		if (cur.x == ex && cur.y == ey) return cur.d;
		if (cur.d > taxiC) continue;

		for (int dir = 0; dir < 4; dir++) {
			int nx = cur.x + dx[dir];
			int ny = cur.y + dy[dir];

			if (oom(nx, ny) || vis[nx][ny]) continue;
			if (road[nx][ny] == 0) {
				q.push({ nx, ny, cur.d + 1 });
				vis[nx][ny] = true;
			}
		}
	}

	return -1;
}

struct prior { int x, y, d, idx; };

bool compare(prior& a, prior& b) {
	if (a.d == b.d) {
		if (a.x == b.x) return a.y < b.y;
		return a.x < b.x;
	}
	return a.d < b.d;
}

pii carry() {
	vector<prior> tmp;

	for (int i = 0; i < passengers.size(); i++) {
		info& cur = passengers[i];
		if (cur.isArrived) continue;

		int distance = getDistance(taxi.X, taxi.Y, cur.sx, cur.sy);
		if (distance == -1) continue;

		tmp.push_back({ cur.sx, cur.sy, distance, i });
	}

	sort(tmp.begin(), tmp.end(), compare);

	if (tmp.size() > 0) return { tmp[0].d, tmp[0].idx };
	else return { -1,-1 };
}

int moveTaxi(info& cur) {
	taxi = { cur.sx, cur.sy };

	int distance = getDistance(taxi.X, taxi.Y, cur.ex, cur.ey);
	if (distance == -1) return -1;

	taxi = { cur.ex, cur.ey };
	cur.isArrived = true;

	return distance;
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	// freopen("input.txt", "r", stdin);
	input();

	while (true) {
		auto target = carry();
		if (target.first == -1) break;
		taxiC -= target.first;
		
		int distance = moveTaxi(passengers[target.second]);
		if (distance == -1) break;
		taxiC += distance;
	}

	for (info& cur : passengers)
		if (!cur.isArrived) {
			cout << -1;
			return 0;
		}

	cout << taxiC;
	return 0;
}
{% endhighlight %}

<br/>

# 6. 소감
<hr>

- 골드2라는 난이도에 비해 문제가 쉽게 느껴졌다.
- BFS 로직에서 어처구니 없는 실수를 해서 맞왜틀을 시전해버렸다.
  - 문제를 풀다가 조금 졸렸는데 아직도 저기에 왜 참조자를 사용한지 모르겠다.
  - 참고로 초기에 값을 참조하고 해당 주소를 pop 해버리는데 이런 걸 댕글링 포인터라고 한다.
    - 포인터가 해제된 영역을 가리키고 있는 상태

  ```cpp
  	while (!q.empty()) {
		  auto& cur = q.front();  // 참조자를 사용해서
		  q.pop();  // 여기에 오면 댕글링 포인터가 됨.
  
		  ...
  
		  for (int dir = 0; dir < 4; dir++) {
		  	...

		  	if (road[nx][ny] == 0) {
		  		q.push({ nx, ny, cur.d + 1 });  // 이 부분에서 cur의 값이 계속 변경되었다!
		  		vis[nx][ny] = true;
		  	}
		  }
    }
  ```