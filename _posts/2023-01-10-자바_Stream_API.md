---
title: "[JAVA] 스트림 API 연산과 내부 반복자 "
excerpt: "Stream API"

categories:
  - Java
tags:
  - [stream]

permalink: /java/2024-01-10-JAVA_Stream_API/

toc: true
toc_sticky: true

date: 2024-01-10
last_modified_at: 2024-01-10
---
# Intro
함수형 프로그래밍 방식에서 스트림 API를 활용한 다양한 연산을 실습해보며 내부 연산자와 외부 연산자를 같이 비교해보자.

---

스트림 API의 연산의 종류는 중간 연산과 최종 연산으로 나눌 수 있다.

<br>

# 1. 중간 연산
> 스트림의 연산 결과가 스트림이며 중간연산을 여러번 연결하여 파이프라인 구성이 가능하다.

- 대표적 : filter(), map(), sorted()

#### 1. filter()
> 조건에 맞는 요소 추출

```java
public class Filter {
    public static void main(String[] args) {
        
        int[] array = {1,5,3,2,4};

        //1. IntStream으로 홀수만 필터링
        IntStream intStream = Arrays.stream(array).filter(number -> number % 2 != 0);
        //2. Stream<Integer>로 홀수만 필터링
        Stream<Integer> integerStream = Arrays.stream(array).filter(number -> number % 2 != 0).boxed();
    }
}
```
##### IntStream VS Stream&lt;Integer&gt;
여기서 IntStream과 Stream&lt;Integer&gt;의 차이가 궁금할 것이다.

우선 두 개 모두 Stream API에서 제공되는 형태의 스트림이지만, 내부 요소 타입이 다르다.

IntStream은 기본타입인 int의 스트림을 나타내는 인터페이스다.<br>
때문에 데이터의 처리 요소가 기본 타입인 경우 사용한다.

Stream&lt;Integer&gt;는 객체 타입인 Integer의 스트림을 나타내는 인터페이스다.<br>
객체 타입의 데이터 처리가 필요한 경우에 사용한다.<br>

현재 작성한 코드는 int형 배열의 요소를 필터링 하기 때문에 IntStream을 사용해야 한다.
만약 IntStream을 Stream&lt;Integer&gt;로 변환하는 경우에는 boxed()메서드를 사용해야 한다.


##### 데이터 처리 상태
현재 IntStream객체에는 int값들이 들어가 있는 상태지만 이를 배열로 간주하지 않는다.

즉, IntStream은 배열이 아닌 스트링 형태로 데이터를 처리하기 때문에 필터링 연산을 수행할 수 있는 것이다.

만약 배열로 변환하고 싶으면 toArray() 메서드를 사용해야 한다.

intStream, integerStream 출력 : **<U>1 5 3</U>** 
{: .notice--danger}

#### 2. sorted()
> 기본 정렬, 사용자 정의 정렬

```java
public class Sorted {
    public static void main(String[] args) {
        
        List<String> stringList = Arrays.asList("엄마는 외계인", "베리베리 스트로베리", "아몬드 봉봉", "슈팅스타");

        //1. 기본 오름차순 정렬
        Stream<String> stringStream1 = stringList.stream().sorted();
        //2. 사용자 정의 정렬 : 문자열 길이
        Stream<String> stringStream2 = stringList.stream().sorted(Comparator.comparing(String::length));
    }
}
```
##### 기본 정렬
sorted()메서드를 사용 시 Comparator을 지정하지 않으면 자동으로 Comparable을 기준으로 정렬한다.<br>
Comparable은 모든 기본타입의 래퍼클래스에서 오름차순으로 구현되어 있다.

stringStream1 출력 : **<U>ㄱ,ㄴ,ㄷ 순</U>**
{: .notice--danger}

##### 사용자 정의 정렬
sorted()에 Comparator 클래스의 comparing 메서드를 사용하면서 String의 length를 기준으로 오름차순 정렬을 했다.
  - `String::length` -> 문자열의 길이를 반환하는 메서드 참조

stringStream2 출력 : **<U>문자열이 짧은 순</U>**
{: .notice--danger}

<br>
**Comparator.comparing**<br>
그러면 여기서 궁금한 점이 발생할 수 있다.<br>
원래 Comparator 인터페이스를 직접 구현하여 compare메서드를 정의해야 하지만 여기서는 comparing 메서드를 사용하였다.<br>
그 이유는 comparing() 메서드가 내부적으로 해당 키를 추출하는 방식으로 처리하기 때문이다.

>comparing() : 특정 키를 추출하고 그 키를 기반으로 정렬하는데 사용한다.

#### 3. Map()
> 조건에 맞는 요소 반환

```java
public class Map {
    public static void main(String[] args) {

        List<String> list = Arrays.asList("apple", "pear", "orange", "banana", "tomato");
        
        //문자열 대문자 변경
        Stream<String> stringStream = list.stream().map(String :: toUpperCase);
    }
}
```
list의 있는 각각의 요소를 모두 대문자로 변경하여 반환했다.

stringStream 출력 : **<U>APPLE PEAR ORANGE BANANA TOMATO</U>**
{: .notice--danger}

<br>

