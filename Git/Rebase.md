
# **Rebase**

## **1. Rebase란?**

`rebase`는 현재 브랜치의 커밋들을 다른 브랜치의 최신 커밋 뒤로 다시 붙이는 작업이다.

쉽게 말하면, 브랜치가 갈라진 상태에서 내 브랜치의 시작점을 최신 기준으로 바꾸는 것이다.

예를 들어 처음 상태가 아래와 같다고 하자.

```text
main:    A --- B
              \
feature:       C --- D
```

여기서 `feature` 브랜치에서 `main`을 기준으로 rebase하면 다음처럼 된다.

```text
main:    A --- B
                  \
feature:           C' --- D'
```

겉보기에는 `C`, `D` 커밋이 그대로 이동한 것 같지만 실제로는 `C'`, `D'`처럼 새로운 커밋으로 다시 만들어진다.

즉, rebase는 기존 커밋을 그대로 옮기는 것이 아니라, 같은 변경 내용을 새 기준 위에 다시 적용하는 작업이다.

---

## **2. Rebase를 왜 사용할까?**

Rebase를 사용하는 가장 큰 이유는 커밋 기록을 깔끔하게 만들기 위해서이다.

`merge`를 사용하면 브랜치가 갈라졌다가 합쳐진 기록이 그대로 남는다.

```text
A --- B -------- M
 \             /
  C --- D ----
```

반면 `rebase`를 사용하면 커밋들이 한 줄로 정리된다.

```text
A --- B --- C' --- D'
```

그래서 작업 흐름을 나중에 확인할 때 더 단순하게 볼 수 있다.

다만 rebase는 커밋을 새로 만들기 때문에 이미 다른 사람과 공유한 커밋에는 조심해서 사용해야 한다.

---

## **3. Merge와 Rebase의 차이**

### **Merge**

`merge`는 두 브랜치의 변경 내용을 합친다.

예를 들어 `main` 브랜치에서 아래 명령어를 실행하면:

```bash
git merge feature
```

현재 브랜치인 `main`에 `feature` 브랜치의 변경 내용이 합쳐진다.

이때 상황에 따라 `merge commit`이라는 새로운 커밋이 생길 수 있다.

```text
main:    A --- B -------- M
              \        /
feature:       C --- D
```

`merge`는 기존 커밋 기록을 그대로 보존한다.

---

### **Rebase**

`rebase`는 현재 브랜치의 커밋들을 다른 브랜치 뒤로 다시 붙인다.

예를 들어 `feature` 브랜치에서 아래 명령어를 실행하면:

```bash
git rebase main
```

`feature` 브랜치의 커밋들이 `main`의 최신 커밋 뒤로 다시 적용된다.

```text
main:    A --- B
                  \
feature:           C' --- D'
```

`rebase`는 커밋 기록을 한 줄로 정리하는 데 유리하다.

---

## **4. Merge 방향과 Rebase 방향**

Merge와 rebase는 방향을 헷갈리기 쉽다.

### **main에서 feature를 merge하는 경우**

```bash
git switch main
git merge feature
```

이 명령은 `feature`의 내용을 `main`에 합치는 것이다.

즉, 최종 결과는 `main` 브랜치가 바뀐다.

---

### **feature에서 main을 merge하는 경우**

```bash
git switch feature
git merge main
```

이 명령은 `main`의 내용을 `feature`에 합치는 것이다.

즉, 최종 결과는 `feature` 브랜치가 바뀐다.

---

### **feature에서 main을 rebase하는 경우**

```bash
git switch feature
git rebase main
```

이 명령은 `feature`의 커밋들을 `main` 뒤로 다시 붙이는 것이다.

즉, `main`이 바뀌는 것이 아니라 `feature`의 커밋 위치가 바뀐다.

---

## **5. Rebase는 언제 사용하면 좋을까?**

Rebase는 보통 다음과 같은 상황에서 사용한다.

```text
main 브랜치가 먼저 변경되었고,
내 feature 브랜치도 따로 작업이 진행된 상태
```

예를 들면 다음과 같다.

```text
main:    A --- B --- C
              \
feature:       D --- E
```

이때 `feature` 브랜치에서:

```bash
git rebase main
```

을 실행하면 다음처럼 된다.

```text
main:    A --- B --- C
                      \
feature:               D' --- E'
```

이렇게 하면 내 작업 커밋들이 최신 `main` 뒤에 정리된다.

---

## **6. Rebase 중 충돌이 발생하는 이유**

Rebase 중 충돌이 발생하는 이유는 같은 파일의 같은 부분을 서로 다르게 수정했기 때문이다.

예를 들어 `main`에서는 어떤 줄을 이렇게 바꾸고:

```java
return "main";
```

`feature`에서는 같은 줄을 이렇게 바꿨다면:

```java
return "feature";
```

Git은 어떤 내용을 선택해야 할지 자동으로 판단할 수 없다.

