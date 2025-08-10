# 3장: 움직이는 월드 (Movable World)

## 학습 목표

이 장을 완료하면 다음을 할 수 있습니다:
- 속도와 가속도 개념을 구현할 수 있습니다
- 시간 기반 애니메이션을 구현할 수 있습니다
- JavaFX AnimationTimer를 사용할 수 있습니다
- 델타 타임을 이용한 프레임 독립적 움직임을 구현할 수 있습니다
- 상속 계층을 통해 기능을 확장할 수 있습니다

## 핵심 개념

**프레임(Frame)과 FPS**

게임에서 프레임은 화면에 표시되는 하나의 정지 화면입니다:
- **프레임**: 애니메이션의 한 장면
- **FPS (Frames Per Second)**: 초당 프레임 수
- **60 FPS**: 1초에 60번 화면이 갱신됨 (약 16.67ms마다)

영화처럼 빠르게 변하는 정지 화면들이 연속적으로 보이면 움직이는 것처럼 보입니다.

### 3.1 MovableBall - 움직임 추가

**MovableBall 클래스 설계**

PaintableBall을 상속받아 움직임 기능을 추가합니다:

**추가 필드:**
- `velocity`: 속도 벡터 (Vector2D)

**생성자:**
- 2개 매개변수: center, radius (속도는 영벡터로 초기화, 색은 PointableBall의 기본값)
- 3개 매개변수: center, radius, color (속도는 영벡터로 초기화)
- 4개 매개변수: center, radius, color, velocity (모든 속성 지정)

**추가 메서드:**
- `getVelocity()`: 속도 벡터 반환
- `setVelocity(Vector2D velocity)`: 속도 벡터 설정
- `move()`: 프레임 단위 이동 (기본적으로 1/60초 가정)
- `move(double deltaTime)`: 시간 기반 이동

**구현 힌트:**
```java
public class MovableBall extends PaintableBall {
    private Vector2D velocity;

    public MovableBall(Point center, double radius) {
        super(center, radius);
        // 속도 초기화
    }

    public MovableBall(Point center, double radius, Color color) {
        super(center, radius, color);
        // 속도 초기화
    }

    public MovableBall(Point center, double radius, Color color, Vector2D velocity) {
        super(center, radius, color);
        // 속도 초기화
    }

    // 시간 기반 이동
    public void move(double deltaTime) {
        Point currentCenter = getCenter();
        Vector2D displacement = new Vector(velocity.getX() * deltaTime, velocity.getY() * deltaTime);
        Point newCenter = currentCenter.add(displacement);
        moveTo(newCenter);
    }

    public void move() {
        // 기본 60 FPS 가정
        // 메서드 활용
    }
}
```

### 3.2 시간 기반 애니메이션

**델타 타임(Delta Time)이란?**

이전 프레임과 현재 프레임 사이의 시간 간격입니다:
- 컴퓨터 성능에 관계없이 일정한 속도 유지
- 단위: 보통 초(second) 사용
- 예: 60 FPS에서 델타 타임 ≈ 0.0167초

```java
// 프레임 의존적 (잘못된 방법)
Point oldCenter = ball.getCenter();
ball.moveTo(new Point(oldCenter.getX() + 5, oldCenter.getY())); // 빠른 컴퓨터에서는 더 빨리 움직임

// 시간 기반 (올바른 방법)
Vector2D displacement = new Vector2D(velocity.getX() * deltaTime, velocity.getY() * deltaTime);
Point newCenter = oldCenter.add(displacement); // 모든 컴퓨터에서 같은 속도
```

**AnimationTimer란?**

JavaFX에서 제공하는 게임 루프 클래스입니다:
- 자동으로 매 프레임마다 handle() 메서드 호출
- 나노초 단위의 정확한 시간 제공
- start()로 시작, stop()으로 중지

**GameLoop 클래스 설계**

AnimationTimer를 상속받아 게임 루프를 구현합니다:

**중요 참고사항:**
- AnimationTimer는 JavaFX UI 스레드에서만 동작합니다
- 따라서 단위 테스트(JUnit)로는 테스트할 수 없습니다
- 실제 JavaFX 애플리케이션 실행을 통해서만 동작 확인이 가능합니다

**필드:**
- `lastUpdate`: 이전 프레임의 시간 (나노초)
- `world`: World 객체 참조
- `gc`: GraphicsContext 참조

**핵심 메서드:**
- `handle(long now)`: AnimationTimer가 매 프레임 호출
  - 델타 타임 계산
  - update 호출
  - render 호출
