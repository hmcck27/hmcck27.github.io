---

title: @Trasactional과 entity manager의 생성
categories: [JPA]
tags: [JPA, transaction]     # TAG names should always be lowercase
author:
name: Choi Jin Kyu
link: https://github.com/hmcck27
toc: true
# date: 2022-02-05

img_path: /assets/img/mdImage/

---


최근 테스트 코드를 작성하다가 하나의 문제를 겪었다.  

내가 만든 테스트 시나리오는 다음과 같다.

1. entitymanager를 통해서 Order를 가져온다.
2. Order의 status를 미확인결제로 update한다.
3. orderRepository.flush()를 통해서 db에 변경사항을 반영한다.
4. orderRepository.findById()를 통해서 다시 order를 갖고온다
5. order의 status가 미확인결제인지 체크한다.  

하지만 계속해서 5번 검증 과정에서 orderRepository.findById()의 결과인 order 엔티티의 status가 같지 않았다.   
가능성은 두가지였다.  
1. 이미 order의 status가 미확인결제였다.  
2. 어떠한 이유에서 flush()가 제대로 작동하지 않음. 즉 dirty checking으로 entity가 변경되었는데, flush()에서 변경이 반영되지 않았다.  

직접 해보니 flush() 시점에서 update로그가 찍히지 않음을 확인했다.  

전체 테스트 코드를 한번 붙여넣기 해보면,  

```java
    @Test
    public void 결제status검증_결제확인() {

        // 가장 최신 order 갖고 오기
        long count = orderRepositoryJpa.count();
        Optional<Order> findOrder = orderRepositoryJpa.findById(count);
        Order order = findOrder.get();
        order.updateOrderStatus(OrderStatus.결제완료);

        // flush
        orderRepositoryJpa.flush();
        // detach -> 다시 order를 찾아와야 하니까.
        orderRepository.detach(order);

        // orderValidateRequestDto 생성
        OrderValidateRequestDto orderValidateRequestDto = new OrderValidateRequestDto();
        orderValidateRequestDto.setOrderId(count);
        orderValidateRequestDto.setOrderStatus(OrderStatus.결제완료.toString());

        // ordervalidate 실행
        Order returnOrder = orderService.validateOrder(orderValidateRequestDto.toServiceDto());
        OrderValidateResponse orderValidateResponse = OrderValidateResponse.createDto(returnOrder);
        
        // then
        assertThat(orderValidateResponse.getOrderId()).isEqualTo(order.getId());
        assertThat(orderValidateResponse.getOrderStatus()).isEqualTo(order.getStatus().toString());
    }
```

분명히 flush()를 하는 시점에서 쿼리가 나가야 하는데, 나가지 않았다.  

그런데 이걸 곰곰히 생각해보면, 당연히 되지 않음을 알 수 있다.  

**Transaction이 없기 때문이었다....  **

결국에는 entity manager와 transaction의 연관성을 까먹었기 때문에 이런 삽질을 반복하면서 왜 쿼리가 안나가는건지 의아해하고 있었던 것이다.  

JPA에서 transaction과 eneitymanager의 연관성을 다음과 같다.  

1. entitymanger의 변경 사항은 flush()에 의해서 db에 반영된다. 하지만 이는 transaction안에서의 이야기이다.  
2. pending되었던 변경사항은 transaction 끝나면서 db에 commit된다.  
3. find/select 하는 건 transaction이 필요하지 않다. 하지만 transaction없이 find/select하면, entity를 lock하지 않는다.  

즉 entity manager에서 영속 상태로 존재하는 entity들의 변경사항들은 dirty-checking에 의해서 변경이 감지되고, 이러한 변경은 transaction 안에서만 db에 반영된다.  

자 여기까지는 정말 단순한 생각이었다.  
반전에 반전이 반복되는데, 
  
spring data jpa에서는 jpaRepository를 상속받아서 interface를 만는데, 
이 JpaRepository의 구현체 중에서 기본적으로 SimpleJpaRepository를 사용하게 된다.  

이 SimpleJpaRepository의 모든 메소드 위에는 transactional이 붙어있다.  

그런데 왜 꼭 flush()를 위해서 @Transactional을 붙여줘야 하는걸까..?  

위의 코드에서는 em.flush()를 사용한게 아니라, 
orderRepository.flush()를 사용했다.  

