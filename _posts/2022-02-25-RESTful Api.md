---

title: Restful Api란
categories: [Networkd]
tags: [RESTful API, API]     # TAG names should always be lowercase
author:
name: Choi Jin Kyu
link: https://github.com/hmcck27
toc: true
# date: 2022-02-05

img_path: /assets/img/mdImage/

---

# RestApi란 ?  

Rest Api란 말은 개발하면서 참 많이 들어보는 단어일 것이다.  
특히 웹 서비스를 개발한다면 클라이언트, 서버간의 통신에 api를 사용하게 된다.  

클라이언트는 원하는 데이터를 담아서 api 요청을 보내게 되고, 서버는 해당 요청을 받아서  
응답을 내려주는 것을 api를 통한 통신이라고 흔히 지칭하는데,  

api가 정확하게 무엇이고,  
또 Rest Api로 개발하는것의 중요성을 많이 얘기하는데,  
정확하게 Rest Api의 규격이 무엇인지 잘 모르는것 같아서 한번 정리해보려고 한다.  

---

## API

API란 application programming interface의 약자이다.  
응용 프로그램에서 사용할 수 있도록 운영 체제나 프로그래밍 언어가 제공하는 기능을 제어할 수 있게 만든 인터페이스이다.  

인터페이스라...  
처음 api의 정의에 대해서 들으면 당황스러울 것 같다.  
우리가 사용하는 api란 말은 클라이언트, 서버간의 통신을 의미했기 때문이다.  

하지만 실제로 api란 좀 더 다양하고 큰 의미로 사용한다.  

일단 interface란 말에 집중해보자.  
GUI는 그래픽을 통한 화면으로 시스템, 프로그램을 제어할 수 있는 인터페이스이다.  
즉 interface는 내가 원하는 기능을 정의하고 사용할 수 있게 해준다.  

api = application programming interface 도 interface이다.  
application간의 interface라고 보면 된다.  

즉 특 application 과 통신하거나 내가 원하는 기능을 위임하려면  
해당 application이 제공하는 interface인 api를 통해서 사용할 수 있다.  

---

## RESTFUL API

RESTFUL API란 REST 아키텍쳐의 제약 조건을 준수하는 application programming interface이다.  
REST는 Representational state tranfer를 의미한다.  

### REST란 ?

REST는 protocol이나 표준이 아니라 아키텍쳐의 원칙 세트이다.  
그래서 누구나 rest api를 개발하지만 개발하는 방식이나 범위는 다르다.  

앞서서 우리는 api에 대해서 알아봤는데,  
웹 어플리케이션을 개발하는 사람들이 사용하는 api는 정확하게는 restful api를 지칭하는 일이 많다.  
물론 정확하게 restful하게 api를 설계하는 사람은 많이 없고,   
restful을 지향하면서 개발하게 되는것 같다.  

restful에서 rest란 다음의 약자이다.  
> Representational State transfer
 
자원의 표현을 통해서 해당 자원에 대한 정보를 주고 받는것을 의미한다.  

여기서 자원이란 소프트웨어가 관리하는 모든 것을 의미한다.  

예를 들어서 내가 간이 spring web을 만들었는데,  
해당 web의 기능으로는 다음이 있다.  
1. 회원가입
2. 내 게시물
3. 내 댓글  

그렇다면 이 소프트웨어에서 관리하는 자원에는 분명히
1. 나의 id
2. 나의 password
3. 나의 게시물에 대한 정보
4. 나의 댓글에 대한 정보

등을 관리할 것이다. (데이터베이스나 nosql등으로...)  
그렇다면 클라이언트 입장에서 데이터를 받아서 화면에 렌더링해주고 싶다면,  
해당 정보를 서버에 요청해야 한다.  

그럴려면 두가지의 전제가 있어야 한다.  

1. 자원을 어떻게 표현할 것인가 ?

여기서 interface의 특징이 나온다.  
클라이언트에서는 id라고 하고, 서버에서는 id대신에 uid라고 관리한다면 둘의 네이밍은 맞지 않게 된다.  
즉 자원에 대한 표현은 클라이언트는 서버의 표현을, 서버는 클라이언트의 표현에 대해서 알고 있어야 한다는 것이다.  

그래서 자원을 얻고 싶을때는 서버가 제공하는 interface에 따라서 질문을 던지게 된다.  

2. state의 전달
예를 들어서 내가 원하는 정보가 회원에 대한 정보라고 쳐보자.  
그렇다면 어떤 회원인지 데이터를 관리하고 있는 서버에게 질문을 던져야 한다.  

예를 들어서  
그 회원의 id는 1번이다.
그 회원의 name은 jk이다.  
그 회원의 age는 25이다. 등으로 말이다.  

즉 해당 회원 데이터를 갖고 오기 위해서는 해당 회원의 state를 전달해야 한다.  

여러가지 방식으로 state를 전달할 수 있다.  

URI를 생각해보자. URI는 자원의 위치뿐만 아니라 자원의 id(고유값)을 통해서 자원을 명시한다.  
여기서 자원의 id가 state가 될 수 있다.  

또 http body에 다음과 같은 json을 담을 수도 있다.  
```json
{
    "id" : "1"
}
```

이런 방식으로 state를 전달할 수 있다.  

정리해보면 rest란 http URI나 http body에 state를 담고  
여러가지 http method, GET, POST, PUT, DELETE를 통해서   
해당 자원에 대한 interface를 제공하고 처리하는것을 의미한다.  

우리는 http protocol을 사용하기 때문에 REST API또한 http를 활용하면 된다.  

## REST의 정의

조금 더 정리해서 설명해보면

1. Http URI를 통해서 자원을 명시한다.
2. HTTP method = GET, POST, PUT, DELETE를 사용한다.
3. 자원에 대한 CRUD처리를 적용한다.  

## REST의 구성요소

1. 자원 - resource(어떤 자원인지)
2. 행위 - GET, POST, PUT, DELETE
3. 행위의 내용 - message(행위를 수행할 때 필요한 정보)

## RESTFUL API의 조건

1. 늘 동일한 interface
    어디서 요청이 오던 동일한 리소스에 대한 api 요청의 interface는 어떤 클라이언트에게나 적용되야 한다.  
2. 독립적인 clinet, server
    두 application은 완전히 독립적이어야 한다. 
3. stateless하다.
    서버의 세션을 필요로 하지 않는다.  
    서버 또한 client에 대한 정보를 저장하지 않는다. 
4. 캐싱이 가능해야 한다. 
    가급적이면 리소스를 클라이언트, 혹은 서버에서 캐싱할 수 있어야 한다.  
5. 계층 구조
    클라이언트-서버 통신 중간에 다른 중개자가 있을 수 있고, 해당 중개자의 정보에 대해서는  
    클라이언트, 서버 둘다 고려하지 않아도 api가 작동해야 한다.  
    즉 interface만의 역할을 해야한다. 