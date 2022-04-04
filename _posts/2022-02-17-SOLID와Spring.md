---

title: SOLID와 Spring
categories: [Spring]
tags: [Spring, OOP]     # TAG names should always be lowercase
author:
name: Choi Jin Kyu
link: https://github.com/hmcck27
toc: true
# date: 2022-02-05

img_path: /assets/img/mdImage/

---

# 스프링 기초

## 스프링의 기술들

스프링의 기술을 다양하다.  
가장 쉽게 접할 수 있는 Spring Framework, Spring Boot 부터  
각기 필요한 기능들을 구현하고 싶을때 사용하는 Spring Security, Spring Data, Spring session, Spring Rest Docs, Spring Batch, Spring Cloud,등이 있다.  

특히 스프링 부트 같은 경우에는 이 모든 기술들을 편리하게 사용할 수 있도록 도와준다.  

스프링 부트 같은 경우는 모든 실무에서 기본적으로 깔고 간다고 한다 - 김영한선생님.  

스프링 부트의 장점은 다음과 같다.  
1. 스프링을 편리하게 사용할 수 있도록 지원한다. 최근에는 모든 실무에서 기본으로 사용한다.
2. 단독으로 실행 할 수 있는 스프링 어플리케이션을 쉽게 생성한다.
3. Tomcat 같은 웹 서버를 내장해서 별도의 웹 서버를 설치하지 않아도 된다. 
4. 손쉬운 빌드 구성을 위한 starter 종속성을 제공한다.
5. 스프링과 3rd party 라이브러리를 자동 구성한다.
6. 메트릭, 상태 확인, 외부 구성같은 프로덕션 준비 기능을 제공한다. 
7. 관례에 의한 간결한 설정.

---

## 왜 스프링이 필요할까 ?

스프링을 사용하는 이유는 큰게 아니다.  
물론 웹 서버를 띄우고 데이터 베이스 커넥션을 잡아서 CRUD를 하고,  
다양한 기능을 제공하지만,  
스프링의 가장 큰 핵심은 객체 지향이다.  

결국 대규모의 자바 프로젝트에서 객체 지향을 지키면서 개발할 수 있게 해준다.  

스프링은 객체 지향의 특징중에서 다형성을 극대화해서 지원한다.  
의존관계 주입, IoC 컨테이너가 이런 역할을 해준다.  

---

## SOLID와 스프링

SOLID에 대해서 먼저 알아보자.  

### SRP

Single Responsibility Principle의 약자이다.  
즉 하나의 클래스는 하나의 책임만 져야 한다.  

클래스가 여러 책임을 갖게 되면, 그 클래스는 각 책임에 따른 변경 이유가 발생하기 때문이다.  

즉 하나의 책임만 지면 하나의 변경 이유만 갖게 되는데, 책임이 여러개면 그만큼 변경될 일이 여러가지가 된다는 뜻이다.  

사실 *책임*이라는 말은 참 모호한 말이다.  

그래서 우리는 변경을 기준으로 잡으면 된다.  

예를 들어서 다음과 같은 클래스가 있다고 가정하자.  

```java
    public class service{

        public service makeProduct() {
            List<Produce> produces = makeProduce();
            List<Design> designs = design(produces);
            return develop(produces, designs);
        }
    }
```

다음과 같은 클래스는 service하나를 만드는 과정이다.  
이 클래스에서는 produce를 만들고, produces를 받아서 design도 하고,  
그 결과로 develop을 해서 service를 반환한다.  

해당 클래스의 경우 1개 이상의 책임을 지고 있다.  
기획, 디자인, 개발 3개 이상의 책임을 지고 있는 상황에서,  
만약에 design에 해당하는 코드가 수정된다면,  
design만 수정하는것이 아닌 더 큰 클래스인 service 전체를 수정해야 한다.  

즉 service의 경우 변경이 발생할 수 있는 지점은 = 책임을 지고 있는 부분은  

produce, design, develop 세군데인 것이다.  

이런 클래스는 SRP를 잘 지켜서 만든 클래스가 아니다.  

그러면 이런 경우에 우리는 어떻게 클래스를 다시 설계해야 할까..?  

```java

    public class service {

        private Producer producer;
        private Designer designer;
        private Developer developer;

        public service makeProduct() {

            List<Produce> produces = producer.makeProduce();
            List<Design> designs = designer.design(produces);
            return developer.develop(produces, designs);
        }
    }
```

다음과 같이 각 책임을 따로 지고 있는 클래스를 만들어야 한다.  
이렇게 책임을 분산시키면 만약에 design의 코드가 수정된다고 하더라도,  
큰 클래스인 service가 수정될 일은 없다.  

즉 service의 책임 3개를 클래스 producer, designer, developer가 나눠갖는 것이다.  