- `update(double deltaTime)`: 공들의 위치 업데이트
- `render()`: 화면에 그리기

**구현 순서:**
1. 델타 타임 계산 (나노초를 초로 변환)
2. 모든 MovableBall의 move(deltaTime) 호출
3. 화면 지우기
4. 배경 그리기
5. 모든 공 그리기

**델타 타임 계산 힌트:**
```java
// 나노초를 초로 변환: 1초 = 1,000,000,000 나노초
double deltaTime = (now - lastUpdate) / 1_000_000_000.0;
```

### 3.3 속도 벡터와 물리 계산

**벡터(Vector)란?**

크기와 방향을 가진 양입니다:
- **스칼라**: 크기만 있음 (예: 온도, 질량)
- **벡터**: 크기 + 방향 (예: 속도, 힘)

```
      ↑ (3, 4)
     /|
    / | 4
   /  |
  /   |
 →-----+
    3

크기 = √(3² + 4²) = 5
방향 = atan2(4, 3) ≈ 53.13°
```

**속도 vs 속력**
- **속력(Speed)**: 크기만 있음 (예: 50km/h)
- **속도(Velocity)**: 크기 + 방향 (예: 동쪽으로 50km/h)

**Vector2D 클래스 설계**

2차원 벡터를 표현하는 불변(immutable) 클래스입니다:

**필드 (final):**
- `x`: x 구성 요소
- `y`: y 구성 요소

**메서드:**
- `add(Vector2D other)`: 벡터 덧셈
- `subtract(Vector2D other)`: 벡터 뺄셈
- `multiply(double scalar)`: 스칼라 곱셈
- `magnitude()`: 벡터의 크기 (√(x² + y²))
- `normalize()`: 정규화 (크기를 1로 만들기)
- `dot(Vector2D other)`: 내적 (x₁×x₂ + y₁×y₂)
- `getX()`, `getY()`: Getter 메서드

**구현 힌트:**
```java
// 모든 연산은 새 Vector2D 객체 반환 (immutable)
// normalize에서 magnitude가 0일 때 처리 필요
// 내적 = x₁×x₂ + y₁×y₂
```

**Point와 Vector2D의 관계**

Point는 위치를 나타내고, Vector2D는 변위(이동량)를 나타냅니다:
- **Point + Vector2D = Point**: 위치에서 이동하여 새로운 위치
- **Point - Point = Vector2D**: 두 위치 간의 변위
- **Vector2D + Vector2D = Vector2D**: 변위의 합성

이 관계를 활용하기 위해 Point 클래스를 확장해야 합니다:

```java
public class Point {
    private final double x;
    private final double y;

    // 기존 생성자와 메서드...

    // Vector2D와의 연산을 위해 추가할 메서드
    public Point add(Vector2D vector) {
        return new Point(x + vector.getX(), y + vector.getY());
    }

    public Vector2D subtract(Point other) {
        return new Vector2D(x - other.x, y - other.y);
    }
}
```

**중요:** 이 메서드들을 Point 클래스에 추가하면, 위치와 속도를 더 자연스럽게 다룰 수 있습니다.

### 3.4 MovableWorld 클래스 설계

**MovableWorld 클래스란?**

MovableWorld는 World 클래스를 상속받아 움직이는 공들을 관리하는 확장된 세계입니다:

**상속 구조:**
```
World (기본 세계)
  ↓ 상속
MovableWorld (움직이는 공들을 관리하는 세계)
```

**MovableWorld의 역할:**
1. **움직이는 공들의 위치 업데이트**: 모든 MovableBall의 이동을 관리
2. **프레임 단위 시뮬레이션**: 매 프레임마다 물리 업데이트 수행
3. **렌더링 최적화**: 움직이는 객체들의 효율적인 그리기

**World로부터 상속받은 메서드들:**
- `add(Ball ball)`: 공을 월드에 추가 (2장에서 정의)
- `remove(Ball ball)`: 특정 공 제거
- `clear()`: 모든 공 제거
- `getBalls()`: 공 목록 반환 (방어적 복사)
- `getBallCount()`: 현재 공의 개수 반환
- `getWidth()`, `getHeight()`: 월드 크기 반환
- `draw(GraphicsContext gc)`: 모든 공을 화면에 그리기

**클래스 설계:**

**생성자:**
```java
public MovableWorld(double width, double height) {
    super(width, height); // 부모 클래스 생성자 호출
}
```

