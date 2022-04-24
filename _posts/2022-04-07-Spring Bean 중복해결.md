# Spring Bean 중복 해결  

spring bean은 singleton으로 유지된다.  
DI container는 각 interface를 구현한 구현체 클래스를 spring bean으로 등록하게 되는데,  
의존성을 주입해줄 때는 구현체가 아니라 interface기준으로 주입하게 된다.  

---

## 중복을 허용하되 필요한 빈은 1개.
하나의 interface를 두개의 빈이 동시에 구현하게 하고 싶다면 어떻게 해야할까 ?  
예를 들어보자.  
  
```java
@Configuration
@ComponentScan
public class AutoAppConfig {
}

@Component
@RequiredArgsConstructor
public class MemberServiceImpl implements MemberService {
    private final MemberRepository memberRepository;
}

@Component
public class MemoryMemberRepository implements MemberRepository {
}

@Component
public class JdbcMemberRepository implements MemberRepository {
}

```

만약 이런 상황에서 autoappconfig를 읽고 component scan을 한다고 가정하자.  
그러면 MemberServiceImpl을 spring bean으로 올리고, 해당 bean이 의존하는 MemberRepository를 주입하게 된다.  

하지만 현재 상황은 memberRepository의 구현체가 두개이다.  
둘다 spring bean으로 올라가 있는 상황에서 어떤 구현체에 의존할것인지  
명확하지 않은 상황이고, 결국에 spring은 다음과 같은 error를 뱉는다.  

```
NoUniqueBeanDefinitionException: No qualifying bean of type 'MemberRepository' available: expected single matching bean but found 2: jdbcMemberRepository,memoryMemberRepository
```

즉 single을 예상했는데 2개의 bean을 찾았고, 어떤것을 채택하지 몰라 에러가 발생한 것이다.  

이런 상황에서는 3개의 해결 방법이 있다.  

1. @Autowired의 필드명 매칭
```java
@Component
public class MemberServiceImpl implements MemberService {

    @Autowired
    private MemberRepository jdbcMemberRepository;
}

@Component
public class MemoryMemberRepository implements MemberRepository {
}

@Component
public class JdbcMemberRepository implements MemberRepository {
}

```
다음과 같이 MemberServiceImpl이 의존하는 구현체를 @Autowired를 통해서 직접 엮어준다.  
@Autowired는 필드명을 보고 필요한 인스턴스를 주입해준다.  

DI container는 일단 interface = 타입을 보고 먼저 bean을 찾고, 만약에 해당하는 bean이 여러개라면 필드이름으로 bean을 찾는다.

2. @Autowired를 이용한 생성자 파라미터 매칭
```java
@Component
public class MemberServiceImpl implements MemberService {

    @Autowired
    public MemberServiceImpl(MemberRepository jdbcMemberRepository) {
        this.memberRepository = jdbcMemberRepository;
    }
}

@Component
public class MemoryMemberRepository implements MemberRepository {
}

@Component
public class JdbcMemberRepository implements MemberRepository {
}
```
생성자를 통한 의존성 주입을 해줄때 파리미터로 구현체를 지정해서 넘기면 자동으로 jdbcMemberRepository가 주입된다.  

3. @Qualifier를 통한 매칭
@Qualifier는 빈의 추가 구분자를 지정해주는 방식이다.  
원래 bean은 bean name으로 구분되지만, @Qualifier를 달아주면 해당 bean의 새로운 이름이 하나 더 생긴것과 같다.  

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

겹치는 구현체에 @Qualifier를 달아준다.  
그러면 value로 주어진 name으로 새로운 bean의 구분자가 생성이 되고,  
만약에 겹치는 경우 내가 원하는 구현체의 Qualifier name을 주면 해당 qualifier를 가진 bean으로 주입한다.

만약에 해당 qualifier를 가진 bean이 없다면 qualfier value를 name으로 가진 bean을 다시 찾아본다.  

그런데도 없으면 당연히 예외가 발생한다.  

4. primary bean을 지정.

말그래도 여러 bean이 동일한 interface를 구현하고 있는 경우,  
primary가 붙은 빈이 가장 우선권을 가진다.

```java
@Component
public class MemberServiceImpl implements MemberService {

    @Autowired
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}

@Component
@Qualifier("memeoryDB")
public class MemoryMemberRepository implements MemberRepository {
}

@Component
@Primary
public class JdbcMemberRepository implements MemberRepository {
}
```

---

## 중복을 허용하되 여러개의 빈이 동시에 필요
개발하다보면 동적으로 interface의 구현체를 선택해야 할일이 있다.  
따라서 빈을 중복으로 등록하고 로직에 따라서 빈을 선택하게 하면 된다.  

(당연히 중복으로 등록하기 위해서는 @Qualifier 또는 @Primary를 사용해야 한다.)

코드로 한번 보자.

```java
@Component
@Qualifier("memeoryDB")
public class MemoryMemberRepository implements MemberRepository {
}

@Component
@Primary
public class JdbcMemberRepository implements MemberRepository {
}

@Component
@RequiredArgsConstructor
public class SelectRepositoryService{

    private final Map<String, MemberRepository> repositoryMap;

    public MemberRepository selectOne(String code) {

        MemberRepository memberRepository = repositoryMap.get(code);
        return memberRepository;
    }
}

public enum RepositoryEnum {
    memoryMemberRepository, jdbcRepository
}
```  

@RequiredArgsConstructor를 통해서 생성자 주입을 해주는데,  
문제는 `Map<String, MemberRepository>`를 주입해준다.  
현재 스프링이 관리하는 MemberRepository.class의 bean은 두개다.  
스프링은 자동으로 {bean이름 : repositoryInterface를 가진 bean} 페어를 등록해준다.  
따라서 selectOne()으로 원하는 repository를 가져올 수 있다.  
즉 동적으로 저장되는 저장소가 다를경우에 유용하게 사용할 수 있다.  

## 정리  
우리가 여러개의 bean을 띄우는 경우는 있을 수 있다.  
예를 들어서 로컬에서는 h2 데이터베이스를, 그리고 운영 테스트에서는 원격 postgres를 사용한다고 가정했을때, 두 데이터베이스를 번갈아 끼면서 테스트 한다던지 등등 다양한 상황이 있을 수 있다.  

즉 구현체를 바꿔끼는 경우가 있는 경우, 혹은 특정한 상황에서만 특정 빈을 이용하는 경우에서는 이렇게 중복되는 빈을 대처하기 위한 방법이 있어야 한다.

@Primary와 @Qualifier를 잘 사용해서 조합하면 좋을 것 같다..!  

또한 의외로 동적으로 관리하는건 많이 사용되는 것 같다..!  
-> 지금 내가 짠 코드에서 수정할 부분이 스쳐지나간다.  

스프링이 제공해주는 전략패턴을 잘 활용해보자.