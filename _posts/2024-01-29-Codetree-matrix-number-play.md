---
layout: post
title: "Codetree. 격자 숫자 놀이"
date: 2024-01-29
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/matrix-number-play) <br/><br/>


# 1. 문제설명
<hr>

- `3x3`크기의 격자판이 주어진다.
- 배열의 위치 `(x, y)`의 값이 K가 되려면 몇 번의 연산을 수행해야 하는가?
- 연산 과정
  - 행의 개수 >= 열의 개수
    - 모든 행에 대해 숫자 정렬
	  - 1순위 : 출현 빈도 수가 적은 순서대로 
	  - 2순위 : 값이 작은 순서대로
    - 이후 숫자와 숫자의 빈도 수를 출력
  - 행의 개수 < 열의 개수
    - 모든 열에 대해 과정 수행
  - 행 또는 열의 길이가 100이 넘어가면 100개 이후는 모두 버림.
  - 길이가 맞지 않는 경우는 최대 길이로 맞추고 0으로 채움.

- example

```
// input
3 1 2
1 1 2
5 5 5

// 행의 개수 >= 열의 개수
1 1 2 1 3 1
2 1 1 2 0 0
5 3 0 0 0 0

// 행의 개수 < 열의 개수
1 3 1 1 3 1
1 1 1 1 1 1
2 1 2 2 0 0
1 2 1 1 0 0
5 0 0 0 0 0
1 0 0 0 0 0
```


- 문제 분류
  - Implementation, dx/dy,


<br/>

# 2. 알고리즘 설계
<hr>

- 숫자의 빈도수 계산
  - 1~100까지 어떤 숫자가 들어올지 알 수 없다.
  - 하나의 숫자가 N개라면 100의 크기의 배열은 매우 비효율적
  - `unordered_map<int,int>`로 카운팅
    - 0은 사용하지 않기 때문에 건너뛰기
  - `map`의 `first`는 숫자, `second`는 빈도 수 저장
  - `map`은 `sort`함수가 적용되지 않아 벡터에 값을 그대로 복사 후 정렬
- 행에 대한 연산, 열에 대한 연산
  - 행 인덱스 `r_idx`, 열 인덱스 `c_idx`를 하나씩 증가시켜주며 원래 배열에 복사
  - 값의 행 또는 열이 최대 100이 되므로, 배열의 크기를 100으로 고정
  - 값을 복사하는 과정에서 행, 열의 연산이 바뀌게 되면 일부 값이 남아 있을 수 있어 배열의 초기화 진행


<br/>

# 3. 전체 코드
<hr>

{% highlight cpp %}
#define _CRT_SECURE_NO_WARNINGS
#include <bits/stdc++.h>
using namespace std;
using pii = pair<int, int>;

int A[101][101];
int r, c, k, row, col;

void print() {
	for (int i = 1; i <= row; i++) {
		for (int j = 1; j <= col; j++)
			cout << A[i][j] << ' ';
		cout << '\n';
	}
	cout << '\n';
}

void init() {
	for (int i = 1; i <= 100; i++)
		memset(A[i], 0, sizeof(A[i]));
}

bool compare(pii& a, pii& b) {
	if (a.second == b.second) return a.first < b.first;
	return a.second < b.second;
}

void play_row() {
	int mx = 0;
	vector<vector<pii>> res;

	for (int i = 1; i <= row; i++) {
		unordered_map<int, int> um;

		for (int j = 1; j <= col; j++) {
			int cur = A[i][j];
			if (cur == 0) continue;;

			um[cur]++;
		}

		vector<pii> v(um.begin(), um.end());
		sort(v.begin(), v.end(), compare);

		res.push_back(v);
	}

	init();

	int r_idx = 1, c_idx;
	for (auto nxt : res) {
		c_idx = 1;
		for (auto elem : nxt) {
			A[r_idx][c_idx++] = elem.first;
			A[r_idx][c_idx++] = elem.second;
			if (c_idx == 100) break;
		}
		r_idx++;
		mx = max(mx, c_idx);
	}

	col = mx - 1;
}

void play_col() {
	int mx = 0;
	vector<vector<pii>> res;

	for (int j = 1; j <= col; j++) {
		unordered_map<int, int> um;

		for (int i = 1; i <= row; i++) {
			int cur = A[i][j];
			if (cur == 0) continue;;

			um[cur]++;
		}

		vector<pii> v(um.begin(), um.end());
		sort(v.begin(), v.end(), compare);

		res.push_back(v);
	}

	init();

	int r_idx, c_idx = 1;
	for (auto nxt : res) {
		r_idx = 1;
		for (auto elem : nxt) {
			A[r_idx++][c_idx] = elem.first;
			A[r_idx++][c_idx] = elem.second;
			if (r_idx == 100) break;
		}
		c_idx++;
		mx = max(mx, r_idx);
	}

	row = mx - 1;
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	// freopen("input.txt", "r", stdin);

	row = col = 3;

	cin >> r >> c >> k;
	for (int i = 1; i <= row; i++)
		for (int j = 1; j <= col; j++)
			cin >> A[i][j];

	int ans = -1;

	for(int t = 0; t <= 100; t++) {
		if (A[r][c] == k) {
			ans = t;
			break;
		}
		if (row >= col) play_row();
		else play_col();
	}
	
	cout << ans;

	return 0;
}
{% endhighlight %}

<br/>

# 4. 소감
<hr>

- 행과 열에 대한 연산을 별도로 구현하였다.
- 행과 열만 바뀌고 로직이 똑같기 때문에 복붙해서 오타가 발생했다..
- 아래와 같이 `mx` 값 갱신을 서로 반대로 만들어야 하는데 둘다 똑같이 복사해서..
  - 맞왜틀을 20분은 시전한 거 같다...

```cpp
r_idx++;
mx = max(mx, c_idx);

c_idx++;
mx = max(mx, r_idx);
```