**추가 메서드:**
```java
public void update(double deltaTime) {
    // 모든 공들의 위치 업데이트
    // 각 공이 MovableBall인지 확인하고 move() 호출
}
```

**update() 메서드 구현 힌트:**
```java
// instanceof를 사용하여 MovableBall 타입 확인
for (Ball ball : balls) {
    if (ball instanceof MovableBall movableBall) {
        movableBall.move(deltaTime);
    }
}
```

**MovableWorld 사용 예시:**
```java
// MovableWorld 생성
MovableWorld world = new MovableWorld(800, 600);

// 움직이는 공 추가 (상속받은 add 메서드 사용)
Point initialCenter = new Point(100, 100);
MovableBall ball = new MovableBall(initialCenter, 20, Color.RED);
ball.setVelocity(new Vector2D(50, 30)); // 속도 벡터 설정
world.add(ball); // World 클래스의 add() 메서드 사용

// 게임 루프에서 업데이트
world.update(0.016); // 약 60FPS (1/60초)
```

### 3.5 개선된 MovableBall with Vector

**물리 시뮬레이션 기초**

뉴턴의 운동 법칙을 코드로 구현:
1. **제1법칙 (관성)**: 힘이 없으면 속도 유지
2. **제2법칙**: F = ma (힘 = 질량 × 가속도)
3. **제3법칙**: 작용-반작용

```java
// 물리 업데이트 순서
1. 힘 적용 → 가속도 계산 (F = ma)
2. 가속도로 속도 변경 (v = v + a × Δt)
3. 속도로 위치 변경 (p = p + v × Δt)
4. 가속도 초기화 (다음 프레임 준비)
```

**MovableBallV2 클래스 설계**

Vector2D를 사용하여 더 나은 물리 시뮬레이션을 구현합니다:

```java
public class MovableBallV2 extends PaintableBall {
    private Vector2D velocity;
    private Vector2D acceleration;

    public MovableBallV2(Point center, double radius, Color color) {
        super(center, radius, color);
        this.velocity = new Vector2D(0, 0);
        this.acceleration = new Vector2D(0, 0);
    }

    // 힘 적용 (F = ma, a = F/m)
    public void applyForce(Vector2D force) {
        // 질량을 반지름에 비례한다고 가정 (간단화)
        double mass = getRadius();
        Vector2D acc = force.multiply(1.0 / mass);
        acceleration = acceleration.add(acc);
    }

    // 물리 업데이트
    public void update(double deltaTime) {
        // 1. 속도 업데이트: v = v + a × Δt
        velocity = velocity.add(acceleration.multiply(deltaTime));

        // 2. 위치 업데이트: p = p + v × Δt
        Point currentCenter = getCenter();
        Vector2D displacement = velocity.multiply(deltaTime);
        Point newCenter = currentCenter.add(displacement);
        moveTo(newCenter);

        // 3. 가속도 초기화 (중요!)
        acceleration = new Vector2D(0, 0);
    }

    // 속도 제한
    public void limitSpeed(double maxSpeed) {
        double currentSpeed = velocity.magnitude();
        if (currentSpeed > maxSpeed) {
            velocity = velocity.normalize().multiply(maxSpeed);
        }
    }
}
```

**구현 힌트:**
```java
// 중력 적용 예시
Vector2D gravity = new Vector2D(0, 9.8 * 10); // 10픽셀 = 1미터로 스케일링
ball.applyForce(gravity);

// 바람 적용 예시
Vector2D wind = new Vector2D(5, 0);
ball.applyForce(wind);
```

## 실습 과제

**instanceof 연산자**

객체가 특정 클래스의 인스턴스인지 확인하는 연산자:
```java
if (ball instanceof MovableBall) {
    // ball이 MovableBall 또는 그 자식 클래스의 인스턴스인 경우
    MovableBall movable = (MovableBall) ball; // 안전한 형변환
    movable.move();
}
```

### Lab 3-1: 기본 MovableBall 구현
`MovableBall` 클래스를 구현하고 테스트하세요:
- PaintableBall을 상속
- 속도 벡터 필드 추가 (Vector2D)
- move() 메서드 구현
- Point와 Vector2D를 활용한 위치 업데이트

