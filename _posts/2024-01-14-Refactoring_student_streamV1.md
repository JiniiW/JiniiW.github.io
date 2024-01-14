---
title: "[리팩터링] 학생별 총점 평균 stream으로 구하기 : V1"
excerpt: "Version.1"

categories:
  - SimpleProgram
tags:
  - [stream, student, v1, re]

permalink: /Refactoring/2024-01-14_Refactoring_SimpleProgram_StudentGradeWithStreamV1/

toc: true
toc_sticky: true

date: 2024-01-14
last_modified_at: 2024-01-14
---
## 프로그램 설명
학생들의 이름, 국어, 영어, 수학 점수를 입력받아 stream을 사용하여 총점, 평균 구하기
Student 객체 생성, StudentMain 작성

## 요구조건
**stream** 사용
1. 학생 정보를 저장하기 위한 List 선언 : stuList
2. 5명의 이름, 국어, 영어, 수학 점수 입력받아 stuList에 추가
3. 입력받은 학생들의 이름 출력
4. 학생들의 국어 점수 List에 저장 : korScoreList
5. 학생들의 국어 점수 총합과 평균 출력
6. 학색들의 영어 점수 List에 저장 : engScoreList
7. 학생들의 영어 점수 총합과 평균 출력
8. 학색들의 수학 점수 List에 저장 : MathScoreList
9. 학생들의 수학 점수 총합과 평균 출력
10. 학생별 과목 총점과 평균을 totalExam에 저장
11. 출력

<img src = "{{url}}/images/2024-01-14-Refactoring_StudentStream/실행결과.png" width="50%">


## 코드 작성
1. Student<br>
<a href="https://github.com/JiniiW/Code-Refactoring/blob/main/SimpleProgram/src/studentgradgewitthstream/v1/Student.java"><mark>GitHub</mark></a> 에서 확인 가능하다.

2. StudentMain

```java

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class StudentMain {

    public static void main(String[] args) {
        
        List<Student> stuList = new ArrayList<>();

        
        Scanner input = new Scanner(System.in);
        System.out.println("5명 학생의 이름과 점수를 국어 영어 수학 순서대로 입력해주세요.");
        for (int i = 0; i < 5; i++) {
            String stuName = input.next();
            Integer korScore = input.nextInt();
            Integer mathScore = input.nextInt();
            Integer engScore = input.nextInt();

            input.nextLine(); //추가

            stuList.add(new Student(stuName, korScore, mathScore, engScore));

        }

        
        System.out.print("\n학생들의 이름 : ");
        stuList.stream().map(Student::getName).forEach(student -> System.out.print(student + " "));

        System.out.println("\n\n===== 과목별 총점수와 평균 =====");
        List<Integer> studentsKorScore = stuList.stream().map(Student::getKorScore).toList();
        int korTotal = studentsKorScore.stream().mapToInt(Integer::intValue).sum();
        Double korAverage = studentsKorScore.stream().mapToInt(Integer::intValue).average().orElse(0.0);
        System.out.printf(" 국어 점수 총합 : %d점 평균 : %.2f점", korTotal, korAverage);


        List<Integer> studentsEngScore = stuList.stream().map(Student::getEngScore).toList();
        int engTotal = studentsEngScore.stream().mapToInt(Integer::intValue).sum();
        Double engAverage = studentsEngScore.stream().mapToInt(Integer::intValue).average().orElse(0.0);
        System.out.printf("\n 영어 점수 총합 : %d점 평균 : %.2f점", engTotal, engAverage);

        List<Integer> studentsMathScore = stuList.stream().map(Student::getMathScore).toList();
        int mathTotal = studentsMathScore.stream().mapToInt(Integer::intValue).sum();
        Double mathAverage = studentsMathScore.stream().mapToInt(Integer::intValue).average().orElse(0.0);
        System.out.printf("\n 수학 점수 총합 : %d점 평균 : %.2f점", mathTotal, mathAverage);

        //toatlExam에 String으로 모두 저장
        List<String> totalExam = stuList.stream().map(student -> (stuList.indexOf(student) + 1) + "번  " + student.getName() + "   " + student.getKorScore() + "   " + student.getEngScore() + "   " + student.getMathScore()
                + "   " + (student.getKorScore() + student.getEngScore() + student.getMathScore())
                + "   " + ((student.getKorScore() + student.getEngScore() + student.getMathScore()) / 3.0)).toList();

        System.out.println("\n\n========= 학생별 점수 =========");
        System.out.println("번호    이름     국어  수학  영어   총점   평균");
        totalExam.stream().forEach(System.out::println);
    }
}

```
## 느낀점
코드를 작성할 때 2가지의 궁금증이 생겼다.
### 1. 개행문자
처음 학생 정보를 5명 입력받을 때 for문을 사용해서 Scanner를 통해 입력받았다.<br>
그런데 2번째 학생을 입력받을 때 InputMismatchException이 계속 발생했다.<br>
<img src = "{{url}}/images/2024-01-14-Refactoring_StudentStream/InputMismatchExecption.png" width="70%"><br>
찾아보니 nextInt()를 입력받고 난 후에 입력 버퍼에서 개행 문자를 소비하지 않고 남아있기 때문에 예외가 발생했을 수 있다고 한다.<br>
그래서 이름과 점수를 입력받고 난 후 제일 마지막에 input.nextLine()을 추가하였다.<br>
평소에 생각지도 못한 개행문자 때문에 예외가 발생해서 원인을 찾느라 좀 힘들었다. <br>
앞으로 연속해서 입력받는 상황이 생기면 nextLine()을 통해 입력버퍼를 비워서 사용해야 겠다.

### 2. map()과 mapToInt()
처음에 학생들의 국어점수만 따로 List에 작성할때 mapToInt()를 사용해서 List에 값을 저장했는데 map과 달리 boxed()의 과정을 중간에 추가해주어야 했다.<br>
```java
List<Integer> studentsKorScore = stuList.stream().map(Student::getKorScore).toList(); 
List<Integer> studentsKorScore2 = stuList.stream().mapToInt(Student::getKorScore).boxed().toList(); 
```
때문에 과정이 하나 더 적은 map()을 작성하면 어떨까? 라는 생각으로 사용했다.<br>

하지만 확실하게 개념을 알고 작성하고 싶어서 둘의 차이점에 대해 알아봤다.<br>
map()은 객체를 반환하고 mapToInt()는 int타입을 반환하기 때문에 차이가 나는 것이였다.<br>
평소에는 map()을 사용하고, int의 값을 직접적으로 필요할 때 mapToInt()를 사용해야 겠다.

이 외에 한 가지 만족스럽지 못한 부분이 있다.
### 3. totalExam
요구조건에서 totalExam에 학생별 과목 총점과 평균만 저장 하라고 했다.<br>
그래서 처음에는 total과 average만 저장하려고 했는데 List의 타입을 뭘로 정할지 모르겠어서 요구조건을 만족시키지 못하고 String으로 선언해서 학생의 정보를 모두 작성했다.

후에 다른 사람들이 작성한 코드를 보니까 Student 객체에 총점과 평균의 필드를 따로 추가해주거나 객체를 새로 생성한 경우도 있었다.<br>
나는 왜 그런 생각을 하지 못했을까?<br>
이 부분은 리펙터링을 통해 수정해야 겠다.


## 리펙터링 할 것
- 처음 학생별 3과목 점수를 입력받을 때 배열로 받아보자
- 과목별 점수 총합과 평균을 출력하는 부분만 따로 모아서 한번에 출력하자
- totalExam에 저장하고 출력하는 방식을 개선하자