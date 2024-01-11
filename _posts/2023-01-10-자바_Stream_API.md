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
함수형 프로그래밍 방식에서 스트림 API를 활용한 다양한 연산을 할 수 있다.<br>
하지만 너무 많은 메서드가 존재해 파이프라인을 작성할 때 마다 멈칫한다.<br>
스트림 연산의 대표적인 터링(filtering), 매핑(mapping), 정렬(sorting), 집계(Aggregation)기능을 중점으로 공부해보자.

---

스트림 API의 연산의 종류는 중간 연산과 최종 연산으로 나눌 수 있다.

<br>

# 1. 중간 연산
> 스트림의 연산 결과가 스트림이며 중간연산을 여러번 연결하여 파이프라인 구성이 가능하다.

- 대표적 : filter(), map(), sorted()

#### 1. 필터링 : filter()
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

현재 IntStream객체에는 int값들이 들어가 있는 상태지만 이를 배열로 간주하지 않는다.

즉, IntStream은 배열이 아닌 스트링 형태로 데이터를 처리하기 때문에 필터링 연산을 수행할 수 있는 것이다.

만약 배열로 변환하고 싶으면 toArray() 메서드를 사용해야 한다.

intStream, integerStream 출력 : **<U>1 5 3</U>** 
{: .notice--danger}


##### IntStream VS Stream&lt;Integer&gt;
여기서 IntStream과 Stream&lt;Integer&gt;의 차이가 궁금할 것이다.

우선 두 개 모두 Stream API에서 제공되는 형태의 스트림이지만, 내부 요소 타입이 다르다.

<span style = "color : red">IntStream</span>은 기본타입인 int의 스트림을 나타내는 인터페이스다.<br>
때문에 데이터의 처리 요소가 <span style = "color : red">기본 타입</span>인 경우 사용한다.

<span style = "color : red">Stream&lt;Integer&gt;</span>는 객체 타입인 Integer의 스트림을 나타내는 인터페이스다.<br>
<span style = "color : red">객체 타입</span>의 데이터 처리가 필요한 경우에 사용한다.<br>

현재 작성한 코드는 int형 배열의 요소를 필터링 하기 때문에 IntStream을 사용해야 한다.
만약 <mark>IntStream을 Stream&lt;Integer&gt;로 변환하는 경우에는 boxed()메서드</mark>를 사용해야 한다.

> **IntStream** : 기본형(int) / **Stream&lt;Integer&gt;** : 객체(Integer)



#### 2. 정렬 : sorted()
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
sorted()에 Comparator 클래스의 comparing() 메서드를 사용하면서 String의 length를 기준으로 오름차순 정렬을 했다.
  - `String::length` -> 문자열의 길이를 반환하는 메서드 참조

stringStream2 출력 : **<U>문자열이 짧은 순</U>**
{: .notice--danger}

<br>
**comparing() VS compare()**<br>
그러면 여기서 궁금한 점이 발생할 수 있다.<br>
원래 Comparator 인터페이스를 직접 구현하여 compare()메서드를 정의해야 하지만 여기서는 comparing() 메서드를 사용하였다.<br>

그 이유는 comparing() 메서드가 내부적으로 해당 키를 추출하는 방식으로 처리하기 때문이다.<br>
compare()는 구현할 때 외부에서 두 객체를 받아와 직접 비교해야하지만 comparing()은 람다 표현식을 통해 키를 추출한다.


#### 3. 매핑 : Map()
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

추가로 mapToInt(), asDoubleStream(), boxed, flatMapInt() 등이 존재한다.
<br>

### 2. 최종 연산
> 스트림의 연산결과가 스트림이 아니며, 스트림 파이프라인의 끝에 위치하며 스트림 요소를 소모한다.

- 연산 결과 : 단일값, 배열, 컬렉션 등
- 대표적 : forEach(), of(), reduce

#### 1. 루핑 : forEach()

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


**forEach() VS peek()**<br>
forEach()는 최종 연산으로 요소을 소모한다.<br>
하지만 peek()는 중간연산으로 요소를 소모하지 않는다.<br>
때문에 peek()은 중간에 결과값을 확인할 수 있는 디버깅 용으로 많이 사용한다. -> filter()나 map()의 결과를 확인할 때 유용하다.


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