**테스트 코드:**
```java
@Test
public void testMovableBallMovement() {
    Point initialCenter = new Point(100, 100);
    MovableBall ball = new MovableBall(initialCenter, 20, Color.RED);
    ball.setVelocity(new Vector2D(50, 30));

    ball.move(1.0); // 1초 동안 이동

    Point newCenter = ball.getCenter();
    assertEquals(150, newCenter.getX(), 0.001);
    assertEquals(130, newCenter.getY(), 0.001);
}

@Test
public void testVelocityMagnitude() {
    MovableBall ball = new MovableBall(new Point(0, 0), 10, Color.BLUE);
    Vector2D velocity = new Vector2D(3, 4);
    ball.setVelocity(velocity);

    assertEquals(5.0, ball.getVelocity().magnitude(), 0.001); // 3-4-5 삼각형
}
```

### Lab 3-2: AnimationTimer 구현
JavaFX AnimationTimer를 사용하여 애니메이션 구현:
- 여러 개의 공이 화면에서 움직이기
- 델타 타임을 사용한 부드러운 움직임
- FPS 표시

**MovableWorldApp 구현 가이드**

AnimationTimer를 사용하여 움직이는 공들을 표시합니다:

**필요한 기능:**
1. World 생성 (800×600)
2. 랜덤한 속도로 움직이는 공 10개 생성
3. FPS 표시
4. GameLoop 시작

**createMovingBalls() 메서드:**
- 랜덤 위치: 가장자리에서 반지름만큼 떨어진 곳
- 랜덤 크기: 10~30 픽셀
- 랜덤 색상: RGB 각각 0~255
- 랜덤 속도: -100~100 pixels/second

**FPS 계산 힌트:**
```java
// handle() 메서드에서
frameCount++;
long currentTime = System.currentTimeMillis();
if (currentTime - lastFpsTime >= 1000) { // 1초마다
    int fps = frameCount;
    frameCount = 0;
    lastFpsTime = currentTime;
    Platform.runLater(() -> fpsLabel.setText("FPS: " + fps));
}
```

### Lab 3-3: Vector2D 클래스 활용
Vector2D를 사용하여 더 나은 물리 구현:
- 마우스 클릭 방향으로 공 발사
- 중력 효과 추가
- 마찰력 구현

### Lab 3-4: 궤적 그리기
공의 이동 경로를 시각화:
- 최근 50개 위치 저장
- 반투명한 선으로 궤적 표시
- 속도에 따른 색상 변화

## JUnit 테스트 예제

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import static org.junit.jupiter.api.Assertions.*;

public class MovableBallTest {
    private MovableBall ball;

    @BeforeEach
    public void setUp() {
        ball = new MovableBall(new Point(100, 100), 20, Color.RED);
        ball.setVelocity(new Vector2D(60, 80)); // 60, 80 pixels/second
    }

    @Test
    public void testInitialVelocity() {
        MovableBall newBall = new MovableBall(new Point(0, 0), 10, Color.RED);
        assertEquals(0, newBall.getVelocity().getX());
        assertEquals(0, newBall.getVelocity().getY());
    }

    @Test
    public void testMove() {
        Point originalCenter = ball.getCenter();

        ball.move(0.5); // 0.5초 이동

        Point newCenter = ball.getCenter();
        assertEquals(originalCenter.getX() + 30, newCenter.getX(), 0.001);
        assertEquals(originalCenter.getY() + 40, newCenter.getY(), 0.001);
    }

    @Test
    public void testVelocityMagnitude() {
        assertEquals(100, ball.getVelocity().magnitude(), 0.001); // sqrt(60^2 + 80^2) = 100
    }

    @Test
    public void testVelocityDirection() {
        MovableBall rightBall = new MovableBall(new Point(0, 0), 10, Color.RED);
        rightBall.setVelocity(new Vector2D(10, 0));
        double rightDirection = Math.atan2(0, 10);
        assertEquals(0, rightDirection, 0.001); // 오른쪽 = 0 라디안

        MovableBall upBall = new MovableBall(new Point(0, 0), 10, Color.BLUE);
        upBall.setVelocity(new Vector2D(0, -10));
        double upDirection = Math.atan2(-10, 0);
        assertEquals(-Math.PI/2, upDirection, 0.001); // 위쪽 = -π/2
    }
}
```

## 자가 평가 문제

1. **델타 타임을 사용하는 이유는?**
   - 프레임률 독립적인 움직임
   - 다양한 하드웨어에서 일관된 속도
   - 부드러운 애니메이션

2. **속도와 가속도의 차이는?**
   - 속도: 위치의 변화율
   - 가속도: 속도의 변화율

3. **AnimationTimer의 handle 메서드는 언제 호출되나요?**
   - 매 프레임마다 (보통 60FPS)

4. **벡터 정규화(normalize)의 의미는?**
   - 방향은 유지하고 크기를 1로 만들기

## 자주 하는 실수와 해결 방법

### 1. 델타 타임 단위 혼동
```java
// 잘못된 코드 - 나노초를 그대로 사용
double deltaTime = now - lastUpdate;

