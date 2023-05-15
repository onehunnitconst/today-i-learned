# 계층형 쿼리

### 재귀 쿼리

- WITH문에 RECURSIVE 키워드를 붙이면 기존 표준 SQL에서는 수행할 수 없는 재귀적인 쿼리문을 수행할 수 있다.
- WITH RECURSIVE 문은 다음과 같이 작성할 수 있다.
    
    ```sql
    WITH RECURSIVE table_name(n) AS (
    		--Non-Recursive Term    
      UNION --(혹은 UNION ALL)
        --Recursive Term
    )
    ```
    
- 비재귀항에 해당하는 쿼리는 초기 값을 설정한다. 재귀항에 해당하는 쿼리는 재귀적으로 가져오는 table_name 테이블을 조회함으로써 재귀적인 CTE를 만든다.
- 만약 1에서 100까지의 숫자를 더하는 쿼리를 수행하고자 한다면 다음과 같이 작성한다.
    
    ```sql
    WITH RECURSIVE t(n) AS (
        VALUES (1)
      UNION ALL
        SELECT n+1 FROM t WHERE n < 100
    )
    SELECT sum(n) FROM t;
    ```
    
- 일반적으로 계층 구조로 되어 있는 테이블에 대한 쿼리가 필요할 때 사용한다.