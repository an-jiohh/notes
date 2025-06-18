# **Prim 알고리즘 정리**

Prim 알고리즘은 하나의 정점에서 출발하여, **아직 방문하지 않은 정점 중 가장 짧은 간선을 선택하며 확장**하는 방식이다.

### **핵심 아이디어**

- 방문하지 않은 노드 중, **가장 비용이 적은 간선을 선택**
    
- 이미 연결된 노드 집합에서 **인접한 최소 간선을 우선순위 큐로 관리**
    
- **사이클 방지**를 위해 이미 방문한 노드는 제외
    

---

## **알고리즘 절차**

1. 임의의 정점에서 시작
    
2. 연결된 간선 중 가중치가 가장 작은 간선을 선택
    
3. 해당 간선이 연결된 노드가 아직 방문되지 않았다면:
    
    - 해당 노드를 MST에 포함
        
    - 해당 노드에서 연결된 간선을 우선순위 큐에 추가
        
    
4. 모든 정점이 연결될 때까지 반복
    

---

## **우선순위 큐(PriorityQueue) 활용**

  

Prim 알고리즘에서 가장 작은 가중치 간선을 빠르게 찾기 위해 **우선순위 큐(힙)**를 사용한다.

  

파이썬에서는 heapq 모듈을 사용하여 최소 힙을 구현할 수 있다.

```
import heapq

heapq.heappush(queue, (weight, node))
weight, node = heapq.heappop(queue)
```

---

## **Python 코드 예시**

```
import heapq

def prim(V, graph, start=1):
    visited = [False] * (V + 1)
    min_heap = []
    total_cost = 0

    # 시작 노드에서 연결된 간선들을 힙에 추가
    visited[start] = True
    for to, weight in graph[start]:
        heapq.heappush(min_heap, (weight, to))

    while min_heap:
        weight, node = heapq.heappop(min_heap)
        if not visited[node]:
            visited[node] = True
            total_cost += weight
            for to, w in graph[node]:
                if not visited[to]:
                    heapq.heappush(min_heap, (w, to))

    return total_cost
```

**입력 예시**
```
V = 4
graph = {
    1: [(2, 3), (3, 5), (4, 4)],
    2: [(1, 3), (3, 2)],
    3: [(1, 5), (2, 2), (4, 1)],
    4: [(1, 4), (3, 1)]
}

result = prim(V, graph)
print("최소 스패닝 트리의 총 비용:", result)
```

---
## **시간 복잡도 분석**

| **연산** | **복잡도**    |
| ------ | ---------- |
| 간선 탐색  | O(E log V) |
| 힙 연산   | O(log V)   |

Prim은 **Priority Queue + Adjacency List** 조합을 사용하면 성능이 매우 좋으며, 특히 **밀집 그래프(Dense Graph)**에 적합하다.

---

## **Kruskal과의 비교**

|**기준**|**Kruskal**|**Prim**|
|---|---|---|
|접근 방식|간선 중심|정점 중심|
|정렬 필요|간선 정렬|불필요|
|사이클 확인|Union-Find|visited 배열|
|적합한 그래프|희소 그래프|밀집 그래프|
|자료구조|Union-Find|우선순위 큐|