그리고 orderRepository는 JpaRepository를 상속받았다.  
그러면 orderRepository.flush() 또한 JpaRepository의 flush이고, 이때 사용하는 구현체는 SimpleJpaRepository이다. 그리고 SimpleJpaRepository의 모든 메소드는 @Transactional이 붙어있다.  

실제로 SimpleJpaRepository의 flush() 메소드는 다음과 같다.  

```java
	@Transactional
	@Override
	public void flush() {
		em.flush();
	}
```  

즉, 해당 flush()는 분명히 transaction 안에서 동작한다.  

다시 상황을 정리해보면  
1. no-transaction 에서 order라는 entity의 dirty-checking이 동작하고 있고 값이 바뀌었다.  
2. 그리고 flush()를 사용하면서 transaction이 새로 열린다.  
3. 하지만 여기서 변경사항이 update 쿼리로 나가지 않는다.  

코드를 간략하게 보자.  

```java
    @Test
    @Rollback
    public void transaction테스트() {
        // 기본 트랜잭션 존재하지 않음.

        // order fetch
        Optional<Order> findOrder = orderRepositoryJpa.findById(2L);
        Order order = findOrder.get();

        // order status 변경
        if (order.getStatus().equals(OrderStatus.미확인결제)) {
            order.updateOrderStatus(OrderStatus.결제완료);
        } else {
            order.updateOrderStatus(OrderStatus.미확인결제);
        }

        // 영속성 컨텍스트 flush()
        orderRepositoryJpa.flush();
        
    }
```

실행한 로그를 훑어보자.  
```
2022-06-12 22:02:40.621  INFO 76379 --- [           main] t.m.h.service.OrderControllerTest        : Started OrderControllerTest in 9.239 seconds (JVM running for 10.487)
2022-06-12 22:02:40.873 DEBUG 76379 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Creating new transaction with name [org.springframework.data.jpa.repository.support.SimpleJpaRepository.findById]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT,readOnly
2022-06-12 22:02:40.873 DEBUG 76379 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Opened new EntityManager [SessionImpl(2002285602<open>)] for JPA transaction
2022-06-12 22:02:40.897 DEBUG 76379 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Exposing JPA transaction as JDBC [org.springframework.orm.jpa.vendor.HibernateJpaDialect$HibernateConnectionHandle@3f523dae]
2022-06-12 22:02:40.910 DEBUG 76379 --- [           main] org.hibernate.SQL                        : 
    select
        order0_.order_id as order_id1_8_0_,
        order0_.created_at as created_2_8_0_,
        order0_.updated_at as updated_3_8_0_,
        order0_.complex_type as complex_4_8_0_,
        order0_.detail_address as detail_a5_8_0_,
        order0_.house_id as house_i17_8_0_,
        order0_.pg_uid as pg_uid6_8_0_,
        order0_.is_living as is_livin7_8_0_,
        order0_.land_code as land_cod8_8_0_,
        order0_.merchant_uid as merchant9_8_0_,
        order0_.price as price10_8_0_,
        order0_.product_id as product11_8_0_,
        order0_.order_status as order_s12_8_0_,
        order0_.trans_amount as trans_a13_8_0_,
        order0_.trans_period as trans_p14_8_0_,
        order0_.trans_type as trans_t15_8_0_,
        order0_.unique_no as unique_16_8_0_,
        order0_.user_id as user_id18_8_0_ 
    from
        orders order0_ 
    where
        order0_.order_id=?
2022-06-12 22:02:40.953 DEBUG 76379 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Initiating transaction commit
2022-06-12 22:02:40.953 DEBUG 76379 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Committing JPA transaction on EntityManager [SessionImpl(2002285602<open>)]
2022-06-12 22:02:40.963 DEBUG 76379 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Closing JPA EntityManager [SessionImpl(2002285602<open>)] after transaction
2022-06-12 22:02:40.967 DEBUG 76379 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Creating new transaction with name [org.springframework.data.jpa.repository.support.SimpleJpaRepository.flush]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
2022-06-12 22:02:40.967 DEBUG 76379 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Opened new EntityManager [SessionImpl(126560570<open>)] for JPA transaction
2022-06-12 22:02:40.967 DEBUG 76379 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Exposing JPA transaction as JDBC [org.springframework.orm.jpa.vendor.HibernateJpaDialect$HibernateConnectionHandle@146b4c6c]
2022-06-12 22:02:40.969 DEBUG 76379 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Initiating transaction commit
2022-06-12 22:02:40.970 DEBUG 76379 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Committing JPA transaction on EntityManager [SessionImpl(126560570<open>)]
2022-06-12 22:02:40.970 DEBUG 76379 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Closing JPA EntityManager [SessionImpl(126560570<open>)] after transaction
2022-06-12 22:02:40.996  INFO 76379 --- [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2022-06-12 22:02:40.998 TRACE 76379 --- [ionShutdownHook] o.h.type.spi.TypeConfiguration$Scope     : Handling #sessionFactoryClosed from [org.hibernate.internal.SessionFactoryImpl@7221539] for TypeConfiguration
2022-06-12 22:02:40.998 DEBUG 76379 --- [ionShutdownHook] o.h.type.spi.TypeConfiguration$Scope     : Un-scoping TypeConfiguration [org.hibernate.type.spi.TypeConfiguration$Scope@2a09eb5f] from SessionFactory [org.hibernate.internal.SessionFactoryImpl@7221539]
2022-06-12 22:02:41.001  INFO 76379 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2022-06-12 22:02:41.016  INFO 76379 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```  

