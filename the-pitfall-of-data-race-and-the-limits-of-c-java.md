---
theme: gaia
---
<!-- _class: lead -->
# Data Race의 함정과 C/Java의 한계
### 발표자: 이재현 (jaehylee)
---
## 목차
 - Data Race의 정의
 - 함정 :rotating_light:
 - C/C++의 경우
 - Java의 경우
 - 해결책
 - 이걸 신경써야 하는 이유
 - 결론
---
## Data Race의 정의
1. 두 개 이상의 스레드가 동일한 메모리에 **_동시에_** 접근할 때
2. 그 중 하나 이상의 스레드가 쓰기 작업을 수행할 때
3. 그 중 하나 이상의 스레드가 **_동기화되지_** 않았을 때
---
<!-- _class: lead gaia -->

# `두 개 이상의` 스레드?
---
## 두 개 이상의 스레드?
```java
List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
names.add("Charlie");

for (String name : names) {
    if ("Bob".equals(name)) {
        names.remove(name); // ???
    }
}
```
---
## 함정 :rotating_light:
```
Exception in thread "main" java.util.ConcurrentModificationException
...
```
### **동시** 수정???
```
This exception may be thrown by methods that have detected
concurrent modification of an object when such modification is not permissible.
...
Note that this exception does not always indicate that an object has been
concurrently modified by a different thread...
```
---
## 왜 이런 일이 발생하는가
- `foreach`로 순회하며 도는 동안에는 `List`가 변할 거라고 생각하지 않는다.
  (동일한 메모리 주소로 고정될 것이라 가정한다)
- 그런데 `list.remove()`를 하면 객체 내부의 메모리 관리로 인해 메모리 주소의 변경이 일어날 수 있다.
- 그런데 `foreach`의 다음 부분에서는 동일한 메모리 주소를 가정하므로 이미 해제된 메모리에 접근할 수 있다.
  (**Dangling Pointer**) :boom:
- 이 모든 것은 하나의 전제에서 출발:
**"읽는 동안에는 데이터가 변하지 않을 것이다"**
---
## C/C++은 어떻게 대응하는가?
- `mutex`를 안쓰면 몰?루
- 개발자를 믿는 언어
- **책임도 모두 개발자에게**
---
## Java는 어떻게 대응하는가?
- `foreach`문이 돌 때마다 `List`의 길이를 측정
- 길이가 달라질 경우 데이터나 메모리가 바뀌었다고 판단
- `ConcurrentModificationException` 투척
- 하지만 결국 **원천적으로 막을 수는 없다**
---
## 그럼 어떻게 하지?
- 대부분의 코드는 데이터를 읽는 동안에는 변경되지 않을 거라 가정한다
- 따라서 읽는 것이 모두 끝나기 전에는 변해서는 **안된다**
- 읽는 것이 잠깐 끊겼다가 다시 재개될 때에도 마찬가지
- 이 모든 규칙을 코드를 **실행하기 전에** 검사해야 안전하다 :bug:
---
## 이걸 왜 신경써야 함?
### 예상되는 최소 피해
- 데이터가 내가 예상 못한 이상한 상태가 됨
- 실행할 때마다 달라짐, 프로그램의 작동 예측/제어 불가
### 예상되는 최대 피해
- 접근하면 안되는 메모리에 접근, `SEGFAULT` 발생
- `undefined behavior`로 인한 **_보안 취약점 발생_**
---
## 이걸 왜 신경써야 함?
### CVE-2025-49844 ("RediShell" 취약점)
- 2025년 10월 3일 발견
- Redis의 `Lua` 스크립트 엔진에서 해제된 메모리에 접근하는 취약점
- 위험도: 치명적, **원격으로 어떤 코드도 실행 가능** (RCE)
- Redis에서 `Lua` 스크립트 지원 시작 때부터 **13년간** 발견 못함
- 아마도 해커들은 다크웹을 통해 알고, 몇번 공격해서 이득을 취했을 듯 (얼마나 되는지는 모름)
---
## 결론
- 이런 취약점이 더 이상 남 일이 아님
- 해킹에도 AI 사용으로 예상하기 힘든 공격 증가
- 내가 만드는 소프트웨어는 과연 안전할까?
- 이런 취약점을 처음부터 막을 수는 없을까?
---
<!-- _class: lead gaia -->
# Q&A
