# 2장: 볼 월드 (Ball World)

## 학습 목표

이 장을 완료하면 다음을 할 수 있습니다:
- 게임 객체를 표현하는 클래스를 설계하고 구현할 수 있습니다
- 생성자 오버로딩과 유효성 검사를 구현할 수 있습니다
- JavaFX Canvas를 사용하여 그래픽을 렌더링할 수 있습니다
- 상속을 통해 기능을 확장할 수 있습니다
- 예외 처리를 통해 견고한 코드를 작성할 수 있습니다

## 핵심 개념

### 2.1 Ball 클래스 설계

**상속(Inheritance)이란?**

상속은 기존 클래스(부모 클래스)의 속성과 메서드를 새로운 클래스(자식 클래스)가 물려받는 객체 지향 프로그래밍의 핵심 개념입니다.

```java
// 부모 클래스
public class Animal {
    protected String name;

    public void eat() {
        System.out.println(name + "이(가) 먹습니다.");
    }
}

// 자식 클래스 - extends 키워드로 상속
public class Dog extends Animal {
    public void bark() {
        System.out.println(name + "이(가) 짖습니다.");
    }
}
```

**상속의 장점:**
1. **코드 재사용**: 부모 클래스의 코드를 다시 작성할 필요 없음
2. **확장성**: 기존 기능에 새로운 기능 추가 가능
3. **유지보수**: 공통 코드를 한 곳에서 관리

**Ball 클래스 설계**

Ball 클래스는 1장에서 만든 Point 클래스를 활용하여 다음 기능을 가져야 합니다:

**필드 (private):**
- `center`: Point 타입의 중심점
- `radius`: 반지름

**생성자:**
- 매개변수 생성자: 위치(Point)와 반지름을 지정하여 생성 (필수)
- 편의 생성자: x, y 좌표와 반지름을 받아서 생성
- 기본 생성자 없음 - Ball은 생성 시 위치를 반드시 지정해야 함
- 반지름이 0 이하일 때 `IllegalArgumentException` 발생

**메서드:**
- Getter: `getRadius()`, `getCenter()`
- 위치 변경: `moveTo(Point center)` - 공을 새로운 위치로 이동
- `getArea()`: 공의 면적 계산 (π × r²)
- `contains(Point p)`: 점이 공 안에 있는지 확인
- `contains(double x, double y)`: 점이 공 안에 있는지 확인

**구현 힌트:**
```java
public class Ball {
    private Point center;
    private double radius;

    // 생성자 - 위치 지정 필수
    public Ball(Point center, double radius) {
        // TODO: null 체크, 유효성 검사
    }

    public Ball(double x, double y, double radius) {
        // TODO: Point 생성하여 다른 생성자 호출
    }

    // Getter 메서드
    public Point getCenter() {
        // TODO: center 반환
    }

    public double getRadius() {
        // TODO: radius 반환
    }

    // 위치 이동 메서드
    public void moveTo(Point newCenter) {
        // TODO: null 체크 후 center 업데이트
    }

    // contains 메서드
    public boolean contains(Point p) {
        // TODO: center.distanceTo(p) 활용
    }

    // contains 메서드
    public boolean contains(double x, double y) {
    }
}
```

### 2.2 PaintableBall - 상속을 통한 확장

**super 키워드란?**

`super`는 부모 클래스를 참조하는 특별한 키워드입니다:
- `super()`: 부모 클래스의 생성자 호출
- `super.메서드()`: 부모 클래스의 메서드 호출
- `super.필드`: 부모 클래스의 필드 접근

**PaintableBall 클래스 설계**

Ball을 상속받아 색상 정보를 추가합니다:

**추가 필드:**
- `color`: JavaFX Color 타입

**생성자:**
- 3개 매개변수: 위치와 반지름만 받고 기본 색상(빨간색) 사용
- 4개 매개변수: 위치, 반지름, 색상 모두 지정
- color가 null일 때 `IllegalArgumentException` 발생

**추가 메서드:**
- `getColor()`: 색상 반환
- `setColor(Color color)`: 색상 변경 (null 체크 필요)