로그를 보니까 단서를 찾아볼 수 있었다.  
1. select쿼리에서는 전후로, 새로운 transaction, entity manager를 만든다.  
2. flush()에서도 전후로, 새로운 transaction, entity manager를 만든다.  

즉, 처음의 상황에서 entity manager는 다르다.  
이러면 select로 갖고온 order entity는 entity manager를 통해서 관리가 되고 있지 않다는 의심이 들었다.  
왜냐하면 select시 새로운 entity manager가 생성이 되고, 그때 잠깐 괸라가 되고 entity manager는 삭제되니까.

다시 한번 코드를 수정해서 확인해보자.  

```java
    @Test
    @Rollback
    public void transaction테스트() {
        // 기본 트랜잭션 존재하지 않음.

        // order fetch
        Optional<Order> findOrder = orderRepositoryJpa.findById(2L);
        Order order = findOrder.get();
        boolean contains = em.contains(order);
        System.out.println("contains = " + contains);
        // order status 변경
        if (order.getStatus().equals(OrderStatus.미확인결제)) {
            order.updateOrderStatus(OrderStatus.결제완료);
        } else {
            order.updateOrderStatus(OrderStatus.미확인결제);
        }
        // 영속성 컨텍스트 flush()
        orderRepositoryJpa.flush();
    }
```  
이번에 직접, entity manager를 주입받아서 order라는 entity가 em에 의해서 관리되고 있는지를 찍어보았다.  
결과는 다음과 같다.  

```
contains = false
```  
즉 order는 entity manager에 의해서 관리가 되고 있지 않은 상황이다.  

이래서야 dirty-checking이 동작하지 않는 것이 당연하다.  

그러면 @Transactional을 메소드에 붙이면 어떻게 로그가 찍히는지 확인해보자.  