그래서 충돌이 발생한다.

---

## **7. Rebase 충돌 해결 흐름**

Rebase 중 충돌이 발생하면 보통 아래 순서로 해결한다.

```bash
git status
```

먼저 어떤 파일에서 충돌이 났는지 확인한다.

충돌 파일을 열어보면 보통 아래처럼 표시된다.

```text
<<<<<<< HEAD
main 브랜치 쪽 내용
=======
feature 브랜치 쪽 내용
>>>>>>> 커밋해시
```

여기서 필요한 내용만 남기고 충돌 표시를 삭제한다.

예를 들어 최종적으로 아래처럼 정리한다.

```java
return "resolved";
```

그다음 수정한 파일을 스테이징한다.

```bash
git add 파일명
```

그리고 rebase를 계속 진행한다.

```bash
git rebase --continue
```

---


## **8. 왜 Rebase는 rebase --continue를 추가적으로 실행해야할까?

Merge 중 충돌을 해결하면 보통 아래처럼 commit을 한 뒤 바로 Merge가 진행되게 된다.

```bash
git commit
```

하지만 rebase 중에는 다음 명령어를 추가적으로 사용해야한다.

```bash
git rebase --continue
```

이유는 rebase가 여러 커밋을 하나씩 다시 적용하는 작업이기 때문이다.

즉, Git은 현재 커밋 하나만 처리하는 것이 아니라, 남아 있는 커밋들을 순서대로 계속 적용해야 한다.

그래서 사용자가 충돌을 해결하고 `git add`를 하면, Git에게 이렇게 알려줘야 한다.

```text
이 커밋의 충돌은 해결했으니, 다음 커밋 rebase를 계속 진행해라.
```

이후에 커밋을 하나하나씩 추가적으로 해결해나가야하기 때문에 git rebase --continue로 진행하게 된다.

---

## **9. Rebase 중 자주 쓰는 명령어**

### **현재 상태 확인**

```bash
git status
```

현재 rebase 중인지, 어떤 파일에서 충돌이 났는지 확인할 수 있다.

---

### **충돌 해결 후 계속 진행**

```bash
git add 파일명
git rebase --continue
```

충돌 파일을 수정한 뒤 rebase를 이어간다.

---

### **Rebase 취소**

```bash
git rebase --abort
```

rebase를 시작하기 전 상태로 되돌린다.

충돌 해결이 너무 꼬였거나 처음부터 다시 하고 싶을 때 사용한다.

---

### **현재 커밋은 건너뛰기**

```bash
git rebase --skip
```

현재 적용 중인 커밋을 건너뛴다.

다만 해당 커밋의 변경 내용이 빠질 수 있으므로 조심해서 사용해야 한다.

---

## **10. Merge 중에는 왜 switch가 안 될까?**

Merge나 rebase가 진행 중인 상태에서는 브랜치를 마음대로 바꾸기 어렵다.

이유는 Git이 현재 작업을 완료하지 않은 상태로 기억하고 있기 때문이다.

예를 들어 merge 중 충돌이 발생하면 Git은 이런 상태가 된다.

```text
아직 merge가 끝나지 않았다.
충돌을 해결하고 commit을 해야 한다.
```

이 상태에서 다른 브랜치로 이동하면 작업 상태가 꼬일 수 있다.

그래서 먼저 다음 중 하나를 해야 한다.

### **충돌을 해결하고 merge 완료**

```bash
git add 파일명
git commit
```

### **merge 취소**

```bash
git merge --abort
```

rebase 중이라면:

```bash
git rebase --abort
```

로 취소할 수 있다.

---

## **11. Rebase를 사용할 때 주의할 점**

Rebase는 커밋 기록을 다시 만드는 작업이다.

그래서 혼자 작업 중인 브랜치에서는 비교적 안전하게 사용할 수 있다.

하지만 이미 다른 사람과 공유한 커밋을 rebase하면 커밋 해시가 바뀌기 때문에 문제가 생길 수 있다.

정리하면 다음과 같다.

```text
혼자 작업 중인 브랜치 → rebase 사용 가능
이미 공유된 커밋 → rebase 조심
기록을 그대로 남기고 싶음 → merge
기록을 깔끔하게 정리하고 싶음 → rebase
```

---

## **12. Merge와 Rebase 선택 기준**

### **Merge를 쓰면 좋은 경우**

- 브랜치가 합쳐진 기록을 남기고 싶을 때
- 협업 중인 브랜치의 기록을 안전하게 보존하고 싶을 때
- 커밋 기록이 조금 복잡해져도 상관없을 때

### **Rebase를 쓰면 좋은 경우**

- 커밋 기록을 한 줄로 깔끔하게 정리하고 싶을 때
- 내 작업 브랜치를 최신 기준 위로 올리고 싶을 때
- 아직 다른 사람과 공유하지 않은 개인 작업 브랜치일 때