**구현 힌트:**
```java
// 부모 클래스 생성자 호출 예시
super(x, y, radius);

// null 체크 예시
if (color == null) {
    throw new IllegalArgumentException("색상은 null일 수 없습니다");
}
```

### 2.3 JavaFX Canvas에 그리기

**Canvas와 GraphicsContext**

Canvas는 그림을 그릴 수 있는 빈 화면이고, GraphicsContext는 그림을 그리는 도구입니다:
- **Canvas**: 도화지
- **GraphicsContext**: 붓, 펜, 물감 등의 그리기 도구

**좌표계 이해하기**
```
(0,0) ────────► X축
│
│    ● (100, 50)
│
▼
Y축
```
- 원점(0,0)은 왼쪽 상단
- X축은 오른쪽으로 증가
- Y축은 아래쪽으로 증가

**PaintableBall에 그리기 기능 추가**

PaintableBall 클래스에 자신을 그리는 메서드를 추가합니다:

**추가 메서드:**
- `draw(GraphicsContext gc)`: GraphicsContext를 받아서 자신을 그리기

**구현 요구사항:**
- 공의 중심 좌표를 기준으로 그리기
- `fillOval`의 매개변수는 왼쪽 상단 좌표와 너비, 높이
- 색상으로 채우고 검은색 테두리 추가

**JavaFX 그리기 메서드 예시:**
```java
public void draw(GraphicsContext gc) {
    // 공의 왼쪽 상단 좌표 계산
    Point center = getCenter();
    double leftX = center.getX() - getRadius();
    double topY = center.getY() - getRadius();
    double diameter = getRadius() * 2;

    // 공 채우기
    gc.setFill(this.color);
    gc.fillOval(leftX, topY, diameter, diameter);
}
```

### 2.4 World 클래스 - 공들을 관리하는 컨테이너

**컨테이너(Container) 클래스란?**

여러 객체를 담고 관리하는 클래스입니다. 마치 상자에 여러 물건을 담는 것과 같습니다:
- **List**: 순서가 있는 객체들의 모음
- **ArrayList**: List의 구현체, 크기가 자동으로 조절됨

**방어적 복사(Defensive Copy)**

내부 데이터를 보호하기 위해 복사본을 반환하는 기법:
```java
// 위험한 코드 - 외부에서 리스트 수정 가능
public List<Ball> getBalls() {
    return balls; // 원본 반환
}

// 안전한 코드 - 복사본 반환
public List<Ball> getBalls() {
    return new ArrayList<>(balls); // 복사본 반환
}
```

**World 클래스 설계**

여러 개의 Ball을 관리하는 컨테이너 클래스입니다:

**필드:**
- `width`, `height`: 월드의 크기 (final)
- `balls`: Ball 객체들을 저장하는 List

**생성자:**
- 너비와 높이를 받아 초기화
- 크기가 0 이하면 예외 발생

**메서드:**
- `add(Ball ball)`: 공 추가
  - null 체크
  - 경계 체크 (공 전체가 월드 안에 있어야 함)
- `remove(Ball ball)`: 특정 공 제거
  - 해당 공이 없으면 `NoSuchElementException` 발생
- `clear()`: 모든 공 제거
- `getBalls()`: 공 목록 반환 (방어적 복사 필요)
- `getBallCount()`: 현재 공의 개수 반환
- `getWidth()`, `getHeight()`: 크기 반환
- `draw(GraphicsContext gc)`: 모든 공을 화면에 그리기
  - 배경 지우기
  - 각 PaintableBall의 draw 메서드 호출

**private 메서드:**
- `isInBounds(Ball ball)`: 공이 경계 안에 있는지 확인

**구현 힌트:**
```java
// ArrayList 생성
private final List<Ball> balls = new ArrayList<>();

// 방어적 복사
return new ArrayList<>(balls);
```

## 실습 과제

**생성자 오버로딩(Constructor Overloading)**

