# Spring Bean 중복 해결 (2)

앞서서 spring bean이 중복되는 상황 해결을 위한 여러가지 방법에 대해서 알아보았다.  
1. @Autowired의 필드명 매칭
2. @Autowired의 생성자 파라미터 매칭
3. @Qualifier 사용
4. @Qualifier와 @Primary 사용

결국 우리가 유용하게 사용하게 되는 방법은 3번과 4번이다.  
직접 빈 네임 이외에도 빈을 구분하는 구분자를 추가로 만들어주기 때문에,  
더욱 자유도가 높다고 할 수 있다.  

예를 들어서 1,2번은 직접 생성자나 변수 선언의 코드를 건드리지만, 3번과 4번은 코드 수정에서 조금 더 자유롭기 때문이다.  

하지만 3번 4번을 사용하게 되면 하나의 문제점이 존재한다.  

바로 빈의 구분자를 string으로 선언하기 때문에 컴파일 타이밍에서 직접 에러를 잡기 어렵다는 점이다.  

이번에는 이 에러를 잡을 custom한 annotation을 직접 만들어보자.  

```java
@Component
public class MemberServiceImpl implements MemberService {

    @Autowired
    public MemberServiceImpl(@Qulifier("remoteDB") MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}

@Component
@Qualifier("memeoryDB")
public class MemoryMemberRepository implements MemberRepository {
}

@Component
@Qualifier("remoteDB")
public class JdbcMemberRepository implements MemberRepository {
}
```  

코드를 살펴보면 우리가 하나의 인터페이스를 구현하는 두개의 구현체를 동시에 구현했었다.  
원래는 동일한 이름의 스프링 빈이 생성되면 에러가 발생하나,  
qualifier를 통해서 빈이름을 제외하고도 빈을 구분하는 구분자를 추가로 구현하였다.  

결국에는 빈을 주입할때, @Qualifier를 통해서 중복되는 빈중 어떤 빈을 사용할지 명시할 수 있었다.  

하지만 결국에 @Qualifier를 사용하는 방법은 다음과 같다.  

```java
@Autowired
public MemberServiceImpl(@Qulifier("remoteDB") MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
}
```

즉 @Qualifier의 내부 파라미터로 들어가는 string은 컴파일 타임에서 type check가 되지 않는다.  
따라서 아예 string을 제외한 annotation을 추가로 만들면 해결된다.  

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier("remoteDB")
public @interface RemoteDB {
}
```

RemoteDB라는 새로운 annotation을 만든다.  
그러고 이 annotation에 @Qualifier가 붙는 대상을 명시하고 지속 시간을 명시하는 @Target, @Retention을 똑같이 붙인다.  
당연히 @Inherited랑 @Documented도 붙인다.  

그리고 해당 annotation이 @Qualifier의 역할을 동일하게 수행할 수 있도록 @Qualifier도 추가한다.  

그러면 이제 @RemoteDB라는 annotation은 @Qualifier의 역할을 동일하게 수행하고, 추가로 @Qualifier의 파라미터인 "remoteDB"을 아예  
붙임으로써 이제는 컴파일 타이밍에서 사전에 에러를 방지할 수 있다.  

다음과 같은 사용하면 된다.  
RemoteDB 

```java
@Autowired
public MemberServiceImpl(@RemoteDB MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
}
```

이렇게 붙임으로써 해당 생성자를 통해서 의존관계를 주입할때 jdbcMemberRepository를 사용할 것임을 명시할 수 있다.  
그리고 개발자가 직접 @Qualifier의 파라미터 스트링을 작성하지 않기 때문에 오타가 발생해도 컴파일 타이밍에 바로 에러를 잡을 수 있다.  

## 정리
하지만 너무 남발하지는 말자 !  
이것도 관리의 일환이 되버리거나 개발자가 annotation을 follow하지 못하는 경우가 있을 수 있다...!