```
2022-06-12 22:11:07.419  INFO 76941 --- [           main] t.m.h.service.OrderControllerTest        : Started OrderControllerTest in 8.969 seconds (JVM running for 10.495)
2022-06-12 22:11:07.464 DEBUG 76941 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Creating new transaction with name [team.means.housingsolution.service.OrderControllerTest.transaction테스트]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
2022-06-12 22:11:07.465 DEBUG 76941 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Opened new EntityManager [SessionImpl(2124327048<open>)] for JPA transaction
2022-06-12 22:11:07.486 DEBUG 76941 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Exposing JPA transaction as JDBC [org.springframework.orm.jpa.vendor.HibernateJpaDialect$HibernateConnectionHandle@8318808]
2022-06-12 22:11:07.487  INFO 76941 --- [           main] o.s.t.c.transaction.TransactionContext   : Began transaction (1) for test context [DefaultTestContext@359df09a testClass = OrderControllerTest, testInstance = team.means.housingsolution.service.OrderControllerTest@408bb173, testMethod = transaction테스트@OrderControllerTest, testException = [null], mergedContextConfiguration = [WebMergedContextConfiguration@43df23d3 testClass = OrderControllerTest, locations = '{}', classes = '{class team.means.housingsolution.HousingSolutionApplication}', contextInitializerClasses = '[]', activeProfiles = '{prod1}', propertySourceLocations = '{}', propertySourceProperties = '{spring.config.location = classpath:application-prod1.yml, org.springframework.boot.test.context.SpringBootTestContextBootstrapper=true}', contextCustomizers = set[org.springframework.boot.test.autoconfigure.actuate.metrics.MetricsExportContextCustomizerFactory$DisableMetricExportContextCustomizer@4b44655e, org.springframework.boot.test.autoconfigure.properties.PropertyMappingContextCustomizer@0, org.springframework.boot.test.autoconfigure.web.servlet.WebDriverContextCustomizerFactory$Customizer@4fe767f3, org.springframework.boot.test.context.filter.ExcludeFilterContextCustomizer@5db250b4, org.springframework.boot.test.json.DuplicateJsonObjectContextCustomizerFactory$DuplicateJsonObjectContextCustomizer@37918c79, org.springframework.boot.test.mock.mockito.MockitoContextCustomizer@0, org.springframework.boot.test.web.client.TestRestTemplateContextCustomizer@124c278f, org.springframework.boot.test.context.SpringBootTestArgs@1, org.springframework.boot.test.context.SpringBootTestWebEnvironment@1efee8e7], resourceBasePath = 'src/main/webapp', contextLoader = 'org.springframework.boot.test.context.SpringBootContextLoader', parent = [null]], attributes = map['org.springframework.test.context.web.ServletTestExecutionListener.activateListener' -> true, 'org.springframework.test.context.web.ServletTestExecutionListener.populatedRequestContextHolder' -> true, 'org.springframework.test.context.web.ServletTestExecutionListener.resetRequestContextHolder' -> true, 'org.springframework.test.context.event.ApplicationEventsTestExecutionListener.recordApplicationEvents' -> false]]; transaction manager [org.springframework.orm.jpa.JpaTransactionManager@193792e6]; rollback [true]
2022-06-12 22:11:07.676 DEBUG 76941 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Found thread-bound EntityManager [SessionImpl(2124327048<open>)] for JPA transaction
2022-06-12 22:11:07.676 DEBUG 76941 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Participating in existing transaction
2022-06-12 22:11:07.690 DEBUG 76941 --- [           main] org.hibernate.SQL                        : 
    select
        order0_.order_id as order_id1_8_0_,
        order0_.created_at as created_2_8_0_,
        order0_.updated_at as updated_3_8_0_,
        order0_.complex_type as complex_4_8_0_,
        order0_.detail_address as detail_a5_8_0_,
        order0_.house_id as house_i17_8_0_,
        order0_.pg_uid as pg_uid6_8_0_,
        order0_.is_living as is_livin7_8_0_,
        order0_.land_code as land_cod8_8_0_,
        order0_.merchant_uid as merchant9_8_0_,
        order0_.price as price10_8_0_,
        order0_.product_id as product11_8_0_,
        order0_.order_status as order_s12_8_0_,
        order0_.trans_amount as trans_a13_8_0_,
        order0_.trans_period as trans_p14_8_0_,
        order0_.trans_type as trans_t15_8_0_,
        order0_.unique_no as unique_16_8_0_,
        order0_.user_id as user_id18_8_0_ 
    from
        orders order0_ 
    where
        order0_.order_id=?
contains = true
2022-06-12 22:11:07.744 DEBUG 76941 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Found thread-bound EntityManager [SessionImpl(2124327048<open>)] for JPA transaction
2022-06-12 22:11:07.744 DEBUG 76941 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Participating in existing transaction
2022-06-12 22:11:07.809 DEBUG 76941 --- [           main] org.hibernate.SQL                        : 
    update
        orders 
    set
        complex_type=?,
        detail_address=?,
        house_id=?,
        pg_uid=?,
        is_living=?,
        land_code=?,
        merchant_uid=?,
        price=?,
        product_id=?,
        order_status=?,
        trans_amount=?,
        trans_period=?,
        trans_type=?,
        unique_no=?,
        user_id=? 
    where
        order_id=?
2022-06-12 22:11:07.846 DEBUG 76941 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Initiating transaction rollback
2022-06-12 22:11:07.846 DEBUG 76941 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Rolling back JPA transaction on EntityManager [SessionImpl(2124327048<open>)]
2022-06-12 22:11:07.857 DEBUG 76941 --- [           main] o.s.orm.jpa.JpaTransactionManager        : Closing JPA EntityManager [SessionImpl(2124327048<open>)] after transaction
2022-06-12 22:11:07.858  INFO 76941 --- [           main] o.s.t.c.transaction.TransactionContext   : Rolled back transaction for test: [DefaultTestContext@359df09a testClass = OrderControllerTest, testInstance = team.means.housingsolution.service.OrderControllerTest@408bb173, testMethod = transaction테스트@OrderControllerTest, testException = [null], mergedContextConfiguration = [WebMergedContextConfiguration@43df23d3 testClass = OrderControllerTest, locations = '{}', classes = '{class team.means.housingsolution.HousingSolutionApplication}', contextInitializerClasses = '[]', activeProfiles = '{prod1}', propertySourceLocations = '{}', propertySourceProperties = '{spring.config.location = classpath:application-prod1.yml, org.springframework.boot.test.context.SpringBootTestContextBootstrapper=true}', contextCustomizers = set[org.springframework.boot.test.autoconfigure.actuate.metrics.MetricsExportContextCustomizerFactory$DisableMetricExportContextCustomizer@4b44655e, org.springframework.boot.test.autoconfigure.properties.PropertyMappingContextCustomizer@0, org.springframework.boot.test.autoconfigure.web.servlet.WebDriverContextCustomizerFactory$Customizer@4fe767f3, org.springframework.boot.test.context.filter.ExcludeFilterContextCustomizer@5db250b4, org.springframework.boot.test.json.DuplicateJsonObjectContextCustomizerFactory$DuplicateJsonObjectContextCustomizer@37918c79, org.springframework.boot.test.mock.mockito.MockitoContextCustomizer@0, org.springframework.boot.test.web.client.TestRestTemplateContextCustomizer@124c278f, org.springframework.boot.test.context.SpringBootTestArgs@1, org.springframework.boot.test.context.SpringBootTestWebEnvironment@1efee8e7], resourceBasePath = 'src/main/webapp', contextLoader = 'org.springframework.boot.test.context.SpringBootContextLoader', parent = [null]], attributes = map['org.springframework.test.context.web.ServletTestExecutionListener.activateListener' -> true, 'org.springframework.test.context.web.ServletTestExecutionListener.populatedRequestContextHolder' -> true, 'org.springframework.test.context.web.ServletTestExecutionListener.resetRequestContextHolder' -> true, 'org.springframework.test.context.event.ApplicationEventsTestExecutionListener.recordApplicationEvents' -> false]]
2022-06-12 22:11:07.876  INFO 76941 --- [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2022-06-12 22:11:07.876 TRACE 76941 --- [ionShutdownHook] o.h.type.spi.TypeConfiguration$Scope     : Handling #sessionFactoryClosed from [org.hibernate.internal.SessionFactoryImpl@7a0f06ad] for TypeConfiguration
2022-06-12 22:11:07.876 DEBUG 76941 --- [ionShutdownHook] o.h.type.spi.TypeConfiguration$Scope     : Un-scoping TypeConfiguration [org.hibernate.type.spi.TypeConfiguration$Scope@21c3f35b] from SessionFactory [org.hibernate.internal.SessionFactoryImpl@7a0f06ad]
2022-06-12 22:11:07.878  INFO 76941 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2022-06-12 22:11:07.888  INFO 76941 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```

