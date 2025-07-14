# **배낭 문제(Knapsack Problem) 정리**
  

배낭 문제(Knapsack Problem)는 동적계획법(DP)이 자주 사용되는 대표적인 최적화 문제 중 하나이다. 제한된 무게를 가진 배낭에 담을 수 있는 물건들을 선택할 때, **전체 가치(value)의 합을 최대화**하는 조합을 찾는 것이 목표이다.

---

## **문제 종류**

  

### **1. 0/1 배낭 문제 (0/1 Knapsack)**

- 각 물건을 **한 번만 선택하거나 선택하지 않을 수 있음**
    
- 예: “책을 고르되 같은 책은 두 번 넣을 수 없음”
    

  

### **2. 완전 배낭 문제 (Unbounded Knapsack)**

- 각 물건을 **무제한으로 선택 가능**
    
- 예: “같은 종류의 사탕을 여러 번 넣어도 됨”
    

  

### **3. 분할 배낭 문제 (Fractional Knapsack)**

- 물건을 **쪼개어 일부만 선택 가능**
    
- 예: “곡물 같은 것을 무게 단위로 나누어 담을 수 있음”
    
- 보통 **그리디 알고리즘**으로 해결 가능
    

---

## **1. 0/1 배낭 문제**

- 각 물건을 **한 번만 선택할 수 있음**
    
- 부분집합 조합을 찾아야 함
    
- **“넣거나 안 넣거나”** 의 이진 선택

**목표:** 무게 합이 W 이하가 되도록 물건을 골라, **가치 합을 최대**로 만들기

### **알고리즘**

- **동적계획법 (DP) 사용**
    
- 상태는 (i, w) → i번째 물건까지 고려하고 현재 배낭 무게가 w일 때 최대 가치
### 왜 DP인가? 핵심 아이디어
- n개의 물건
- weight 배열에 각 물건의 무게
- w의 가방 무게 제한

ns(n, w)의 문제를 푸는 것이라고 할때,
ns점화식으로 진행하며 ns(n-1, w-?)하위로 내려가면서 문제를 접근

이때 물건이 없을 때랑 있는상태로 하위 문제를 분리하여 접근하는 것이 핵심
"ABCD"물건이 있고 w = 5, weight = {1,2,3,4} 이라는 상황에서

ns("ABCD", 5)
-> ns("ABC", 1),  D가 포함됨
-> ns("ABC", 5), D가 포함되지 않음
하위 문제를 이렇게 나누어 DP를 진행


### **점화식 (Bottom-up)**

```
nx(n, w) = max (
					nx(n-1, w-weight[n]) + val[n],
					nx(n-1, w) + 0
)

```

dp[i][w] = i번째까지의 물건을 고려했을 때, 총 무게 w 이하로 담을 수 있는 최대 가치

```python
n, k = map(int, input().split())

weight = [0]
value = [0]

dp = [[0]*(k+1) for i in range(n+1)]


for i in range(n):
	a,b = map(int, input().split())
	weight.append(a)
	value.append(b)

for i in range(1, n+1):
	for j in range(k+1):
		if weight[i] > j :
			dp[i][j] = dp[i-1][j]
		else :
			dp[i][j] = max(dp[i-1][j], dp[i-1][j - weight[i]]+value[i])
print(dp[n][k])
```

#### 수도 코드
1. dp 배열 설정
2. 물건 여부를 바탕으로 순회, for(1,n+1) = 아무것도 안넣는 것을 고려하여 1부터 순회
3. 무게를 바탕로 순회 for(k+1) 
4. if 다음 무게를 추가할 수 있는지 판별 if weight{i} > j , weight가 큼으로 넣는 상황 고려 못함
	1. 넣지 못함으로 이전의 값을 그대로 반영 dp{i-1}{j}
5. weight를 넣는 것을 고려할 수 있음으로 다음 항목중 최대치를 현재배열에 설정
	1. dp{i-1}{j} = 이전값
	2. dp{i-1}{j-weight{i}}+value{i} = 이전값중 넣을 수 있는값 + 현재 value
 
### **공간 최적화 (1차원 DP 배열 사용)**

```
def knapsack_optimized(N, W, weights, values):
    dp = [0] * (W + 1)

    for i in range(N):
        for w in range(W, weights[i] - 1, -1):
            dp[w] = max(dp[w], dp[w - weights[i]] + values[i])

    return dp[W]
```

- 내부 반복을 거꾸로 하는 이유는 **같은 i에서 중복 계산을 피하기 위해서**다.


---

## **2. 완전 배낭 문제 (Unbounded Knapsack)


- 각 물건을 **여러 번 선택 가능**
    
- 예: 사탕, 쌀처럼 개수 제한 없이 고를 수 있는 경우
    
- **“넣고 또 넣을 수 있음”**

### **알고리즘**

- **동적계획법 사용 (DP 배열 1차원으로 가능)**
    
- 순서를 조절해 **중복 사용을 허용**함

### **점화식 (1차원 DP)**

```python
dp[w] = max(dp[w], dp[w - weight[i]] + value[i])
```

### **구현 방식**
```python
def knapsack_unbounded(n, w, weights, values):
    dp = [0] * (w + 1)
    for i in range(n):
        for j in range(weights[i], w + 1):
            dp[j] = max(dp[j], dp[j - weights[i]] + values[i])
    return dp[w]
```

---

## **3. 분할 배낭 문제 (Fractional Knapsack)**

- 그리디를 이용하여 풀이

가격 / 무게 = 1단위당 가격으로 한다음
1단위당 가격이 가장 높은 물건부터 그리디로 넣으면 풀이 가능