같은 이름의 생성자를 여러 개 만드는 것:
```java
public class Ball {
    private Point center;
    private double radius;

    // 생성자 1: Point와 반지름 지정
    public Ball(Point center, double radius) {
        if (center == null) {
            throw new IllegalArgumentException("중심점은 null일 수 없습니다");
        }
        if (radius <= 0) {
            throw new IllegalArgumentException("반지름은 양수여야 합니다");
        }
        this.center = center;
        this.radius = radius;
    }

    // 생성자 2: x, y 좌표와 반지름 지정 (편의 생성자)
    public Ball(double x, double y, double radius) {
        this(new Point(x, y), radius); // 생성자 1 호출
    }
}
```

### Lab 2-1: Ball 클래스 구현
다음 요구사항을 만족하는 `Ball` 클래스를 구현하세요:
- Point 객체를 사용하여 위치를 저장
- 생성 시 위치 지정 필수 (기본 생성자 없음)
- 반지름은 항상 양수
- 면적과 둘레를 계산하는 메서드 추가
- `toString()` 메서드 오버라이드

**테스트 코드:**
```java
@Test
public void testBallCreation() {
    Ball ball = new Ball(new Point(100, 200), 30);
    assertEquals(100, ball.getCenter().getX());
    assertEquals(200, ball.getCenter().getY());
    assertEquals(30, ball.getRadius());
}

@Test
public void testBallArea() {
    Ball ball = new Ball(new Point(0, 0), 10);
    assertEquals(Math.PI * 100, ball.getArea(), 0.001);
}
```

### Lab 2-2: PaintableBall 확장
`Ball`을 상속받아 색상 정보를 추가한 `PaintableBall` 구현:
- Color 필드 추가
- 여러 생성자 오버로딩
- 색상 변경 메서드
- 유효성 검사
- GraphicContext를 받아서 그리기 추가

### Lab 2-3: BallWorld 애플리케이션
JavaFX를 사용하여 여러 개의 공을 화면에 표시:
- 5개의 랜덤한 위치와 크기의 공 생성
- 각각 다른 색상 적용
- 마우스 클릭으로 새로운 공 추가

**BallWorldApp 구현 가이드**

JavaFX Application을 상속받아 구현합니다:

**필요한 필드:**
- World 객체
- Canvas 객체
- GraphicsContext 객체

**start 메서드에서 할 일:**
1. World 생성 (800×600)
2. Canvas 생성
3. 랜덤한 공 5개 생성
4. 마우스 클릭 이벤트 처리 설정
5. Scene 구성하고 Stage에 설정
6. 초기 화면 그리기

**구현해야 할 메서드:**
- `createRandomBalls(int count)`: 랜덤 위치, 크기, 색상의 공 생성
- `handleMouseClick(MouseEvent event)`: 클릭 위치에 새 공 추가
- `draw()`: 화면 전체 다시 그리기

**JavaFX 이벤트 처리:**
```java
// 마우스 클릭 이벤트 설정
canvas.setOnMouseClicked(event -> {
    double x = event.getX();
    double y = event.getY();
    // 새 공 생성 및 추가
});
```

> 📖 **이벤트 처리에 대한 자세한 설명은 [Java 이벤트 처리 가이드](./java_event_handling.md)를 참고하세요.**

### Lab 2-4: 충돌 감지 기초

Ball 클래스에 두 공이 겹치는지 확인하는 메서드를 추가하세요:

**메서드 시그니처:**
```java
public boolean isColliding(Ball other)
```

**구현 요구사항:**
- 두 공의 중심 사이 거리 계산 (Point의 distanceTo 메서드 활용)
- 거리가 두 반지름의 합보다 작으면 충돌
- other가 null인 경우 처리

**구현 힌트:**
```java
public boolean isColliding(Ball other) {
    if (other == null) {
        return false;
    }
    // TODO: center.distanceTo(other.getCenter()) 활용
    // TODO: 두 반지름의 합과 비교
}
```

## JUnit 테스트 예제

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import static org.junit.jupiter.api.Assertions.*;

public class BallTest {

