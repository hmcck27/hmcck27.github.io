# Spring bean Scope

이전 ioc에 대해 알아보면서 간단하게 scope가 뭔지, 어떤 종류가 있는지 알아보았다.  
좀 더 자세하게 정리해보자 !!

가장 먼저 bean의 scope는 다음과 같다.
1. Singleton
2. prototype
3. request
4. session

우리가 가장 흔하게 사용하는 scope는 당연히 singleton이다.  
아무런 설정을 하지 않으면 spring은 자동으로 bean을 singleton으로 생성, 관리한다.  

하지만 우리가 명시적으로 scope를 지정할 수도 있다.

다음과 같은 방식이다.  

```java
@Scope("prototype")
@Component
public class HelloBean {
}
```

만약 component scan을 사용한다면,  
bean으로 선언하고 싶은 클래스위에   
@Scope annotation과 원하는 scope를 설정해서 붙여주면 된다.  

이제 각 scope의 특징을 한번 정리해보자.  

## singleton
spring container는 아무런 설정을 하지 않은 bean은 자동으로 singleton으로 설정한다.  
이 singleton bean은 말그대로 singleton이다.  

application context가 load되면서 container가 한번 생성을 하게 되고,  
이렇게 생성된 instance는 singleton으로 관리되기 때문에,  
client가 해당 instance를 호출하면 새로 생성하는게 아니라  
기존에 생성해두었던 bean을 application context에서 가져오게 된다.  

이렇게 만든 singleton은 내부의 변수를 수정하는 메소드를 만들게 되면  
stateless가 깨지게 되니까 내부 변수를 만든다해도 해당 변수를 수정하는 일을 최대한 지양해야 한다.  
동시성 문제가 발생하게 되기 때문이다.  

그리고 spring container의 가장 큰 핵심 역할이 바로 개발자가 직접 해당 bean을 singleton으로 만들지 않아도,  
자동으로 singleton으로 관리해준다는 것이다.  

이전에도 알아봤듯이 singleton에는 여러 문제가 있었다.  

1. 코드를 구현해줘야 한다.  
2. DIP 위반이다. -> 즉 클라이언트 코드가 해당 객체를 부르기 위해서는 구체화된 인스턴스를 가져오기 때문이다.  
3. OCP 위빈이다. -> 구체화된 클래스에 의존하기 때문에 사용할 클래스가 바뀌면 인스턴스도 바꿔줘야 한다.
4. 테스트하기 어렵다.
5. 내부 속성을 변경하거나 초기화하기 어렵다. -> 전역에서 관리되기 때문에 함부로 속성을 변경하면 이걸 사용하는 다른 클라이언트 코드들에 문제가 생길 여지가 많다.  
6. 유연성이 떨어진다.   

다음과 같으 문제들이 있었다.  
하지만 이런 문제들을 해결하는 것이 spring의 가장 큰 장점이다.  

또 이러한 singleton bean은 
1. 생성
2. 의존관계 주입
3. 초기화 callback
4. 사용
5. 소멸 callback
6. 소멸
까지의 모든 step을 spring contatiner가 관리해준다.  

## prototype
singleton이 항상 같은 instance를 반환하는 것에 비해서,  
prototype은 조금 다르다.  
prototype은 다음과 같은 특징이 있다.  

client가 prototype bean을 요청하는 순간에 객체를 생성하며,  
언제나 새로운 객체를 반환해준다.  

즉 singleton이 늘 하나의 instance로 관리되었지만,  
prototype은 늘 새로운 객체를 생성하게 된다.  

prototype bean은 spring container가 모든 bean lifecycle을 책임지지 않는다.  
생성 - 의존관계 주입 - 초기화 callback까지는 해주지만 그 이후에는 container의 손을 떠나기 때문에,  
만약에 소멸 callback을 꼭 해줘야 한다면 그 부분은 객체를 호출한 client의 역할이 된다.  

따라서 뭐 @PreDestroy를 명시해준다고 해도 소멸 callback은 client가 알아서 호출해야 한다.  

이런 prototype bean은 그냥 알아서 잘 사용하면 큰 문제가 없지만,  
만약에 singleton bean과 같이 사용하면 문제가 생길 수 있다.  

