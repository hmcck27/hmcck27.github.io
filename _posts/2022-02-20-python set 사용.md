
---

title: Python Set 자료형
categories: [Python]
tags: [Python, Set]     # TAG names should always be lowercase
author:
name: Choi Jin Kyu
link: https://github.com/hmcck27
toc: true
# date: 2022-02-05

img_path: /assets/img/mdImage/

---


# Python Set 다루기

알고리즘을 공부하다 보면 set을 사용할 일이 은근 많은 것 같다.  
그래서 이번 기회에 공부한걸 깔끔하게 정리하고자 한다.  

---

## 집합 자료형이란

set은 순서가 없고, 인덱스가 없는 데이터의 모음이다.  
파이썬 2,3부터 지원되는 자료형이다.  

set의 가장 큰 특징은 다음과 같다.  
1. 중복이 허용되지 않는다.
2. 같은 set 내의 원소들 사이에는 인덱스(순서)가 존재하지 않는다.

---

## 집합 자료형 선언하기

1. 빈 집합 선언하기
    ```python
    setA = set()
    ```
2. 집합을 초기화하면서 선언하기  
    집합을 초기화하면서 선언하는 방법은 두개가 있다.  
    집합 생성자 set() 함수의 파라미터를 봐보자.
    ```python
    def __init__(self, seq=()): # known special case of set.__init__
    """
    set() -> new empty set object
    set(iterable) -> new set object
    
    Build an unordered collection of unique elements.
    # (copied from class doc)
    """
    pass
    ```
    set(iterable)에서 iterable 한 객체는 뭐든 집합의 생성자 파라미터가 될 수 있다.  
    iterable한 객체의 종류에는 뭐가 있을까 ?  
    list, dict, set, str, bytes, tuple, range 이 대표적인 iterable 객체이다.  
    즉 위의 객체들이 iterable하며 이 객체들을 통해서 set을 만들 수 있다.  
    - list 를 통한 선언
        ```python
        setA = set([1,2,3])
        print(setA) -> {1,2,3}
        ```
    - dict 를 통한 선언
        ```python
        setA = set({"a" : 1, "b" : 2})
        print(setA) -> {"a", "b"}
        ```
        dict 를 통한 선언은 다음과 같이 key 값들이 set의 원소가 됨을 알 수 있다.
    - set 을 통한 선언
        ```python
        setA = set({1,2})
        print(setA) -> {1,2}
        ```
    - str을 통한 선언
        ```python
        setA = set("aabcd")
        print(setA)  -> {'c', 'b', 'd', 'a'}
        ```
    유용하게 사용할 만한 친구들만 정리해봤다.  

---

## 집합 연산

집합은 여러 연산들이 있다.  
이건 딱히 프로그래밍한에서의 얘기가 아니라,  
수학에서 많이 들어본 연산이다.  
교집합, 차집합, 합집합들이 그 연산들이다.  

이런 연산들 같은 경우에는 중복이 되지 않는다는 집합의 성질때문에 굉장히 유용하게 쓰일 일이 많다.  
일단 샘플 집합 두개를 만들어서 하나씩 알아보자.  

```python
set1 = set(["a", "b" , 1, "2", "c", "a"]) -> {"a","b",1,"2","c"}
set2 = set("ab221") -> {"a","b","2","1"}
```

1. 교집합
    ```python
    set1.intersection(set2)
    set1 & set2
    ```
    결과는 다음과 같다.
    > {'2', 'a', 'b'}  
    > {'2', 'a', 'b'}

2. 차집합
    ```python
    set1.difference(set2)
    set2.difference(set1)
    set1 - set2
    set2 - set1
    ```
    결과는 다음과 같다.
    > {1, 'c'}  
    > {'1'}  
    > {1, 'c'}  
    > {'1'}  

3. 합집합
    ```python
    set1.union(set2)
    set1 | set2
    ```
    > {1, 'c', '1', '2', 'b', 'a'}  
    > {1, 'c', '1', '2', 'b', 'a'}

---

## set 내장 메소드

set 내장 메소드에는 굉장히 유용한게 많다 !  
나도 기억해뒀다가 나중에 기회가 되면 유용하게 쓰려고 한다.  

1. add()  
기존 set에 새로운 원소를 추가하고 싶을때 사용한다.
    ```python
    set1 = set(["a", "b" , 1, "2", "c", "a"])
    set1.add(3)
    ```
    > {1, 3, '2', 'b', 'a', 'c'}


2. update()  
여러개의 값을 한꺼번에 추가하고 싶을때 사용한다.
    ```python
    set1 = set(["a", "b" , 1, "2", "c", "a"])
    set1.update("sav")
    print(set1)
    ```
    > {'a', 1, 'b', 'v', '2', 'c', 's'}

3. remove(), discard()  
특정 값을 제거할때 사용한다.  
둘은 조금의 차이가 있다.  

    - remove 에서 없는 원소 삭제할 경우
        ```python
        set1 = set(["a", "b" , 1, "2", "c", "a"])
        set1.remove(3)
        print(set1)
        ```
        > key error 발생   

        없는 원소를 삭제하려고 하면 key error가 발생한다.

    - remove에서 있는 원소 삭제
        ```python
        set1 = set(["a", "b" , 1, "2", "c", "a"])
        set1.remove("a")
        print(set1)
        ```
        > {1, 'b', '2', 'c'}

    - discard에서 없는 원소 삭제할 경우
        ```python
        set1 = set(["a", "b" , 1, "2", "c", "a"])
        set1.discard(3)
        print(set1)
        ```
        > {'a', 1, 'b', '2', 'c'}
        
        없는 원소를 삭제해도 key error가 발생하지 않는다.  
        없으면 아무 행동도 하지 않는다.  
        discard를 사용하면 이런 차이가 있다.  

    - discard에서 있는 원소 삭제할 경우
        ```python
        set1 = set(["a", "b" , 1, "2", "c", "a"])
        set1.discard("a")
        print(set1)
        ```
        > {1, 'b', '2', 'c'}


4. isdisjoint()   
두 집합이 disjoint한지 = 겹치는 원소가 없는지

    ```python
    set1 = set(["a", "b" , 1, "2", "c", "a"])
    print(set1.isdisjoint("df"))
    ```
    > True

    ```python
    set1 = set(["a", "b" , 1, "2", "c", "a"])
    print(set1.isdisjoint("ab"))
    ```
    > False

5. a.issubset(b)  

    a가 b의 subset인지 확인한다.
    
    ```python
    set1 = set(["a", "b" , 1, "2", "c", "a"])
    print(set1.issubset(set(["a", "b" , 1, "2", "c", "a", 3])))
    ```
    > True

    ```python
    set1 = set(["a", "b" , 1, "2", "c", "a"])
    print(set1.issubset(set(["a", "b" , 1, "2"])))
    ```
    > False

6. a.issuperset(b)  

    a가 b의 super set인지 확인한다.

    ```python
    set1 = set(["a", "b" , 1, "2", "c", "a"])
    print(set1.issuperset(set(["a","b"])))
    ```
    > True

    ```python
    set1 = set(["a", "b" , 1, "2", "c", "a"])
    print(set1.issuperset(set(["b", "f"])))
    ```
    > False

---

## 정리

set은 연산 속도도 빠르고, 중복이 없다는 큰 장점이 있다.  
하지만 인덱싱은 불가능하기 때문에 그걸 고려해서 사용해야 한다.  
