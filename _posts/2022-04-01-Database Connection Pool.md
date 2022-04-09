---

title: Database Connection Pool
categories: [DBCP]
tags: [DBCP]     # TAG names should always be lowercase
author:
name: Choi Jin Kyu
link: https://github.com/hmcck27
toc: true
# date: 2022-02-05

img_path: /assets/img/mdImage/

---

# Database Connection Pool

오늘 면접을 봤는데 DBCP에 대한 질문을 받았다.  
이번 기회에 한번 정리해서 알아보려고 한다.  

Database Connection Pool이란 was가 실행되면서 DB와 미리 connection을 맺어놓은 객체들을 저장한다.  
그러니까 미리 connection을 맺어놓고 필요할때마다 갖다가 꺼내쓰는 방식이다.  
물론 사용이 끝나면 다시 pool로 반환한다.  

## 왜 사용할까 ?  
만약에 DBCP를 사용하지 않으면 어떻게 될까 ?  
그러면 매 요청마다 커넥션을 잡아야 한다.  

conn = DriverManager.getConnetion("asdfasdf")  
을 통해서 conn을 잡고 해당 conn을 통해서 직접 쿼리를 날려야 한다.  

이렇게 말이다.  

```java
try {

    Class.forName(JDBC_DRIVER);
    conn = DriverManager.getConnection(DB_URL, USERNAME, PASSWORD);
    stmt = conn.createStatement();

    String sql1 = "select * from test";

    rs = stmt.executeQuery(sql1);

    while(rs.next()) {
        String no = rs.getString("no");
        String name = rs.getString("name");
        System.out.println("no = "+no+" , "+"name = "+name);
    }

}catch(SQLException se1) {
    se1.printStackTrace();
}catch(Exception ex) {
    ex.printStackTrace();
}finally {
    rs.close();
    stmt.close();
    conn.close();
}
```

생각해보면 매번 쿼리를 날릴때마다 직접 connection을 만들고 처리하고 반환까지 해야 한다고 생각하면 참 귀찮고 번거로운 작업이다.  

그래서 우리는 Database Connecion Pool을 사용한다.  

## 특징

1. was가 실행되면서 미리 connection 객채들을 생성하고 pool에 저장한다.  
2. 요청에 따라서 pool에서 connection 객체를 갖고와서 사용하고 반환한다.
3. 미리 connection이 맺어져 있기 때문에, 연결 시간이 소요되지 않는다.  
4. connection의 수는 제한적이다.  
5. 이용자 수에 따라서 connection의 수를 직접 조절할 수 있다.
6. 많이 설정하면 메모리 소모가 많은대신 대기시간이 줄어든다.

## HikariCP

Database Connection Pool을 관리한다.  