// 올바른 코드 - 초 단위로 변환
double deltaTime = (now - lastUpdate) / 1_000_000_000.0;
```

### 2. 프레임 의존적 움직임
```java
// 잘못된 코드 - 고정된 이동량
ball.setX(ball.getX() + 5);

// 올바른 코드 - 시간 기반 이동
ball.setX(ball.getX() + velocity * deltaTime);
```

### 3. 가속도 누적
```java
// 잘못된 코드 - 가속도가 계속 누적됨
acceleration = acceleration.add(force);

// 올바른 코드 - 매 프레임 초기화
public void update(double deltaTime) {
    velocity = velocity.add(acceleration.multiply(deltaTime));
    position = position.add(velocity.multiply(deltaTime));
    acceleration = new Vector2D(0, 0); // 초기화!
}
```

## 구현 검증용 테스트 코드

아래 테스트 코드를 사용하여 구현한 클래스들이 올바르게 작동하는지 확인하세요:

### Vector2D 클래스 테스트

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class Vector2DTest {

    @Test
    public void testVectorCreation() {
        Vector2D vector = new Vector2D(3.0, 4.0);
        assertEquals(3.0, vector.getX(), 0.001, "X 성분이 올바르게 설정되지 않았습니다");
        assertEquals(4.0, vector.getY(), 0.001, "Y 성분이 올바르게 설정되지 않았습니다");
    }

    @Test
    public void testDefaultConstructor() {
        Vector2D vector = new Vector2D();
        assertEquals(0.0, vector.getX(), 0.001, "기본 X 성분은 0이어야 합니다");
        assertEquals(0.0, vector.getY(), 0.001, "기본 Y 성분은 0이어야 합니다");
    }

    @Test
    public void testVectorAddition() {
        Vector2D v1 = new Vector2D(2.0, 3.0);
        Vector2D v2 = new Vector2D(1.0, 4.0);

        Vector2D result = v1.add(v2);

        assertEquals(3.0, result.getX(), 0.001, "벡터 덧셈 X 성분이 잘못되었습니다");
        assertEquals(7.0, result.getY(), 0.001, "벡터 덧셈 Y 성분이 잘못되었습니다");

        // 원본 벡터는 변경되지 않아야 함
        assertEquals(2.0, v1.getX(), 0.001, "원본 벡터가 변경되었습니다");
        assertEquals(3.0, v1.getY(), 0.001, "원본 벡터가 변경되었습니다");
    }

    @Test
    public void testVectorSubtraction() {
        Vector2D v1 = new Vector2D(5.0, 8.0);
        Vector2D v2 = new Vector2D(2.0, 3.0);

        Vector2D result = v1.subtract(v2);

        assertEquals(3.0, result.getX(), 0.001, "벡터 뺄셈 X 성분이 잘못되었습니다");
        assertEquals(5.0, result.getY(), 0.001, "벡터 뺄셈 Y 성분이 잘못되었습니다");
    }

    @Test
    public void testVectorMultiplication() {
        Vector2D vector = new Vector2D(3.0, 4.0);
        Vector2D result = vector.multiply(2.0);

        assertEquals(6.0, result.getX(), 0.001, "벡터 곱셈 X 성분이 잘못되었습니다");
        assertEquals(8.0, result.getY(), 0.001, "벡터 곱셈 Y 성분이 잘못되었습니다");
    }

    @Test
    public void testVectorMagnitude() {
        Vector2D vector = new Vector2D(3.0, 4.0);
        double magnitude = vector.magnitude();

        assertEquals(5.0, magnitude, 0.001, "벡터 크기 계산이 잘못되었습니다");

        // 영벡터 테스트
        Vector2D zeroVector = new Vector2D(0.0, 0.0);
        assertEquals(0.0, zeroVector.magnitude(), 0.001, "영벡터의 크기는 0이어야 합니다");
    }

    @Test
    public void testVectorNormalize() {
        Vector2D vector = new Vector2D(3.0, 4.0);
        Vector2D normalized = vector.normalize();

        assertEquals(1.0, normalized.magnitude(), 0.001, "정규화된 벡터의 크기는 1이어야 합니다");
        assertEquals(0.6, normalized.getX(), 0.001, "정규화된 벡터 X 성분이 잘못되었습니다");
        assertEquals(0.8, normalized.getY(), 0.001, "정규화된 벡터 Y 성분이 잘못되었습니다");
    }

    @Test
    public void testZeroVectorNormalize() {
        Vector2D zeroVector = new Vector2D(0.0, 0.0);
        Vector2D normalized = zeroVector.normalize();

        // 영벡터의 정규화는 영벡터를 반환해야 함
        assertEquals(0.0, normalized.getX(), 0.001, "영벡터 정규화 X 성분이 잘못되었습니다");
        assertEquals(0.0, normalized.getY(), 0.001, "영벡터 정규화 Y 성분이 잘못되었습니다");
    }

    @Test
    public void testDotProduct() {
        Vector2D v1 = new Vector2D(3.0, 4.0);
        Vector2D v2 = new Vector2D(2.0, 1.0);

        double dotProduct = v1.dot(v2);

        assertEquals(10.0, dotProduct, 0.001, "내적 계산이 잘못되었습니다");
    }
}
```

