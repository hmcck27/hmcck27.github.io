---

title: JAVA enum valueof overriding
categories: [Java]
tags: [Java, enumertaion]     # TAG names should always be lowercase
author:
name: Choi Jin Kyu
link: https://github.com/hmcck27
toc: true
# date: 2022-02-05

img_path: /assets/img/mdImage/

---

## Java의 Enum

java은 의외로 많이 사용한다.  
나같은 경우에는 주로 status를 관리하기 위해서 사용하는데 그 이유는 다음과 같다.

1. 대부분의 status를 갖는 객체는 특정 상태에 대한 status 값은 하나를 가진다.
   예를 들면 결제의 status는 "결제시작", "결제완료"와 같이 항상 하나의 값을 갖는다.   
   만약에 status가 여러개를 동시에 갖는다면 enum이 아닌 테이블 다대다 매칭을 통해서 풀어야 한다.  
2. status는 개발 초기에 결정되서 수정할 일이 많이 없다.  
   내 주관이지만 status라 함은 초기 기획에서 결정되고 이후에는 수정되지 않을 확률이 높다.  
   생산성을 위해서는 굳이 데이터베이스까지 끌고 가지 말고 application 레벨에서 enum으로 관리하는 것이 좋다.  
   물론 나중에 수정될 일이 잦은 status가 있다면 데이터베이스에서의 매핑으로 해결해야 할것 이다.  

따라서 status값을 enum으로 해결하는 것이 효율적인 경우가 많은 것 같다. (물론 아닐수도 있다. 상황에 적절하게 사용하면 된다.)  

기본적인 java의 enum은 다음과 같이 사용한다.  

```java
public enum OrderStatus {

    결제완료, 결제취소, 결제실패, 미확인결제

}
```

주문의 status를 다음과 같이 4종류로 나누어보았다.  
결제기 시작되고 나서는 미확인결제, 결제가 성공하고 validate에 성공하면 결제성공, pg사에서 callback 혹은 webhook으로 결제가 취소되거나 실패되면 결제취소, 결제 실패가 된다.  

## enum valueof  

java에서는 기본적으로 valueof라는 함수를 제공한다.  
다음과 같은 경우에 사용한다.  
1. String 인 "결제완료"를 OrderStatus 타입으로 형변환하고 싶은 경우

예를 들면 다음과 같은 쓰임이다.

```java
    public OrderStatus toEnum() {
        
        orderStatus = new String("결제완료");
        OrderStatus enumOrderStatus = OrderStatus.valueof(orderStatus);
        return enumOrderStatus;

    }
```

문자열이 있을때 해당 문자열과 일치하는 enum을 반환한다.  

## enum valueof의 override

사실 가장 다루고 싶은 부분은 이 부분이다.  
기본적으로 enum에서 string -> enum class 를 찾는 과정에서 사용하는 valueof를 override할 일이 있었다.  

예를 들어보자.  
우리가 결제를 구현할때는 pg사를 이용하게 된다.  

즉 pg사에서 다루는 결제 status와 가맹점 서버의 결제 status가 다른 경우가 굉장히 높다.  
우리가 실제로 결제가 잘 이루어 졌는지 검증하는 api가 있다고 가정해보자.  

이때 pg사에서 webhook을 이용해서 해당 결제가 완료되고 나서 가맹점 서버의 결제 완료 api를 호출한다고 가정해보자.  
이런 경우 pg사에서 보내는 request의 http body에 담긴 json의 형태가 다음과 같다고 해보자.

```json
{
    "orderId" : 1234,
    "status" : "paid" or "cancelled" or "failed"
}
```

가맹점 서버에서는 다음과 같이 결제 status를 관리한다고 해보자.    
```java

public enum OrderStatus {

    결제완료,
    결제취소,
    결제실패,
    미확인결제;
}

```  

이런 경우에 가맹점 서버는 해당 json을 받아서 가맹점 서버에 저장된 결제의 status를 검증하고 update하는 과정이 필요하고,  
이 과정에서 "paid"와 같은 pg사의 status를 가맹점 서버의 **결제완료** 와 같은 enum status로 변환해야 한다.  

물론 enum class에서 일일히 매핑하는 메소드를 만들어 줄 수 있겠지만, enum이 어떻게 변화, 확장될지는 아무도 모르니  
결국에는 valueof()와 같은 동작이 새로 필요하다.  

예를 들어서 다음과 같이 사용하는 것이다.  

```java


// orderCompleteReqeustDto.getStatus() => String "paid"
OrderStatus newStatus = OrderStatus.valueOf(orderCompleteRequestDto.getStatus());

// 예상되는 newStatus => 결제완료
```

따라서 우리는 enum class에서 기본 제공하는 valueof()를 overriding할 필요성이 있다.  

하지만 enum class에서 @Override valueof()를 실제로 작성해보면 overriding이 제대로 작동하지 않는다.  

찾아보니, enum class는 기본적으로 final, static하다고 한다.  

즉 우리는 valudof()를 overriding하지 못하니까 이런 경우에는 custom한 method를 작성해야 한다.  

다음의 코드를 통해서 custom하게 valueof()를 대체하는 method를 정의할 수 있다.  

```java
public enum OrderStatus {

    결제완료("paid"),
    결제취소("cancelled"),
    결제실패("failed"),
    미확인결제("unpaid");

    private final String value;

    OrderStatus(String impStatus) {
        this.value = impStatus;
    }

    public String getValue() {
        return this.value;
    }

    public static OrderStatus getOrderStatusFromPgStatus(String pgStatus) {
        for (OrderStatus os : values()) {
            if (os.getValue().equals(pgStatus)) {
                return os;
            }
        }
        throw new WrongOrderStatusException();
    }
}
```  

코드를 보면, 생성자를 통해서 enum은 value라는 final String을 갖는다.  

그리고 getValue라는 method는 해당 enum의 value를 반환한다.  
getOrderStatusFromPgStatus()는 일차하는 enum value를 찾고, 해당 enum을 반환한다.  

다음의 코드를 정의하면, 우리가 처음 원했던  
```java
// orderCompleteReqeustDto.getStatus() => String "paid"
this.orderStatus = OrderStatus.getOrderStatusFromImpStatus(orderCompleteRequestDto.getStatus());
// 예상되는 newStatus => 결제완료
```

이런 방식의 동작이 가능하다 !!
