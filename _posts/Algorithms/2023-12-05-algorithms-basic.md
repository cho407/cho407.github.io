---
title: "알고리즘 기본 이론(with Swift)"
description: "알고리즘 기본 이론(with Swift)"
categories: Algorithms
tags: [Swift, Algorithms]
sidebar: 
  nav: "docs"
header:
  teaser: /assets/images/algorithms.jpg
---
## 알고리즘의 기본 구성 요소

1. **입력(Input)**: 알고리즘은 0개 이상의 입력을 받을 수 있다. 이 입력은 처리되어야 할 데이터이다.
2. **출력(Output)**: 적어도 하나 이상의 출력을 생성한다. 이 출력은 문제의 해답이다.
3. **명확성(Clearness)**: 각 단계는 명확하고 모호하지 않아야 한다.
4. **유한성(Finiteness)**: 알고리즘은 유한한 수의 단계를 거쳐 종료되어야 한다.
5. **효과성(Effectiveness)**: 모든 연산은 충분히 기본적이어서 수행 가능해야 한다.

## 알고리즘의 중요한 특성

1. **정확성(Correctness)**: 알고리즘은 정확한 결과를 내야 한다.
2. **효율성(Efficiency)**: 자원(시간, 메모리 등)을 최소한으로 사용하여야 한다.
3. **독립성(Independence)**: 알고리즘은 어떤 프로그래밍 언어나 기계에 의존하지 않는 독립적인 문제 해결 방법이어야 한다.

## 알고리즘의 성능 평가

### 시간 복잡도(Time Complexity)