### 2. 최종 연산
> 스트림의 연산결과가 스트림이 아니며, 스트림 파이프라인의 끝에 위치하며 스트림 요소를 소모한다.

- 연산 결과 : 단일값, 배열, 컬렉션 등
- 대표적 : forEach(), of(), reduce

#### 1. forEach()

> 요소를 하나씩 꺼낸다.

```java
public class ForEach {
    public static void main(String[] args) {
        int[] array = {1,5,3,2,4};

        //배열을 정렬하여 출력
        Arrays.stream(array).sorted().forEach(System.out::println);
    }
}
```
배열의 요소를 정렬한 후 바로 출력해주기 위해 스트림 객체에 저장하지 않고 요소 하나하나를 바로 출력해주었다.

여기서 sorted()는 중간연산이고, forEach()는 최종연산이다.

출력 : **<U>1 2 3 4 5</U>**
{: .notice--danger}

#### 2. of()

> 여러 입력 스트림 생성

```java
public class Of {
    public static void main(String[] args) {
        //1부터 5의 정수 스트림 생성하고 출력
        IntStream.of(1,2,3,4,5).forEach(System.out::println);
    }
}
```

of() 메서드를 사용하여 1부터 5까지의 정수로 이루어진 스트림을 생성하여 요소를 출력해주었다.

출력 : **<U>1 2 3 4 5</U>**
{: .notice--danger}

##### 주의

IntStream은 정수값만 입력할 수 있으므로 컬렉션은 들어갈 수 없다.

컬렉션은 객체를 다루기 때문에 Stream을 사용해야 한다.

그래도 IntStream에 값을 입력하려면 컬렉션을 mapToInt()를 통해 정수 값으로 바꿔주고 toArray()로 배열 형태를 만들어서 스트림을 생성할 수 있다.

```java
List<Integer> numbers = Arrays.asList(1,2,3,4,5,6,77,88,99);
IntStream.of(numbers.stream().mapToInt(Integer::intValue).toArray());
```

- `Integer::intValue` -> Integer 객체를 int로 변환하는 메서드 참조

#### 3. reduce()

> 스트림의 모든 요소 조합하여 단일 결과값 생성

```java
public class Reduce {
    public static void main(String[] args) {

        List<Integer> numbers = Arrays.asList(1,2,3,4,5);
        
        //1. 누적합 출력
        int total = numbers.stream().reduce(0, Integer::sum);
        //2. 누적곱 출력
        int multiply = numbers.stream().reduce(1, ((num1, num2) -> num1 * num2));
    }
}
```
reduce()는 초기값을 지정하고 연산을 수행하는데 합과 곱에 맞춰 초기값을 알맞게 부여해야 한다.

요소를 조함하는 방식은 (초기값, **이항연산자**)를 사용하여 순차적으로 조합한다.

출력 : **<U>15 120</U>**
{: .notice--danger}

<br>

### 추가

이러한 메서들 말고도 더 많은 스트림 메서드가 존재한다.

- 중간연산 : distinct(), limit()
- 최종연산 : count(), sum(), average(), min(), max(), joining()

<br>

### 3. 내부반복자

지금까지 다양한 스트림 API 메서드를 활용하면서 데이터를 처리하는 방법을 배웠다.

이렇게 메서드들을 활용하여 연산이 컬렉션 내부에서 요소를 반복하는 작업을 내부적으로 처리한다고 의미한다.

>내부 반복자 : 컬렉션 내부에서 반복 작업을 처리하는 반복자

<img src = "{{url}}/images/2024-01-10-Java_StreamAPI/내부반복자.png" width = "60%">

풀어서 설명해보자면 사용자가 직접 반복 작업을 제어하지 않고 스트림 API가 컬렉션의 요소를 처리하는데 사용되는 것이다.

기존의 외부 반복자와 비교해서 코드를 보자

```java
List<String> stringList = Arrays.asList("apple", "orange", "banana");

// 내부 반복자 사용
stringList.stream().forEach(System.out::println);

// 외부 반복자 사용
for (String fruit : stringList) {
    System.out.println(fruit);
}
```

내부 반복자는 직접 루프문을 작성하지 않고 간결한 코드로 요소를 처리할 수 있지만, 외부 반복자는 직접 루프를 작성해야 한다.

때문에 내부 반복자를 사용하는 스트림은 코드가 간결해지고 처리속도가 빠르기 때문에 <mark>병렬처리</mark>에 효율적이다.<br>
또한 <mark>람다식</mark>을 활용하여 다양한 요소 처리를 정의하고, 이를 통해 중간 처리와 최종 처리를 수행하는 <mark>파이프라인</mark>을 형성하는 특징을 가진다.

---

### Outro

요약해보자면 함수형 프로그래밍에서 스트림 API를 통해 데이터 처리를 간결하게 표현할 수 있다.

현재는 실습을 할 때 함수형 인터페이스를 사용하지 않았지만 앞으로는 기존에 구현되어 있는 인터페이스를 사용하여 람다 표현식을 통해 간편하게 구현해야 겠다.

또한 앞으로 스트림 API를 사용할 때 중간연산과 최종연산의 개념이 확실히 와닿아 파이프라인을 헷갈리지 않고 잘 구현할 수 있을 것 같다.
