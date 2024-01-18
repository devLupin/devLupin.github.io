---
layout: post
title: "Codetree. 전투 로봇"
date: 2024-01-17
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/fighting-robot) <br/><br/>


# 1. 문제설명
<hr>

- `n * n`개의 격자에 `m`개의 몬스터와 하나의 전투로봇이 주어진다.
  - 한칸에는 몬스터가 하나 이상
  - 전투로봇과 몬스터 모두 자연수 레벨을 가진다.
  - 초기 전투로봇의 레벨은 2
- 전투로봇은 1차에 상하좌우로 인접한 한칸씩 이동할 수 있다.
  - 자신의 레벨보다 큰 몬스터가 있는 칸은 지나칠 수 없고, 나머지는 지날 수 있다.
  - 자신보다 레벨이 적은 몬스터만 없앨 수 있다.
- 이동할지 정하는 규칙
  - 없앨 수 있는 몬스터가 있다면 없애러 간다.
  - 없앨 수 있는 몬스터가 하나 이상이라면, 거리가 가장 가까운 몬스터를 없앤다.
    - 거리는 해당 칸으로 이동할 때 지나야 하는 칸의 개수 의미
    - 거리가 같은 몬스터가 여러개 있다면 상단/좌측의 순으로 우선순위가 부여된다.
  - 없앨 몬스터가 없다면 종료
- 전투로봇은 본인의 레벨과 같은 수의 몬스터를 없앨 때마다 레벨 상승


- 문제 분류
  - Implementation, Simulation, BFS


<br/>

# 2. 알고리즘 설계
<hr>

- 크게 3가지로 분할 구현하였다.
  - 전투로봇과 몬스터의 거리 구하기, 없앨 몬스터 찾기, 몬스터 없애기
- 자신의 레벨과 같은 수의 몬스터를 없애면 레벨업을 하기 때문에 `robot_exp`라는 변수로 관리해주었다.
- 없앨 몬스터 찾기
  - 현재 전투로봇의 위치와 살아남은 몬스터 간 거리를 구하고, 위치와 거리를 전부 벡터에 담는다.
  - 거리/상단/좌측 순으로 정렬 기준을 세워 정렬한다.
  - 정렬된 벡터의 첫번째 원소가 타겟이 된다.
  - 만약, 벡터의 크기가 0이라면 타겟 몬스터가 없다는 의미 (종료조건)
- 거리 구하기
  - 단순하게 `|x2-x1| + |y2-y1|`로 구하면 안된다.
  - 레벨이 전투로봇보다 이하인 몬스터만 지나칠 수 있다.
  - BFS를 통해 최단 경로를 명시해주어야 한다.
- 몬스터 없애기
  - 앞서 타겟을 찾았다면, 몬스터에 해당하는 위치의 값을 0으로 만든다.
  - 로봇과 타겟의 위치의 값(0)을 서로 `swap`한다.
  - 로봇의 위치를 타겟의 위치로 변경한다.
  - 타겟의 위치는 `{-1, -1}`로 변경하여 앞의 없앨 몬스터 찾기에서 바로 `continue`되도록 한다.
  - 몬스터를 잡아먹었다면 `robot_exp++`를 해준다.
    - 만약 `robot_exp`와 로봇의 레벨이 같아지면 로봇의 위치의 값을 `++`하면 된다.



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

int n, A[21][21];
vector<pii> monster;
pii robot;
int robot_exp;

int dx[] = { -1,1,0,0 };
int dy[] = { 0,0,-1,1 };

struct pos {
	int x, y, d, idx;
};

int distance(pii& src, pii& dest) {
	int lv = A[src.X][src.Y];
	bool vis[21][21] = { false, };
	queue<pos> q;

	q.push({ src.X, src.Y, 0 });
	vis[src.X][src.Y] = true;

	while (!q.empty()) {
		pos& now = q.front();
		q.pop();

		if (now.x == dest.X && now.y == dest.Y) return now.d;
		
		for (int dir = 0; dir < 4; dir++) {
			int nx = now.x + dx[dir];
			int ny = now.y + dy[dir];

			if (nx < 0 || ny < 0 || nx >= n || ny >= n) continue;
			if (!vis[nx][ny] && lv >= A[nx][ny]) {
				q.push({ nx, ny, now.d + 1 });
				vis[nx][ny] = true;
			}
		}
	}

	return -1;
}

bool compare(const pos& a, const pos& b) {
	if (a.d == b.d) {
		if (a.x == b.x) return a.y < b.y;
		return a.x < b.x;
	}
	return a.d < b.d;
}

pos get_target() {
	vector<pos> ret;

	for (int i = 0; i < monster.size(); i++) {
		auto& nxt = monster[i];

		if (nxt.X == -1 && nxt.Y == -1) continue;
		if (A[robot.X][robot.Y] <= A[nxt.X][nxt.Y]) continue;
		
		int dist = distance(robot, nxt);
		if (dist == -1) continue;

		ret.push_back({ nxt.X, nxt.Y, dist, i });
	}

	if (ret.size() == 0) return { -1,-1 };
	
	sort(ret.begin(), ret.end(), compare);
	return ret[0];
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	// freopen("input.txt", "r", stdin);

	cin >> n;
	for (int i = 0; i < n; i++) {
		for (int j = 0; j < n; j++) {
			cin >> A[i][j];
			if (A[i][j] == 9) {
				robot = { i,j };
				A[i][j] = 2;
			}
			else if (A[i][j] > 0) monster.push_back({ i,j });
		}
	}
	
	int ans = 0;

	while (true) {
		auto target = get_target();
		if (target.x == -1 && target.y == -1) break;

		A[target.x][target.y] = 0;
		swap(A[robot.X][robot.Y], A[target.x][target.y]);
		robot = { target.x, target.y };
		monster[target.idx] = { -1,-1 };

		robot_exp++;
		if (robot_exp == A[robot.X][robot.Y]) {
			A[robot.X][robot.Y]++;
			robot_exp = 0;
		}

		ans += target.d;
	}

	cout << ans;
	return 0;
}
{% endhighlight %}

<br/>

# 4. 소감
<hr>

- 문제에 있는 것을 정말 그대로 구현하면 되는 문제였다.
- 다만 전투로봇 레벨 **이하**의 몬스터를 지나칠 수 있다는 점에 주의한다.
  - 레벨이 같다면 지나칠 수 있지만, 잡아먹을 수는 없다.
- 참조자와 속도의 관계
  - 내 코드를 보면 몬스터 정보 및 그 외 정보들을 벡터에 담아 관리하였다.
  - 벡터를 참조자가 아닌 형태로 사용했을 때와 참조자로 사용했을 때의 속도 차이가 분명했다.
  - `auto nxt = monster[i]`와 `auto& nxt = monster[i]`는 각각 489ms, 290ms를 기록했다.
  - 참조자를 사용하는 습관을 가질 필요성을 느꼈다.