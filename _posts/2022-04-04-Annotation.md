---

title: Annotation
categories: [Java]
tags: [Java, Annotation]     # TAG names should always be lowercase
author:
name: Choi Jin Kyu
link: https://github.com/hmcck27
toc: true
# date: 2022-02-05

img_path: /assets/img/mdImage/

---

# Annotation

우리가 자바로 개발하면 필요할때마다 annotation을 사용한다.  
interface를 만들면 @Override를 사용하게 되고
lombok을 사용하면 @Getter, @Setter등을 사용하게 된다.
또 validation에서는 @NotNull과 같은 annotation을 사용한다.    

문제는 @Getter라고 적어준다고 해도 실제 우리 코드에는 get 함수가 구현되어 있지 않다.  

하지만 난 단순히 code에 annotation을 작성했을 뿐인데,  
어떻게 실제 코드가 변화할 수 있는건지 의문이 있었다.  

---

## Annotation이 뭘까 ?  
1. Annotation은 metadata이다. 즉 data에 대한 data이다.  
예를 들어서 다음 코드를 보자.  
```java
@Override
public String toString() {
return "This is String Representation of current object.";
}
```
코드에서 우리는 toString()함수를 override했다.  
그리고 메소드 위에 @Override를 작성했다.  
@Override annotation을 붙이지 않아도 물론 override된다.  
그렇다면 왜 Annotation을 사용하는 걸까 ?  
@Override는 javac에게 해당 메소드는 override하는 메소드임을 알려준다.  
즉 메소드에 대한 metadata이다.  
그리고 만약에 parent class에 override할 method가 없다면 compile 에러를 만든다.  

만약에 @Override를 붙이지 않았다면 실제로 parent에 toString()이 있는지 체크하지 않을것이고, 개발자가 원하는 방향 = 이 메소드는 override 메소드야 임을 compiler에서는 알 수가 없다. 물론 compile은 제대로 될겠지만 워하는 방향으로 작동하지 않을 것이다.  

따라서 왜 annotation을 붙이는지 명확하게 알 수 있다.  

2. Annotation은 적절한 코드를 생성한다.  
Lombok의 @Getter같은 경우는 개발자가 직접 getter를 구현하지 않아도 실제로 컴파일 타이밍시 코드를 생성해준다.  
만약에 boilerplate code를 줄이고 생산성을 늘리고 싶다면 코드를 삽입해주는 Annotation은 큰 도움이 된다.  

3. run time시 특정 기능을 실행하도록 정보를 제공한다. spring 같은 framework에서는 DI를 위한 annotation을 다는 경우가 많다. 이런 annotation은 run time에서 annotation이 적용된 element의 역할을 정의하게 된다.

---

## 왜 Annotation이 필요할까 ?
annotation이전에는 xml을 사용했다.  
하지만 xml은 유지보수가 쉽지 않았다.  
xml보다 코드에 친숙한 metadata를 위해서 annotation을 만들게 되었다.  

물론 xml과 annotation은 각각 장단점이 있다.  
xml은 코드 자체에서 loosely coupled 즉 의도적으로 느슨한 결합을 만들 수 있다.  
즉 configuration과 code의 결합성을 약하게 하는것이다.  

만약에 application 전역에서 적용되는 상수, 파라미터를 설정하고 싶은 경우 code에 묶여있지 않은 xml를 사용하는것이 더 좋은 선택일 수 있다.  
하지만 code의 일부분에만 적용되도록 하고 싶다면 그런 경우에는 xml보다는 annotation을 사용하게 나을 것이다.  

---

## Custom Annotation을 만드는 방법
일단은 @Override 어노테이션을 확인해보자.  
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

해당 어노테이션에서는 별일 안한다.  
그냥 단순하게 parent class에 해당 method가 존재하는지 체크할 뿐이다.  
Annotation에는 본질적으로 buisness logic이 들어가 있지 않다.  
마치 interface처럼 logic은 제외하고 target element가 사용할 수 있어야 한다.  

해당 annotation을 보고 jvm은 bytecode level에서 동작할 수 있게끔 작동한다.  

일단은 Annotation이 들어갈 수 있는 여러 기본적인 Annotation들에 대해서 알아보자.  

1. @Target - 어노테이션의 적용 대상
2. @Retention - 어노테이션이 유지 되는 기간

Target의 경우는 다음과 같다.
```java
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```

보면 적용되는 대상에 따라서 element를 설정해주면 됨을 알 수 있다.  
예를 들어서 @Getter는 클래스대상이니까 TYPE,  
@Override의 경우는 method대상이니까 METHOD이다.  

