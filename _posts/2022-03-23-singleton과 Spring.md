---

title: Singleton과 Spring
categories: [Spring]
tags: [Spring, Singleton]     # TAG names should always be lowercase
author:
name: Choi Jin Kyu
link: https://github.com/hmcck27
toc: true
# date: 2022-02-05

img_path: /assets/img/mdImage/

---

# Singleton Pattern

여기서 자바 싱글톤 = spring singleton을 예시로 생각해보자.  
Singleton은 어플리케이션 전역에서 하나만 존재하는 객체이다.  

자바 JVM에서 딱 하나만 존재하는 객체 혹은 이런 디자인 패턴을 singleton이라고 한다.  

---

## Singleton Design Pattern

클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴이다.  
그래서 복수로 인스턴스를 생성하는 것을 막는 로직이 객체 생성에 필요하다.  
다음과 같은 패턴을 말한다.  

```java
public class SingletonService {

    private static final SingletonService instance =new SingletonService();

    public static SingletonService getInstance(){
        return instance;
    }

    private SingletonService() {

    }

    public void logic() {
        System.out.println("call singleton instance");
    }
}
```

이렇게 클래스를 만들면,  
처음 static으로 선언하면서 static 영역에 객체가 생성된다.  
그리고 해당 클래스의 생성자는 private이기 때문에 외부에서 생성자 호출이 불가능하다.  
즉 그냥 생성자 자체를 막아놓은 것이다.  
그리고 해당 객체를 가져오려면 getInstance()를 호출해야만 한다.  
instance는 이미 static 영역에 선언되어 있고 해당 instance를 가져오게 된다.  

즉 외부에서는 늘 static 영역에 미리 선언되었던 instance만을 가져올 수 있게 된다.  

```java
    @Test
    @DisplayName("singleton pattern obejct use")
    void singletonServiceTest() {

        SingletonService instance1 = SingletonService.getInstance();
        SingletonService instance2 = SingletonService.getInstance();

        System.out.println("instance1 = " + instance1);
        System.out.println("instance2 = " + instance2);

        assertThat(instance1).isSameAs(instance2);

    }
```

테스트 통과 !

---

## singleton pattern 구현의 문제점.  

1. 코드를 구현해줘야 한다.  
2. DIP 위반이다. -> 즉 클라이언트 코드가 해당 객체를 부르기 위해서는 구체화된 인스턴스를 가져오기 때문이다.  
3. OCP 위빈이다. -> 구체화된 클래스에 의존하기 때문에 사용할 클래스가 바뀌면 인스턴스도 바꿔줘야 한다.
4. 테스트하기 어렵다.
5. 내부 속성을 변경하거나 초기화하기 어렵다. -> 전역에서 관리되기 때문에 함부로 속성을 변경하면 이걸 사용하는 다른 클라이언트 코드들에 문제가 생길 여지가 많다.  
6. 유연성이 떨어진다.   

---

## web application과 singleton  
현대적인 application의 대부분은 web application이지만,  
사실 application이라 함은 web만을 지칭하는 것은 아니다.  

서버내에서 즉 컴퓨터내에서 특정 작업을 처리해주는 데몬같은 것도 어플리케이션이고  
일정 시간마다 대용량의 작업을 처리하는 배치 프로그램도 어플리케이션이다.  

스프링으로도 이렇게 데몬이나 배치같은 어플리케이션을 만들 수 있다.  
스프링 배치라는 모듈이 있다 ! 물론 내가 개발해보지는 않았다...

대부분은 스프링 어플리케이션은 웹이다.  
웹에서는 여러 고객이 동시에 요청한다.  
그러면 요청마다 새로운 객체를 생성해서 반환하게 된다.  

만약에 스프링의 DI container가 없는 상태에서 요청마다 객체를 생성하게 되면 다음과 같다.  

```java
public class SingletonTest {

    @Test
    @DisplayName("no spring only pure di container")
    void pureContainer() {
        AppConfig appConfig = new AppConfig();

        // 1. 조회
        MemberService memberService1 = appConfig.memberService();
        MemberService memberService2 = appConfig.memberService();

        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);
    }
}
```
테스트 코드를 실행시켜보면,  
다음과 같은 결과가 나온다.  

