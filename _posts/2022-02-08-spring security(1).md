---

title: spring security (1)
categories: [Spring]
tags: [인증/인가, Spring Security]     # TAG names should always be lowercase
author:
name: Choi Jin Kyu
link: https://github.com/hmcck27
toc: true
# date: 2022-02-05

img_path: /assets/img/mdImage/

---

# Spring Security

Spring Security에 대해서 알아볼 시간이 됬다 !

지난 시간동안 개발하면서 스프링을 사용했고, 스프링 시큐리티는 필수로 사용했는데 깊은 이해가 없이 진행했었다.  
따라서 이번 기회에 한번 정리해볼 생각이다.

지난 포스트들에서 우리는 인증/인가, 토큰/세션, 대표적인 토큰 JWT에 대해서 정리했다.

이번에는 스프링 시큐리티에 대해서 정리하고, 이 스프링의 하위 프레임워크가 인증/인가, 토큰/세션이라는
키워드와 어떤 연관이 있는지 알아보자.

그리고 이번 spring security를 정리하면서 코드를 통해서 기본적인 security를 구현해볼 생각이고,  
다음 포스트에서는 oauth2 -> spring security + oauth2 에 대해서 알아보고 구현해볼 생각이다.

여기에 정리하는 대부분의 내용은 spring 공식 문서에 정리되어 있는 내용이다.  
필요한 사람은 공식 문서를 보면 되겠다.  
https://docs.spring.io/spring-security/reference/index.html

---

## Spring Security란 뭘까 ?

Spring 이라는 걸출한 프레임워크는 거대한 생태계로 유명하다.  
정말로 여러가지 기능들을 인터페이스로 제공한다.  
우리는 필요한 인터페이스를 구현하면 된다.

스프리에서 보안을 담당하는 하위 프레임워크가 spring security이다.  
여기서의 보안은 인증/인가/공격방어 를 의미한다.

이런 기능들을 개발자 입장에서 편리하고 간편하게 구현할 수 있도록 인터페이스를 제공한다.

많은 보안 관련 옵션을 제공해주기 때문에 개발자 입장에서는 정말 간편하다.

---

## Spring security의 종류

총 두가지이다.
1. **Servlet application**
2. Reactive application

그 중 우리는 Servlet application에 대해서 다뤄볼 예정이다.  
간단하게 종류에 대해서만 설명해보면,

Servlet applcation은
> Spring Security integrates with the Servlet Container by using a standard Servlet Filter. This means it works with any application that runs in a Servlet Container. More concretely, you do not need to use Spring in your Servlet-based application to take advantage of Spring Security.

다음과 같다. 무슨 말이냐면, 스프링 시큐리티는 서블릿 컨테이너와 결합되는데, 뭘 이용해서 결합하냐, 표쥰 서블릿 필터를 이용해서다.  
이게 무슨 말이냐면 서블릿 컨테이너에서는 스프링을 굳이 사용하지 않아도 스프링 시큐리티를 사용할 수 있다는 의미이다.

핵심만 요약하면
1. 서블릿 컨테이너에서의 보안
2. 서블릿 필터를 사용

여기에 있는 용어중에 서블릿 필터는 모를 수 있어도 서블릿 컨테이너는 알고 가야 한다.  
서블릿 컨테이너는 나중에 was + spring application 에 대해서 다루면서 자세히 정리할 예정이지만,  
간단하게만 설명하면, 서블릿들의 컨테이너. 서블릿들을 모아서 관리하는게 서블릿 컨테이너이다.

이게 얘기가 길어질 수 밖에 없는데,  
서블릿은 클라이언트에서 서버로 보낸 요청을 처리하고 서버가 반환해주는 일련의 프로세스를 처리하는 자바의 인터페이스이다.  
그냥 요청이 오면 서블릿이 생긴다고 편하게 생각하자 !

그런 서블릿을 모아서 관리하고 동작시킬 수 있는 환경을 제공하는게 서블릿 컨테이너이며,  
대표적인 얘로는 톰캣이 있다. 톰캣 설치할때 JRE이 필요한 이유가 자바에 종속되있기 때문이다.

서블릿 컨테이너는 요청이 올때마다 새로운 스레드를 생성하고 요청에 맞는 서블릿 객체를 생성한다.  
그리고 스레드가 이 서블릿 객체를 맡게 된다.  
그래서 클라이언트 요청 반환의 프로세스를 이 서블릿 객체를 통해서 스레드가 처리한다.

어쨋든 다시 돌아가면, 서블릿 컨테이너에서의 보안을 할 수 있게 해준다.  
스프링을 사용하지 않아도 자바 진영에서는 사용할 수 있다느 얘기이다.

뭐 이건 그렇게 중요한 이야기는 아니고  
스프링 시큐리티의 핵심은 서블릿 필터이다.

핵심으로 들어가기 전에 코드로 한번 정체를 확인해보자.

---

## 서블릿 필터

위에서 spring security의 servlet application에 대해서 이야기 했다.  

스프링 시큐리티는 서블렛 필터에 기반하고 있다.  
단순하게 이해해보자.  
서블렛은 위에서 요청, 반환의 과정을 처리해주는 자바 인터페이스였다.  

