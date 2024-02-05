---
layout: post
title: "Codetree. 드래곤 커브"
date: 2024-02-02
---

[문제 출처](https://www.codetree.ai/training-field/frequent-problems/problems/dragon-curve) <br/><br/>


# 1. 문제설명
<hr>

- 좌표평면은 x 값이 위에서 아래로 갈수록 증가하며 0에서부터 시작하며, y는 왼쪽에서 오른쪽으로 갈수록 증가하며 0에서부터 시작한다.
- 0차 드래곤 커브는 길이가 1인 선분이다.
- 1차 드래곤 커브는 0차 드래곤 커브를 복제한 후 해당 드래곤 커브의 끝점을 기준으로 시계 방향으로 90도 회전하여 연결한 것이다.
  - 끝점은 해당 드래곤 커브에서 시작점으로부터 가장 멀리 떨어진 점(=그려지며 도착한 마지막 지점)을 뜻한다.
- n차 드래곤 커브는 n-1차 드래곤 커브의 끝점에 n-1차 드래곤 커브를 복제한 뒤 시계 방향으로 90도 회전시킨 뒤 연결한 도형이다.

# 2. 입/출력
<hr>

- 드래곤 커브의 개수 `n`
- `n`개의 드래곤 커브 정보
  - 시작점 `x, y`, 시작 방향 `d`, 차수 `g`
- 방향은 0이상 3이하의 정수
  - 각각 오른쪽, 위쪽, 왼쪽, 아래쪽
- 입력으로 주어지는 드래곤 커브의 모든 꼭지점은 좌표평면의 범위를 넘어서지 않는다.
- 좌표평면 범위 내에 있는 꼭짓점이 모두 드래곤 커브의 일부인 크기가 1인 정사각형의 개수를 출력하라

# 3. 문제 분류
- Simulation


<br/>

# 4. 알고리즘 설계
<hr>

- 입력으로 주어지는 0차 드래곤 커브부터 드래곤 커브의 세대까지 기억할 필요가 있다.
- 본인은 드래곤 커브의 세대 `g`만큼 드래곤 커브를 생성하고, 좌표를 방문 표시를 하였다.
- 0차 드래곤 커브는 `x, y` 좌표와 이에 방향 `d`를 더한 `dx, dy`로 구성된다.
- 1차 드래곤 커브는 0차 드래곤 커브에서 방향 `d`를 더한 `dx, dy`로 구성된다.
- 2차 드래곤 커브는 1차 드래곤 커브에서 방향 `d`를 더한 `dx, dy`로 구성된다.
- 이런식으로 이전 방향들을 기억하고 이를 90도 회전(`d`를 더한)한 값으로 구성된다.
- 마지막으로 방문 표시가 되어 있으면서 연결된 좌표들의 개수를 세어주면 된다.


<br/>

# 5. 전체 코드
<hr>

{% highlight cpp %}
#include <bits/stdc++.h>
using namespace std;

const int SZ = 105;
const int dx[] = {0,-1,0,1};
const int dy[] = {1,0,-1,0};
int n, x, y, d, g;
bool board[SZ][SZ];
vector<int> dir;

int count() {
    int ret = 0;

    for(int i=0; i<SZ; i++)
        for(int j=0; j<SZ; j++)
            ret += board[i][j] && board[i+1][j] && board[i][j+1] && board[i+1][j+1];
    
    return ret;
}

void make() {
    int sz = dir.size();
    for(int i=sz-1; i>=0; i--) {
        int nd = (dir[i] + 1) % 4;

        x += dx[nd];
        y += dy[nd];
        board[x][y] = true;

        dir.push_back(nd);
    }
}

int main(void) {
    ios::sync_with_stdio(false);
    cin.tie(NULL);

    cin >> n;
    
    while(n--) {
        dir.clear();

        cin >> x >> y >> d >> g;

        board[x][y] = true;

        x += dx[d];
        y += dy[d];
        board[x][y] = true;

        dir.push_back(d);
        while(g--) make();
    }

    cout << count();
    return 0;
}
{% endhighlight %}

<br/>

# 4. 소감
<hr>

- 좌표를 어떻게 기억하고 그 좌표들을 90도 회전할지 아이디어를 떠올리는 게 관건인 문제였다.