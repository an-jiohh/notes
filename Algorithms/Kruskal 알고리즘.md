# **Kruskal 알고리즘 정리**

Kruskal 알고리즘은 **간선을 가중치 오름차순으로 정렬한 뒤**, **사이클이 생기지 않도록** 가장 작은 간선을 하나씩 선택하며 MST를 구성한다.
  

### **핵심 아이디어**

- **가장 짧은 간선부터** 선택
    
- **사이클이 생기면 제외**(Union-Find 사용)


---

## **알고리즘 절차**

1. 모든 간선을 가중치 기준으로 **오름차순 정렬**
    
2. Union-Find 자료구조를 이용해 **사이클 여부 판별**
    
3. 간선을 하나씩 확인하면서,
    
    - 서로 연결되지 않은 두 정점을 연결하고
        
    - MST에 포함시키며
        
    - 총 비용을 누적
        
    
4. 간선 수가 (정점 수 - 1)이 되면 종료
    

---

## **주요 자료구조: Union-Find**

Union-Find는 **집합의 대표 노드를 찾아서**, 두 노드가 같은 집합에 속해 있는지 판단하고, 두 집합을 하나로 합치는 데 사용된다.

---
## **Python 구현**

```
# Union-Find 구현
def find(parent, x):
    if parent[x] != x:
        parent[x] = find(parent, parent[x])  # 경로 압축
    return parent[x]

def union(parent, a, b):
    root_a = find(parent, a)
    root_b = find(parent, b)
    if root_a < root_b:
        parent[root_b] = root_a
    else:
        parent[root_a] = root_b

# Kruskal 메인 함수
def kruskal(V, edges):
    # 1. 간선 리스트 정렬
    edges.sort(key=lambda x: x[2])  # 가중치 기준 정렬
    parent = [i for i in range(V + 1)]  # 1-based index

    total_cost = 0
    count = 0

    for u, v, weight in edges:
        if find(parent, u) != find(parent, v):
            union(parent, u, v)
            total_cost += weight
            count += 1
            if count == V - 1:
                break

    return total_cost
```

---
## **자바 코드 예시**

```
class Edge implements Comparable<Edge> {
    int from, to, weight;

    Edge(int from, int to, int weight) {
        this.from = from;
        this.to = to;
        this.weight = weight;
    }

    @Override
    public int compareTo(Edge o) {
        return this.weight - o.weight;
    }
}

int kruskal(int V, List<Edge> edges) {
    Collections.sort(edges);
    int[] parent = new int[V + 1];
    for (int i = 1; i <= V; i++) parent[i] = i;

    int totalWeight = 0;
    int count = 0;

    for (Edge e : edges) {
        int uRoot = find(parent, e.from);
        int vRoot = find(parent, e.to);

        if (uRoot != vRoot) {
            union(parent, uRoot, vRoot);
            totalWeight += e.weight;
            count++;
            if (count == V - 1) break;
        }
    }
    return totalWeight;
}
```

---

## **시간 복잡도 분석**

| **단계**                             | **시간 복잡도**               |
| ---------------------------------- | ------------------------ |
| 간선 정렬                              | O(E log E)               |
| Union-Find (with path compression) | O(α(N)) ≈ O(1)           |
| 전체 수행 시간                           | **O(E log E)** (E는 간선 수) |

> 대부분의 경우에서 Kruskal은 빠르게 동작하며, 특히 **희소 그래프(sparse graph)**에 효과적이다.

---

## **특징 요약**

|**항목**|**내용**|
|---|---|
|알고리즘 분류|그리디 (Greedy)|
|입력 형태|간선 리스트|
|필요 자료구조|Union-Find|
|적합한 그래프|희소 그래프|
|사이클 방지 방법|두 정점이 같은 집합인지 확인|
|핵심 연산|간선 정렬 + 집합 병합|