변경이 일어나도 하나의 클래스만 수정하면 된다. !!

### OCP

Open/closed Principle  

1. 소프트웨어 요소는 확장에는 열려있으나, 변경에는 닫혀있어야 한다.  
2. 다형성을 활용하면 가능하다.  
3. 인터페이스를 만들고 해당 인터페이스를 재사용한다.  

실제로 개발을 하면서 가장 중요하게 느껴지는 원칙이다.  
이게 지켜지지 않으면 유지보수가 힘들어진다.  

인터페이스를 구현한 구현체를 새로 만들고 이를 적용하면 된다.  
그러면 기존 코드를 변경하지 않고도 확장이 가능하다.  

```java
public class Service {
    
    private Producer producer;
    private Designer designer;
    private Developer developer;

    public service makeProduct() {

        List<Produce> produces = producer.makeProduce();
        List<Design> designs = designer.design(produces);
        return developer.develop(produces, designs);
    }
}
```

해당 클래스에서 makeProduct가 바뀐다고 가정해보자.  
이제 단순히 produce -> desing -> develop의 절차에서  
조금 더 확장되서 우리가 중간에 새로운... 예를 들어서 develop의 절차를 조금 수정한다고 해보자.  

```java
public class Service {
    
    private Producer producer;
    private Designer designer;
    private Developer developer;

    public service makeProduct() {

        List<Produce> produces = producer.makeProduce();
        List<Design> designs = designer.design(produces);
        Develop = developer.develop(produces, designs)
        return Develop
    }
}
```

그러면 여기서 develop함수를 수정해야 한다.

즉 변경은 불가피한것이다.  

그러면 OCP를 지키는 방향을 무엇일까..?  
일단은 makeProduct()를 인터페이스로 추상화하자.  

```java
public interface MakeProductInterface {
    public service makeProduct();
}
```

그리고 이 인터페이스를 구현한 구현체를 만들자.  
아마 기존의 코드가 구현체로 구성되있다면 다음과 같을 것이다.  

```java
public class Service {
    
    private Producer producer;
    private Designer designer;
    private Developer developer;
    private MakeProductInterface makeProduct = new MakeProductBefore();

    public Service(Producer producer, Designer designer, Developer developer){
        this.makeProduct = makeProduct(producer, designer, developer);
    }

}

public class MakeProductBefore implements MakeProductInterface{
    @Override
    public service makeProduct(Producer producer, Designer designer, Developer developer){
        List<Produce> produces = producer.makeProduce();
        List<Design> designs = designer.design(produces);
        return developer.develop(produces, designs);
    }
}
```

여기서 새로운 구현체인 MakeProductAfter를 만들고 **갈아끼우면** 된다.
```java
public class Service {
    
    private Producer producer;
    private Designer designer;
    private Developer developer;
    private MakeProductInterface makeProduct = new MakeProductAfter();

    public Service(Producer producer, Designer designer, Developer developer){
        this.makeProduct = makeProduct(producer, designer, developer);
    }

}

public class MakeProductAfter implements MakeProductInterface{
    @Override
    public service makeProduct(Producer producer, Designer designer, Developer developer){
        List<Produce> produces = producer.makeProduce();
        List<Design> designs = designer.design(produces);
        // 새로운 절차 뭔가 추가했음
        return developer.develop(produces, designs);
    }
}
```

즉 우리는 기존 코드를 수정하지 않고 (물론 after로 수정하는 부분은 어쩔 수 없다. -> 스프링에서는 이걸 IoC로 더 편하게 해결한다.)
새로운 구현체인 after를 만들어서 갈아끼우는 방식으로 확장에 성공했다.  

다형성을 사용한다고 해도 OCP를 완전하게 지키는건 어려웠다.  
다음에 스프링 IoC를 설명하면 다형성을 사용해서 OCP를 완전하게 지키는 방법에 대해서 알아보자.  

### LSP

Liskov Substitution Principle  

프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야 한다.  

다형성을 사용하려면 상속, 구현하는 클래스, 인터페이스 내부 규약을 잘 지켜야 한다는 의미이다.  

예를 들어서

```java

public interface MakeProductInterface {
    public service makeProduct();
}

public class MakeProductBefore implements MakeProductInterface{
    @Override
    public service makeProduct(Producer producer, Designer designer, Developer developer){
        List<Produce> produces = producer.makeProduce();
        List<Design> designs = designer.design(produces);
        return developer.develop(produces, designs);
    }
}

public class MakeProductAfter implements MakeProductInterface{
    @Override
    public service makeProduct(Producer producer, Designer designer, Developer developer){
        List<Produce> produces = producer.makeProduce();
        List<Design> designs = designer.design(produces);
        // 새로운 절차 뭔가 추가했음
        return developer.develop(produces, designs);
    }
}
```

