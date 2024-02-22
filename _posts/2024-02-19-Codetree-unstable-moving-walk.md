---
layout: post
title: "Codetree. 불안한 무빙워크"
date: 2024-02-20
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/unstable-moving-walk) <br/><br/>


# 1. 문제설명
<hr>

- 길이가 `n`의 무빙 워크의 안정성 테스트를 하고자 한다.
- 총 `2n`개의 판으로 구성된다.
- 무빙워크의 레일은 시계 방향으로 회전한다.
- 각 사람은 1번 칸에 올라서서 n번 칸에서 내리게 된다.
- 사람이 어떤 칸에 올라가거나 이동하면 그 칸의 안정성은 즉시 1만큼 감소하게 되며 안정성이 0인 칸에는 올라갈 수 없다.
- 과정이 종료될 때 몇 번째 실험을 하고 있었는지 계산하라

<br/>


# 2. 입/출력
<hr>

- 입력
  - 무빙워크의 길이 `n`
  - 실험을 종료하게 하는 안정성이 0인 판의 개수 `k`
- 출력
  - 무빙워크가 종료될 때 몇 번째 실험 중이었는지를 출력

<br/>


# 3. 문제 분류
- Simulation

<br/>


# 4. 알고리즘 설계
<hr>

- 이런 류의 회전 문제는 deque가 유리하다.
  - 전체를 움직일 필요없이 양 끝을 조정해줌으로써 기능을 구현할 수 있기 때문
- queue는 선형 자료구조가 아니지만, deque는 queue 자료구조 중 유일하게 인덱스 접근이 가능하다.
  - 무빙워크의 수, 방문 여부를 체크하기 위해 2개의 deque 선언!


<br/>

# 5. 전체 코드
<hr>

{% highlight cpp %}
#include <bits/stdc++.h>
using namespace std;

int n, k, ans;
deque<int> dq;
deque<bool> chk;

int count() {
	int ret = 0;
	for(int i=0; i<dq.size(); i++)
		ret += (dq[i] == 0);
	return ret;
}

void rotate() {
	dq.push_front(dq.back());
	dq.pop_back();
	chk.push_front(chk.back());
	chk.pop_back();
}

void solve() {
	while(count() < k) {
		ans++;

		rotate();

		if(chk[n-1]) chk[n-1] = false;
		
		for(int i=n-2; i>=0; i--) {
			if(dq[i+1] > 0 && !chk[i+1] && chk[i]) {
				dq[i+1]--;
				chk[i] = false;

				if(i == n-2) continue;
				chk[i+1] = true;
			}
		}

		if(dq[0] > 0 && !chk[0]) {
			dq[0]--;
			chk[0] = true;
		}
	}
}

int main(void)
{
	ios::sync_with_stdio(false);
	cin.tie(NULL);

	cin >> n >> k;
	for(int x, i=0; i<n*2; i++) {
		cin >> x;
		dq.push_back(x);
		chk.push_back(false);
	}

	solve();
	cout << ans;

	return 0;
}
{% endhighlight %}

<br/>

# 6. 소감
<hr>

- deque를 활용할 수 있다면 쉽게 풀리는 문제였다.