##### 컬렉션 요소 주입

```java
List<Integer> numbers = Arrays.asList(1,2,3,4,5,6,77,88,99);
IntStream.of(numbers.stream().mapToInt(Integer::intValue).toArray());
```

- `Integer::intValue` -> Integer 객체를 int로 변환하는 메서드 참조

IntStream.of()에 list 컬렉션의 요소로 스트림을 생성하고 싶다면 list의 요소를 먼저 기본형으로 변환해주어야 한다.
-> `numbers.stream().mapToInt(Integer::intValue)`

이렇게 기본형으로 변환된 요소는 현재 IntStream 타입이다.

하지만 IntStream.of()에는 객체 자체를 전달할 수 없기 때문에 toArray를 통해 int형 배열로 변경하여 값을 전달해야 한다.

IntStream.of()는 기본형 int 값을 직접 전달받아 새로운 IntStream을 생성하는 목적을 가지고 있기 때문에 객체 자체를 전달하는 것은 의도와 맞지 않다.
{: .notice}

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

#### 4. 집계 : collect()

> 스트림이 요소를 수집하는 방법에 대한 정의

collect()메서드는 Collector 인터페이스를 구현한 것으로, Collectors 클래스는 다양한 종류의 메서드를 가지고 있다.

용어들을 간단하게 정리해보자<br>
- collect() : 스트림의 최종연산으로 요소를 수집하며, 수집하는 방식은 Collector 인터페이스를 통해 정의
- Collector : 인터페이스, collect()가 구현
- Collectors : 클래스, Collector 인터페이스를 구현한 static 메서드를 제공

정리하자면 collect()메서드는 매개변수 타입이 Collector인데, 매개변수가 Collector를 구현한 클래스 객체여야 한다.

**이 문장이 쉽게 이해되지 않으므로 예시를 들어보겠다.<br>**
일반적으로 가장 많이 사용하는 수집 방식중 하나는 Collectors.toList()이다.<br>
해당 방식은 toList()가 List를 반환하는 Collector를 생성하는 것이다.
즉, collect(Collectors.toList())는 스트림의 각 요소를 리스트에 추가하고 최종적으로 리스트를 반환한다.

> 따라서 collect()메서드의 매개변수로는 이러한 수집 기능을 제공하는 Collector 인터페이스를 구현한 클래스의 객체가 전달되어야 한다는 의미이다.

**collect(Collectors.toList()) VS toList()**<br>
해당 메서드는 기능적으로 동일하다.<br>
collect(Collectors.toList())를 간결하게 사용할 수 있도록 Collectors 클래스에 추가된 메서드가 toList()인 것이다.

Collectors의 다양한 메서드는<a href="https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/Collectors.html#method-detail"><mark>JAVA API문서</mark></a>를 찾아보면 된다.


<br>

### 추가

이러한 메서들 말고도 더 많은 스트림 메서드가 존재한다.

- 중간연산 : distinct(), limit(), peek()
- 최종연산 : count(), sum(), average(), min(), max(), joining(), allMatch()

<br>

### 3. 내부반복자

지금까지 다양한 스트림 API 메서드를 활용하면서 데이터를 처리하는 방법을 배웠다.

이렇게 메서드들을 활용하여 연산이 컬렉션 내부에서 요소를 반복하는 작업을 내부적으로 처리한다고 의미한다.

>내부 반복자 : 컬렉션 내부에서 반복 작업을 처리하는 반복자

<img src = "{{url}}/images/2024-01-10-Java_StreamAPI/내부반복자.png" width = "60%">

풀어서 설명해보자면 사용자가 직접 반복 작업을 제어하지 않고 스트림 API가 컬렉션의 요소를 처리하는 것이다.

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

요약해보자면 함수형 프로그래밍은 스트림 API를 통해 데이터 처리를 간결하게 표현할 수 있다.

현재는 실습을 할 때 함수형 인터페이스를 사용하지 않았지만 앞으로는 기존에 구현되어 있는 인터페이스를 사용하여 람다 표현식을 통해 간편하게 구현해야 겠다.

중간연산과 최종연산의 개념이 확실히 와닿아 파이프라인을 헷갈리지 않고 목적에 맞게 구현할 수 있을 것 같다.