    @Test
    @DisplayName("공 생성 시 반지름이 음수면 예외 발생")
    public void testNegativeRadius() {
        assertThrows(IllegalArgumentException.class,
            () -> new Ball(0, 0, -5),
            "반지름이 음수일 때 예외가 발생해야 합니다"
        );
    }

    @Test
    @DisplayName("contains 메서드 테스트")
    public void testContains() {
        Ball ball = new Ball(new Point(100, 100), 50);

        // 중심점은 포함
        assertTrue(ball.contains(new Point(100, 100)));

        // 경계 내부의 점
        assertTrue(ball.contains(new Point(120, 120)));

        // 경계 외부의 점
        assertFalse(ball.contains(new Point(200, 200)));
    }

    @Test
    @DisplayName("World에 공 추가 테스트")
    public void testWorldAddBall() {
        World world = new World(500, 500);
        Ball ball = new Ball(new Point(250, 250), 20);

        world.add(ball);
        assertEquals(1, world.getBalls().size());
    }
}
```

## 자가 평가 문제

1. **상속을 사용하는 이유는 무엇인가요?**
   - 코드 재사용
   - 다형성 구현
   - 계층적 관계 표현

2. **생성자 오버로딩의 장점은?**
   - 다양한 초기화 방법 제공
   - 사용자 편의성 증가
   - 기본값 설정 가능

3. **방어적 복사(Defensive Copy)를 하는 이유는?**
   - 내부 상태 보호
   - 불변성 유지
   - 예상치 못한 변경 방지

4. **JavaFX 좌표계의 원점은 어디인가요?**
   - 왼쪽 상단 (0, 0)

## 자주 하는 실수와 해결 방법

### 1. 좌표계 혼동
```java
// 잘못된 코드 - 중심이 아닌 왼쪽 상단에서 그리기
Point center = ball.getCenter();
gc.fillOval(center.getX(), center.getY(), radius * 2, radius * 2);

// 올바른 코드 - 중심에서 반지름만큼 이동
Point center = ball.getCenter();
gc.fillOval(
    center.getX() - ball.getRadius(),
    center.getY() - ball.getRadius(),
    ball.getRadius() * 2,
    ball.getRadius() * 2
);
```

### 2. 리스트 직접 반환
```java
// 잘못된 코드 - 내부 리스트 노출
public List<Ball> getBalls() {
    return balls;
}

// 올바른 코드 - 방어적 복사
public List<Ball> getBalls() {
    return new ArrayList<>(balls);
}
```

### 3. null 체크 누락
```java
// 잘못된 코드
public void setColor(Color color) {
    this.color = color; // color가 null일 수 있음
}