MakeProductInterface를 구현한 두개의 구현체는 무조건적으로 MakeProductInterface가 의도한 결과물을 내야 한다.  
before의 결과물인 Develop과 after의 결과물인 Develop의 내부 규약이 다르거나 기능이 의도한 방향으로 작동하지 않으면 안된다.  

조금 더 쉬운 예시로는 우리가 스마트폰 인터페이스를 구현한 두개의 구현체 아이폰과 갤럭시가 있다고 가정했을때,  

새로운 구현체 jk폰을 낸다고 해보자.  

그런데 jk폰에서는 전화버튼을 눌렀을때 카카오톡을 실행한다.  
이건 LSP를 어기는 케이스이다.  

스마트폰 인터페이스의 내부 규약은 전화 버튼을 눌렀을때 전화가 가능해야 한다.  

이건 단순히 코딩의 단계에 문제가 아니라 객체를 설계하고 개발하는 단계의 이야기이다.  

### ISP

interface segragation principle  

특정 클라이언트를 위한 인터페이스 여러개가 범용 인터페이스 하나보다 낫다.  
즉 하나의 거대한 인터페이스보다는 여러개의 인터페이스로 나누는것이 유지보수에 유리하다.  

예시를 들어보자.  
이전의 예시에서 우리는 Develop이라는 클래스가 있었다.  

```java
public class Service {
    
    private Producer producer;
    private Designer designer;
    private Developer developer;
    private MakeProductInterface makeProduct = new MakeProduct();

    public Service(Producer producer, Designer designer, Developer developer){
        this.makeProduct = makeProduct(producer, designer, developer);
    }
}

public class MakeProduct implements MakeProductInterface{
    @Override
    public service makeProduct(Producer producer, Designer designer, Developer developer){
        List<Produce> produces = producer.makeProduce();
        List<Design> designs = designer.design(produces);
        // 새로운 절차 뭔가 추가했음
        return developer.develop(produces, designs);
    }
}
```

해당 클래스의 develop을 하나의 함수로 만들었다.  
음 develop 예시를 들어보면 다음과 같다.  

```java
public class Developer implements DevelopInterface{
    @Override
    public Develop develop() {
        return new Develop();
    }    
}

public interface DevelopInterface {

    public Develop develop();

}
```

develop은 사실 front develop, backend develop으로 나눠질 수 있다.  
그니까 기존의 develop interface를 두개의 interface로 나눈다.

```java
public interface FrontDevelopInterface {
    public FrontDevelop frontDevelop();
}

public interface BackeDevelopInterface {

    public BackDevelop backdevelop();

}
```

이렇게 큰 인터페이스를 작은 인터페이스로 나누면 작은 부분의 수정은 작은 부분에서 머무르게 된다.  
더 큰 develop 인터페이스를 수정하는 것보다 작은 interface를 수정하는것이 유지보수에 유리하다.  

실제로 스프링 코드를 까보면 철저하게 interface가 분리된것을 알 수 있다.  

### DIP

Dependency Inversion Principle

프로그래머는 추상화에 의존해야지 구체화에 의존하면 안된다.  
의존성 주입은 이 원칙에 따르는 방법이다.   

쉽게 설명하면 구현 클래스에 의존하지 말고 추상화된 인터페이스에 의존하라는 의미이다.  

예를 들어보자.  

```java
public class Service {
    
    private Producer producer;
    private Designer designer;
    private Developer developer;
    private MakeProductInterface makeProduct = new MakeProduct();

    public Service(Producer producer, Designer designer, Developer developer){
        this.makeProduct = makeProduct(producer, designer, developer);
    }
}

public class MakeProduct implements MakeProductInterface{
    @Override
    public service makeProduct(Producer producer, Designer designer, Developer developer){
        List<Produce> produces = producer.makeProduce();
        List<Design> designs = designer.design(produces);
        // 새로운 절차 뭔가 추가했음
        return developer.develop(produces, designs);
    }
}
```

의 코드에서 클라이언트 코드는 service class이다.  
그리고 service 코드는 interface에도, 구현체인 MakeProduct에도 동시에 의존하고 있다.  

이건 DIP를 위반한 케이스이다.  

즉 우리는 다형성 하나 갖고는 DIP를 지키기는 어렵다.  
아무리 interface를 잘 만든다고 해도 결국에는 기존 코드를 수정해야 한다.  

그래서 우리가 스프링을 사용하게 된다.  

---

## 정리
우리는 객체 지향적으로 설계하고 싶지만 다형성 하나로는 부족했다.  
아무리 다형성을 잘 활용한다고 해도 SOLID전부를 지킬 수는 없다....

따라서 스프링이라는 프레임워크를 사용하게 된것이다. 