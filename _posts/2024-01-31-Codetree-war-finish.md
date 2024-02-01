---
layout: post
title: "Codetree. 종전"
date: 2024-01-31
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/war-finish) <br/><br/>


# 1. 문제설명
<hr>

- `NxN`크기의 격자 정보가 주어진다.
- 격자를 다섯개의 부족의 땅으로 나누기로 했다.
- 땅을 나누기 위해 기울어진 직사각형을 이용한다.

- 직사각형은 다음과 같은 형태이다.
  - <p align="center"><img src="https://contents.codetree.ai/problems/136/images/3fb27249-82d6-4cc3-88fa-382ace91d150.png"></p>

- 1~5번 부족은 다음 지역을 가지게 된다.
  - 1번 부족은 기울어진 직사각형의 경계와 그 안에 있는 지역을 가지게 됩니다.
  - 2번 부족은 기울어진 직사각형의 좌측 상단 경계의 윗부분에 해당하는 지역을 가지게 됩니다. 이때 위쪽 꼭짓점의 위에 있는 칸들은 모두 포함하지만 왼쪽 꼭짓점의 왼쪽에 있는 칸들은 포함하지 않습니다.
  - 3번 부족은 기울어진 직사각형의 우측 상단 경계의 윗부분에 해당하는 지역을 가지게 됩니다. 이때 오른쪽 꼭짓점의 오른쪽에 있는 칸들은 모두 포함하지만 윗쪽 꼭짓점의 위쪽에 있는 칸들은 포함하지 않습니다.
  - 4번 부족은 기울어진 직사각형의 좌측 하단 경계의 아랫부분에 해당하는 지역을 가지게 됩니다. 이때 왼쪽 꼭짓점의 왼쪽애 있는 칸들은 모두 포함하지만 아랫쪽 꼭짓점의 아래쪽에 있는 칸들은 포함하지 않습니다.
  - 5번 부족은 기울어진 직사각형의 우측 하단 경계의 아랫부분에 해당하는 지역을 가지게 됩니다. 이때 아랫쪽 꼭짓점의 아랫쪽에 있는 칸들은 모두 포함하지만 오른쪽 꼭짓점의 오른쪽에 있는 칸들은 포함하지 않습니다.

- 사각형을 적절히 지정하여 부족 간 차지하는 지역의 차이가 가장 적도록 만든다.
  - 이 때의 최대, 최소의 차이를 구하라!

- 문제 분류
  - Simulation


<br/>

# 2. 알고리즘 설계
<hr>

- 사각형의 크기 지정
  - 세로방향 길이(그림 속 1, 3번), 가로방향 길이(그림 속 2, 4번)
  - 각각은 가로 축의 길이 `j`, `n-j`가 최대 값이 된다.
  - 하나씩 탐색한 후 `is_squre` 함수를 통해 격자를 벗어나는 경우 예외처리
- 영역 칠하기
  - 1번 부터 5번 부족까지 맵에 마킹을 해주면된다.
  - 1번은 사각형의 테두리 및 안쪽까지 칠해야하므로, 정답 배열의 초기값을 1로 해주었다.
- 이후, 앞서 언급한 2~5번 부족의 영역을 for문을 통해 칠해주면 된다.
  - 그림 예제만으로는 헷갈릴 수 있는 부분이 있어
  - 여러 그림을 그려보고 실제 영역을 계산해보자.
  - 특히, 어떤 꼭지점은 포함하고 어떤 꼭지점은 포함하지 않는 부분에 주의해야 한다.
- 영역 계산
  - 칠해진 영역 중 최대값과 최소값을 계산


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

int n, A[20][20], B[20][20];
int ans = 0x3f3f3f;

bool is_square(int i, int j, int d1, int d2) {
	int diff = abs(d1 - d2);
	if (i + d1 >= n || j - d1 < 0) return false;
	if (i + d2 >= n || j + d2 >= n) return false;
	if (i + d2 + d1 >= n || j + d2 - d1 < 0) return false;
	return true;
}

void labeling(vector<pii> pos) {
	fill_n(&B[0][0], 20 * 20, 1);

	// area2
	int tmp = 0;
	for (int i = 0; i < pos[1].X; i++) {
		if (i >= pos[0].X) tmp++;
		for (int j = 0; j <= pos[0].Y - tmp; j++)
			B[i][j] = 2;
	}

	// area3
	tmp = 0;
	for (int i = 0; i <= pos[2].X; i++) {
		if (i > pos[0].X) tmp++;
		for (int j = pos[0].Y + 1 + tmp; j < n; j++)
			B[i][j] = 3;
	}

	// area4
	tmp = 0;
	for (int i = n - 1; i >= pos[1].X; i--) {
		if (i < pos[3].X) tmp++;

		for (int j = 0; j < pos[3].Y - tmp; j++)
			B[i][j] = 4;
	}

	// area5
	tmp = 0;
	for (int i = n - 1; i > pos[2].X; i--) {
		if (i <= pos[3].X) tmp++;
		for (int j = pos[3].Y + tmp; j < n; j++)
			B[i][j] = 5;
	}
}

int calc() {
	vector<int> area(5, 0);
	
	for (int i = 0; i < n; i++)
		for (int j = 0; j < n; j++)
			area[B[i][j] - 1] += A[i][j];

	sort(area.begin(), area.end());

	return area[4] - area[0];
}

void solve() {
	for (int i = 0; i < n; i++) {
		for (int j = 0; j < n; j++) {
			for (int d1 = 1; d1 <= j; d1++) {  // 세로방향 길이
				for (int d2 = 1; d2 < n - j; d2++) {  // 가로방향 길이
					if (is_square(i, j, d1, d2)) {
						vector<pii> pos;
						pos.push_back({ i,j });
						pos.push_back({ i + d1, j - d1 });
						pos.push_back({ i + d2, j + d2 });
						pos.push_back({ i + d1 + d2, j - d1 + d2 });

						labeling(pos);
						ans = min(calc(), ans);
					}
				}
			}
		}
	}
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	// freopen("input.txt", "r", stdin);

	cin >> n;
	for (int i = 0; i < n; i++)
		for (int j = 0; j < n; j++)
			cin >> A[i][j];

	solve();
	cout << ans;

	return 0;
}
{% endhighlight %}

<br/>

# 4. 소감
<hr>

- 예제만 보고 구현했다가, 코너케이스 때문에 꽤 많은 시간을 들이게 되었다.
  - 꼭지점 포함, 미포함....