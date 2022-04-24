# python sort 활용법

기본적인 sort활용법은 다음과 같다.

```python
list.sort()
list = sorted(list)
```

파이썬에서 제공하는 sort는 다양하게 활용가능하다.

1. reverse로 sort하기
```python
list.sort(reverse=True)
```
위와 같이 reverse인자를 true로 주면, 원래의 sort방향(증가하는 방향)의 역순으로 정렬이 된다.  

2. key를 주고 sort하기
만약에 다음과 같은 list가 있다고 가정하자.  
```python
list = [[1,2], [1,9], [8,2], [5,5]]
```
이런 상황에서 list의 원소인 개별 list의 첫번째 원소를 기준으로 정렬할 수 있다.  
```python
list = [[1,2], [1,9], [8,2], [5,5]]
list.sort(key=lamdba x:x[0])
```
3. custom한 기준으로 sort하기
reverse, key를 통해서 정렬하기 보다 더욱 custom한 기준으로 정렬하고 싶다면 어떻게 해야 할까 ?  

예를 들어서 list의 첫번째 원소는 내림차순, 첫번째 원소가 같다면 두번째의 원소를 오름차순으로 정렬하기...  

이런 경우는 python의 cmp변수를 활용하면 된다.  

key로 넘기는 parameter로 functools.cmp_to_key를 넘기면 된다.

예시를 보자.  

```python

def compare(a,b):
    if a[0] > b[0]:
        return 1
    elif a[0] < b[0]:
        return -1
    else:
        if a[1] < b[1]:
            return -1
        elif a[1] > b[1]:
            return 1
        else:
            return 0

list = [[1,2], [1,9], [8,2], [5,5]]
list.sort(key=cmp_to_map(compare))
```

cmp_to_map의 인자로 넘겨주는 함수는 다음과 같이 구현하면 된다.  

1. a,b가 순서대로 있을때, a[0]이 b[0]보다 더 큰 경우는 1을 반환 -> a가 앞으로 간다. 내림차순
2. b[1]가 a[1]보다 큰 경우는 -1을 반환 -> b가 뒤로 간다. 오름차순   
3. 같은 경우에는 0을 반환

즉 cmp_to_map함수의 파라미터로 들어가는 함수는 1,-1,0 셋중 하나를 반환하면 된다.  
그리고 1은 앞으로 가는 경우,  
-1은 뒤로 가는 경우,  
0은 그대로 유지하는 경우  

세가지 경우를 신경써서 구현하면 된다.  

