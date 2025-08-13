# HashMap

HashMap 코드 분석

코드: [openjdk-jdk8u](https://github.com/AdoptOpenJDK/openjdk-jdk8u/blob/master/jdk/src/share/classes/java/util/HashMap.java)

## putVal 함수

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

```java
else if (p instanceof TreeNode)
    e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
else {
    for (int binCount = 0; ; ++binCount) {
        if ((e = p.next) == null) {
            p.next = newNode(hash, key, value, null);
            if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                treeifyBin(tab, hash);
            break;
        }
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            break;
        p = e;
    }
}
```

- `else if (p instanceof TreeNode)`
  - p의 타입이 TreeNode이면 p에 새로운 노드를 추가합니다.
- `else`
  - 이제 p는 연결 리스트의 요소입니다. 노드를 순서대로 순회하며 마지막 노드일 때(next가 null일 때) next에 새로운 노드를 추가합니다.
  - `if (binCount >= TREEIFY_THRESHOLD - 1)`
    - 마지막 노드의 순서가 `TREEIFY_THRESHOLD`에서 1을 뺀 숫자보다 크거나 같으면 버킷을 트리로 변경합니다.
    - **1을 빼는 이유**: 반복문을 처음 순회할 때 이미 p에는 첫 번째 노드가 남겨져 있습니다. 그렇기 때문에 마지막 노드에 새 노드를 추가할 때의 binCount는 첫 번째 노드가 포함되어 있지 않습니다. 따라서 TREEIFY_THRESHOLD에서 1을 뺀 값과 비교를 하게 됩니다.
    - `TREEIFY_THRESHOLD`
      - 버킷에 담긴 노드 수가 이 값 이상이면 버킷의 자료구조를 트리로 변경합니다.
      - 반대로 `UNTREEIFY_THRESHOLD`가 있습니다. 이 값 이하이면 버킷의 자료구조를 연결 리스트로 변경합니다.
- HashMap 버킷(bin)의 자료 구조 2가지
  - **왜 2개의 자료 구조를 사용할까?**
    - 연결 리스트보다 레드-블랙 트리의 시간 복잡도가 낮기 때문에 노드 수가 많아지면 레드-블랙 트리를 사용하는 것이 효율적이기 때문입니다.
  - 시간 복잡도
    - 연결 리스트: O(n)
    - 레드-블랙 트리: O(logn)

## resize 함수

## hash 함수

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

- key의 해시값과 해시값을 오른쪽으로 16번 비트 이동시킨 값을 xor 연산한 값을 새로운 해시값으로 사용합니다.
- 그 이유는 HashMap의 버킷 인덱스를 선택할 때 `putVal` 함수에서 본 것 처럼 `(n - 1)`과 AND 비트 연산을 하게 되는데 이때 이 연산에서는 항상 상위 비트가 버려지게 됩니다.
- 그래서 hash 함수에서는 상위 비트도 인덱스를 선택할 때 영향을 미치게 하기 위하여 기존 해시값의 상위 비트(`h >>> 16`의 결과)를 기존 해시값과 xor 연산하게 된 것입니다.
- xor 연산을 선택한 이유는 비트 연산자 중 가장 고른 비트 분포를 보장하기 때문입니다.