### MovableBall 클래스 테스트

```java
import javafx.scene.paint.Color;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import static org.junit.jupiter.api.Assertions.*;

public class MovableBallTest {

    private MovableBall ball;

    @BeforeEach
    public void setUp() {
        ball = new MovableBall(new Point(100, 100), 20, Color.RED);
    }

    @Test
    public void testMovableBallCreation() {
        MovableBall newBall = new MovableBall(new Point(100, 100), 20, Color.RED);

        // 부모 클래스 속성 확인
        Point center = newBall.getCenter();
        assertEquals(100, center.getX(), 0.001, "X 좌표가 올바르게 설정되지 않았습니다");
        assertEquals(100, center.getY(), 0.001, "Y 좌표가 올바르게 설정되지 않았습니다");
        assertEquals(20, newBall.getRadius(), 0.001, "반지름이 올바르게 설정되지 않았습니다");
        assertEquals(Color.RED, newBall.getColor(), "색상이 올바르게 설정되지 않았습니다");

        // 초기 속도는 0이어야 함
        Vector2D velocity = newBall.getVelocity();
        assertEquals(0.0, velocity.getX(), 0.001, "초기 X 속도는 0이어야 합니다");
        assertEquals(0.0, velocity.getY(), 0.001, "초기 Y 속도는 0이어야 합니다");
    }

    @Test
    public void testVelocitySetterAndGetter() {
        Vector2D newVelocity = new Vector2D(50.0, 30.0);
        ball.setVelocity(newVelocity);

        Vector2D velocity = ball.getVelocity();
        assertEquals(50.0, velocity.getX(), 0.001, "X 속도 설정이 올바르지 않습니다");
        assertEquals(30.0, velocity.getY(), 0.001, "Y 속도 설정이 올바르지 않습니다");
    }

    @Test
    public void testVelocityImmutability() {
        Vector2D originalVelocity = new Vector2D(100.0, 75.0);
        ball.setVelocity(originalVelocity);

        // 원본 벡터를 변경해도 ball의 속도는 영향받지 않아야 함
        // (Vector2D가 불변 객체이므로)
        Vector2D retrievedVelocity = ball.getVelocity();
        assertEquals(100.0, retrievedVelocity.getX(), 0.001, "벡터 속도 X가 올바르지 않습니다");
        assertEquals(75.0, retrievedVelocity.getY(), 0.001, "벡터 속도 Y가 올바르지 않습니다");
    }

    @Test
    public void testMove() {
        ball.setVelocity(new Vector2D(60.0, 80.0)); // 60, 80 pixels/second

        double deltaTime = 0.5; // 0.5초
        ball.move(deltaTime);

        // 0.5초 동안 이동한 거리 계산
        Point newCenter = ball.getCenter();
        assertEquals(130.0, newCenter.getX(), 0.001, "X 방향 이동이 올바르지 않습니다"); // 100 + 60*0.5
        assertEquals(140.0, newCenter.getY(), 0.001, "Y 방향 이동이 올바르지 않습니다"); // 100 + 80*0.5
    }

    @Test
    public void testMoveWithZeroVelocity() {
        MovableBall zeroVelBall = new MovableBall(new Point(50, 50), 10, Color.GREEN);
        // 기본 속도는 영벡터

        Point originalCenter = zeroVelBall.getCenter();
        zeroVelBall.move(1.0); // 1초 경과

        Point newCenter = zeroVelBall.getCenter();
        assertEquals(originalCenter.getX(), newCenter.getX(), 0.001, "속도가 0일 때 X 위치가 변경되었습니다");
        assertEquals(originalCenter.getY(), newCenter.getY(), 0.001, "속도가 0일 때 Y 위치가 변경되었습니다");
    }

    @Test
    public void testMoveWithNegativeVelocity() {
        ball.setVelocity(new Vector2D(-40.0, -30.0));

        ball.move(1.0);

        Point newCenter = ball.getCenter();
        assertEquals(60.0, newCenter.getX(), 0.001, "음수 X 속도 이동이 올바르지 않습니다");
        assertEquals(70.0, newCenter.getY(), 0.001, "음수 Y 속도 이동이 올바르지 않습니다");
    }

    @Test
    public void testInheritance() {
        // MovableBall이 PaintableBall을 상속받는지 확인
        assertTrue(ball instanceof PaintableBall, "MovableBall은 PaintableBall을 상속받아야 합니다");
        assertTrue(ball instanceof Ball, "MovableBall은 Ball을 상속받아야 합니다");

        // 부모 클래스의 메서드 사용 가능한지 확인
        ball.setColor(Color.BLUE);
        assertEquals(Color.BLUE, ball.getColor(), "상속받은 색상 변경이 작동하지 않습니다");

        assertTrue(ball.contains(100, 100), "상속받은 contains 메서드가 작동하지 않습니다");
    }

    @Test
    public void testVelocityMagnitudeCalculation() {
        ball.setVelocity(new Vector2D(30.0, 40.0));

        double magnitude = ball.getVelocity().magnitude();
        assertEquals(50.0, magnitude, 0.001, "속력 계산이 올바르지 않습니다"); // √(30² + 40²) = 50
    }

    @Test
    public void testVelocityDirectionCalculation() {
        ball.setVelocity(new Vector2D(10.0, 10.0));

        Vector2D vel = ball.getVelocity();
        double direction = Math.atan2(vel.getY(), vel.getX());
        assertEquals(Math.PI / 4, direction, 0.001, "방향 계산이 올바르지 않습니다"); // 45도 = π/4 라디안
    }
}
```