// 올바른 코드
public void setColor(Color color) {
    if (color == null) {
        throw new IllegalArgumentException("색상은 null일 수 없습니다");
    }
    this.color = color;
}
```

## 구현 검증용 테스트 코드

아래 테스트 코드를 사용하여 구현한 클래스들이 올바르게 작동하는지 확인하세요:

### Ball 클래스 테스트

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import static org.junit.jupiter.api.Assertions.*;

public class BallTest {

    @Test
    public void testBallCreation() {
        // x, y 좌표로 생성하는 편의 생성자 테스트
        Ball ball = new Ball(new Point(100, 200), 30);
        Point center = ball.getCenter();
        assertEquals(100, center.getCenter().getX(), 0.001, "X 좌표가 올바르게 설정되지 않았습니다");
        assertEquals(200, center.getCenter().getY(), 0.001, "Y 좌표가 올바르게 설정되지 않았습니다");
        assertEquals(30, ball.getRadius(), 0.001, "반지름이 올바르게 설정되지 않았습니다");
    }

    @Test
    public void testBallWithPoint() {
        // Point를 사용한 생성자 테스트
        Point originalCenter = new Point(100, 200);
        Ball ball = new Ball(originalCenter, 30);
        Point center = ball.getCenter();
        assertEquals(100, center.getX(), 0.001, "X 좌표가 올바르게 설정되지 않았습니다");
        assertEquals(200, center.getY(), 0.001, "Y 좌표가 올바르게 설정되지 않았습니다");
        assertEquals(30, ball.getRadius(), 0.001, "반지름이 올바르게 설정되지 않았습니다");
    }

    @Test
    public void testMoveTo() {
        Ball ball = new Ball(new Point(50, 75), 20);

        // moveTo 테스트
        Point newCenter = new Point(150, 175);
        ball.moveTo(newCenter);

        Point center = ball.getCenter();
        assertEquals(150, center.getX(), 0.001, "moveTo가 올바르게 작동하지 않습니다");
        assertEquals(175, center.getY(), 0.001, "moveTo가 올바르게 작동하지 않습니다");
    }

    @Test
    public void testRadiusValidation() {
        Point center = new Point(0, 0);

        // 유효하지 않은 반지름 테스트
        assertThrows(IllegalArgumentException.class, () -> {
            new Ball(ceneter, -5); // 음수 반지름은 예외 발생해야 함
        }, "음수 반지름에 대해 예외가 발생하지 않았습니다");

        assertThrows(IllegalArgumentException.class, () -> {
            new Ball(center, 0); // 0 반지름도 예외 발생해야 함
        }, "0 반지름에 대해 예외가 발생하지 않았습니다");
    }


    @Test
    public void testContains() {
        Point center = new Point(100, 100);
        Point innerPoint = new Point(120, 120);
        Point outerPoint1 = new Point(200, 200);
        Point outerPoint2 = new Point(50, 50);
        Point linePoint = new Point(150, 100);

        Ball ball = new Ball(center, 50);

        // 공 내부의 점들
        assertTrue(ball.contains(100, 100), "중심점이 포함되지 않았습니다");
        assertTrue(ball.contains(120, 120), "공 내부 점이 포함되지 않았습니다");

        // Point를 사용한 contains 테스트
        assertTrue(ball.contains(center)), "중심점이 포함되지 않았습니다");
        assertTrue(ball.contains(innerPoint), "공 내부 점이 포함되지 않았습니다");

        // 공 외부의 점들
        assertFalse(ball.contains(outerPoint1), "공 외부 점이 포함되었습니다");
        assertFalse(ball.contains(outerPoint2), "공 외부 점이 포함되었습니다");

        // 경계선상의 점 (반지름 거리)
        assertTrue(ball.contains(150, 100), "경계선상의 점이 포함되지 않았습니다");
        assertTrue(ball.contains(linePoint), "경계선상의 점이 포함되지 않았습니다");
    }
}
```

### PaintableBall 클래스 테스트

