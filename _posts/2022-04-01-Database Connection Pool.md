---

title: JDBC와 Database Connection Pool
categories: [DBCP]
tags: [DBCP, JDBC, Hikari CP]     # TAG names should always be lowercase
author:
name: Choi Jin Kyu
link: https://github.com/hmcck27
toc: true
# date: 2022-02-05

img_path: /assets/img/mdImage/

---

## JDBC

자바의 database connection은 보통 jdbc(java Database Connectivity)를 사용한다.  
jdbc란 자바 프로그램 내부에서 sql을 실행하기 위한 데이터베이스를 연결해주는 application programming interface이다.  
즉 데이터베이스 사용을 위한 api이다.  

jdbc는 interface이기 때문에 실제 어떠한 데이터 베이스를 사용하던지, 즉 데이터베이스의 종류에 종속되지 않는다.  

음 만약에 이런 jdbc api가 없었더라면 = interface가 없다면,  
우리는 사용하는 database 종류에 맞춘 sql문을 각각 작성해야 했을것이다.  

![jdbc](./img/jdbc.jpeg)

jdbc의 역할은 다음과 같다.  
1. java application 에서 DB서버에 접속
2. sql을 수행하고 그 결과를 resultSet에 받아온다.
3. DB 자체에 대한 정보를 갖고온다.

즉 그림을 보면 알 수 있지만 외부 DB에 접근하는 작업을 jdbc api를 통해서 하게 된다.  

jdbc interface는 다음과 같다.  
1. JDBC driver manager : java application과 jdbc driver간의 접속을 공급한다.
2. JDBC driver API : Driver manager와 driver간의 접속 interface
3. JDBC driver : DBMS 접속을 제어하는 모듈, DBMS와의 통신 담당

정리하면 다음과 같은 순서대로 java application에서 DBMS로 연결된다.
1. Java application
2. JDBC Driver Manager
3. JDBC Driver
4. DBMS

# Database Connection 

어플리케이션과 Database어플리케이션 간의 통신을 위한 객체이다.  

우리의 어플리케이션에서 외부 서버에 존재하는 database에 접근하려면 다음과 같은 정보가 필요할 것이다.  

1. 어떤 데이터베이스인지 -> 어떤 driver가 필요할지(h2, postgreSql, 등등)
2. 데이터베이스가 위치한 서버의 url + port -> 데이터베이스의 url
3. 데이터 베이스 접근을 위한 username, password

JDBC를 통해서 Database Connection을 가져와 보자.

```java
public class DatabaseConn {

    static final String driverName = "org.h2.Driver";
	static final String dbUrl = "jdbc:h2:~/test";
	static final String username = "sa";
	static final String password = "";

    public static void main(){

        PreparedStatement  pstmt = null;
        ResultSet rs = null;

        try {
            sql = ""

            // 1. h2 driver 등록
            Class.forName(driverName); 

            // 2. get connection
            connection = DriverManager.getConnection(dbUrl, username, password);
            
            // 3. PreparedStatement 객체 생성
            pstmt = conn.createStatement();

            // 4. execute query
            rs = pstmt.executeQuery(sql);
            } catch (Exception e) {
            } finally {
                conn.close();
                pstmt.close();
                rs.close();
            }
        }
    }
}
```

# Database Connection Pool

Database Connection Pool이란, 말 그대로  
database와의 connection을 담고 있는 pool이다.   

위 코드를 살펴보면 다음과 같은 절차로 이루어져 있다.  
1. Driver를 등록하고
2. DriverManager를 통해서 connection을 얻고
3. PreparedStatement 객체를 생성하고 이를 통해서 쿼리를 수행한다.

즉, 개발자가 직접 connection을 생성하고 쿼리를 날려야 함을 알 수 있었는데,  
만약에 쿼리를 날리는 모든 부분에 다 이렇게 코드를 작성해야 한다면 이건 조금 골치가 아파진다.  

이런 순간을 위해서 Database Connection Pool이 존재한다.  

쿼리를 날리거나 DB에 뭔가를 반영하고 싶을때, 이미 pool안에 만들어둔 Database Connection을 가져와서 수행하면 된다.  

DBCP의 장점은 무엇일까 ?  
1. 미리 객체를 생성해뒀기 때문에 런타임에 동적으로 객체 생성, 소멸의 cost가 없고 빠르게 메모리에서 이미 만들어진 객체를 이용할 수 있다.
2. 직접 코드 작성을 하지 않아도 된다.  
3. connection의 수를 관리 또는 제한해서 부하를 줄이고, 효율적으로 connection들을 이용하게 한다.  

DBCP의 경우 다음과 같은 특징을 가진다.
1. WAS가 실행하면서 connection 객체를 미리 만들고 pool에 저장해놓는다.
2. request를 수행하면서 db connection이 필요할때마다 가져다 사용하고 다시 반환한다.
3. connection의 수는 설정가능하다.
4. 만약 현재 사용가능한 connection이 없을 경우, request는 connection을 기다린다.

그렇다면 connection의 수를 크게 설정하면 꼭 좋은 것일까 ?  
일단 connection객체를 생성한다는 것은 그만큼 메모리를 사용한다는것과 동일하다.  

만약에 너무 많은 connection을 만들어두면, 메모리를 많이 사용하게 되고,  
그렇다고 너무 적게 만들어두면, thread가 connection을 갖기 위해 대기하는 시간이 길어진다.

그럼 어떻게 connection의 수를 관리해야 할까 ?  

결국에 connection을 이용하는건 thread이다.  
1. thread pool의 크기가 connection pool의 크기보다 크다면 대기하는 시간이 있을 수 있다.
2. thread pool의 크기가 connection pool의 크기보다 작다면 대기하는 시간은 없겠지만, 남은 connection이 있을 수 밖에 없다.  

즉 connection pool의 크기를 조절해서 성능을 증가시키는것은 어쨋든 한계가 있다.  
결국에 thread의 수보다 크면 그 이상 증가해도 의미가 없다.  

## Hikari CP

Database Connection Pool을 관리하는 기술이다.  
hikari cp가 어떻게 connection pool을 관리하는지 알아보자.  

만약에 request가 하나들어온다고 생각해보자.  
1. request 요청됨
2. 해당 request는 thread1에서 실행된다.
3. thread1에서 database connection을 필요로 하게 되면, 
4. 이전에 사용했던 Connection이 존재하는지 확인한다.
5. 있다면 현재 사용가능한 connection이 있는지 확인한다.
6. 없다면 전체 connection pool에서 사용가능한 connection이 있는지 확인한다.
7. 아예 connection을 get하지 못하면, 가능한 connection이 생길때까지, handoffqueue에서 대기한다.
8. 다른 thread에서 connection을 반환하면 해당 connection의 사용정보를 pool에 기록하고
9. 반납된 connection은 handoffqueue에 삽입된다.
10. thread1에서 connection을 get한다.

hikari cp에서는 connection의 수를 어떻게 설정하는게 좋을지 가이드를 제시한다.  
> num of connections = core_count * 2 + effective spindle count  
즉 cpu 갯수 * 2 + DB가 동시에 관리할 수 있는 io 요청 수  

로 설정하는게 좋다고 한다.  

spring boot를 사용하게 되면 default connection pool로 hikari cp를 사용하게 되는데,  
이때 위의 가이드에서 제시하는 것처럼, connection의 수를 조절해보자.  

spring의 properties나 yaml로 해당 설정이 가능하다.  

