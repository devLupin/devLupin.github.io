---
layout: post
title: "Codetree. 바이러스 실험"
date: 2024-02-02
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/virus-experiment) <br/><br/>


# 1. 문제설명
<hr>

- `n * n` 격자 무늬의 배지에 바이러스를 배양하는 실험을 진행한다.
- 초기에 각 칸에 5만큼의 양분이 들어있으며 `m`개의 바이러스로 시작한다.
- 실험 사이클
  - 각 바이러스는 본인이 속한 `1 * 1` 크기의 칸에 있는 양분을 섭취한다.
    - 본인의 나이만큼 양분을 섭취한다.
    - 같은 칸에 여러 개의 바이러스가 있을 때에는 나이가 어린 바이러스부터 양분을 섭취한다.
    - 양분을 섭취한 바이러스는 나이가 1 증가한다.
    - 만약 양분이 부족하여 본인의 나이만큼 양분을 섭취할 수 없다면 그 즉시 죽는다.
  - 모든 바이러스가 섭취를 끝낸 후 죽은 바이러스가 양분으로 변한다.
    - 죽은 바이러스마다 나이를 2로 나눈 값이 바이러스가 있던 칸에 양분으로 추가된다.
    - 편의를 위해 소숫점 아래는 버린다.
  - 바이러스가 번식을 진행한다.
    - 5의 배수의 나이를 가진 바이러스에게만 진행된다.
    - 인접한 8개의 칸에 나이가 1인 바이러스가 생긴다.
    - 배지 범위를 벗어난 곳에는 바이러스가 번식하지 않는다.
  - 과정이 순서대로 끝난 뒤에는 주어진 양분의 양에 따라 칸에 양분이 추가된다.

# 2. 입/출력
<hr>

- 격자의 크기 `n`, 바이러스의 개수 `m`, 총 사이클의 수 `k`
- `n`개의 줄에 마지막에 추가되는 양분의 양이 주어진다.
- `m`개의 줄에 바이러스의 정보가 주어진다.
  - 바이러스의 위치(`r, c`), 바이러스의 나이 
- 입력으로 주어지는 바이러스의 위치는 모두 서로 다르다.
- k 사이클 이후에 살아남은 바이러스의 수를 구하라

# 3. 문제 분류
- Simulation


<br/>

# 4. 알고리즘 설계
<hr>

- 바이러스가 양분을 먹고, 번식시키고, 추가 양분을 배분하는 총 3가지 과정으로 나누었다.
- 바이러스 정보 관리
  - `기존 바이러스`, `확산될 바이러스`, `죽은 바이러스`, `새로운 바이러스`로 관리하였다.
  - 양분을 먹는 과정에서 죽게되는 바이러스를 `죽은 바이러스` 벡터에 담는다.
  - 양분을 먹는 과정에서 살아남은 바이러스
    - 살아남은 바이러스 전체를 `새로운 바이러스`에 담는다
    - 만약 나이가 5의 배수라면 `확산될 바이러스` 벡터에 담는다.
  - 번식 과정에서 확산될 바이러스의 8가지 방향을 통해 나이가 1인 바이러스를 `새로운 바이러스`에 담는다.
  - `기존 바이러스 = 새로운 바이러스` 대입을 통해 바이러스 정보를 갱신한다.


<br/>

# 5. 전체 코드
<hr>

{% highlight cpp %}
#define _CRT_SECURE_NO_WARNINGS
#include <bits/stdc++.h>
using namespace std;

int n, m, k, r, c, a;
int A[15][15], B[15][15];

int dx[] = { -1,-1,-1,0,1,1,1,0 };
int dy[] = { -1,0,1,1,1,0,-1,-1 };

struct info {
	int x, y, age;
};
vector<info> virus, spread_virus, dead_virus, new_virus;

bool oom(int x, int y) { return x < 1 || y < 1 || x > n || y > n; }

bool compare(info& a, info& b) { return a.age < b.age; }

void eat() {
	sort(virus.begin(), virus.end(), compare);

	for (int i = 0; i < virus.size(); i++) {
		info& nxt = virus[i];

		if (nxt.age <= A[nxt.x][nxt.y]) {
			A[nxt.x][nxt.y] -= nxt.age;
			virus[i].age++;

			if (virus[i].age % 5 == 0)
				spread_virus.push_back(virus[i]);

			new_virus.push_back(virus[i]);
		}
		else
			dead_virus.push_back(nxt);
	}
}

void update() {
	for (auto& nxt : dead_virus)
		A[nxt.x][nxt.y] += nxt.age / 2;

	for (auto& nxt : spread_virus) {
		for (int dir = 0; dir < 8; dir++) {
			int nx = nxt.x + dx[dir];
			int ny = nxt.y + dy[dir];

			if (!oom(nx, ny))
				new_virus.push_back({ nx,ny,1 });
		}
	}

	virus = new_virus;
}

void add() {
	for (int i = 1; i <= n; i++)
		for (int j = 1; j <= n; j++)
			A[i][j] += B[i][j];
}

void init() {
	spread_virus.clear();
	dead_virus.clear();
	new_virus.clear();
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	// freopen("input.txt", "r", stdin);

	cin >> n >> m >> k;

	for (int i = 1; i <= n; i++) {
		for (int x, j = 1; j <= n; j++) {
			cin >> x;
			A[i][j] = 5;
			B[i][j] = x;
		}
	}

	while (m--) {
		cin >> r >> c >> a;
		virus.push_back({ r, c, a });
	}

	while (k--) {
		init();
		eat();
		update();
		add();
	}

	int ans = 0;
	for (auto& nxt : virus)
		ans += (nxt.x != -1 && nxt.y != -1);

	cout << ans;
	return 0;
}
{% endhighlight %}

<br/>

# 4. 소감
<hr>

- 평소 죽는거나 없애야 하는 정보에 대해 좌표값을 `(-1, -1)`로 처리했고, 아래와 같이 for문을 돌면서 건너뛰었다.

```cpp
for (int i = 0; i < virus.size(); i++) {
    info& nxt = virus[i];
    if (nxt.x < 0 && nxt.y < 0) continue;
    ....
}
```

- 바이러스가 많이 생성되다 보니, 이렇게 하면 시간초과를 발생시켰다.
- 검색을 해봤더니 구현 문제에서는 값을 복사하는 방식을 많이 사용하고 있었다.
  - 그래야 탐색 시간을 줄일 수 있으니까!