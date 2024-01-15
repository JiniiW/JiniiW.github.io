---
title: "[리팩터링] 학생별 총점 평균 stream으로 구하기 : V2"
excerpt: "Version.2"

categories:
  - SimpleProgram
tags:
  - [stream, student, v2, re]

permalink: /Refactoring/2024-01-14_Refactoring_SimpleProgram_StudentGradeWithStreamV2/

toc: true
toc_sticky: true

date: 2024-01-14
last_modified_at: 2024-01-14
---
## 기존 코드
<a href="{{url}}/Refactoring/2024-01-14_Refactoring_SimpleProgram_StudentGradeWithStreamV1/"><mark>Version1</mark></a> 


## 코드 작성
추가된 파일 : ExamResult

1.Student<br>
<a href="https://github.com/JiniiW/Code-Refactoring/blob/main/SimpleProgram/src/studentgradgewitthstream/v1/Student.java"><mark>GitHub</mark></a> 에서 확인 가능하다.

2.ExamResult<br>

```java
package studentgradgewitthstream.v2;

public class ExamResult{

    //Student student;

    private Integer totalScore;
    private Double  averageScore;

    public ExamResult(Integer totalScore, Double averageScore) {
        this.totalScore = totalScore;
        this.averageScore = averageScore;
        //calculatorScore();
    }

    public Integer getTotalScore() {
        return totalScore;
    }

    public Double getAverageScore() {
        return averageScore;
    }

      //메서드 -> main에서 구현
    /*  public void calculatorScore(){
        totalScore = student.getKorScore() + student.getEngScore() + student.getMathScore();
        averageScore = totalScore / 3.0;
   }*/
}

```

3.StudentMain

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class StudentMainV2 {

    public static void main(String[] args) {
        List<Student> stuList = new ArrayList<>();

        Scanner input = new Scanner(System.in);
        System.out.println("5명 학생의 이름과 점수를 국어 영어 수학 순서대로 입력해주세요.");

        //String 배열 선언
        for (int i = 0; i < 5; i++) {
            String[] student = input.nextLine().split(" ");
            stuList.add(new Student(student[0], Integer.parseInt(student[1]), Integer.parseInt(student[2]), Integer.parseInt(student[3])));
        }

        System.out.print("\n학생들의 이름 : ");
        stuList.stream().map(Student::getName).forEach(student -> System.out.print(student + " "));

        List<Integer> korScoreList = stuList.stream().map(Student::getKorScore).toList();
        List<Integer> engScoreList = stuList.stream().map(Student::getEngScore).toList();
        List<Integer> mathScoreList = stuList.stream().map(Student::getMathScore).toList();

        System.out.println("\n\n===== 과목별 총 점수와 평균 =====");
        int korTotal = korScoreList.stream().mapToInt(Integer::valueOf).sum();
        int engTotal = engScoreList.stream().mapToInt(Integer::valueOf).sum();
        int mathTotal = mathScoreList.stream().mapToInt(Integer::valueOf).sum();

        Double korAverage = korScoreList.stream().mapToInt(Integer::intValue).average().orElse(0.0);
        Double engAverage = engScoreList.stream().mapToInt(Integer::intValue).average().orElse(0.0);
        Double mathAverage = mathScoreList.stream().mapToInt(Integer::intValue).average().orElse(0.0);

        //출력
        System.out.printf("국어 점수 총합 : %d점, 평균 : %.2f점", korTotal, korAverage);
        System.out.printf("\n영어 점수 총합 : %d점, 평균 : %.2f점", engTotal, engAverage);
        System.out.printf("\n수학 점수 총합 : %d점, 평균 : %.2f점", mathTotal, mathAverage);

        System.out.println("\n\n========= 학생별 점수 =========");

        List<ExamResult> totalExam = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            int totalScore = (korScoreList.get(i) + engScoreList.get(i) + mathScoreList.get(i));
            double averageScore = totalScore / 3.0;
            totalExam.add(new ExamResult(totalScore, averageScore));
        }

        System.out.println("번호    이름     국어  영어  수학  총점   평균");
        for (int i = 0; i < 5; i++) {
            System.out.printf("%d번  %3s %4d %4d %4d %4d  %3.2f\n",
                    i + 1,
                    stuList.get(i).getName(),
                    korScoreList.get(i),
                    engScoreList.get(i),
                    mathScoreList.get(i),
                    totalExam.get(i).getTotalScore(),
                    totalExam.get(i).getAverageScore());
        }
    }
}
```
## 수정사항
### 1. String[] 선언
처음에 리펙터링을 할 때 5명 학생의 점수를 입력받는 부분이 중복되는게 많아서 줄이고 싶었다.<br>
그래서 String 배열로 이름부터 점수까지 한번에 문자열로 입력받고 띄어쓰기를 기준으로 split을 하여 인데스에 값을 하나씩 넣어주었다.<br>
그 후 리스트에 추가할 때 정수형으로 들어가야 하는 문자들은 `Integer.parseInt`를 통해 변환해주었다.<br>
확실히 배열을 사용하여 전보다 코드가 더 간결해졌다.

### 2. ExamResult객체 추가
요구조건에서 totalExam 리스트에 학생들의 점수 총합과 평균값을 따로 추가해달라고 했다.<br>
그래서 처음에는 Student 객체에 total과 avg 필드를 추가하려 했지만 의도와는 맞지 않는것 같아 객체를 새로 생성해 구조화 했다.
**메서드 추가 X**<br>
ExamResult를 만든 후에는 Student 객체를 선언해 메서드를 바로 구현해서 생성자를 통해 총점과 평균을 구하려했다. 하지만 Student 객체를 새로 생성하는 작업 자체가 불필하게 느껴져서 Main에서 변수를 선언하여 계산했다.

### 3. 출력
기존에는 List하나에 모든 값을 String으로 저장하여 forEach로 출력했지만 수정 후에는 각각의 리스트의 값을 getter 메서드를 통해 접근하여 형식화된 출력을 구현했다.

## 느낀점
리팩터링을 진행하며 코드가 이전에 비해 중복성이 감소하고 가독성이 향상되었다는 것을 확인하니 뿌듯하다.<br> 
기존 코드에 비해 수정된 코드에서는 이미 존재하는 메서드들을 효과적으로 활용할 수 있어서 작업이 좀 더 편리하게 이루어진 것 같다.<br> 그러나 아직도 코드 중복이 남아있어서, 이 부분을 메서드로 추출하고 최종적으로 리팩터링을 진행해야 겠다.




## 리펙터링 할 것
- 중복 부분 메서드 따로 분리