```java
import javafx.scene.canvas.GraphicsContext;
import javafx.scene.paint.Color;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

public class PaintableBallTest {

    @Test
    public void testPaintableBallCreation() {
        PaintableBall ball = new PaintableBall(new Point(100, 100), 20, Color.RED);

        // 상속받은 Ball의 속성들 확인
        Point center = ball.getCenter();
        assertEquals(100, center.getX(), 0.001, "X 좌표가 올바르게 설정되지 않았습니다");
        assertEquals(100, center.getY(), 0.001, "Y 좌표가 올바르게 설정되지 않았습니다");
        assertEquals(20, ball.getRadius(), 0.001, "반지름이 올바르게 설정되지 않았습니다");

        // PaintableBall의 고유 속성 확인
        assertEquals(Color.RED, ball.getColor(), "색상이 올바르게 설정되지 않았습니다");
    }

    @Test
    public void testPaintableBallInheritance() {
        PaintableBall paintableBall = new PaintableBall(new Point(100, 100), 20, Color.BLUE);

        // Ball의 메서드들이 작동하는지 확인 (상속 확인)
        assertTrue(paintableBall instanceof Ball, "PaintableBall은 Ball을 상속받아야 합니다");

        // Ball의 메서드 사용 가능한지 확인
        Point newCenter = new Point(200, 300);
        paintableBall.moveTo(newCenter);
        Point center = paintableBall.getCenter();
        assertEquals(200, center.getX(), 0.001, "상속받은 moveTo가 작동하지 않습니다");
        assertEquals(300, center.getY(), 0.001, "상속받은 moveTo가 작동하지 않습니다");

        // contains 메서드도 사용 가능한지 확인
        assertTrue(paintableBall.contains(200, 300), "상속받은 contains가 작동하지 않습니다");
    }

    @Test
    public void testColorHandling() {
        PaintableBall ball = new PaintableBall(new Point(0, 0), 10, Color.GREEN);

        // 색상 변경
        ball.setColor(Color.YELLOW);
        assertEquals(Color.YELLOW, ball.getColor(), "색상 변경이 올바르게 작동하지 않습니다");

        // null 색상 처리 (구현에 따라 기본 색상으로 설정하거나 예외 발생)
        ball.setColor(null);
        assertNotNull(ball.getColor(), "null 색상 설정 후에도 색상이 있어야 합니다");
    }

    @Test
    public void testDefaultColor() {
        // 색상 없이 생성하는 생성자 테스트
        PaintableBall ball = new PaintableBall(new Point(100, 100), 10);
        assertNotNull(ball.getColor(), "기본 색상이 설정되어야 합니다");
        assertEquals(Color.RED, ball.getColor(), "기본 색상은 빨간색이어야 합니다");
    }

    @Test
    public void testDraw() {
        // Mock GraphicsContext 생성 (mockito-inline 필요)
        GraphicsContext gc = Mockito.mock(GraphicsContext.class);
        PaintableBall ball = new PaintableBall(new Point(100, 100), 25, Color.BLUE);

        // draw 메서드 호출
        ball.draw(gc);

        // 적절한 메서드들이 호출되었는지 확인
        verify(gc).setFill(Color.BLUE);
        verify(gc).fillOval(getCenter().getX() - getRadius(), getCenter.getY() - getRadius(), getRadius() * 2, getRadius() * 2);
    }
}
```


### World 클래스 테스트

```java
import javafx.scene.canvas.GraphicsContext;
import javafx.scene.paint.Color;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import java.util.List;
import java.util.NoSuchElementException;
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

public class WorldTest {

    private World world;

    @BeforeEach
    public void setUp() {
        world = new World(800, 600);
    }

    @Test
    public void testWorldCreation() {
        assertEquals(800, world.getWidth(), "World의 너비가 올바르게 설정되지 않았습니다");
        assertEquals(600, world.getHeight(), "World의 높이가 올바르게 설정되지 않았습니다");
    }

    @Test
    public void testAddBall() {
        Ball ball = new Ball(100, 100, 20);
        world.add(ball);

        assertEquals(1, world.getBalls().size(), "Ball이 추가되지 않았습니다");
    }

    @Test
    public void testAddMultipleBalls() {
        Ball ball1 = new Ball(new Point(100, 100), 20);
        Ball ball2 = new Ball(new Point(200, 200), 30);
        PaintableBall ball3 = new PaintableBall(new Point(300, 300), 25, Color.RED);

        world.add(ball1);
        world.add(ball2);
        world.add(ball3);

        assertEquals(3, world.getBalls().size(), "여러 Ball이 올바르게 추가되지 않았습니다");
    }

    @Test
    public void testBoundaryCheck() {
        // 경계 내부의 공 - 추가 성공
        Ball validBall = new Ball(new Point(400, 300), 50);
        world.add(validBall);
        assertEquals(1, world.getBalls().size(), "경계 내부의 공이 추가되어야 합니다");

        // 경계 외부의 공 - 추가 실패 (예외 발생)
        Ball invalidBall = new Ball(new Point(50, 50), 100); // 왼쪽 상단 밖으로 나감
        assertThrows(IllegalArgumentException.class, () -> {
            world.add(invalidBall);
        }, "경계 외부의 공은 추가되면 안됩니다");
    }

    @Test
    public void testDefensiveCopy() {
        PaintableBall ball = new PaintableBall(new Point(400, 300), 25, Color.BLUE);
        world.add(ball);

        List<Ball> balls = world.getBalls();
        balls.clear(); // 반환된 리스트를 수정

        // 원본 리스트는 영향받지 않아야 함
        assertEquals(1, world.getBalls().size(), "방어적 복사가 제대로 되지 않았습니다");
    }

    @Test
    public void testRemoveBall() {
        Ball ball = new Ball(new Point(100, 100), 20);
        world.add(ball);
        assertEquals(1, world.getBallCount(), "Ball 추가 확인");

        world.remove(ball);
        assertEquals(0, world.getBallCount(), "Ball이 제거되지 않았습니다");

        // 존재하지 않는 공 제거 시 예외 발생
        assertThrows(NoSuchElementException.class, () -> {
            world.remove(ball);
        }, "존재하지 않는 공 제거 시 예외가 발생해야 합니다");
    }

    @Test
    public void testDraw() {
        GraphicsContext gc = Mockito.mock(GraphicsContext.class);

        // PaintableBall들 추가
        world.add(new PaintableBall(new Point(100, 100), 20, Color.RED));
        world.add(new PaintableBall(new Point(200, 200), 30, Color.GREEN));

        // 그리기 수행
        world.draw(gc);

        // clearRect가 호출되었는지 확인
        verify(gc).clearRect(0, 0, world.getWidth(), world.getHeight());
        // 두 개의 공이 그려졌는지 확인 (각 공당 fillOval 호출)
        verify(gc, times(2)).fillOval(anyDouble(), anyDouble(), anyDouble(), anyDouble());
    }
}
```