Retention의 경우에는 다음과 같다.
```java
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

예시로 몇개의 annotation을 살펴보자.
1. @Override  
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

target은 method이다.  
retention은 source이다. -> compile되고 이후에는 버려진다.

2. @Getter  
```java
@Target({ElementType.FIELD, ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface Getter {
	/**
	 * If you want your getter to be non-public, you can specify an alternate access level here.
	 * 
	 * @return The getter method will be generated with this access modifier.
	 */
	lombok.AccessLevel value() default lombok.AccessLevel.PUBLIC;
	
	/**
	 * Any annotations listed here are put on the generated method.
	 * The syntax for this feature depends on JDK version (nothing we can do about that; it's to work around javac bugs).<br>
	 * up to JDK7:<br>
	 *  {@code @Getter(onMethod=@__({@AnnotationsGoHere}))}<br>
	 * from JDK8:<br>
	 *  {@code @Getter(onMethod_={@AnnotationsGohere})} // note the underscore after {@code onMethod}.
	 *  
	 * @return List of annotations to apply to the generated getter method.
	 */
	AnyAnnotation[] onMethod() default {};
	
	boolean lazy() default false;
	
	/**
	 * Placeholder annotation to enable the placement of annotations on the generated code.
	 * @deprecated Don't use this annotation, ever - Read the documentation.
	 */
	@Deprecated
	@Retention(RetentionPolicy.SOURCE)
	@Target({})
	@interface AnyAnnotation {}
}
```
Target은 FIELD, TYPE으로 field과 class, enum, interface에 선언가능하다.   
Retention은 source이다.  

3. @SpringBootApplication

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface SpringBootApplication {
    // 생략
}

```

target은 class이고 retention은 runtime이다.  
spring이 작동되고 있는 runtime에도 해당 annotation은 유지된다.  

4. javax의 @NotNull

```java
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE })
@Retention(RUNTIME)
public @interface NotNull {
    // 생략
}
```

validation이라 그런지 다양한 target에 적용이 가능하고,  
validation은 runtime에 필요하니까 Retention은 run time이다.

한번 custom annotation을 만들어보자.  
우리가 만들 annotation은 메소드 위에 붙여서 해당 메소드의 진행 시간을 체크하는 메소드이다.  
aop를 통해서 구현하고, 해당 메소드가 끝나면 log로 시간을 찍어보자.  

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
}
```
일단 annotation을 만들어주자.  
1. method가 target이니까 method로 설정해준다.
2. 로그를 남기려면 runtime으로 설정해야 한다.

```java
@Component
@Aspect
public class LogAspect {
    Logger logger = LoggerFactory.getLogger(LogAspect.class);

    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable{
        StopWatch stopWatch = new StopWatch();
        stopWatch.start(joinPoint.getSignature().toString());

        Object ret = joinPoint.proceed();
        stopWatch.stop();
        logger.info(stopWatch.prettyPrint());
        return ret;
    }
}
```

1. LogAspect에서는 logger를 선언한다.
2. stopwatch 객체를 활용해서 시간을 측정하고, start()의 taskname으로 joinPoint의 signature로 설정한다.
3. proceed가 끝나면 logger.info로 log를 출력한다.

```java
@Controller
public class TestController {

    @LogExecutionTime
    @GetMapping("/logCheck")
    public String logCheck(logCheckRequestDto dto) {
        return dto.getName();
    }
}
```

그리고 해당 url로 request를 날리면 다음과 같이 log가 남겨진다.  
```
---------------------------------------------
ns         %     Task name
---------------------------------------------
013841600  100%  String com.example.demo.TestController.logCheck(logCheckRequestDto)
```

끝 !

---

## Annotaion processor

Annotation을 붙이고 컴파일 타이밍때 우리가 원하는 동작을 하게 하는 핵심은 annotation processor이다.  

annotation processor는 javac의 일부이다.  
즉 컴파일 타이밍에 작동한다.  

1. javac가 소스 코드를 compile한다.
2. 이때 annotation processor는 annotation을 보고 기존의 코드를 수정하거나 변경한다.

한번 직접 실습으로 확인해보자.  
여기서 우리는 custom한 annotation과 기존 annotation으로 샘플 클래스를 만들어보자.  

```java
@Getter
@Setter
@NoArgsConstructor
public class MyClass extends MyParentClass{

    @NotNull
    private String name;
    private String age;

    @Override
    @LogExecutionTime
    public void printMyName() {
        System.out.println("this is child class name");
    }
}
```

해당 클래스를 작성하고나서 build를 해보자.  
즉 compile이후에 코드가 어떻게 작성되는 확인해보는 것이다.  

```java
// 해당 코드는 compile되고 나서 byte code를 decompile한 결과입니다.

public class MyClass extends MyParentClass {
    @NotNull
    private String name;
    private String age;

    @LogExecutionTime
    public void printMyName() {
        System.out.println("this is child class name");
    }

    public String getName() {
        return this.name;
    }

    public String getAge() {
        return this.age;
    }

    public void setName(final String name) {
        this.name = name;
    }

    public void setAge(final String age) {
        this.age = age;
    }

    public MyClass() {
    }
}
```

다음과 같이 getter, setter같은 경우는 retention이 source였기 때문에
compile된 이후에는 사라지는 것을 확인할 수 있다.  

그리고 @LogExecutionTime과 @NotNull같은 경우는 retention이 run time이었기 때문에 compile이후에도 유지됨을 알 수 있다.
