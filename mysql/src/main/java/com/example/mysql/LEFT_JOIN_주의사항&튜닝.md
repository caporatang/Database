## LEFT JOIN 주의사항 & 튜닝
대표적으로 사용되는 JOIN은 INNER JOIN, LEFT JOIN이 있는데, INNER JOIN은 교집합, LEFT 조인은 교집합과 왼쪽 테이블에 있는 데이터의 결과가 반환된다.  

## LEFT JOIN 예시
````sql
CREATE TABLE user (
    id int NOT NULL AUTO_INCREMENT,
    name varchar(20) NOT NULL,
    PRIMARY KEY (id)
);

CREATE TABLE user_coupon (
    user_id int NOT NULL ,
    coupon_id int NOT NULL,
    PRIMARY KEY (user_id, coupon_id)
    KEY ix_couponid (coupon_id)
);


/* 전체 유저 목록을 조회하는데, 3번 쿠폰을 가진 유저들에 대해서는 쿠폰 사용 여부를 같이 조회하는 경우 */
SELECT u.id, uc.coupon_id, uc.use_yn
FROM user u
LEFT JOIN user_coupon uc ON uc.user_id=u.id AND uc.coupon_id=3;
````
위와 같은 예시와 다르게 LEFT JOIN이 INNER JOIN처럼 동작하는 경우도 있는데,  
user 테이블에 데이터가 30000건, user_coupon 테이블에 데이터가 3000건, coupon_id=1 .. 2 .. 3 이런식으로 적재되어 있을때 예시 쿼리와 동일하게 실행하면 30000건의 데이터가 반환된다.  
````sql
/* -> 30000건 조회 */
SELECT u.id, uc.coupon_id, uc.use_yn
FROM user u
LEFT JOIN user_coupon uc ON uc.user_id=u.id AND uc.coupon_id=3;

/* 일단 30000건이 조회되는 이유는 WHERE 조건에 필터링 하는 조건이 없어서이다. */
/* where절에 조건을 추가하게 되면 조회결과는 3000건만 반환된다 */
SELECT u.id, uc.coupon_id, uc.use_yn
FROM user u
LEFT JOIN user_coupon uc ON uc.user_id=u.id
WHERE uc.coupon_id=3;
````
조건이 붙는 위치에 따라 조건이 수행하는 역할이 다르기 때문에 건수 차이가 발생한다. ON절에 조건이 부여됐을때는  user 테이블에 대해서 user_coupon 테이블을 연결하는 역할을 수행했지만, where절에 조건을 부여했을때는 실제 반환되는 데이터를 필터링 하는 역할을 수행했기 때문이다.  
즉, user_coupon_id가 3인 데이터를 조회하게 되므로 inner join과 동일한 결과가 출력된다고 볼 수 있다.  
그리고 mysql에서는 바로 위 쿼리처럼 LEFT JOIN에 WHERE절에 필터링을 사용하는 경우에 옵티마이저가 쿼리 최적화를 하면서 INNER JOIN으로 쿼리를 자동으로 변경하기 때문에 주의가 필요하다.  

## COUNT(*) with LEFT JOIN
LEFT JOIN 자체가 왼쪽에 있는 테이블의 데이터를 모두 반환하기 떄문에, 굳이 LEFT JOIN을 사용할 필요가 없다. 어차피 결과는 동일하기 떄문이다. 굳이 LEFT JOIN을 붙여서 쿼리 실행 속도를 늦출 필요없다.  

## 정리
1. LEFT JOIN을 사용하고자 한다면 Driven Table(Inner Table) 컬럼의 조건(조인 조건)은 반드시 ON절에 명시해서 사용해야 한다.(IS NULL 조건은 예외다.)  
2. LEFT JOIN과 INNER JOIN은 결과 데이터 및 쿼리 처리 방식 등이 매우 다르므로, 필요에 맞게 올바르게 사용하는 것이 중요하다.
3. LEFT JOIN 쿼리에서 COUNT를 사용하는 경우 LEFT JOIN이 굳이 필요하지 않다면 JOIN은 제거한다. 
