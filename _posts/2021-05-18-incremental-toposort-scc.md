
---
layout: post
title:  "Incremental Topological Ordering and Strong Component Maintenance"
date:   2021-05-18 23:59:00
author: koosaga
tags: [algorithm, graph-theory, data-structure, scc]
---

# Incremental Topological Ordering and Strong Component Maintenance

방향 그래프 $G$ 에 대해서, $G$ 의 *위상 정렬* $O: V \rightarrow [n]$ 은 모든 간선 $u \rightarrow v$ 에 대해서 $O(u) < O(v)$ 가 성립하는 순열로 정의된다. $G$ 의 위상 정렬이 존재하기 위해서는 $G$ 가 사이클이 없어야 한다는 사실이 잘 알려져 있다 (Directed Acyclic Graph, DAG). 

위상 정렬은 방향 그래프에서 사용하는 가장 기초적인 알고리즘 중 하나이다. 그래프는 일반적으로 순서가 없이 표현되는데, 문제를 풀거나 처리를 하는 데 있어서 편리한 순서를 준다는 데 큰 의미가 있다. 현실에서 마주치는 문제를 많이 모델링할 수 있으며, 공학 분야에도 다양하게 응용될 수 있다. 대단히 많은 적용 사례가 있어서 굳이 모든 것을 언급할 필요는 없겠지만주요한 적용 사례는 프로그래밍 언어의 control flow, `npm` 과 같은 유닉스 패키지 관리자 혹은 컴파일러의 dependency 관리, 산업 공학에서의 critical path method 등이 된다.

방향 그래프에 사이클이 없다면, 위상 정렬을 적용할 수 없다. 이 경우에는 그래프에 대한 강연결 컴포넌트 (SCC) 를 구해서 사이클을 하나의 정점으로 묶어주고, 이 DAG에서 위상 정렬을 적용하는 방법이 자주 사용된다. 위상 정렬과 SCC 모두 방향 그래프에서 해결할 수 있는 가장 근본적인 문제 중 하나이다. 

그래프의 일부가 바뀔 때에도 여러 잘 알려진 문제를 효율적으로 해결하는 알고리즘들을 Dynamic Graph Algorithm이라고 한다. 무방향 그래프에서는 많은 Dynamic Algorithm들이 발견되었고 지금도 연구되고 있으나, 방향 그래프는 대칭성이 없기 때문에 효율적인 알고리즘을 개발하기가 상대적으로 어려운 편이다. 하지만:

* Offline, Incremental (간선 추가만을 허용)  이라는 제약 조건 하에 $O(m \log m)$ 시간에 Incremental SCC를 구할 수 있음이 알려져 있다.
* Incremental 이라는 제약 조건 하에 위상 정렬과 SCC에 대해서 총 $O(m^{1.5})$ (amortized $O(m^{0.5})$) 시간에 Incremental 위상 정렬과 SCC를 구할 수 있다.

이것은 방향성 그래프에서 찾기 드문 강력한 결과이다. 단순히 말해서, 간선이 추가되었을 때 위상 정렬과 SCC를 관리하는 쿼리 문제를 해결할 수 있다. 위상 정렬과 SCC는 방향 그래프 분석에 있어서 가장 중요한 도구이며, 방향 그래프의 복잡성 때문에 사실상 유일한 도구라고 볼 수 있다. 고로 이와 같은 결과는 간선이 추가되었을 때도 방향 그래프에서 관리하던 유용한 정보들을 유지할 수 있게끔 하는 시초석이 된다. 또한, 알고리즘이 생각보다 간단하고 재미있는 아이디어에 기반하기 때문에 대회 준비 레벨에서도 배워볼만 하다.