### GameLoop 클래스 테스트에 대한 중요 참고사항

**GameLoop는 AnimationTimer를 상속받는 클래스로, 단위 테스트가 불가능합니다.**

**이유:**
- AnimationTimer는 JavaFX의 UI 스레드(JavaFX Application Thread)에서만 실행됩니다
- JUnit 테스트 환경에서는 JavaFX UI 스레드가 실행되지 않습니다
- `start()`, `stop()`, `handle()` 메서드 호출 시 다음과 같은 예외가 발생합니다:
  ```
  java.lang.IllegalStateException: Not on FX application thread
  ```

**대안:**
1. **통합 테스트**: 실제 JavaFX 애플리케이션을 실행하면서 테스트
2. **수동 테스트**: 애플리케이션을 실행하여 직접 동작 확인
3. **TestFX 프레임워크**: JavaFX 애플리케이션 테스트 전용 프레임워크 사용

**GameLoop 클래스 구현 시 검증 방법:**
```java
// MovableWorldApp의 start() 메서드에서 직접 확인
@Override
public void start(Stage stage) {
    // ... 초기화 코드 ...

    GameLoop gameLoop = new GameLoop(world, gc);

    // FPS 표시를 통해 게임 루프 동작 확인
    Label fpsLabel = new Label("FPS: 0");
    gameLoop.setFpsListener(fps ->
        Platform.runLater(() -> fpsLabel.setText("FPS: " + fps))
    );

    gameLoop.start(); // 실제 실행하여 동작 확인
}
```

### MovableWorld 클래스 테스트