예를 들어보자.  
만약에 어떤 singleton bean의 의존관계 주입에서 prototype bean을 주입한다고 가정해보자.  

그러면 우리는 당연히 이 singleton의 prototype bean을 호출할때 늘 새로운 bean일거라고 착각하기 쉽다.  

하지만 singleton bean의 의존관계 주입에서 주입된 prototype bean은 하나이고  
여러 client가 singleton bean을 통해서 prototype bean을 부른다고 해도  
이미 singleton이 생성될때 새로 받은 prototype bean이기 때문에  
늘 생성되는것이 아니다.  

따라서 이런 문제를 해결하기 위한 방법이 provider이다.  

## provider
만약에 singleton bean이 prototype bean을 사용할때마다 새로운 bean을 받고 싶다면  
우리는 다음과 같은 방법을 사용할 수 있다.  
```java
static class ClientBean {
    @Autowired
    private ApplicationContext ac;
        public int logic() {
        PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
        prototypeBean.addCount();
        int count = prototypeBean.getCount();
        return count;
    }
 }


```

코드를 보면 logic()이라는 함수를 실행할때마다 우리는 application context에서 새롭게 bean을 조회한다.  
우리가 prototype bean은 조회될때마다 새로 생성된다고 했으니  
이렇게 코드를 작성하면 singleton bean에서 logic()을 호출하면 늘 새로운 bean을 container에게서 받아오게 된다.  

하지만 이렇게 코드를 작성하는것은 조금 이상하다.  
이런 방식을 dependency lookup이라고 한다.  

하지만 이제는 DL은 많이 사용하지 않는다.  

따라서 provider를 사용하면 이를 해결할 수 있다.  

일단 문제가 있는 상황부터 먼저 확인해보자.  

```java
public class PrototypeTest {

    @Test
    void singletonClientUsePrototype() {
        AnnotationConfigApplicationContext ac =
                new AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);

        ClientBean clientBean1 = ac.getBean(ClientBean.class);
        int count1 = clientBean1.logic();
        assertThat(count1).isEqualTo(1);

        ClientBean clientBean2 = ac.getBean(ClientBean.class);
        int count2 = clientBean2.logic();
        assertThat(count2).isEqualTo(2);

    }

    @Scope("singleton")
    static class ClientBean {
        private final PrototypeBean prototypeBean;       

        @Autowired
        public ClientBean(PrototypeBean prototypeBean) {
            this.prototypeBean = prototypeBean;
        }


        public int logic() {
            prototypeBean.addCount();
            int count = prototypeBean.getCount();
            return count;
        }

    }

    @Scope("prototype")
    static class PrototypeBean {
        private int count = 0;

        public void addCount() {
            count++;
        }

        public int getCount() {
            return this.count;
        }

        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init " + this);
        }
    }
}

```

코드를 보면,  
singleton bean이 prototype bean을 사용하고 있다. 
테스트 코드에서는 client bean을 두개 받은 다음에, 둘은 singleton이기 때문에  
같은 인스턴스이다.  

각 client에서 logic()를 호출을 하게된다.  
이때 이미 singleton이 갖고 있는 prototype은 새로 다시 생성되는게 아니기 때문에,  
singleton의 생성 시점에 한번 생성되고 쭉 유지되는 마치 그냥 변수와 같다.  

따라서 우리가 원하는 작동 방식인 1, 1로 값이 나오지 않는다.  

만약에 우리가 provider를 사용해서 늘 호출할때마다 새로운 prototype bean을 생성하게 하고 싶다면  
다음과 같이 코드를 작성하면 된다.  

```java
public class PrototypeTest {

    @Test
    void singletonClientUsePrototype() {
        AnnotationConfigApplicationContext ac =
                new AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);

        ClientBean clientBean1 = ac.getBean(ClientBean.class);
        int count1 = clientBean1.logic();
        assertThat(count1).isEqualTo(1);

        ClientBean clientBean2 = ac.getBean(ClientBean.class);
        int count2 = clientBean2.logic();
        assertThat(count2).isEqualTo(1);

    }

    @Scope("singleton")
    static class ClientBean {
        
        @Autowired
        private ObjectProvider<PrototypeBean> prototypeBeanProvider;
        
        public int logic() {
            PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
            prototypeBean.addCount();
            int count = prototypeBean.getCount();
            return count;
        }
    }

    @Scope("prototype")
    static class PrototypeBean {
        private int count = 0;

        public void addCount() {
            count++;
        }

        public int getCount() {
            return this.count;
        }

        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init " + this);
        }
    }
}
```