### 테스트 실행 방법

1. **필요한 의존성 추가** (pom.xml):
   ```xml
   <dependencies>
       <dependency>
           <groupId>org.junit.jupiter</groupId>
           <artifactId>junit-jupiter</artifactId>
           <version>5.8.2</version>
           <scope>test</scope>
       </dependency>
       <dependency>
           <groupId>org.mockito</groupId>
           <artifactId>mockito-inline</artifactId>
           <version>4.6.1</version>
           <scope>test</scope>
       </dependency>
   </dependencies>
   ```

   **주의**: GraphicsContext는 final 클래스이므로 mockito-inline이 필요합니다.

2. **Maven 프로젝트의 경우**:
   ```bash
   mvn test
   ```

3. **IDE에서 실행**:
   - 테스트 클래스에서 우클릭 → "Run Tests"
   - 개별 테스트 메서드 실행 가능

4. **테스트 성공 기준**:
   - 모든 테스트가 통과하면 구현이 올바름
   - 실패하는 테스트가 있다면 해당 기능을 다시 구현

### 테스트 해석 가이드

- **AssertionError**: 예상한 값과 실제 값이 다름 → 로직 수정 필요
- **NullPointerException**: null 체크 누락 → 방어 코드 추가 필요
- **IllegalArgumentException**: 유효성 검사 누락 → 검증 로직 추가 필요

## 다음 장 미리보기

3장에서는 공에 움직임을 추가합니다:
- 속도 벡터 구현
- 시간 기반 애니메이션
- AnimationTimer 사용법
- 부드러운 움직임 구현

## 추가 학습 자료

- [Java 이벤트 처리 가이드](./java_event_handling.md) - 이 장에서 사용하는 이벤트 처리 방법에 대한 상세 설명
- [JavaFX Canvas Tutorial](https://docs.oracle.com/javafx/2/canvas/jfxpub-canvas.htm)
- [Java 상속과 다형성](https://docs.oracle.com/javase/tutorial/java/IandI/subclasses.html)
- [Oracle Java 생성자 가이드](https://docs.oracle.com/javase/tutorial/java/javaOO/constructors.html)
- [JavaFX Graphics와 Animation](https://openjfx.io/javadoc/17/javafx.graphics/module-summary.html)
- [Java 예외 처리 베스트 프랙티스](https://docs.oracle.com/javase/tutorial/essential/exceptions/)

## 학습 체크포인트

- [ ] Ball 클래스를 구현하고 테스트했습니다
- [ ] 상속을 사용하여 PaintableBall을 구현했습니다
- [ ] JavaFX Canvas에 도형을 그릴 수 있습니다
- [ ] World 클래스로 여러 객체를 관리할 수 있습니다
- [ ] 예외 처리와 유효성 검사를 구현했습니다