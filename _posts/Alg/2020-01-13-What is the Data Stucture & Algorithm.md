자료구조는 무엇이며 알고리즘은 무엇인가에 대해 생각해보며 여러 포스트를 진행해 보려 합니다.

# 정의

***Program = Data structure + Algorithm***

프로그램은 입력된 데이터를 어떤식으로 저장하고 (자료구조) 처리하는지 (알고리즘) 에 따라 그 성능이 확연히 달라집니다.
적은 양의 데이터를 처리하는 프로그램이라면 자료구조, 알고리즘은 잊어도 될 것 같습니다. 하지만 대용량의 데이터를 지속적으로 처리하는 프로그램이라면 반드시 자료구조, 알고리즘을 고려해야 합니다.

## 자료구조

자료구조는 **데이터를 표현하는 방법 및 구조** 라고 정의할 수 있습니다.

![](https://qph.fs.quoracdn.net/main-qimg-b39bae5b6be3d7f21738eb7ec04aae13)

위의 그림은 자료구조가 어떤 식으로 분류될 수 있는가를 보여줍니다.

- Primitive type
  - Int
  - Char
  - Bool
- Non-primitive type
  - Linear
    - Array
    - Linked List
    - Stack
    - Queue
    - Deque
  - Non-Linear
    - Tree
    - Graph

## 알고리즘

알고리즘은 **데이터를 처리하는 방법** 이라고 정의할 수 있겠네요.

아래는 여러 기준으로 [알고리즘을 구분 및 분류](http://www.ktword.co.kr/word/abbr_view.php?m_temp1=5735) 해본 것 입니다.

### 1. 알고리즘의 주제별 분류

- 기초적인 알고리즘
  - 최대값 또는 최소값 찾기 등
    - 가장 큰 숫자를 기억해가며 진행함 
  - 유클리드 알고리즘
    - 두 정수의 최대공약수를 빠르게 구하기

- 탐색 알고리즘 (Searching Algorithm)
  - 탐색 문제 
    - 순서화된 리스트(ordered list)에서 어떤 원소의 위치 및 존재유무를 찾는 것
  - 탐색문제의 해 또는 결과
    - 원소의 위치
  - 주요 종류
    - 순차 탐색(sequential search)/선형탐색(linear search)
    - 이진 탐색 (binary search) 등

- 정렬 알고리즘 (Sorting Algorithm)
  - 정렬 문제
    - 수 많은 자료를 특정 목적에 맞게 순서있게 재 배열하는 것
  - 정렬문제의 해 또는 결과
    - 정렬된 배열
  - 주요 종류
    - 선택 정렬 (Selection Sort)
    - 버블 정렬 (Bubble Sort)
    - 삽입 정렬 (Insertion Sort)
    - 퀵 정렬 (Quick Sort)
    - 병합 정렬 (Merge Sort)

- 그래프 알고리즘 (Graph Algorithm)
  - 주요 종류
    - 그래프 순회(Graph Traversal)/탐색,검색(Graph Search) 방법
    - 신장 트리(생성 트리) 작성 방법
    - 최소비용 생성트리 작성 알고리즘
    - 최단경로 알고리즘

- 해시 알고리즘 (Hash Algorithm)

- 최적화 알고리즘(Optimizing Algorithm) 등니다.

### 2. 알고리즘에 확률의 개입 여부에 따른 분류

- 결정 알고리즘 (Deterministic Algorithm)
  - 항상 정확한 답을 반환하는 알고리즘
  - 例) 위 1.항에 언급된 대부분의 알고리즘들

- 확률 알고리즘 (Probabilistic Algorithm), 무작위 알고리즘 (Randomized Algorithm)
  - 실행될 명령이 무작위로 결정되는 알고리즘
    - 몬테칼로 알고리즘 : 무작위로 난수를 발생시켜 근사적인 해를 구하는 알고리즘
    - 라스베가스 알고리즘 : 무작위 시간으로 정확한 해를 구하는 알고리즘

### 3. 알고리즘의 설계기법/전략/파라다임에 의한 분류

  ※ ☞ 알고리즘 설계 참조
     - 분할정복(Divide and Conquer) 전략
        . 규모가 큰 문제를 만만한 작은 조각으로 나눠 각개격파 
     - 동적계획법(Dynamic Programming) 전략
     - 탐욕 알고리즘(Greedy Algorithm) 전략 등
     
# 시간 복잡도 (Time complexity)

시간 복잡도란 핵심 연산의 반복이 얼만큼 일어나는 가를 보고 판단합니다. 자료구조와 알고리즘을 적절히 사용함으로서 시간복잡도를 줄이는 것이 목적이라면 목적이라 할 수 있습니다. 

![](https://joshuajangblog.files.wordpress.com/2016/09/1.jpg?w=638)
![](https://miro.medium.com/max/843/0*A5SQeJqDoLGhCYPg.png)

볼게 많습니다. 시간이 걸리겠지만 하나하나 살펴보고 깊이있게 들여다볼 생각입니다.