코드를 보면, singleton 내부에서는 ObjectProvider를 사용한다.  
그리고 logic()을 수행할때마다 늘 이 provider에서 새로운 prototype bean을 가져오게 된다.  
이때 getObject()를 수행하면 그때 spring container에게서 새로운 prototype instance를 받아서 반환해주게 되기 때문에  
우리가 의도한 대로 1,1의 결과값을 받을 수 있다.  

여기서 provider대신에 factory를 사용해도 된다.  
provider는 factory를 상속받고 좀 더 다양한 stream 기능이라던지를 제공해준다.  

하지만 두가지 방법다 spring 에 의존적이다.  
provider나 factory나 둘다 오로지 spring container에 접근해서 bean을 조회하는 것을 대리하는 기능이기 때문이다.  

따라서 우리가 spring에서 더 독립적인 provider를 사용하고 싶다면  
다음과 같은 방식이 가능하다.  

```java
@Scope("singleton")
    static class ClientBean {
    @Autowired
    private Provider<PrototypeBean> prototypeBeanProvider;

    public int logic() {
        PrototypeBean prototypeBean = prototypeBeanProvider.get();
        prototypeBean.addCount();
        int count = prototypeBean.getCount();
        return count;
    }
}
```

javax.inject의 provider를 사용하게 되면,  
좀더 spring에서 독립적으로 provider 패턴을 사용할 수 있다.  

## Web Scope
web 환경에서만 동작하는 scope이다.  
spring이 종료까지 다 관리를 해준다.  

웹 scope의 종류는 다음과 같다.
1. request : http 요청이 들어와서 나갈때까지 유지되는 scope이다. 요청마다 별도의 bean이 생성된다.
2. session : http session과 동일한 scope를 갖는 bean이다. 
3. application : 서블릿 컨텍스트와 동일한 scope를 가진다.  
4. web socket : 웹 소켓과 동일한 생명주기를 가진다.  

## request scope

먼저 request scope를 가진 bean을 언제 활용할까 ??  
바로 http 요청이 올때마다 로그가 쌓일텐데,  
tomcat은 요청당 thread를 만들어서 처리한다.  
이때 thread가 많아져서 요청마다 로그를 구분하기가 힘들어진다면,  
request scope로 logger를 만들어서 구현하면 된다.  
-> 옛날에는 이걸 몰라서 로그를 제대로 알아보지 못했던 기억이 난다.  

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {

    private final LogDemoServicce logDemoServicce;
    private final Provider<MyLogger> myLoggerProvider;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        String url = request.getRequestURL().toString();

        MyLogger myLogger = myLoggerProvider.get();
        myLogger.setRequestUrl(url);

        myLogger.log("controller test");

        logDemoServicce.logic("testId");

        return "OK";

    }

}

@Component
@Scope("request")
public class MyLogger {

    private String uuid;
    private String requestUrl;

    public void setRequestUrl(String requestUrl) {
        this.requestUrl = requestUrl;
    }

    public void log(String message) {
        System.out.println("[" + uuid + "]" + "[" + requestUrl + "]" + message);
    }

    @PostConstruct
    public void init() {
        uuid = randomUUID().toString();
        System.out.println("[" + uuid + "]" + "request scope bean create: " + this);
    }

    @PreDestroy
    public void close() {
        System.out.println("[" + uuid + "]" + "request scope bean close: " + this);
    }

}

@Service
@RequiredArgsConstructor
public class LogDemoServicce {

    private final Provider<MyLogger> myLoggerProvider;

