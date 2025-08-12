# HashMap

HashMap 코드 분석

코드: [openjdk-jdk8u](https://github.com/AdoptOpenJDK/openjdk-jdk8u/blob/master/jdk/src/share/classes/java/util/HashMap.java)

## putVal 메서드

```java
if ((tab = table) == null || (n = tab.length) == 0)
    n = (tab = resize()).length;
```

- `(tab = table) == null`: tab 변수에 table 변수의 값을 할당하고 tab 변수가 null인지 확인합니다.
  - `(tab = table)`: 조건절에서 변수에 할당하는 것을 **할당식**(assignment expression)이라 합니다. 값을 할당하는 행위 자체를 하나의 표현식으로 간주합니다. 이 표현식은 할당된 값 자체를 반환합니다. 여기서 이 할당식의 결과는 변수 tab에 담긴 값입니다.

```java
if ((p = tab[i = (n - 1) & hash]) == null)
    tab[i] = newNode(hash, key, value, null);
```

- `p = tab[i = (n - 1) & hash]`
  - 변수 i에 (n - 1)과 hash의 비트와이즈 AND 연산자 결과를 대입합니다.
  - 그리고 tab의 i 번째 요소를 p에 대입합니다.
  - 만약 p가 null이면 해당 요소에 새로운 노드를 생성합니다.
- `(n - 1) & hash`의 의미
  - HashMap 설계에 따라 n(해시맵의 버킷 크기)은 항상 2의 거듭제곱입니다. 그리고 버킷의 인덱스를 설정할 때에는 hash 값이 무엇이냐에 상관없이 항상 버킷 크기보다 작은 수를 고르게 분포시키기 위해 나머지 연산을 사용합니다. (`hash % n`)
  - n이 항상 2의 거듭제곱이므로 (n - 1) & hash와 hash % n이 같은 결과를 내게 됩니다.
  - 그런데 나머지 연산보다 비트 연산자의 성능이 우수하기 때문에 비트 연산자를 사용한 것입니다.

## resize 메서드

## hash 메서드