```java
import javafx.scene.canvas.GraphicsContext;
import javafx.scene.paint.Color;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import static org.junit.jupiter.api.Assertions.*;

public class MovableWorldTest {

    private MovableWorld world;

    @BeforeEach
    public void setUp() {
        world = new MovableWorld(800, 600);
    }

    @Test
    public void testWorldCreation() {
        assertEquals(800, world.getWidth(), "World 너비가 올바르지 않습니다");
        assertEquals(600, world.getHeight(), "World 높이가 올바르지 않습니다");
    }

    @Test
    public void testAddMovableBall() {
        MovableBall ball = new MovableBall(new Point(100, 100), 20, Color.RED);
        world.add(ball);

        assertEquals(1, world.getBallCount(), "MovableBall이 추가되지 않았습니다");
    }

    @Test
    public void testUpdate() {
        MovableBall ball = new MovableBall(new Point(100, 100), 20, Color.RED);
        ball.setVelocity(new Vector2D(50.0, 30.0));
        world.add(ball);

        double deltaTime = 0.1; // 0.1초
        world.update(deltaTime);

        // 공이 이동했는지 확인
        Point newCenter = ball.getCenter();
        assertEquals(105.0, newCenter.getX(), 0.001, "update 후 공이 X 방향으로 이동하지 않았습니다");
        assertEquals(103.0, newCenter.getY(), 0.001, "update 후 공이 Y 방향으로 이동하지 않았습니다");
    }

    @Test
    public void testUpdateMultipleBalls() {
        MovableBall ball1 = new MovableBall(new Point(100, 100), 20, Color.RED);
        MovableBall ball2 = new MovableBall(new Point(200, 200), 30, Color.BLUE);

        ball1.setVelocity(new Vector2D(10.0, 20.0));
        ball2.setVelocity(new Vector2D(-15.0, 25.0));

        world.add(ball1);
        world.add(ball2);

        world.update(1.0); // 1초

        Point center1 = ball1.getCenter();
        Point center2 = ball2.getCenter();
        assertEquals(110.0, center1.getX(), 0.001, "첫 번째 공의 X 이동이 올바르지 않습니다");
        assertEquals(120.0, center1.getY(), 0.001, "첫 번째 공의 Y 이동이 올바르지 않습니다");
        assertEquals(185.0, center2.getX(), 0.001, "두 번째 공의 X 이동이 올바르지 않습니다");
        assertEquals(225.0, center2.getY(), 0.001, "두 번째 공의 Y 이동이 올바르지 않습니다");
    }

    @Test
    public void testRender() {
        // GraphicsContext는 final 클래스이므로 mockito-inline 의존성이 필요합니다
        GraphicsContext gc = Mockito.mock(GraphicsContext.class);
        MovableBall ball = new MovableBall(new Point(100, 100), 20, Color.GREEN);
        world.add(ball);

        assertDoesNotThrow(() -> {
            world.draw(gc);
        }, "MovableWorld 렌더링 중 예외가 발생했습니다");
    }

    @Test
    public void testInheritance() {
        // MovableWorld가 World를 상속받는지 확인
        assertTrue(world instanceof World, "MovableWorld는 World를 상속받아야 합니다");

        // 부모 클래스의 메서드 사용 가능한지 확인
        Ball staticBall = new Ball(new Point(50, 50), 15);
        world.add(staticBall);
        assertEquals(1, world.getBallCount(), "상속받은 add 메서드가 작동하지 않습니다");
    }
}
```

### 테스트 실행 방법 및 가이드

1. **의존성 추가** (pom.xml에 추가):
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

   **중요: Mockito 의존성 관련 참고사항**
   - GraphicsContext는 final 클래스이므로 일반적인 mockito-core로는 mock 생성이 불가능합니다
   - **mockito-inline**을 사용해야 final 클래스를 mock할 수 있습니다
   - mockito-inline은 mockito-core의 모든 기능을 포함하므로 mockito-core 대신 사용하면 됩니다

2. **테스트 실행**:
   ```bash
   mvn test
   ```

3. **성공 기준**:
   - 모든 테스트가 PASSED 상태
   - 실패한 테스트가 있다면 해당 기능 재구현 필요

## 다음 장 미리보기

4장에서는 경계와 충돌을 다룹니다:
- 벽과의 충돌 감지
- 반사 구현
- 공 간의 충돌
- 탄성 충돌 시뮬레이션

## 추가 학습 자료

- [게임 물리 기초](https://gafferongames.com/post/integration_basics/)
- [JavaFX AnimationTimer 가이드](https://docs.oracle.com/javafx/2/animation/jfxpub-animation.htm)
- [벡터 수학 기초](https://www.khanacademy.org/math/linear-algebra/vectors-and-spaces)

## 학습 체크포인트

- [ ] MovableBall 클래스를 구현했습니다
- [ ] 시간 기반 애니메이션을 이해했습니다
- [ ] AnimationTimer를 사용할 수 있습니다
- [ ] 벡터 연산을 구현했습니다
- [ ] 델타 타임의 중요성을 이해했습니다