    public void logic(String id) {
        MyLogger myLogger = myLoggerProvider.get();
        myLogger.log("service id = "+id);
    }
}
```

코드를 보면 이제 logger클래스는 request의 scope를 갖고 있다.  
그리고 요청이 올때마다 새로운 logger를 생성하고, logger의 id를 uuid로 구분하기 때문에,  
이는 요청마다 구분할 수 있는 구분자가 된다.  

따라서 실제로 log를 찍어보면 다음과 같다.  

```
[39d8f7ec-fd19-4ef0-98d4-d496b2ff9bbc]request scope bean create: hello.core.common.MyLogger@1f0bbad5
[39d8f7ec-fd19-4ef0-98d4-d496b2ff9bbc][http://localhost:8080/log-demo]controller test
[39d8f7ec-fd19-4ef0-98d4-d496b2ff9bbc][http://localhost:8080/log-demo]service id = testId
[39d8f7ec-fd19-4ef0-98d4-d496b2ff9bbc]request scope bean close: hello.core.common.MyLogger@1f0bbad5
[93ae1fef-decf-4857-919d-6ec19c5b97ba]request scope bean create: hello.core.common.MyLogger@2956bcba
[93ae1fef-decf-4857-919d-6ec19c5b97ba][http://localhost:8080/log-demo]controller test
[93ae1fef-decf-4857-919d-6ec19c5b97ba][http://localhost:8080/log-demo]service id = testId
[93ae1fef-decf-4857-919d-6ec19c5b97ba]request scope bean close: hello.core.common.MyLogger@2956bcba
[26fddf2a-5ce5-4ad3-9291-206902f86a2d]request scope bean create: hello.core.common.MyLogger@7e3f6487
[26fddf2a-5ce5-4ad3-9291-206902f86a2d][http://localhost:8080/log-demo]controller test
[26fddf2a-5ce5-4ad3-9291-206902f86a2d][http://localhost:8080/log-demo]service id = testId
[26fddf2a-5ce5-4ad3-9291-206902f86a2d]request scope bean close: hello.core.common.MyLogger@7e3f6487
```

이런식으로 총 요청이 3번있었음을 알수있고 어떤 url에 대한 요청인지도 알 수 있다.  

물론 log를 이렇게 남기기보다 여러가지 설정과 관심사 분리를 통해서 좀 더 예쁘게 구현할 수 있다.  
1. tomcat 로그 날짜별 설정
2. aop나 interceptor를 통한 로그 구현 등등...

다음 시간이 로그를 예쁘게 남기는 방법에 대해서 알아볼 예정이다.  

그리고 이런 코드를 좀더 발전시킬 수 있는 방법이 있다.  
proxy를 사용하면 굳이 provider를 사용하지 않아도 된다.  

코드로 살펴보자.  

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger {

    private String uuid;
    private String requestUrl;

    public void setRequestUrl(String requestUrl) {
        this.requestUrl = requestUrl;
    }

    public void log(String message) {
        System.out.println("[" + uuid + "]" + "[" + requestUrl + "]" + message);
    }

    @PostConstruct
    public void init() {
        uuid = randomUUID().toString();
        System.out.println("[" + uuid + "]" + "request scope bean create: " + this);
    }

    @PreDestroy
    public void close() {
        System.out.println("[" + uuid + "]" + "request scope bean close: " + this);
    }

}
```

이렇게 코드를 작성하면 proxy 객체로 만들 수 있다.  
jpa에서도 값이 초기회되지 않은 빈 객체를 proxy라고 하는데, 비슷하게 이해하면 될것 같다.  

그리고 controller와 service의 코드에서 provider를 없에고 기존의 mylogger를 주입받자.  
우리가 provider를 사용해야 했던 이유는 controller나 service의 경우는 singleton bean이다.  
따라서 application이 로드되면서 객체를 생성하고 의존관계를 주입하는데,  
이때 주입하는 객체인 mylogger는 scope가 request이기 때문에 현시점에는 존재하지 않는다.  
따라서 에러가 발생했다.  

이번에는 mylogger를 프록시로 만들면 controller나 service에서 객체를 주입받을때, 프록시 객체를 주입받는다.  
이 프록시객체는 진짜로 요청이 들어오는 내부에서 빈을 요청하는 로직이 들어가 있기 때문에  
런타임에서 request가 오면 그때 logger를 호출하게 된다.  

