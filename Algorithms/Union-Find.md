# Union-Find

- 여러 개의 원소가 있을 때, **어떤 원소가 어떤 집합에 속하는지를 빠르게 확인**하고
- **두 집합을 합치는 연산**을 효율적으로 수행하는 자료구조

##  **왜 필요했을까?**

- 어떤 원소들이 **같은 그룹**에 속해 있는지 **빠르게** 알고 싶다.
- 새로운 관계가 생기면 **그룹끼리 합치고 싶다**.

없다면?
A, B가 같은 그룹에 속해있는지 직접 DFS,BFS탐색을 통해서 확인해야함  
또, C가 추가된다면 이를 위해서 다시 또 DFS, BFS탐색을 통해서 확인해야함  

---
## 핵심 연산

**find(x)**  
원소 x가 속한 **집합의 대표(root)를 찾는** 연산

**union(x, y)**  
원소 x가 속한 집합과 원소 y가 속한 집합을 **하나로 합치는** 연산

A, B가 같은 그룹인지 알고싶다면,  
find로 집합의 대표를 각각 찾은 후, 같다면 같은 집합으로 판별

A, B에 새로운 관계가 생겨,
그룹끼리 합치고 싶다면,
union연산으로, 같은 집합으로 합치는 것이다.  

그러면 구현은 어떻게 하였을까?  

---

## 핵심 아이디어 

원소들을 **트리 구조로 표현**하면 빠르게 연결 관계를 추적할 수 있도록 구현

각 원소는 **부모 포인터(parent)**를 가지고,
- 자기 자신이 부모이면 root
- 아니면, **위로 따라가며** root를 찾음

**A -> B의 경우**

| n      | A   | B   |
| ------ | --- | --- |
| arr[n] | B   | B   |

같은 새로운 관계를 추가하고 싶다면, 루트를 찾아 연결시킴(하나의 루트를 자식으로 연결)

**A -> B의 경우 + C의 관계 추가**

| n      | A   | B   | C   |
| ------ | --- | --- | --- |
| arr[n] | B   | B   | B   |

---
## 코드화

#### **초기화**

- 각 원소는 자기 자신이 대표(root)인 독립된 집합
```
	parent = [i for i in range(n+1)]
```
#### **find(x)**

- x의 부모가 자기 자신이면 대표
- 아니면, 재귀적으로 부모를 따라 올라감
```
def find(parent, x):
    if parent[x] != x:
        parent[x] = find(parent, parent[x])  # 경로 압축(Path Compression)
    return parent[x]
```

### **union(x, y)**

- 두 원소의 root를 찾아 서로 다르면 한 쪽을 다른 쪽에 연결
```
def union(parent, a, b):
    root_a = find(parent, a)
    root_b = find(parent, b)
    if root_a != root_b:
        parent[root_b] = root_a  # 또는 parent[root_a] = root_b
```

---
## 성능 문제 → 최적화 연구**

  

초기에는 find()가 느렸음. 트리 구조가 **너무 깊어지면**, 대표를 찾기까지 시간이 오래 걸림.

A -> B -> C -> .......
노드가 한쪽으로 멀어질 수록 root를 찾는대까지 시간이 오래걸림

  

### 경로 압축 (Path Compression)

- find()를 호출할 때마다 **중간 노드들을 대표에 직접 연결**시킴
- → 트리가 얕아짐 → 더 빠르게 대표 찾음
- 재귀 형식으로 찾은 후 return이 이루어지면서 해당값을 반영

### **랭크 기반 합치기 (Union by Rank or Size)**

- 트리의 높이나 크기를 기준으로 **작은 트리를 큰 트리에 붙이기**
- → 트리 높이 유지 → 전체적으로 더 빠름

**필요한 추가 구조**
```
rank = [1] * (n + 1)  # 각 노드의 트리 높이 or 노드 수
```
**개선된 union 함수**
```
def union(parent, rank, a, b):
    root_a = find(parent, a)
    root_b = find(parent, b)

    if root_a == root_b:
        return  # 이미 같은 집합

    # 트리 높이가 낮은 쪽을 높은 쪽에 붙이기
    if rank[root_a] < rank[root_b]:
        parent[root_a] = root_b
    else:
        parent[root_b] = root_a
        if rank[root_a] == rank[root_b]:
            rank[root_a] += 1
```

이 두 가지를 같이 쓰면 **거의 상수 시간(O(α(n)))**에 동작함