1. 테스트 메소드를 실행하면서 트랜잭션이 열린다. 동시에 entity manager도 생성이 된다.
2. select 쿼리도, update 쿼리도 해당 쿼리가 실행되는 SimpleJpaRepository의 메소드들의 @Transactional의 propagation level은 Propagation.REQUIRED 이다. 즉 기존의 transaction이 있으니 새로운 transaction을 열지 않는다.  

즉 테스트 메소드 전체의 @Transactional 때문에 하나의 공통의 entity manager만이 존재하고,  
order는 당연히 entity manager에 의해서 관리된다.  

따라서 dirty-checking이 동작하고, 변경사항이 반영된다.  

그래서 결론 !!

우리가 평소에 개발할때, 디비에 변경되는 사항들 즉 dirty-checking이 동작하게 하려면 @Transactional 이 필요하다.  
너무 당연한건데... 급 SimpleJpaRepository에 트랜잭션이 존재하는데 왜 동작하지 않는지 궁금했다.  

원래 @Transactional이 붙으면 자동으로 entity manager를 생성한다.  
영속성 컨텍스트 자체가 @Transactional이 시작될때 생성되고 닫힌다.  

기본적인 개념을 잊지말자 !!

오늘 삽질은 끝.

사실 삽질하다가 초반에 왜 제대로 동작하지 않았는지 알아버렸지만, 그래서 마무리를 지어야지...  

