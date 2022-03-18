---

title: spring security (2)
categories: [Spring]
tags: [인증/인가, Spring Security]     # TAG names should always be lowercase
author:
name: Choi Jin Kyu
link: https://github.com/hmcck27
toc: true
# date: 2022-02-05

img_path: /assets/img/mdImage/

---

# Servlet Authentication Architecture

이전에 우리는 스프링 시큐리티에서 서블렛 딴의 구조를 알아봤다.  
즉 하나의 요청이 들어오고 어떻게 필터를 거치는지에 대해서 알아봤다.  

이번에는 spring security의 authentication의 구체적인 구조를 알아보자.  

서블렛 객체가 httprequest를 갖기 이전에 필터를 거친다고 했다.  
하지만 필터에서 제대로 걸러진다면 서블렛까지 요청이 도달하지 못한다.  

필터에서는 어떤 기준으로 요청을 거르는 걸까 ??  
전에도 설명했듯이 당연히 인증/인가의 과정이다.  

1. 적합한 유저인지
2. 권한이 있는 유저인지  

스프링은 여러 레이어와 구조를 갖고 해당 인증/인가를 수행한다.  
여기에 어떤 구조들이 존재하는지 알아보자.  

---

## Security Context Holder

Security Context Holder는 스프링 시큐리티에서 핵심적인 역할을 한다.  
말그래도 Security Context를 포함한다.  

무슨 말이냐, 누가 적합한 유저인지 그 디테일을 갖고 있다.  
여기에 유저에 대한 값이 있으면 적합한 유저이다.  

![img_4.png](spring security4.png)

위의 그림을 보자.  
context holder가 context를 갖고 있고,
이 context안에는 authenticaiton이 존재한다.  
그리고 authentication 에는 세가지 종류 Principal, Credential, Authorities가 존재한다.

먼저 각 용어들이 무엇을 의미하는지 살펴보자.  
1. Security Context는 authentication 객체들을 갖고 있다.
2. Authentication은 authenticaiton manager의 input으로 해당 유저의 Principal, Credentials, Authorities에 대한 정보를 갖는다.  
3. Principal은 유저의 id를 의미한다. UserDetail의 인스턴스이다.
4. Credential은 유저의 비밀번호다. 
5. Authorities는 권한이다. roles나 scope를 의미한다.  

일단은 대애충 훑어봤다. 더 자세한건 밑에서 다뤄보자.  

특정 유저가 적합한 유저라고 알려주고 싶다면 가장 빠른 방법은 이 context holder에 해당 유저에 대한 값을 넣어주면 된다.  

코드로 살펴보자.  

```java
SecurityContext context = SecurityContextHolder.createEmptyContext();
Authentication authentication =
new TestingAuthenticationToken("username", "password", "ROLE_USER");
context.setAuthentication(authentication);
SecurityContextHolder.setContext(context);
```

코드에서는 특정 유저에 대한 authentication 객체를 만든다음에 context holder의 빈 컨텍스트에 직접 넣어주고 있다.

