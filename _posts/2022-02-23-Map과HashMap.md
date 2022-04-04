---

title: Map과 HashMap
categories: [Java]
tags: [Java, Map]     # TAG names should always be lowercase
author:
name: Choi Jin Kyu
link: https://github.com/hmcck27
toc: true
# date: 2022-02-05

img_path: /assets/img/mdImage/

---

# Map과 HashMap의 차이

Map이던 HashMap이던 사용할 일이 굉장히 많다.  
실제로 실무에서 둘다 사용하는데 둘의 차이를 잘 모르고 사용하는 경우가 많다.  

하지만 둘은 명백한 차이점이 있다.  

이번 기회에 한번 그 차이점들을 정리해보자 !  

Map과 HashMap의 가장 큰 차이는 키에 대한 값, 즉 key-value를 찾는 과정이 다르다.  
HashMap은 HashTable을 통해서 key-value를 유지하고, Map은 Red-Black tree 를 사용한다.  

둘의 코드를 까보면, HashMap은 Map을 구현하는 구현체이다.  
물론 구현체는 더 다양하다.  
실무에서 static으로 사용할거면 concurrent hashmap을 사용해야 한다. 동시성 이슈가 있을 수 있으니까...

## Map
Map은 key-value를 가진 집합이며, 중복을 허용하지 않는다.  
즉 한개의 key에 한개의 value가 매핑된다.  

1. key-value로 이뤄진 데이터의 집합이다.
2. 순서가 존재하지 않는다.
3. 키는 중복이 허용되지 않는다. 
4. 구현체로 TreeMap, HashTable, HashMap이 있다.

## HashMap

1. HashMap은 Map interface 구현체이다.  
2. key또는 value에 null을 허용한다.
3. 중복을 허용하지 않는다.
4. hash 알고리즘을 통해서 구현되어 있다. -> 각 키값의 해시값을 버킷에 저장하고 그 값을 통해서 객체를 찾는다. O(1)의 속도이다.  

## TreeMap

1. 중복을 허용하지 않는다.
2. key-value로 구성된다.
3. SortedMap을 상속하며 key에 대한 정렬이 이루어진다.

## HashTableMap

1. key-value로 구성된다.
2. null을 허용하지 않는다.

구글링해서 코드를 보다보면 
```java
    Map<String, Object> map = new HashMap<String, Object>();
```

다음과 같이 사용하는 경우가 왕왕 있는데, 다형성 때문에 이렇게 사용한다 !  
즉 어떤 map 구현체를 사용할지 모르니까, -> 변경의 여지가 있기 때문에...
