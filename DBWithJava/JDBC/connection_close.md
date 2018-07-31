# Connection, Statement, ResultSet close 순서에 대한 고찰
보통 커넥션을 맺을때는 Connection -> Statement(PreparedStatdment) -> ResultSet 순서로 객체를 얻어오게 된다. 
Connection 객체로 DB 와 연결한 후 SQL 문을 데이터베이스로 보내기 위한 Statement(PreparedStatement) 객체를 생성한다. (`conn.prepareStatement(sql);`) 

생성된 Statement 에 파라미터에 알맞은 데이터를 바인딩 한 뒤 (`psmt.setString(1, data);`) State(PreparedStatement) 객체를 통해 Sql 쿼리를 실행하여 그 결과값을 ResultSet 으로 돌려준다.(`resultSet = psmt.excuteQuery();`)

이러한 과정을 거쳐 JDBC 에서는 DB 와 통신하여 SQL 을 실행하고 그 결과값을 가져오게 된다. 문제는 DB 와의 통신을 위해 얻어온 Connection, Statement, ResultSet 을 close 할 때, 어떻게 close 를 해야하는지이다.

처음에는 finally 구문에서 그저 전부 닫으면 되겠지, 하는 안일한 생각을 하였다.
```java
try {
    ...
} catch {
    ...
} finally {
    if (conn != null) {
        conn.close;
    }
    if (psmt != null) {
        psmt.close;
    }
    if (resultSet != null) {
        resultSet.close;
    }
}
```
하지만 이 방식에는 치명적인 문제점이 있었으니.. **3가지 모두 close 되기 전 어느 하나를 close 하다 에러가 발생하면 나머지는 close 할 수 없다** 는 점이었다. 예를 들어, conn.close() 실행시 Exception이 발생한다면 에러를 catch 하는 부분이 없기 때문에 코드는 더이상 아래로 흘러가지 못해 psmt, resultSet 이 줄줄이 close 되지 못할 수 있다. 그렇기에 모든 close() 함수 실행시에는 각각 try-catch 문으로 감싸주는 것이 행여나 있을 에러발생시에도 안전하게 연결을 해제시킬 수 있다.
```java
try {
    ...
} catch {
    ...
} finally {
    if (resultSet != null) {
        try {
            resultSet.close();
        } catch (Exception e) {}
    }
    if (psmt != null) {
        try {
            psmt.close();
        } catch (Exception e) {}
    }
    if (conn != null) {
        try {
            conn.close();
        } catch (Exception e) {}
    }
}
```
그렇다면 close 되는 순서는 상관이 없는가?

JDBC의 스펙은 Connection 이 close 되면 Statement 가 close 되고 Statement 가 닫히면 ResultSet 이 닫힌다고 되어있다.  그렇다면 connection 만 close 시키면 줄줄이 다 close 되어야 하는게 아닌가 ?! 싶을수도 있겠다. 

하지만 애석하게도 connection 을 close 시켜 나머지 모두 close 되기를 바란다 하더라도 statement close 시 Exception이 발생한다면 그 뒤의 resultset 은 close 되지 않을 가능성이 높다. 게다가 프로젝트에서 DBCP 를 사용한다면 `conn.close()` 실행시 connection 이 close 되는 것이 아니라 커넥션 풀로 되돌아가는 것이기 때문에 Statement 와 ResultSet 을 close 해준다고 장담할 수 없다. 

이러한 사유로 JDBC 사용시 close 하는 순서는 마지막에 획득한 것부터 close 하는것이 권장된다. (위와같이 ResultSet -> Statement -> Connection 순서)

>요약
>1. 커넥션에서 얻어온 객체는 close() 시 모두 각각 try-catch 문으로 감싸준다.
>2. 객체가 제대로 close() 되지 않는 상황을 방지하기 위해 close() 순서는 ResultSeet -> Statement -> Connectioin 순서대로 clo() 한다.