만약에 context에서 유저에 대한 정보를 갖고 오고 싶을 경우에는 어떻게 할까 ?

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
String username = authentication.getName();
Object principal = authentication.getPrincipal();
Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
```
여기서는 컨텍스트를 갖고와서 authentication객체에서 유저에 대한 정보를 갖고 온다.  

일단은 다음과 같은 방식으로 진행된다.  

---

## Security Context

Security Context Holder에 의해서 유지되고, authentication 객체를 갖는다.  

---

## Authentication

유저는 authentication 객체를 생성한다.  
그리고 이 객체는 secutiry context에 의해서 보관되고 사용된다.  

두가지의 목적이 있는데,  
1. authentication manager의 input이다. 초기에 유저가 만들고 authentication manager에서 인증 과정을 거치기 전에 생성된다고 보면 된다. 이 경우에는 아직 isAuthenticated는 false이다.  
   그니까 무슨 말이냐면 처음에 로그인을 시도하면 아직은 authentication 객체가 없는 상태이다.  
   그래서 처음 만들고 authentication manager의 input으로 들어가서 실제 인증 과정을 거친다는 의미이다.  
2. Security context에 저장되서 적합한 유저의 데이터를 갖고 있다.  

authenticaiton 에는 세가지 부분이 있다고 했다.  

1. principal : 사용자의 아이디를 의미하는 경우가 많다.  
2. credentials : 사용자의 비밀번호
3. authorities : 사용자의 권한  

authentication 인터페이스를 살펴보자.  

```java
public interface Authentication extends Principal, Serializable {
    Collection<? extends GrantedAuthority> getAuthorities(); // Authentication 저장소에 의해 인증된 사용자의 권한 목록
    Object getCredentials();
    Object getDetails();
    Object getPrincipal();
    boolean isAuthenticated();
    void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```
이렇게 ID, 비밀번호, 권한, 인증 여부, 디테일을 반환하는 함수들이 있다.  

---

## Authentication Manager

스프링 시큐리티가 어떻게 인증을 구현할지에 대해 정의하는 api이다.  
authentication manager가 로그인시 authentication을 생성하고 이를 security context에 저장하는 역할을 한다.  
구현체로는 가장 대표적으로 provide manager가 사용된다.  

---

## Provide manager

Authentication Manager의 구현체이다.  

Provide Manager는 다수의 authentication provider를 대리한다.  

![img_5.png](spring security5.png)  

하나의 authentication provider는 해당 authentication의 성공, 실패를 결정할 수 있다.  
만약의 다수의 authentication provider가 다 처리할 수 없는 authentication의 경우  
ProviderNotFoundException이 발생한다.  

각각의 authentication provider는 다른 종류의 authentication을 수행한다.  

---

## Authentication Provider

여러개의 authentication provider가 위의 provider manager에 주입되는데,  
각각의 authentication provider는 다른 종류의 authentication을 수행한다.  


---

## Authentication Entry Point와 

authentication entry point는 말그대로 인증의 진입점이다. 
이 진입점에서는 client에게 response를 보내는 역할을 하게 된다.  
무슨 response냐면, 만약 유저가 인증이 안된 유저이면 인증을 요청하는 response를 보낸다.  
예를 들면 리다이렉를 수행하게 한다던지...

만약 인증이 된 유저라면 인증 요청 response를 보낼 필요가 없고 유저의 요청에 응답하면 되기 때문에  
항상 response를 보내지는 않는다.  

---

## Abstract Authentication Processing Filter

Abstract Authentication Processing Filter는 security filter chain에 포함되서  
인증된 유저인지 필터링한다.  

시나리오는 다음과 같다.  

1. abstract authentication processing filter에서 해당 요청에 인증이 되어 있는지 하기 위해서
2. authentication 객체를 생성한다.  
3. 그리고 이 authentication 객체는 authentication manager로 보내진다.  
4. authentication manager의 구현체인 authentication provider가 실제 인증을 수행하고  
5. 성공과 실패에 따라서 다른 분기를 타게 된다. 실제로는 다른 핸들러가 처리하게 되는거다.  

![img_6.png](spring security6.png)

그림에서 위의 시나리오가 잘 설명되어 있다.  
하지만 실제의 구현의 abstract Authentication Filter를 사용하지 않는다.  
이건 추상화된 필터고 이 추상화 객체를 구현한 구현체 예를 들면,  
UsernamePasswordAuthenticationFilter 같은걸 사용하게 된다.  

위의 그림에서는 전부 구현체가 없다.  

---

## 실제로 구현해보자..!

드디어 구현이다.  
지루한 이론에 대해서 훑어봤으니 실제로 어떻게 작동되는지 직접 구현해보자...!

위에서 설명한 인터페이스들을 실제로 어떤 구현체를 사용할지 정해야하고  
spring security를 좀 편리하게 사용하는 방법에 대해서 알아보자.  

근데 이건 다음 시간에...


