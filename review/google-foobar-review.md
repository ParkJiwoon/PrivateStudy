# Google Foobar Challenge 후기

Foobar Challenge 는 구글이 숨겨놓은 히든 코딩테스트 입니다.

일반적인 방법으로는 접근할 수 없고 Chrome 검색을 하다보면 화면이 깨지면서 초대 요청이 오는 걸로 알고 있습니다.

저는 우연찮게 좋은 기회가 생겨서 접해볼 수 있었습니다.

**사용 가능 언어는 Java 8 과 Python 2.7.13** 입니다.

Java 는 그렇다 쳐도 Python 버전이 너무 낮은 것으로 보아 레거시 시스템인 것으로 보이는데..

다른 사람들의 후기를 찾아봤을 때 연락이 오지 않는 경우도 있었다는 걸로 보아 챌린지를 완료 한다고 인터뷰 기회가 주어지는 것은 아닌 것 같습니다.

<br>

# Problems

Foobar 챌린지에 등장하는 모든 문제는 구글링 하면 찾을 수 있습니다.

보통 코딩 테스트 문제는 기업에서 공개하지 않는 이상 오픈하면 안되는 걸로 알고 있는데..

오픈 마인드인건진 몰라도 문제 전문과 본인 코드까지 올려놓은 사람들이 있더라구요.

문제가 궁금하신 분들은 구글링을 통해 찾으시면 될 것 같아서 간단하게 접근법만 포스팅 하려고 합니다.

문제는 Level 1 부터 시작하며 Level 5 까지 존재합니다.

```
Level 1: 1 문제
Level 2: 2 문제
Level 3: 3 문제
Level 4: 2 문제
Level 5: 1 문제
```

<br>

## Level 1 : Re-ID

소수 (Prime) 를 이어 붙여 만든 "2357111317192329..." 문자열이 존재합니다.

시작 인덱스가 주어지면 해당 인덱스부터 시작하는 5 개의 문자열을 구해야 합니다.

에라토스테네스의 체로 소수를 모두 구해서 이어 붙인 문자열을 만들고 `substring` 으로 답을 구했습니다.

<br>

## Level 2 - 1 : En Route Salute

사람들의 이동 방향이 정해지고 만날 때마다 인사를 한다고 할 때 총 몇번의 인사가 이루어지는 지 구하는 문제입니다.

한쪽 방향 사람들만 그룹으로 묶어서 총 몇 번의 인사를 하는지 구한 뒤 2 배 하면 됩니다.

<br>

## Level 2 - 2 : Power Hungry

주어진 배열의 원소들의 곱 중 가장 큰 값을 구하는 문제입니다.

숫자 크기 때문에 `BigInteger` 로 풀기 귀찮아서 파이썬으로 풀었습니다.

음수, 양수값을 따로 저장해두고 음수 값이 홀수개면 정렬 후에 마지막 값, 즉 절대값이 가장 작은 값을 제거합니다.

그리고 남은 값들을 전부 곱하면 됩니다.

주의할 점은 두 가지 인데 배열의 길이가 1 이고 홀수인 경우가 존재하기 때문에 예외처리가 필요합니다.

그리고 리턴할 때 문자열로 바꿔서 리턴하는 걸 주의해야 합니다.

<br>

## Level 3 - 1 : Find the Access Codes

주어진 배열에서 특정 조건을 이루는 3 개 숫자 그룹에 대한 모든 경우의 수를 구하는 문제입니다.

숫자 3 개의 성립 조건을 예로 들면 `[a, b, c]` 라고 할 때 `c` 는 `b` 로 나누어 떨어지고 `b` 는 `a` 로 나누어 떨어져야 합니다.

크기가 2000 밖에 되지 않아서 `O(n^2)` 으로 구했습니다.

배열을 한번 돌면서 한 원소에 대하여 같은 배열에 있는 약수들의 갯수를 구하고 다시 돌면서 특정 원소의 약수의 약수 갯수를 더해주면 됩니다.

