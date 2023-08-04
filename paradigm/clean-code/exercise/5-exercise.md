# 5장: 형식 맞추기

## 적절한 행 길이를 유지하라
* 대부분의 모듈은 100줄을 넘어가지 않았다.

### 신문 기사처럼 작성하라
* 소스 파일은 신문 기사와 비슷하게 작성한다. 이름은 간단하면서도 설명이 가능하게 짓는다. 이름만 보고도 올바른 모듈을 살펴보고 있는지 아닌지를 판단할 정도로 신경 써서 짓는다.
* 소스 파일의 첫 부분은 고차원 개념과 알고리즘을 설명한다. 아래로 내려갈수록 의도를 세세하게 묘사한다. 마지막에는 가장 저차원 함수와 세부 내역이 나온다.

### 개념은 빈 행으로 분리하라
* 거의 모든 코드는 왼쪽에서 오른쪽으로, 그리고 위에서 아래로 읽힌다. 각 행은 수식이나 절을 나타내고, 일련의 행 묶음은 완결된 생각 하나를 표현한다. 생각 사이는 빈 행을 넣어 분리해야 마땅하다.
* 빈 행은 새로운 개념을 시작한다는 시각적 단서이다.
* 빈 행이 있을때
```java
package fitnesse.wikitext.widgets;

import java.util.regex.*;

public class BoldWidget extends ParentWidget {
  public static final String REGEXP = "'''.+?'''";
  private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
      Pattern.MULTILINE + Pattern.DOTALL);

  public BoldWidget(ParentWidget parent, String text) throws Exception {
    super(parent);
    Matcher match = pattern.matcher(text);
    match.find();
    addChildWidgets(match.group(1));
  }

  public String render() throws Exception {
    StringBuffer html = new StringBuffer("<b>");
    html.append(childHtml()).append("</b>");
    return html.toString();
  }
}
```
* 빈 행을 없애면? 코드 가독성이 떨어져 암호처럼 보인다.
```java
package fitnesse.wikitext.widgets;
import java.util.regex.*;
public class BoldWidget extends ParentWidget {
  public static final String REGEXP = "'''.+?'''";
  private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
      Pattern.MULTILINE + Pattern.DOTALL);
  public BoldWidget(ParentWidget parent, String text) throws Exception {
    super(parent);
    Matcher match = pattern.matcher(text);
    match.find();
    addChildWidgets(match.group(1));}
  public String render() throws Exception {
    StringBuffer html = new StringBuffer("<b>");
    html.append(childHtml()).append("</b>");
    return html.toString();
  }
}
```

### 세로 밀집도
* 줄바꿈이 개념을 분리한다면 세로 밀집도는 연관성을 의미한다. 즉, 서로 밀접한 코드 행은 세로로 가까이 놓여야 한다는 뜻이다.

### 수직 거리
서로 밀접한 개념은 세로로 가까이 둬야 한다. 단, 두 개념이 서로 다른 파일에 속한다면 규칙이 통하지 않는다.

* 변수 선언
  - 변수는 사용하는 위치에 최대한 가까이 선언한다.
  - 짧은 함수의 경우 지역 변수는 각 함수 맨 처음에 선언한다.
    ```java
    private static void readPreferences() {
      InputStream is = null;
      try {
        is = new FileInputStream(getPreferencesFile());
        setPreferences(new Properties(getPreferences()));
        getPreferences().load(is);
      } catch (IOException e) {
        try {
          if (is != null)
            is.close();
        } catch (IOException e1) {
        }
      }
    }
    ```
  - 루프를 제어하는 변수는 흔히 루프 문 내부에 선언한다.
  - 드물지만 다소 긴 함수에서 블록 상단이나 루프 직전에 변수를 선언하는 사례도 있다.
* 인스턴스 변수
  - 인스턴스 변수는 클래스 맨 처음에 선언한다.
  - 변수 간에 세로로 거리르 두지 않는다. 잘 설계한 클래스는 많은 (혹은 대다수) 클래스 메서드가 인스턴스 변수를 사용하기 때문이다.
* 종속 함수
  - 한 함수가 다른 함수를 호출한다면 두 함수는 세로로 가까이 배치한다. 가능하다면 호출하는 함수를 호출되는 함수보다 먼저 배치한다.
  - 규칙을 일관적으로 적용한다면 독자는 방금 호출한 함수가 잠시 후에 정의되리라는 사실을 예측한다.