인증/인가/보안은 어떠한 요청이 들어오면 해당 요청이 적합한 유저이며, 권한이 있고, 보안상 안전한지 평가하는 것을 의미한다.  

즉 서블렛에 필터를 씌우는 과정이 필요하다.  

흔히 스프링 어플리케이션을 띄우면 컨트롤러가 그 요청을 받아서 처리하게 되는데,  
그 이전에 서버딴에서는 was(= 톰캣)에서 우선적으로 네트워크 요청을 처리한다.  
따라서 컨트롤러에 요청이 도달하기 전에 서블렛 컨테이너에 먼저 요청을 보게 된다.  

스프링 시큐리티는 was가 요청을 받으면서 해당 요청이 컨트롤러에 도달하게끔 하는 서블렛위에 필터를 씌운다.  

실제로 스프링 시큐리티를 프로젝트에 적용을 시켜보면, 컨트롤러에 로그를 찍어놔도  
로그가 찍히기 전에 요청이 차단되거나 하는 현상을 확인할 수 있다.  

밑의 이미지는 스프링 시큐리티 공식 문서의 스프링 시큐리티 아키텍처이다.  
클라이언트의 요청이 서블렛에 닫기 전에 필터 체인을 거치는 모습을 볼 수 있다.

![springsecurity](springsecurity.png)

이렇게 생각해보면 무슨 생각이 드는가 ?  
개발자는 스프링 컨트톨러 딴에서 인증/인가/보안에 관한 개발을 상당히 덜 할 수 있게 될것이다.  
필터 설정만 잘 해주면 된다.

---
## 서블릿 필터 체인

위의 그림에서 리퀘스트가 겪는 시나리오를 먼저 살펴보자.  
1. 클라이언트는 요청을 서버로 보낸다.  
2. 서블렛 컨테이너는 필터 다수 + 서블렛에 해당하는 필터 체인을 생성한다.  
3. 필터 체인을 거치면서 인증/인가가 처리되고 서블렛에 도달한다. 이 과정에서 HttpServletRequest가 생성된다.  
4. 서블렛이 리퀘스트의 HttpServletRequest를 생성 컨트롤러에게 전달한다.
5. 비즈니스 로직 처리.
6. HttpServletResponse에 담아서 반환하면
7. 서블렛이 위의 객체를 was에게 전달. was가 네트워크로 통신을 쏜다.

위와 같은 과정에서 필터들은 **순서대로** 처리된다.  

**즉 필터의 순서는 매우 중요하다.**

필터를 어떤 순서로 설정하느냐에 따라서 컨트롤러가 받는 HttpServletRequest가 달라진다.  

---

## 개발자의 필터 구현

그럼 개발자는 필터를 커스터마이징해야할 텐데 이걸 어떻게 할까 ?  

그건 밑의 그림을 보자.  

![springsecurity1.png](springsecurity1.png)

기존의 필터 1이 **DelegatingFilterProxy**로 교체되어 있다.  

스프링이 제공하는 필터의 구현체가 이거다.  
기본적으로 필터는 스프링 빈으로 인식되지 못하기 때문에 이렇게 구현체를 만들 수 있게끔 이런 빈 대체용 필터를 제공한다.  
그러니까 빈 하나로는 안되니까 대체 필터를 만들고 거기에 빈을 넣으면 된다.  

수도 코드로 살펴보자.  

```java
// 공식문서랑 똑같다 !
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    
        // delegate는 스프링 빈을 필터로 만들어준다. 즉 커스터마이징한 필터를 만들 수 있게끔.
	Filter delegate = getFilterBean(someBeanName);

        // 필터링
	delegate.doFilter(request, response);

}
```

자 근데 여기서 끝나기는 아쉽다.  
예를 들어서 개발자가 아 난 /api/v1/jk에는 필터 1,2,3번 걸고
/api/v2/jk2에는 다른 필터들을 걸고 싶어라고 한다면 ?  

스프링 시큐리티에서는 FilterChainProxy와 SecurityFilterChain을 제공한다.


![springsecurity2.png](springsecurity2.png)

그림을 보면 FilterChainProxy가 SecurityFilterChain을 사용하고,  
어떤 Spring security filter가 발동할지 결정한다.  

![springsecurity3.png](springsecurity3.png)

다음과 같이 요청의 종류에 따라서 다른 필터들을 거치게 할 수 있다.  

생각해보자. 이건 필수적으로 해야되는 사항이다.  
우리가 스프링 시큐리티를 사용하지 않고 인증/인가를 구현한다고 했을때,  
각 api마다 특성이 다르기때문에 = 로그인안해도 사용가능, 로그인해야지 사용가능  
각 컨트롤러에서는 인증/인가 로직을 다르게 구현해야 하는데,  

FilterChainProxy와 SecurityFilterChain을 사용해서 상황에 맞게 인증/인가를 설정할 수 있게 된다.  

---

##정리

여기까지 해서 Spring Security의 정말 잘 추상화된 구조를 살펴봤다.  
다음에는 구체적으로 들어가서 설명해볼 생각이다.  

코드 구현까지하면 언제 끝날지 모르겠다..