<br>

## Level 3 - 2 : Bomb, Baby!

M, F 폭탄이 각각 1 개씩 있을 때 주어진 `Input` 만큼의 숫자로 복제하기 위해선 몇 번의 세대가 지나야 하는지 구하는 문제입니다.

한 세대가 지날때마다 M → F 또는 F → M 만큼의 폭탄을 복제할 수 있습니다.

즉 `(M, F)` 의 다음 세대는 `(M + F, F)` 또는 `(M, M + F)` 가 됩니다.

숫자 크기가 10^50 이라서 파이썬으로 풀었습니다.

주어진 input 값부터 시작해서 최종적으로 (1, 1) 이 될 수 있는지를 확인합니다.

단순하게 빼기를 사용하면 시간초과가 나기 때문에 나누기와 나머지 연산을 사용해서 최적화 하면 됩니다.

<br>

## Level 3 - 3 : Prepare the Bunnies' Escape

벽이 존재하는 2 차원 배열에서 오른쪽 아래에서 왼쪽 위로 이동하는 최단 거리를 구하는 문제입니다.

필요 시 벽을 한번 부술 수 있습니다.

백준의 [벽부수고 이동하기](https://www.acmicpc.net/problem/2206) 문제와 완전 동일합니다.

BFS 로 풀었습니다.

<br>

## Level 4 - 1 : Running with Bunnies

영어와 한국어 능력 부족으로 해석에 굉장히 애를 먹은 문제입니다

음수 가중치가 존재하는 그래프에서 제한 시간 내에 시작 노드에서 최대한 많은 다른 노드를 방문하고 도착지에 가야 합니다.

처음 접근한 방법은 Bellman-Ford 알고리즘으로 음수 가중치를 찾고 DFS 를 사용했습니다.

여기서 한가지 간과한 점이 있는데 음수 사이클만 찾으면 된다고 생각했는데 `모든 점에서 나머지 점까지의 최단 거리` 또한 구해야 합니다.

그래서 플로이드 와샬로 최단 거리를 모두 구한 다음 음수 사이클 여부를 체크하고 DFS 로 답을 구했습니다.

<br>

## Level 4 - 2 : Free the Bunny Prisoners

문제를 설명하긴 좀 어려운데 Combination 을 사용해서 푸는 문제입니다.

<br>

## Level 5 : Dodge the Lasers!

문자열 `n` 이 주어지면 `floor(1 * sqrt(2)) + .. + floor(n * sqrt(2))` 를 구하는 문제입니다.

`n` 은 `1 부터 10^100` 까지의 범위입니다.

단순히 sqrt 와 루프를 사용하면 시간초과가 납니다.

접근법을 몰라서 [인터넷에 있는 풀이](https://github.com/arinkverma/google-foobar/blob/master/5.1_dodge_the_lasers.py)를 참고했습니다.

그런데 봐도 무슨 얘긴지 잘 모르겠네요.

<br>

# 챌린지를 마치고

Level 5 까지 제출하고 나면 모든 챌린지가 끝나고 토끼가 뛰어다니는 화면이 나옵니다. (캡쳐는 못했네요)

![](https://github.com/ParkJiwoon/PrivateStudy/blob/master/images/foobar.png?raw=true)

회사 일도 겸하고 있어서 문제 요청만 해두고 널널하게 생각나면 풀고 그랬었는데 Level 4 까지는 풀만한 문제들이 나온 것 같습니다.

지인들 중에는 Level 3 부터 완전 어려운 문제들이 나온 케이스도 있는 걸로 보아 문제 푸는 시간도 고려해서 난이도가 결정되는 것 같기도 합니다. (순전히 개인적인 추측입니다)

Level 3 문제를 전부 풀면 연락처와 본인 정보를 입력할 수 있는데 서론에서도 언급했듯이 실제로 연락받는 케이스는 드물다고 합니다.

재미있는 경험이었습니다.