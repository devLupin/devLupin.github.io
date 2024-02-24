---
layout: post
title: "Codetree. 놀이기구 탑승"
date: 2024-02-22
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/go-on-the-rides) <br/><br/>


# 1. 문제설명
<hr>

- `n*n` 격자에 `n*n`명의 학생을 탑승시키려고 한다.
- 처음에 격자는 모든 칸이 비어있다.
- 각 학생별로 좋아하는 학생이 정확히 4명씩 정해져있다.
  - 자기 자신을 좋아하는 학생은 없고, 동일한 학생의 번호가 2번 주어지는 경우도 없다.
- 아래의 우선순위 중 위에서부터 가장 먼저 만족하는 우선순위 순으로 탑승한다.
  - 격자를 벗어나지 않는 4방향으로 인접한 칸 중 앉아있는 좋아하는 친구의 수가 가장 많은 위치
  - 그 중 인접한 칸 중 비어있는 칸의 수가 가장 많은 위치
  - 행 번호가 가장 작은 위치
  - 열 번호가 가장 작은 위치
- 전부 탑승한 이후 각 학생마다의 점수를 합한 점수 출력
  - 인접한 학생의 수에 따라 `{0, 1, 10, 100, 1000}`점 부여


<br/>


# 2. 입/출력
<hr>

- 입력
  - 격자의 크기를 나타내는 `n`
  - `n * n`개의 줄에 걸쳐 각 학생의 정보 `n0, n1, n2, n3, n4`

- 출력
  - 모든 학생들이 놀이기구에 탑승한 이후의 최종 점수 출력

<br/>


# 3. 문제 분류
- Simulation

<br/>


# 4. 알고리즘 설계
<hr>

- 위치 탐색을 제외한 알고리즘은 간단하니 생략하겠다.
- 정보 저장을 위한 구조체, 배열은 다음과 같이 선언하였다.

  ```cpp
  struct student {
	int num;  // 학생 번호
	vector<int> likes;  // 좋아하는 학생 4명 번호
  };

  int board[SZ][SZ];  // 격자의 정보
  pair<int, int> pos[SZ * SZ];  // i번째 학생의 좌표 정보
  vector<student> v;  // 
  ```

- 격자의 비어있는 칸을 전부 탐색하면서 인접한 4방향을 탐색한다.
- 탐색을 하면서 빈칸의 개수, 좋아하는 학생의 수를 카운팅
- 좋아하는 학생의 수, 비어있는 칸 수, x, y 순으로 저장해서 벡터에 쌓는다.
- 쌓여진 벡터는 다음 칸으로 이동할 수 있는 모든 칸이 된다.
- 우선순위에 따라 정렬하고 0번째의 벡터 원소를 통해 위치를 지정한다.
  - 이 때, `pos`와 `board`에 대한 정보를 갱신한다.


<br/>

# 5. 전체 코드
<hr>

{% highlight cpp %}
#define _CRT_SECURE_NO_WARNINGS
#include <bits/stdc++.h>
using namespace std;

struct student {
	int num;
	vector<int> likes;
};

int N, n0, n1, n2, n3, n4;

const int dx[] = { -1,1,0,0 };
const int dy[] = { 0,0,-1,1 };
const int SZ = 25;
const int SCORE[] = { 0,1,10,100,1000 };

int board[SZ][SZ];
pair<int, int> pos[SZ * SZ];
vector<student> v;

void set_pos(student& cur) {
	vector<tuple<int, int, int, int>> tmp;

	for (int i = 1; i <= N; i++) {
		for (int j = 1; j <= N; j++) {
			if (board[i][j] != 0) continue;

			int like = 0, empty = 0;

			for (int dir = 0; dir < 4; dir++) {
				int nx = i + dx[dir];
				int ny = j + dy[dir];

				if (nx < 1 || ny < 1 || nx > N || ny > N) continue;

				for (int stu : cur.likes) {
					if (board[nx][ny] == stu) like++;
					if (board[nx][ny] == 0) empty++;
				}
			}

			tmp.push_back(make_tuple(~like, ~empty, i, j));
		}
	}

	sort(tmp.begin(), tmp.end());
	auto& [like, cnt, x, y] = tmp[0];

	pos[cur.num] = make_pair(x, y);
	board[x][y] = cur.num;
}

int calc() {
	int sum = 0;

	for (auto& nxt : v) {
		int& num = nxt.num;
		auto& [x, y] = pos[num];

		int cnt = 0;
		for (int dir = 0; dir < 4; dir++) {
			int nx = x + dx[dir];
			int ny = y + dy[dir];

			for (int& cmp : nxt.likes)
				if (board[nx][ny] == cmp) cnt++;
		}

		sum += SCORE[cnt];
	}

	return sum;
}

void solve() {
	for (auto& nxt : v) set_pos(nxt);
	cout << calc();
}

int main(void) {
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	// freopen("input.txt", "r", stdin);

	cin >> N;

	for (int i = 0; i < N * N; i++) {
		cin >> n0 >> n1 >> n2 >> n3 >> n4;
		v.push_back({ n0, {n1, n2, n3, n4} });
	}

	solve();

	return 0;
}
{% endhighlight %}

<br/>

# 6. 소감
<hr>

- `tmp.push_back(make_tuple(~like, ~empty, i, j));` 이 코드의 아이디어가 좋다고 생각한다.
- 다른 언어는 모르겠지만 C++의 STL을 통해 `sort()` 함수를 사용하면 맨 앞에 원소부터 작은 순으로 정렬된다.
- 이 문제에서는 좋아하는 학생 수, 비어있는 칸은 많을 수록 우선순위가 높고 x, y는 낮을수록 우선순위가 높다.
- `~`을 붙여 음수로 만들어주면 원래의 값이 클수록 음수에서는 작아지기 때문에 별도의 `compare` 함수를 작성하지 않아도 된다.
- `-`를 붙여도 될 거 같지만, 그렇게 하면 안된다.
  - `~`는 bit-wise NOT 연산자로 부호 비트를 포함한 비트를 모두 반전시킨다.
  - `-`는 단순히 값을 부정하는 음수 부호를 나타내는 연산자이다.
    - 부호 비트만을 변경하고 나머지 비트는 유지한다.