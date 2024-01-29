---
title: "[MySQL] 딜리미터와 프로시저"
excerpt: "Delimiter & Procedure"

categories:
  - MySQL
tags:
  - [sql, key]

permalink: MySQL_procedure

toc: true
toc_sticky: true

date: 2024-01-24
last_modified_at: 2024-01-24
---

## INTRO
Procedure의 정의와 특징에 대해서 알아보고 왜 DELIMITER가 필요한지 생각해보자

---

## Procedure 의미

프로시저를 한 문장으로 정의한다면 데이터베이스에서 사용되는 저장 로직이나 프로그램단위로, 절차적으로 처리되는 명령어들의 집합이라고 할 수 있다.

말이 어려울 수 있다. 간단하게 설명하자면 데이터에 처리를 위한 일련의 SQL문을 그룹화 하여 저장 후 필요할 때 호출하여 사용하는 것이다.

## 매개변수
프로시저를 호출 할 때 마다 다른 값을 전달하거나 반환이 가능해야 하기 때문에 매개변수를 통해 동적으로 가질 수 있다.

매개변수는 크게 입력 매개변수와 출력 매개변수 두 종류로 나뉜다.

### 1. IN
IN은 입력 매개변수(Input Parameters)로 프로시저에게 값을 전달하는데 사용된다.


```sql
-- 도서 정보 삽입을 위한 프로시저 정의
CREATE PROCEDURE insertBook(IN mybookid INTEGER, IN mybookname VARCHAR(40), IN mypublisher VARCHAR(40), IN myprice INTEGER)
BEGIN
    -- 도서 정보를 Book에 삽입하는 SQL
    INSERT INTO Book(bookid, bookname, publisher, price) VALUES (mybookid, mybookname, mypublisher, myprice);
END

CALL insertBook(13, '스포츠 과학', '과학사', 25000); -- 프로시저 호출
```

insertBook을 호출할 때 데이터를 입력하여 INSERT문에 값을 전달한다.


### 2. OUT
OUT은  출력 매개변수(Output Parameters)로 프로시저에서 계산한 결과나 특정 값을 반환하는데 사용된다.

```sql
-- 평균 가격을 계산하는 프로시저 정의
CREATE PROCEDURE Averageprice(OUT AverageVal INTEGER)
BEGIN
    -- 가격의 평균을 계산하고 결과를 저장하는 SQL
    SELECT AVG(price) INTO AverageVal FROM book WHERE price IS NOT NULL;
END

CALL Averageprice(@myVal); -- 프로시저 호출
SELECT @myVal; -- 출력
```

Averageprice를 호출하여 AVG를 계산한 후 반한된 값을 @myVal이라는 변수에 저장하여 해당 값을 출력해준다.
<br>

이렇게 프로시저를 정의할 때 여러 SQL문이 포함될 수 있기 때문에 보통 DELIMITER를 통해 끝나는 시점을 정의해준다.

## DELIMITER

DELIMITER는 구문 문자로 문법의 끝을 나타내는 역할을 한다.

### 필요성
일반적으로 데이터베이스에서 SQL 문장을 구분할 때 세미콜론(;)을 사용한다.

하지만 프로시저는 여러 SQL문을 포함할 수 있기 때문에 세미콜론을 바로 작성한다면 프로시저의 정의를 종료하게 되므로 구문 오류가 발생할 수 있다.

> DELIMITER를 통해 SQL문의 종결과 프로시저 정의의 종결을 명확히 구분해야 할 필요성이 있다.

코드를 통해 어떻게 사용하는지 살펴보자

```sql
-- 구문문자 변경
DELIMITER //

-- 프로시저 정의
CREATE PROCEDURE addAndPrint(IN a INT, IN b INT, OUT result INT)
BEGIN
    -- 입력받은 두 값을 더하고 결과를 출력
    SET result = a + b;
    SELECT result AS 'Sum';
END // -- 종료

-- 구문문자 변경
DELIMITER ;

-- 프로시저 호출
CALL addAndPrint(3, 5, @sum);
SELECT @sum;
```

저장 프로시저 작성을 위해 DELIMITER 문을 사용하여 문장 종결자를 기본 세미콜론(;)에서 //로 변경했다.<br>
* 여기서 //는 $$으로 작성 가능하다.

그 다음 두 개의 입력 매개변수(a, b)와 하나의 출력 매개변수(result)를 가지는 addAndPrint 저장 프로시저를 정의했다.<br>
여기서는 작성되는 SQL문은 마지막에 세미콜론(;)을 작성한다.

이렇게 프로시저를 정의한 후 다시 DELIMITER ;를 통해 문장 종결자를 다시 세미콜론(;)으로 변경했다.

이후 프로시저를 호출하고, 변수를 출력했다.

> DELIMITER를 통해 SQL문의 종결자를 변경했다.

## 결론

이렇게 DELIMITER와 프로시저를 사용하면 프로그램을 하나의 단위로 묶어서 재사용이 가능하고 성능을 최적하기 때문에 데이터베이스 작업을 효과적으로 처리할 수 있다.

---

## OUTRO
데이터베이스에 '재사용'에 대한 개념을 적용할 수 있다는 것이 놀라웠다.

프로시저를 기존에 정의해놓고 CALL을 통해서 호출을 한다면 사용자는 해당 부분만 볼 수 있기 때문에 보안적인 측면에서도 괜찮을 것 같다.

또한 자주 반복하는 작업에 대해 프로시저를 통해 정의하고 호출하여 사용하면 효율적일 것 같다.

다양한 객체를 통해 데이터베이스 시스템을 더욱 효과적으로 활용할 수 있도록 하자.