> memberService1 = hello.core.member.MemberServiceImpl@71623278
memberService2 = hello.core.member.MemberServiceImpl@768b970c

보다 싶이 새롭게 memberService를 할당받은 모습임을 알 수 있다.  
만약 웹 어플리케이션 같은 경우 이런식으로 객체를 늘 생성하면  
당연히 자원의 소모가 커진다.  

따라서 웹에서는 이렇게 singleton패턴이 아닌 경우 메모리라던가 관리에 지장이 생길 수 밖에 없다.  

singleton같은 경우는 어플리케이션 전역에 하나만 존재한다.  
요청이 동시에 복수가 들어와도 하나만 있는 객체를 반환한다면 늘 새로운 객체를 생성하는 자원 소모를 줄일 수 있다.  

여기서 memberService를 스프링 빈으로 생성하고 spring container가 관리하도록 한다면  
우리는 해당 객체를 다시 생성할 필요없이 전역으로 생성된 인스턴스를 갖다 쓰면 된다.  

---

## Spring Container

spring에서는 spring container가 기본적으로 bean으로 생성된 객체는 singleton으로 만든다.  
이게 singleton의 패턴의 문제점을 해결하면서 객체를 singleton으로 관리하게 해준다.  

DIP, OCP를 지키면서 객체의 단일성을 보장해주고 개발자가 추가적으로 singleton 패턴 디자인을 할 필요가 없다.  

```java
public class SingletonTest {

    @Test
    @DisplayName("spring container and singleton")
    void springContainer() {

        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        // 1. 조회
        MemberService memberService1 = ac.getBean(MemberService.class);
        MemberService memberService2 = ac.getBean(MemberService.class);

        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

        assertThat(memberService1).isSameAs(memberService2);

    }
}
```

> memberService1 = hello.core.member.MemberServiceImpl@1349883
memberService2 = hello.core.member.MemberServiceImpl@1349883

다음과 같이 bean 설정을 주입한 ApplicationContext에서 가져온 객체는 동일한 instance이다.  

spring container가 지원하는 bean들은 대부분 singleton이다.  
물론 bean scope에 맞춰서 다른 bean 종류도 있긴하다.  

그건 IoC Container 포스트에서 자세히 다룰 예정이다.  

---

## singleton 방식의 주의점  

위에서 간단히 짚고 넘어갔듯이 singleton 패턴으로 디자인된 객체들은  
절대로 !! 절대로 state가 존재해서는 안된다.  

만약에 이런 singleton 객체가 있다고 가정해보자.

```java
public class SingletonService {

    private static final SingletonService instance =new SingletonService();
    private int price;

    public static SingletonService getInstance(){
        return instance;
    }

    private SingletonService() {

    }

    public void order(int price) {
        this.price = price;
    }


    public int getPrice() {
        return this.price;
    }

    public void logic() {
        System.out.println("call singleton instance");
    }
}
```

SingletonService는 price라는 가변 변수가 있다.  
해당 변수를 바꾸는 함수 order가 있다면 이는 추후에 큰 문제가 될 수 있다.  

```java
@Test
void statefulServiceSingleton() {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
    SingletonService service1 = ac.getBean(SingletonService.class);
    SingletonService service2 = ac.getBean(SingletonService.class);

    // threadA : 10000원 주문
    service1.order(10000);

    // threadB : 10000원 주문
    service2.order(20000);

    int price1 = service1.getPrice();
    int price2 = service2.getPrice();

    System.out.println("price1 = " + price1);
    System.out.println("price2 = " + price2);

    Assertions.assertThat(service1.getPrice()).isEqualTo(service2.getPrice());

}
```

이 코드에서의 결과물은 
>price1 = 10000  
>price2 = 20000  

가 아니구

다음과 같다.

>price1 = 20000  
price2 = 20000

결국 변수를 함부로 바꾸니까 문제가 생긴것이다.  