이 글에서는 2008년 발표된 두 번째 알고리즘을 소개한다. 이 글의 내용은 Bernhard Haeupler, Siddhartha Sen, Robert E. Tarjan 가 작성한 [Incremental Topological Ordering and Strong Component Maintenance](https://arxiv.org/abs/0803.0792) 논문에 기반해 있다.

## Simple Algorithm without Data Structures

이 단락에서는, 간선이 추가되었을 때 위상 정렬을 관리하는 알고리즘을 소개하고, 정당성과 몇가지 정리를 증명한다. 이 알고리즘의 시간 복잡도는 $O(nm)$ 으로 기존 알고리즘에 비해서 효율적이지 않다. 자료구조를 사용하면 이를 최적화할 수 있지만, 이해의 편의를 위해 간단한 버전을 설명한다.

간선 $u \rightarrow v$를 추가하자. 만약 $u < v$ 일 경우 위상 정렬을 수정할 필요가 없다. $u > v$ 임을 가정하자. 아래 다음과 같이 용어를 정의한다.

* $v$ 에서 도달 가능한 정점들을 *forward vertices* 라고 한다.
* $u$ 로 도달 가능한 정점들을 *backward vertices* 라고 한다.
* 어떠한 정점 $v$ 에 대해서 만약에 $v$ 에서 나가는 (forward의 경우), 혹은 $v$ 로 들어오는 (backward의 경우) 모든 간선을 탐색했다면, $v$ 를 *scanned vertices* 라고 한다.

이제 우리는 $v$ 에서 forward, $u$ 에서 backward 정점을 동시에 양방향으로 탐색한다. 양방향 탐색의 핵심은 forward, backward 방향에 공평하게 시간을 분배하는 것이다. 매 스텝마다, Forward 정점에서 뻗어나가는 간선 하나를 처리한 후 인접 리스트 포인터를 증가시키고, Backward 정점으로 뻗어오는 간선 하나를 처리한 후 인접 리스트 포인터를 증가시킨다. 만약 포인터가 리스트의 끝이라면, 해당 정점은 *scanned* 상태가 되니, 큐 혹은 스택 (DFS/BFS에 따라 달라진다) 에서 다음 정점을 뽑아서 탐색을 계속 진행하면 된다. 

이 탐색이 끝나는 조건은 두 가지이다. 첫 번째 조건은 사이클이 발견되었을 때로, 이 경우에는 실패를 보고하고 알고리즘을 종료한다. 두 번째 경우에는 어떠한 정점 $s$ 가 있어, $s$ 보다 작은 forward vertices와 $s$ 보다 큰 backward vertices들이 모두 *scanned* 상태인 경우이다. 이 경우에는 다음과 같이 처리한다. $X$ 를 $s$보다 작은 forward vertices들의 집합, $Y$ 를 $s$ 보다 큰 backward vertices들의 집합이라고 하자. 만약

* $s$ 가 forward가 아니라면, 현재 정점 순서에서 $X \cup Y$ 를 지운 후, $s$ 를 $s,Y, X$ 로 대체한다. 이 때 $Y$ 와 $X$ 는 기존 위상 정렬 순서를 보존한다.
* $s$ 가 backward가 아니라면, 현재 정점 순서에서 $X \cup Y$ 를 지운 후, $s$ 를 $Y, X, s$ 로 대체한다. 이 때 $Y$ 와 $X$ 는 기존 위상 정렬 순서를 보존한다.

이것으로 간단한 알고리즘의 설명을 마친다. 먼저, 이 알고리즘이 올바른 위상 정렬을 구함을 증명하고, 그 다음 이 알고리즘에서 양방향 탐색을 하는 데 드는 시간이 적음을 증명한다.

**Theorem 1.** 위 알고리즘이 찾는 위상 정렬은 올바르다.

**Proof.** 먼저 알고리즘이 사이클을 올바르게 찾음을 증명한다. 자명하게도, forward이며 backward인 정점을 발견하여 종료했다면 사이클이 존재하는 것이 맞다. 그렇지 않은 경우, $s$ 보다 작은 forward vertices, $s$ 보다 큰 backward vertices들이 모두 스캔되었다. 이 경우 $s$ 가 forward인지 아닌지, backward인지 아닌지가 판별되었기 때문에, $s$가 둘 모두에 속한다면 이미 알고리즘이 종료했을 것이다. 고로 $s$ 는 사이클의 일부가 될 수 없고, 사이클은 $X$ 에 속한 정점 $p, q, r, \ldots, x$ 를 거치다가 바로 $Y$ 에 속한 정점 $y, z, a \ldots $으로 이동해야 한다. 이 때 $x$ 와 $y$ 모두 scanned되었기 때문에, 둘 중 하나를 발견한 시점에서 사이클을 찾았어야 하고, 고로 가정에 모순이다.

이제 새롭게 찾은 위상 정렬이 올바름을 증명한다. 일단 여기서는 $s$ 가 forward가 아니라고 가정한다 (다른 경우에 대해서는 똑같이 해 보면 된다). 그래프의 정점은 $X, Y, V-(X\cup Y)$ 의 세 집합으로 분리할 수 있다. 간선 $x \rightarrow y$ 에 대해서, 시작점과 끝점이 존재할 수 있는 집합의 경우의 수는 9개이다. 이 9개의 케이스 모두에 대해서 새로 찾은 위상 정렬의 순서가 올바름을 따져본다. 

* (1, 2, 3) $\{x, y\} \in X, \{x, y \} \in Y, \{x, y\} \in V - (X \cup Y)$: 기존 위상 정렬 순서가 보존되어서 올바르다.
* (4) $x \in X, y \in Y$: 사이클이다.
* (5) $x \in X, y \notin X \cup Y$: $x$ 가 scanned되었지만 $y \notin X$ 이기 때문에, $y > s$ 이다. 고로 올바르다.
* (6) $x \in Y$, $y \in X$: $(x, y) = (v, w)$ 가 아니라면 이는 기존 위상 정렬 순서 자체에 위배된다. 고로 존재할 수 없고, $(x, y) = (v, w)$ 인 경우에는 올바르다.
* (7) $x \in Y, y \notin X \cup Y$: $x \in Y$ 이니 기존 위상 정렬 순서에 의해 $y > s$ 이다. 고로 올바르다.
* (8) $x \notin X \cup Y, y \in X$: 기존 순서에서 $x < y$, $y < s$ 가 성립했다. 고로 $x \le s$ 이고 이는 새 순서에서도 보존된다. 한편 새 순서에서 $s < y$ 이다. 고로 올바르다.
* (9) $x \notin X \cup Y, y \in Y$: $y$ 가 scanned되었기 때문에, $x \le s$ 가 성립했고 이는 새 순서에서도 보존된다. 한편 새 순서에서 $s < y$ 이다. 고로 올바르다.

이렇게 정당성 증명이 종료되었다. 이제 알고리즘의 양방향 탐색 시간을 계산하자.

두 간선 $a \rightarrow b, c \rightarrow d$이 *related* 라는 것은, 어떠한 간선에서 다른 간선이 도달 가능하다는 것을 뜻한다. 즉, $b$ 에서 $c$ 로 도달가능하거나 $d$ 에서 $a$ 로 도달가능함을 뜻한다. related 간선 쌍은 초기에 0이며, 최대 $\frac{m(m-1)}{2}$ 까지 증가할 수 있다. 

두 간선 $a \rightarrow b, c \rightarrow d$이 *compatible* 하다는 것은, $a < d$ 라는 것이다. 만약 $a \rightarrow b$ 가 forward, $c \rightarrow d$ 가 backward search에서 발견되었다면, 두 간선이 새로이 related되었음을 확인하자. 

양방향 탐색에서, 우리는 항상 탐색하는 두 간선이 *compatible* 함을 보장해야 한다. 이를 우리는 *Compatible Search*라고 부른다. 이를 보장하게끔 양방향 탐색을 하는 단순한 방법은, 다음에 방문할 정점의 리스트를 큐나 스택이 아닌 힙 (priority queue) 를 사용하여 관리하는 것이다. 이렇게 하면 $a$ 가 작고, $d$ 가 큰 순서대로 탐색을 하게 되며, 종료 조건도 쉽게 파악할 수 있다. 여기서, 양방향 탐색 후 $X, Y$ 를 구하는 과정까지 (즉, 위상 정렬 순서를 대체하기 전까지) 든 시간은 (방문한 간선의 개수) * $O(\log N)$ 임을 관찰하자. 논문의 후반부에는 힙을 사용하지 않고 $O(1)$ amortized time에 compatible search를 하는 방법이 적혀 있으나, 여기서 다루지 않는다.

**Theorem 2.** *Compatible Search* 를 사용할 경우, 매 간선 추가 쿼리마다 amortized $O(m^{1/2})$ 개의 간선이 탐색된다.

**Proof.** $2k$ 개의 간선을 탐색한 compatible search를 고려하자. 이 중 $k$ 개의 간선은 forward, $k$ 개의 간선은 backward일 것이다. Forward로 탐색한 간선 $a \rightarrow b$ 를 $a$ 순으로 정렬하자. 이 중 $k/2 + 1, \ldots, k$ 번째 간선에 대해서, 해당 간선과 compatible한 backward 간선이 $k/2$ 개 존재할 것이다. 각각의 쌍들에 대해서 모두 $a < d$ 가 보장된다. 고로, $1 \ldots k / 2$ 번째 forward 간선들도 모두 $a < d$ 가 보장될 것이다. 결론적으로, $1 \ldots k / 2$ 번째 forward 간선, 그리고 $k / 2 + 1, \ldots, k$ 번째 forward 간선에 매칭되는 backward 간선들은 서로 compatible하고, $k^2 / 4$ 개의 related pair가 추가된다. $k \le m^{1/2}$ 이면 위 bound를 만족하며, $k > m^{1/2}$ 일 경우 매 순간 최소 $k m^{1/2} / 4$ 개의 related pair를 추가하게 된다. 고로 $k > m^{1/2}$ 일 경우 전체 쿼리에 대해 $k$ 의 합은 $m^{3/2}$ 를 넘지 않는다. 

**Remark.** 힙을 사용하여 관리할 경우 $k^2 / 4$ 가 아니라 $k^2$ 라는 Bound를 얻을 수 있다. 단순히 모든 forward / backward 간선 쌍에 대해서 compatible함이 쉽게 보장되기 때문이다. 하지만 이는 힙을 사용할 때의 이야기이고, 실제로는 모든 쌍에 대해서 성립하지 않는다.

고로, 위 알고리즘은 매 간선 추가마다 $O(m^{1/2})$ 의 시간을 들여 양방향 탐색을 하고, $O(n)$ 의 시간을 들여서 위상 정렬 순서를 갱신하기 때문에 $O(mn)$ 에 작동한다. 여기까지만 해도 Naive한 방법인 $O(m^2)$ 보다 빠르다는 것을 관찰하자.

### Maintaining the SCC

SCC를 같이 관리하는 방법은 위 알고리즘을 조금 더 응용하면 된다. 새로운 아이디어가 추가되는 것은 아니고, technical한 노력을 더 기울이는 것으로 충분하다.

먼저, 정점 집합에 대한 Union-Find를 관리해서 SCC를 표현하자. 새로운 간선이 추가되었을 때, Union-find를 사용하여 각 끝점이 속하는 SCC를 찾는다. 이후 위에서 사용한 Compatible search를 진행하는데, 사이클이 찾아지는 경우에도 계속 Search를 진행한다는 점에서 차이가 있다. 이 때의 Rule은 다음과 같다.

* 만약 Loop가 발견되었다면, 발견되는 즉시 리스트에서 지우고 계속 탐색한다. (여러 정점이 하나의 SCC로 묶이는 과정에서 Loop가 생길 수 있어서 Search마다 치워줘야 한다.)
* 만약 Forward vertex를 Backward로 찾았다면, Backward vertex로 전환한다.
* 만약 Backward vertex를 Forward로 찾았다면, Forward vertex로 전환한다.

만약 이 과정에서 사이클이 찾아지지 않았다면, 위 알고리즘과 똑같이 Reorder하면 된다. 그렇지 않다면, 위와 같이 집합 $X, Y$ 를 정의한 후, $X \cup Y \cup \{s\}$ 에서 Forward / Backward로 모두 방문이 가능했던 정점을 하나의 SCC로 묶어준다. 이 새로운 SCC를 $Z$ 라고 하고, 남은 정점들은 $s$ 의 Forward / Backward 여부에 따라 $s, Y - Z, X - Z$ 혹은 $Y - Z, X - Z, s$ 의 형태로 배치하면 된다. 새로운 SCC를 만들 때는, 인접 리스트들을 모두 하나의 인접 리스트로 합쳐주고, Union-find를 적절히 사용하여 연결 관계를 재정의하면 된다. 

복잡도 증명도 위 아래와 거의 동일하다. 추가적으로 고려해야 할 것은 Union-Find의 등장인데, `find` 연산의 호출 횟수가 $O(m^{1.5})$ 가 되기 때문에 Path compression이 충분히 된다. 고로 각 연산이 애커만 역함수가 아니라 $O(1)$ 이라고 가정할 수 있어서, 시간 복잡도도 그대로 유지된다.

## Data Structure: Sqrt Decomposition

이제 문제는 위상 정렬 순서를 효율적으로 갱신하는 자료구조를 만드는 것으로 환원되었다. 우리는 아래와 같은 연산을 지원하는 자료구조가 필요하다.

* 두 정점의 위상정렬 대소관계를 $O(1)$ 에 찾을 수 있어야 하고
* 정점 하나를 $O(1)$ 에 지울 수 있어야 하고
* 정점 하나를 $O(1)$ 시간에 삽입할 수 있어야 한다.

이 문제를 [Order-maintenance problem](https://en.wikipedia.org/wiki/Order-maintenance_problem)이라고 부르며, 이를 해결하는 자료구조를 *Dynamic Ordered List* 라고 부른다. Amortized bound를 가정하면, 이미 1982년 Dietz와 Sleator가 이를 해결하는 자료구조를 발견하였다. 고로 전체 문제가 이미 해결되지만, Dynamic Ordered List의 상수가 크고 구현이 복잡하기 때문에 조금 더 간단하면서도 시간 복잡도에서 손해를 보지 않는 대안을 사용한다.

길이가 $\sqrt N$ 인 총 $\sqrt N$ 개의 연결 리스트를 만들며, 이 연결 리스트들을 배열로 관리한다 (즉, 연결 리스트의 배열이다). 배열을 관리하듯이, 중간에 연결 리스트를 삽입하거나 삭제할 일이 있을 경우 인덱스를 완전히 리넘버링해서 관리한다. 연결 리스트 내부에서도 각 원소에 인덱스를 매긴다. 이렇게 관리할 경우 두 정점의 위상정렬 대소관계를 $O(1)$ 에 찾을 수 있으며, 정점 하나를 $ O(1)$ 에 지울 수 있다. 위상정렬 대소 관계는 (연결 리스트의 배열 상 인덱스, 내부 인덱스) 의 쌍을 비교하면 되며, 정점 하나를 지우는 것은 인접 리스트 내부에서 하나씩 지우면 되기 때문이다.

만약에 지우는 과정에서 어떠한 인접한 두 연결 리스트의 길이의 합이 $\sqrt N$ 이하로 줄어들었다고 하자. 그 경우 두 인접한 연결 리스트를 하나의 연결 리스트로 합쳐줄 수 있다. 해당 길이로 줄어들기까지 $\sqrt N$ 개의 연산이 필요했기 때문에, 이 연산에 $O(\sqrt N)$ 의 시간을 소모해도 괜찮다. 고로 새로운 연결 리스트를 만들어 주고, 배열에서 인덱스를 완전히 재배정하여도 시간 복잡도가 유지된다. 이러한 관리를 진행해 주면, 배열의 크기가 $2 \sqrt N$ 이하로 항상 유지된다.

이제 정점을 삽입하는 과정을 처리하자. 여기서 사용할 자료구조는 정점 하나를 $O(1)$ 시간에 삽입할 수 없다. 하지만, 길이 $k$의 리스트를 $O(\sqrt N + k)$ 시간에 삽입할 수는 있다. $s$ 의 위치를 찾은 후, $s$를 지우고 해당 위치에 삽입할 연결 리스트를 만들자. 이후, 배열의 길이를 당겨서, 해당 위치에 새롭게 만든 연결 리스트들을 조각내어 추가해 주면 된다. 이렇게 되면 최대 2개의 연결 리스트에 대해서만 내부적인 relabeling을 진행해 주면 된다. 배열 전체를 relabeling하는 것은 물론 $O(\sqrt N)$ 에 가능하니 그냥 진행해도 된다. 새롭게 삽입한 연결 리스트의 길이가 정확히 $\sqrt N$ 이 아니고 이에 미치지 못할 수 있는데, 이렇다 하더라도 이 단락 3번째 문단에서 논의했던 두 인접 리스트 합치기의 amortization이 유지된다.

이렇게 되면, 문제에서 요구하는 spec의 Order-maintenance problem이 총 $O(M^{1.5} + MN^{0.5})$ 시간에 해결된다.