![](https://i.imgur.com/Lkfeb1p.png)

시간 복잡도는 알고리즘의 성능을 평가하는 데 사용되는 중요한 지표로, 알고리즘 실행에 필요한 시간이 입력 크기에 따라 어떻게 변하는지를 나타낸다. 이는 Big O 표기법으로 표현되며, 알고리즘의 최악의 경우에 대한 시간 복잡도를 나타낸다.

1. **O(1) - 상수 시간**: 알고리즘의 실행 시간이 입력 크기와 상관없이 일정하다. 예를 들어, 배열에서 특정 인덱스의 요소에 접근하는 경우가 이에 해당한다.
    
2. **O(log N) - 로그 시간**: 알고리즘의 실행 시간이 입력 크기의 로그에 비례해 증가한다. 이진 탐색이 대표적인 예시다.
    
3. **O(N) - 선형 시간**: 알고리즘의 실행 시간이 입력 크기에 비례하여 증가한다. 예를 들어, 배열을 순회하거나 선형 탐색을 수행하는 경우가 이에 해당한다.
    
4. **O(N log N)**: 많은 효율적인 정렬 알고리즘들이 이 시간 복잡도를 가진다. 병합 정렬이나 퀵 정렬이 여기에 속한다.
    
5. **O(N^2) - 이차 시간**: 알고리즘의 실행 시간이 입력 크기의 제곱에 비례하여 증가한다. 이중 반복문을 사용하는 경우가 대표적이다.
    
6. **O(2^N) - 지수 시간**: 알고리즘의 실행 시간이 입력 크기에 대해 기하급수적으로 증가한다. 재귀적으로 모든 부분 집합을 생성하는 경우가 이에 해당한다.
    
7. **O(N!) - 팩토리얼 시간**: 알고리즘의 실행 시간이 입력 크기의 팩토리얼에 비례하여 증가한다. 완전 탐색 알고리즘이 이에 속한다.
    

시간 복잡도를 이해하고 고려하는 것은 알고리즘을 설계하고 최적화하는 데 매우 중요하다. 특히, 입력 크기가 클 때 알고리즘의 성능 차이가 크게 나타날 수 있으므로, 효율적인 알고리즘 선택과 구현이 필수적이다.

### 공간 복잡도(Space Complexity)

![](https://i.imgur.com/V1eNHnh.png)


공간 복잡도는 알고리즘을 실행하는 데 필요한 메모리 공간의 양을 나타내는 지표로, 알고리즘의 메모리 사용량을 평가하는 데 사용된다. 이는 알고리즘의 효율성을 이해하는 데 중요한 요소로, 특히 메모리 자원이 제한적인 환경에서 중요하게 고려된다.

1. **O(1) - 상수 공간**: 알고리즘 실행 시 필요한 메모리 공간이 입력 크기와 상관없이 일정하다. 예를 들어, 변수를 몇 개 사용하는 간단한 계산이 이에 해당한다.
    
2. **O(N) - 선형 공간**: 알고리즘 실행에 필요한 메모리 공간이 입력 크기에 비례하여 증가한다. 예를 들어, 입력 배열과 동일한 크기의 배열을 추가로 생성하는 경우가 이에 해당한다.
    
3. **O(N^2) - 이차 공간**: 알고리즘 실행에 필요한 메모리 공간이 입력 크기의 제곱에 비례하여 증가한다. 예를 들어, N x N 크기의 2차원 배열을 사용하는 경우가 이에 해당한다.
    

공간 복잡도를 최적화하는 것은 메모리 사용을 줄이고, 프로그램의 효율성을 높이는 데 중요하다. 특히, 대용량 데이터를 처리하거나 메모리가 제한적인 시스템에서는 공간 복잡도를 낮추는 것이 성능 향상에 크게 기여한다.

알고리즘을 설계할 때는 시간 복잡도와 함께 공간 복잡도도 함께 고려하여, 메모리 사용량과 실행 시간 사이의 균형을 맞추는 것이 중요하다. 때로는 시간 복잡도를 개선하기 위해 약간의 추가 메모리를 사용하는 것이 효과적일 수 있으며, 반대로 메모리 사용을 줄이기 위해 실행 시간이 늘어나는 경우도 있다.

## 알고리즘 설계 기법

### 분할 정복(Divide and Conquer)
분할 정복(Divide and Conquer)은 복잡한 문제를 더 작고 관리하기 쉬운 부분 문제로 나누어 해결하는 알고리즘 설계 전략이다. 이 접근 방식은 크게 세 단계로 나눌 수 있다: 분할(Divide), 정복(Conquer), 결합(Combine).

1. **분할(Divide)**: 원래 문제를 더 작은 부분 문제로 분할한다. 이 부분 문제들은 원래 문제의 축소된 버전이다.
    
2. **정복(Conquer)**: 부분 문제를 재귀적으로 해결한다. 부분 문제의 크기가 충분히 작아져 더 이상 분할할 수 없을 때, 직접 해결한다.
    
3. **결합(Combine)**: 부분 문제의 해결책을 결합하여 원래 문제의 해결책을 얻는다.

**분할 정복의 예시: 병합 정렬(Merge Sort)**

```swift
func mergeSort(_ array: [Int]) -> [Int] {
    // 배열의 크기가 1 이하이면 이미 정렬된 것으로 간주
    guard array.count > 1 else { return array }

    // 배열을 반으로 나눔
    let middleIndex = array.count / 2
    let leftArray = mergeSort(Array(array[..<middleIndex]))
    let rightArray = mergeSort(Array(array[middleIndex...]))

    // 나누어진 배열을 병합
    return merge(leftArray, rightArray)
}

func merge(_ left: [Int], _ right: [Int]) -> [Int] {
    var leftIndex = 0
    var rightIndex = 0
    var result = [Int]()

    // 두 배열의 요소를 비교하여 작은 순서대로 result 배열에 추가
    while leftIndex < left.count && rightIndex < right.count {
        if left[leftIndex] < right[rightIndex] {
            result.append(left[leftIndex])
            leftIndex += 1
        } else {
            result.append(right[rightIndex])
            rightIndex += 1
        }
    }

    // 남은 요소들을 결과 배열에 추가
    if leftIndex < left.count {
        result.append(contentsOf: left[leftIndex...])
    }
    if rightIndex < right.count {
        result.append(contentsOf: right[rightIndex...])
    }

    return result
}

// 예제 사용
let unsortedArray = [34, 7, 23, 32, 5, 62]
let sortedArray = mergeSort(unsortedArray)
print(sortedArray)  // 출력: [5, 7, 23, 32, 34, 62]
```

## 동적 프로그래밍(Dynamic Programming)

동적 프로그래밍(Dynamic Programming)은 복잡한 문제를 간단한 하위 문제로 나누어 해결하는 방법이다. 이 방법은 하위 문제의 해결책을 저장하고 재사용함으로써 중복 계산을 피하고, 전체 문제를 효율적으로 해결한다. 동적 프로그래밍은 주로 최적화 문제에 사용되며, 두 가지 주요 접근 방식이 있다: 탑다운(Top-Down)과 보텀업(Bottom-Up) 방식이다.

피보나치 수열 문제와 함께 탑 다운과 보텀 업의 차이를 살펴보자

-  **탑다운(Top-Down) 접근법**: 재귀를 사용하여 큰 문제를 작은 문제로 나누고, 각 하위 문제의 결과를 메모이제이션(Memoization)을 통해 저장한다.

```swift
// 메모이제이션을 위한 딕셔너리 선언
var memo = [Int: Int]()

// 피보나치 수를 계산하는 함수
func fibonacci(_ n: Int) -> Int {
    // 기본 경우: n이 0 또는 1일 때, n을 반환
    if n <= 1 {
        return n
    }

    // 메모이제이션: 이미 계산된 값이 있으면 그 값을 반환
    if let memoizedValue = memo[n] {
        return memoizedValue
    }

    // 재귀 호출을 통해 피보나치 수 계산
    memo[n] = fibonacci(n - 1) + fibonacci(n - 2)

    // 계산된 피보나치 수 반환
    return memo[n]!
}

// 예제: 10번째 피보나치 수 계산
let result = fibonacci(10)  // 결과: 55
print(result)
```

- **보텀업(Bottom-Up) 접근법**: 작은 문제부터 시작하여 점차 큰 문제로 확장해 나가며, 각 단계의 결과를 테이블에 저장한다.

```swift
func fibonacciBottomUp(_ n: Int) -> Int {
    // n이 0 또는 1인 경우, 바로 n을 반환
    if n <= 1 {
        return n
    }

    // 첫 번째 두 피보나치 수는 0과 1
    var fib = [0, 1]

    // 2부터 n까지 각 피보나치 수를 계산하여 배열에 저장
    for i in 2...n {
        fib.append(fib[i - 1] + fib[i - 2])
    }

    // n번째 피보나치 수 반환
    return fib[n]
}

// 예제: 10번째 피보나치 수 계산
let result = fibonacciBottomUp(10)  // 결과: 55
print(result)
```

## 탐욕 알고리즘(Greedy Algorithm)

탐욕 알고리즘(Greedy Algorithm)은 매 선택에서 지역적으로 최적인 선택을 하여, 최종적으로 전체적인 해답을 찾는 방법이다. 이 알고리즘은 각 단계에서 가장 좋아 보이는 선택을 하며, 이러한 선택이 최종적으로 최적의 결과를 낳을 것이라는 가정 하에 작동한다. 탐욕 알고리즘은 주로 최적화 문제에 사용되며, 각 단계의 선택이 전체 해답에 영향을 미치는 방식으로 문제를 해결한다.

**탐욕 알고리즘의 특징**

- **단순하고 직관적인 접근**: 각 단계에서 가장 좋은 선택을 함으로써 문제를 단순화한다.
- **지역 최적화**: 각 단계에서 최적의 선택을 하지만, 이것이 항상 전체적으로 최적인 해답을 보장하지는 않는다.
- **효율적인 계산**: 탐욕 알고리즘은 종종 다른 방법보다 빠른 해결책을 제공한다.

거스름돈 문제는 탐욕 알고리즘을 사용하여 해결할 수 있는 대표적인 예시다. 고객에게 거슬러 줘야 하는 돈이 주어졌을 때, 가장 적은 수의 동전을 사용하여 거슬러 주는 방법을 찾는 문제다.

```swift
func minCoins(for amount: Int, using denominations: [Int]) -> Int {
    let sortedDenominations = denominations.sorted(by: >)  // 큰 단위부터 정렬
    var remainingAmount = amount
    var coinCount = 0

    for coin in sortedDenominations {
        // 현재 동전으로 최대한 거슬러 줄 수 있는 개수를 계산
        let numCoins = remainingAmount / coin
        remainingAmount -= numCoins * coin
        coinCount += numCoins

        // 모든 금액을 거슬러 줬으면 반복 종료
        if remainingAmount == 0 {
            break
        }
    }

    return coinCount
}

// 예제: 47원을 거슬러 주기 위해 필요한 최소 동전 개수
let coinsNeeded = minCoins(for: 47, using: [1, 5, 10, 25])
print(coinsNeeded)  // 결과: 5 (25원 1개, 10원 2개, 1원 2개)
```

## 백트래킹(Backtracking)

백트래킹(Backtracking)은 해결책에 대한 모든 가능한 조합을 시도해보면서 문제를 해결하는 방법이다. 이 방식은 주로 결정 트리(Decision Tree)를 사용하여 모든 가능한 경우의 수를 탐색한다. 백트래킹은 재귀적으로 문제를 해결하며, 각 단계에서 현재의 선택이 유망하지 않다고 판단되면 이전 단계로 돌아가(Backtrack) 다른 선택을 시도한다.

**백트래킹의 특징**

- **시스템적 탐색**: 모든 가능한 해결책을 시스템적으로 탐색한다.
- **가지치기(Pruning)**: 유망하지 않은 경로는 조기에 배제하여 불필요한 탐색을 줄인다.
- **재귀적 구조**: 대부분의 백트래킹 알고리즘은 재귀적으로 구현된다.

다음은 N-Queens 문제는 N×N 체스판 위에 N개의 퀸을 서로 공격할 수 없게 배치하는 문제다. 이 문제는 백트래킹을 사용하여 해결할 수 있다.

```swift
// N-Queens 문제를 해결하는 함수
func solveNQueens(_ n: Int) -> [[String]] {
    // 체스판 초기화: 모든 칸을 '.'으로 설정
    var board = Array(repeating: Array(repeating: ".", count: n), count: n)
    var solutions = [[String]]() // 해결책을 저장할 배열

    // 퀸을 배치하는 함수 호출
    placeQueens(&board, row: 0, n: n, solutions: &solutions)
    return solutions
}

// 퀸을 배치하는 재귀 함수
func placeQueens(_ board: inout [[String]], row: Int, n: Int, solutions: inout [[String]]) {
    // 모든 행에 퀸을 배치했으면 해결책을 solutions 배열에 추가
    if row == n {
        solutions.append(board.map { $0.joined() })
        return
    }

    // 현재 행의 각 열에 퀸을 시도해 본다
    for col in 0..<n {
        if isValid(board, row: row, col: col, n: n) {
            board[row][col] = "Q" // 퀸 배치
            placeQueens(&board, row: row + 1, n: n, solutions: &solutions) // 다음 행으로 이동
            board[row][col] = "." // 백트래킹: 퀸 제거
        }
    }
}

// 현재 위치에 퀸을 배치할 수 있는지 확인하는 함수
func isValid(_ board: [[String]], row: Int, col: Int, n: Int) -> Bool {
    for i in 0..<row {
        // 같은 열, 대각선에 퀸이 있는지 확인
        if board[i][col] == "Q" || // 같은 열 확인
           (col - i - 1 >= 0 && board[row - i - 1][col - i - 1] == "Q") || // 왼쪽 대각선 확인
           (col + i + 1 < n && board[row - i - 1][col + i + 1] == "Q") { // 오른쪽 대각선 확인
            return false
        }
    }
    return true
}

// 예제: 4-Queens 문제의 해결책 출력
let solutions = solveNQueens(4)
for solution in solutions {
    for row in solution {
        print(row)
    }
    print("---")
}
```