* 개념적 유사성
  - 개념적인 친화도가 높을수록 코드를 가까이 배치한다.
  - 한 함수가 다른 함수를 호출해 생기는 직접적인 종속성, 변수와 그 변수를 사용하는 함수, 비슷한 동작을 수행하는 일군의 함수 등

### 세로 순서
* 일반적으로 함수 호출 종속성은 아래 방향으로 유지한다. 다시 말해, 호출되는 함수를 함수보다 나중에 배치한다.
* 신문 기사와 마찬가지로 가장 중요한 개념을 가장 먼저 표현한다. 가장 중요한 개념을 표현할 때는 세세한 사항을 최대한 배체잫ㄴ다.

## 가로 형식 맞추기
* 프로그래머는 짧은 행을 선호한다. (저자 개인적으로는 120자 이하를 추천)

### 가로 공백과 밀집도
* 가로로는 공백을 사용해 밀접한 개념과 느슨한 개념을 표현한다.
  * 예제 코드
    ```java
    private void measureLine(String line) {
      lineCount++;
      int lineSize = line.length();
      totalChars += lineSize;
      lineWidthHistogram.addLine(lineSize, lineCount);
      recordWidestLine(lineSize);
    }
    ```
  * 할당 연산자를 강조하려고 앞뒤에 공백을 줬다. 할당문은 왼쪽 요소와 오른쪽 요소가 분명히 나뉜다. 공백을 넣으면 두 가지 주요 요소가 확실히 나뉜다는 사실이 더욱 분명해진다.
  * 함수 이름과 이어지는 괄호 사이에는 공백을 넣지 않았다. 함수와 인수는 서로 밀접하기 때문이다.
* 연산자 우선순위를 강조하기 위해서도 공백을 사용한다.
  * 예제 코드
    ```java
    public class Quadratic {
      public static double root1(double a, double b, double c) {
        double determinant = determinant(a, b, c);
        return (-b + Math.sqrt(determinant)) / (2*a);
      }

      public static double root2(int a, int b, int c) {
        double determinant = determinant(a, b, c);
        return (-b - Math.sqrt(determinant)) / (2*a);
      }

      private static double determinant(double a, double b, double c) {
        return b*b - 4*a*c;
      }
    }
    ```
  * 승수 사이에는 공백이 없다. 곱셈은 우선순위가 가장 높기 때문이다.
  * 항 사이에는 공백이 들어간다. 덧셈과 뺄셈은 우선순위가 곱셈보다 낮기 때문이다.
  * 참고: 코드 포매터는 대다수가 연산자 우선순위를 고려하지 못한다. 수식에 똑같은 간격을 적용한다.

### 가로 정렬
* 선언문이나 할당문을 정렬하는 것은 유용하지 못하다. 코드가 엉뚱한 부분을 강조해 진짜 의도가 가려지기 때문이다.
* 정렬이 필요할 정도로 목록이 길다면 문제는 목록 길이지 정렬 부족이 아니다! 클래스를 쪼개야 한다는 의미.

### 들여쓰기
* 소스 파일은 윤곽도(Outline)와 계층이 비슷하다. 파일 전체에 적용되는 정보가 있고, 파일 내 개별 클래스에 적용되는 정보가 있고, 클래스 내 각 메서드에 적용되는 정보가 있고, 블록 내 블록에 재귀적으로 적용되는 정보가 있다. 계층에서 각 수준은 이름을 선언하는 범위이가 선언문과 실행문을 해석하는 범위이다.
* 프로그래머는 들여쓰기 체계에 크게 의존한다. 왼쪽으로 코드를 맞춰 코드가 속하는 범위를 시각적으로 표현한다.
* 때로는 간단한 if문, 짧은 while문, 짧은 함수에서 들여쓰기 규칙을 무시하고픈 유횩이 생긴다. 그럴수록 원점으로 돌아가 들여쓰기를 넣자.

### 가짜 범위
* 때론 빈 while문이나 for문을 접한다. 이런 구조는 가급적 피해야 하나, 피하지 못할 때는 빈 블록을 올바로 들여쓰고 괄호로 감싼다.
* while문 끝에 세미콜론 하나를 살짝 덧붙인 코드의 경우, 세미콜론을 새 행에다 제대로 들여써서 넣어준다.

## 팀 규칙
* 프로그래머라면 각자 선호하는 규칙이 있다. 그러나 팀에 속한다면 자신이 선호해야 할 규칙은 바로 팀 규칙이다.
