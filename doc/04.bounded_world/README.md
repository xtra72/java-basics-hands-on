# 4장: 경계가 있는 월드 (Bounded World)

## 학습 목표

이 장을 완료하면 다음을 할 수 있습니다:
- 경계 충돌 감지를 구현할 수 있습니다
- 물리적으로 정확한 반사를 구현할 수 있습니다
- 공 간의 충돌을 처리할 수 있습니다
- 탄성 및 비탄성 충돌을 시뮬레이션할 수 있습니다
- 상속을 통해 BoundedBall을 구현할 수 있습니다

## 핵심 개념

### 4.1 BoundedBall - 경계 처리 추가

**BoundedBall 클래스 설계**

MovableBall을 상속받아 경계 충돌 처리를 추가합니다:

**추가 필드:**
- `minX`, `minY`: 최소 경계 (공의 중심 기준)
- `maxX`, `maxY`: 최대 경계 (공의 중심 기준)

**메서드:**
- 생성자: 경계 초기값을 Double.MIN_VALUE와 Double.MAX_VALUE로 설정 (경계 없음 상태)
- `setBounds(double minX, double minY, double maxX, double maxY)`:
  - 경계 설정 시 반지름 고려
  - 공의 중심이 이동할 수 있는 범위 계산
- `move(double deltaTime)` 오버라이드:
  - 다음 위치 계산
  - 경계 충돌 검사
  - 충돌 시 속도 반전
  - 위치 보정 (경계 안쪽으로)

**구현 힌트:**
```java
// 생성자에서 경계 초기화 (경계 없음 상태)
public BoundedBall(Point center, double radius, Color color) {
    super(center, radius, color);
    // 초기값: 경계가 설정되지 않은 상태를 나타냄
    this.minX = Double.MIN_VALUE;
    this.minY = Double.MIN_VALUE;
    this.maxX = Double.MAX_VALUE;
    this.maxY = Double.MAX_VALUE;
}

// 경계 설정 시 공의 중심이 이동 가능한 범위
public void setBounds(double minX, double minY, double maxX, double maxY) {
    this.minX = minX + getRadius();
    this.maxX = maxX - getRadius();

this.minY = minY + getRadius();
    this.maxY = maxY - getRadius();
}

// move 메서드에서 경계 충돌 처리
@Override
public void move(double deltaTime) {
    // 다음 위치 계산
    Point nextPoint = getCenter().add(getVelocity().multiply(deltaTime));

    // 경계가 설정된 경우에만 충돌 검사
    // Double.MIN_VALUE와 Double.MAX_VALUE는 경계가 없음을 의미
    if (minX > Double.MIN_VALUE && maxX < Double.MAX_VALUE) {
        if (nextPoint.getX() <= minX || nextPoint.getX() >= maxX) {
            // 1. 속도 반전
            // 2. 위치 보정
        }
    }

    if (minY > Double.MIN_VALUE && maxY < Double.MAX_VALUE) {
        if (nextPoint.getY() <= minY || nextPoint.getY() >= maxY) {
            // 1. 속도 반전
            // 2. 위치 보정
        }
    }

    // 부모 클래스의 move 호출
    super.move(deltaTime);
}
```

### 4.2 개선된 충돌 감지와 반사

**CollisionDetector 클래스 설계**

벽과의 충돌을 감지하고 처리하는 유틸리티 클래스입니다:

**내부 클래스 - WallCollision:**
- `Wall` enum: LEFT, RIGHT, TOP, BOTTOM
- `wall`: 충돌한 벽
- `penetration`: 벽 침투 깊이
- 충돌이 없으면 null 반환

**정적 메서드:**
- `checkWallCollision(BoundedBall ball, double minX, double minY, double maxX, double maxY)`:
  - 각 벽과의 충돌 검사
  - 충돌한 벽과 침투 깊이 반환
  - 충돌이 없으면 null 반환

- `resolveWallCollision(BoundedBall ball, WallCollision collision)`:
  - 충돌한 벽에 따라 속도 반전
  - LEFT/RIGHT: x 속도 반전
  - TOP/BOTTOM: y 속도 반전

**반발 계수(Coefficient of Restitution):**
- 0.0: 완전 비탄성 충돌 (끝적이는 효과)
- 1.0: 완전 탄성 충돌 (에너지 손실 없음)
- 0.8: 일반적인 공의 반발

**참고**: 이 장에서는 반발 계수의 개념을 소개하지만, 실제 구현은 단순화를 위해 완전 탄성 충돌(반발 계수 1.0)로 가정합니다. 반발 계수의 실제 적용은 고급 주제로 다음 장에서 다룹니다.

**구현 힌트:**
```java
// 침투 깊이 계산
// 왼쪽 벽: minX - (ballX - radius)
// 오른쪽 벽: (ballX + radius) - maxX

// 속도 반전 (이 장에서는 단순화를 위해 반발 계수 1.0 가정)
ball.setDx(-ball.getDx());
// 추후 반발 계수 적용 시: ball.setDx(-ball.getDx() * restitution);
```

### 4.3 공 간의 충돌

**BallCollision 클래스 설계**

두 공 사이의 충돌을 감지하고 처리하는 클래스입니다:

**정적 메서드:**
- `areColliding(Ball ball1, Ball ball2)`:
  - 두 공의 중심 거리 계산
  - 거리 < 두 반지름의 합이면 충돌

- `resolveElasticCollision(Ball ball1, Ball ball2)`:
  -  MovableBall인 경우,튕겨남
  - 탄성 충돌 처리 (운동량 보존)
  - 충돌 방향 계산
  - 상대 속도 계산
  - 충격량 계산 및 속도 업데이트
  - 겹침 해결

- `separateBalls(Ball ball1, Ball ball2)` (private):
  - 겹친 공을 분리
  - 각 공을 겹침의 절반만큼 밀어냄

**물리 원리:**
1. **운동량 보존**: m₁v₁ + m₂v₂ = m₁v₁' + m₂v₂'
2. **충격량**: I = Δp = mΔv
3. **탄성 충돌**: 에너지 보존

**구현 힌트:**
```java
// 충돌 감지
거리 = √((x₂-x₁)² + (y₂-y₁)²)
충돌 조건: 거리 < r₁ + r₂

// 충돌 방향 벡터 (정규화)
n = (ball2 - ball1) / distance

// 충격량 계산
impulse = 2 * 상대속도 / (질량합)

// 멀어지고 있는지 확인
if (상대속도 · 충돌방향 <= 0) return;
```

### 4.4 BoundedWorld - 충돌 처리 통합

**BoundedWorld 클래스 설계**

MovableWorld를 상속받아 충돌 처리 기능을 통합한 클래스입니다:

**상속 구조:**
```
World (2장: 기본 세계)
  ↓ 상속
MovableWorld (3장: 움직이는 공 관리)
  ↓ 상속
BoundedWorld (4장: 충돌 처리 추가)
```

**MovableWorld로부터 상속받은 메서드들:**
- `update(double deltaTime)`: 오버라이드하여 충돌 처리 추가
- 모든 공들의 이동 처리 기능

**World로부터 상속받은 메서드들:**
- `add(Ball ball)`: 공을 월드에 추가 (2장에서 정의)
- `remove(Ball ball)`: 특정 공 제거
- `clear()`: 모든 공 제거
- `getBalls()`: 공 목록 반환 (방어적 복사)
- `getBallCount()`: 현재 공의 개수 반환
- `getWidth()`, `getHeight()`: 월드 크기 반환
- `draw(GraphicsContext gc)`: 모든 공을 화면에 그리기

**재정의 메서드들:**
- `add(Ball ball)`: 공 추가될때 영역 설정
-
**메서드:**
- `update(double deltaTime)`: 매 프레임 업데이트
  1. 모든 공 이동
  2. 벽과의 충돌 검사 및 처리
  3. 공 간의 충돌 검사 및 처리

**구현 순서:**
1. **이동 단계**: 모든 MovableBall의 move() 호출
2. **벽 충돌 단계**:
   - BoundedBall인지 확인
   - CollisionDetector.checkWallCollision() 호출
   - 충돌 시 resolveWallCollision() 호출
3. **공 충돌 단계**:
   - 이중 루프로 모든 쌍 검사 (i < j)
   - areColliding() 호출
   - 공 충돌시 resolveCollision() 호출

**최적화 힌트:**
```java
// 이중 충돌 방지
for (int i = 0; i < balls.size(); i++) {
    for (int j = i + 1; j < balls.size(); j++) {
        // 각 쌍을 한 번만 검사
    }
}
```

## 실습 과제

### Lab 4-1: BoundedBall 구현
`BoundedBall` 클래스를 구현하고 테스트하세요:
- MovableBall을 상속
- 경계 충돌 감지
- 반사 구현
- 위치 보정

**테스트 코드:**
```java
@Test
public void testWallBounce() {
    BoundedBall ball = new BoundedBall(new Point(50, 300), 20);
    ball.setBounds(0, 0, 800, 600);
    ball.setVelocity(new Vector2D(-100, 0)); // 왼쪽으로 이동

    // 충돌 전
    assertTrue(ball.getVelocity().getX() < 0);

    // 충분히 이동시켜 충돌 발생
    for (int i = 0; i < 10; i++) {
        ball.move(0.1);
    }

    // 충돌 후 방향 반전
    assertTrue(ball.getVelocity().getX() > 0);
}
```

### Lab 4-2: 다양한 충돌 시나리오
다양한 충돌 상황 구현:
- 벽과의 다중 충돌
- 코너 충돌 처리
- 연속적인 공 간 충돌

### Lab 4-3: 충돌 시뮬레이션
여러 공의 충돌 시뮬레이션:
- 10개의 공을 랜덤 위치에 생성
- 다양한 크기와 속도
- 충돌 효과 시각화 (색상 변화, 사운드 등)

### Lab 4-4: 고급 충돌 처리
특수한 충돌 상황 처리:
- 코너 충돌
- 동시 다중 충돌
- 고속 충돌 (터널링 방지)

**AdvancedCollisionApp 구현 가이드**

고급 충돌 시뮬레이션 앱을 만드세요:

**필요한 기능:**
1. BoundedWorld 사용
2. UI 컨트롤:
   - 중력 활성화 체크박스
   - 공 개수 조절 슬라이더
3. 다양한 크기의 공 생성
4. AnimationTimer로 게임 루프 구현

**추가 메서드:**
- `createBalls()`: 랜덤한 크기와 속도의 공 생성
- `applyGravity()`: 모든 공에 중력 적용 (dy 증가)
- `render()`: Canvas에 그리기

**중력 구현 힌트:**
```java
// 중력 가속도 (예: 500 pixels/s²)
private static final double GRAVITY = 500;

// 모든 공에 적용
for (Ball ball : world.getBalls()) {
    if (ball instanceof MovableBall) {
        MovableBall movable = (MovableBall) ball;
        movable.setDy(movable.getVelocity().getY() + GRAVITY * deltaTime);
    }
}
```

## JUnit 테스트 예제

### 공 간의 충돌 테스트

```java
public class BallCollisionTest {

    @Test
    public void testBallCollisionDetection() {
        Ball ball1 = new Ball(new Point(100, 100), 30);
        Ball ball2 = new Ball(new Point(150, 100), 30);

        assertTrue(BallCollision.areColliding(ball1, ball2));

        ball2.moveTo(new Point(200, 100));
        assertFalse(BallCollision.areColliding(ball1, ball2));
    }

    @Test
    public void testElasticCollision() {
        MovableBall ball1 = new MovableBall(new Point(100, 100), 20);
        ball1.setVelocity(new Vector2D(100, 0));

        MovableBall ball2 = new MovableBall(new Point(140, 100), 20);
        ball2.setVelocity(new Vector2D(-100, 0));

        // 충돌 전 총 운동량
        double totalMomentumBefore =
            ball1.getRadius() * ball1.getVelocity().getX() +
            ball2.getRadius() * ball2.getVelocity().getX();

        BallCollision.resolveCollision(ball1, ball2);

        // 충돌 후 총 운동량
        double totalMomentumAfter =
            ball1.getRadius() * ball1.getVelocity().getX() +
            ball2.getRadius() * ball2.getVelocity().getX();

        // 운동량 보존 확인
        assertEquals(totalMomentumBefore, totalMomentumAfter, 0.001);
    }

    @Test
    public void testBallSeparation() {
        MovableBall ball1 = new MovableBall(new Point(100, 100), 20);
        MovableBall ball2 = new MovableBall(new Point(125, 100), 20);

        // 겹친 상태 (거리: 25, 반지름 합: 40, 겹침: 15)
        assertTrue(BallCollision.areColliding(ball1, ball2), "공들이 겹쳐있어야 합니다");

        BallCollision.separateBalls(ball1, ball2);

        // 분리 후 더 이상 겹치지 않아야 함
        assertFalse(BallCollision.areColliding(ball1, ball2),
                   "분리 후에도 공들이 겹쳐있습니다");

        // 공들 사이의 거리가 반지름 합과 같거나 커야 함
        Point center1 = ball1.getCenter();
        Point center2 = ball2.getCenter();
        double distance = Math.sqrt(Math.pow(center2.getX() - center1.getX(), 2) +
                                   Math.pow(center2.getY() - center1.getY(), 2));
        assertTrue(distance >= 40, "분리 후 거리가 반지름 합보다 작습니다");
    }
}
```

### 벽 충돌 테스트

```java
public class WallCollisionTest {

    @Test
    public void testWallCollisionDetection() {
        BoundedBall ball = new BoundedBall(new Point(50, 300), 20);

        CollisionDetector.WallCollision collision = CollisionDetector.checkWallCollision(
            ball, 0, 0, 800, 600
        );

        assertNull(collision, "충돌이 없을 때는 null이 반환되어야 합니다");

        ball.moveTo(new Point(15, 300)); // 왼쪽 벽에 닿음
        collision = CollisionDetector.checkWallCollision(
            ball, 0, 0, 800, 600
        );

        assertEquals(CollisionDetector.Wall.LEFT, collision.getWall());
        assertEquals(5, collision.getPenetration(), 0.001);
    }
}
```

## 자가 평가 문제

1. **반발 계수(coefficient of restitution)란?**
   - 충돌 후 상대 속도와 충돌 전 상대 속도의 비율
   - 0: 완전 비탄성, 1: 완전 탄성

2. **운동량 보존 법칙이란?**
   - 외력이 없을 때 총 운동량은 보존됨
   - m1v1 + m2v2 = m1v1' + m2v2'

3. **터널링(tunneling) 문제란?**
   - 빠른 속도로 움직이는 물체가 벽을 통과하는 현상
   - 연속 충돌 감지로 해결

4. **충돌 응답의 두 단계는?**
   - 충돌 감지 (Detection)
   - 충돌 해결 (Resolution)

## 자주 하는 실수와 해결 방법

### 1. 충돌 후 물체가 붙어있음
```java
// 잘못된 코드 - 겹침을 해결하지 않음
if (areColliding(ball1, ball2)) {
    resolveCollision(ball1, ball2);
}

// 올바른 코드 - 겹침 해결 포함
if (areColliding(ball1, ball2)) {
    resolveCollision(ball1, ball2);
    separateBalls(ball1, ball2); // 추가!
}
```

### 2. 이중 충돌 처리
```java
// 잘못된 코드 - 같은 충돌을 여러 번 처리
for (Ball ball : balls) {
    checkCollisions(ball, balls);
}

// 올바른 코드 - 각 쌍을 한 번만 검사
for (int i = 0; i < balls.size(); i++) {
    for (int j = i + 1; j < balls.size(); j++) {
        checkCollision(balls.get(i), balls.get(j));
    }
}
```

### 3. 부정확한 반사
```java
// 잘못된 코드 - 속도 변경 전 위치 보정 없음
ball.setVelocity(new Vector2D(-ball.getVelocity().getX(), ball.getVelocity().getY()));

// 올바른 코드 - 위치 보정 후 속도 반전
if (ball.getX() - ball.getRadius() < 0) {
    ball.setX(ball.getRadius());  // 위치 보정
    ball.setVelocity(new Vector2D(-ball.getVelocity().getX(), ball.getVelocity().getY())); // 속도 반전
}
```

## 구현 검증용 테스트 코드

아래 테스트 코드를 사용하여 구현한 클래스들이 올바르게 작동하는지 확인하세요:

### BoundedBall 클래스 테스트

```java
import javafx.scene.paint.Color;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class BoundedBallTest {

    private BoundedBall ball;
    private BoundedWorld world;

    @BeforeEach
    public void setUp() {
        ball = new BoundedBall(new Point(100, 100), 20, Color.RED);
        world = new BoundedWorld(800, 600);
        world.add(ball);
    }

    @Test
    public void testBoundedBallCreation() {
        // 부모 클래스 속성 확인
        Point center = ball.getCenter();
        assertEquals(100, center.getX(), 0.001, "X 좌표가 올바르게 설정되지 않았습니다");
        assertEquals(100, center.getY(), 0.001, "Y 좌표가 올바르게 설정되지 않았습니다");
        assertEquals(20, ball.getRadius(), 0.001, "반지름이 올바르게 설정되지 않았습니다");
        assertEquals(Color.RED, ball.getColor(), "색상이 올바르게 설정되지 않았습니다");
    }

    @Test
    public void testInitialBounds() {
        // 생성 시 경계 초기값 확인 (경계 없음 상태)
        BoundedBall unboundedBall = new BoundedBall(new Point(50, 50), 10, Color.BLUE);

        // 경계가 설정되지 않은 상태에서는 자유롭게 이동 가능
        unboundedBall.setVelocity(new Vector2D(100, 100));
        unboundedBall.move(1.0);

        // 경계가 없으므로 위치가 그대로 변경됨
        Point newCenter = unboundedBall.getCenter();
        assertEquals(150, newCenter.getX(), 0.001, "경계가 없을 때 X 이동이 제한되면 안됩니다");
        assertEquals(150, newCenter.getY(), 0.001, "경계가 없을 때 Y 이동이 제한되면 안됩니다");
    }

    // 반발 계수는 다음 장에서 구현
    /*
    @Test
    public void testRestitutionSetterGetter() {
        ball.setRestitution(0.9);
        assertEquals(0.9, ball.getRestitution(), 0.001, "반발 계수 설정이 올바르지 않습니다");

        // 유효하지 않은 반발 계수
        assertThrows(IllegalArgumentException.class, () -> {
            ball.setRestitution(-0.1); // 음수
        }, "음수 반발 계수에 대해 예외가 발생하지 않았습니다");

        assertThrows(IllegalArgumentException.class, () -> {
            ball.setRestitution(1.1); // 1보다 큰 값
        }, "1보다 큰 반발 계수에 대해 예외가 발생하지 않았습니다");
    }
    */


    @Test
    public void testWallCollisionLeft() {
        ball.moveTo(new Point(10, ball.getCenter().getY())); // 왼쪽 벽 근처
        ball.setVelocity(new Vector2D(-50, 0)); // 왼쪽으로 이동

        Vector2D initialVelocity = ball.getVelocity();
        ball.move(0.1); // 충돌 발생

        assertTrue(ball.getVelocity().getX() > 0, "왼쪽 벽 충돌 후 X 속도가 양수가 되어야 합니다");
    }

    @Test
    public void testWallCollisionRight() {
        ball.moveTo(new Point(780, ball.getCenter().getY())); // 오른쪽 벽 근처 (800 - 20)
        ball.setVelocity(new Vector2D(50, 0)); // 오른쪽으로 이동

        ball.move(0.1); // 충돌 발생

        assertTrue(ball.getVelocity().getX() < 0, "오른쪽 벽 충돌 후 X 속도가 음수가 되어야 합니다");
    }

    @Test
    public void testWallCollisionTop() {
        ball.moveTo(new Point(ball.getCenter().getX(), 10)); // 위쪽 벽 근처
        ball.setVelocity(new Vector2D(0, -50)); // 위쪽으로 이동

        ball.move(0.1); // 충돌 발생

        assertTrue(ball.getVelocity().getY() > 0, "위쪽 벽 충돌 후 Y 속도가 양수가 되어야 합니다");
    }

    @Test
    public void testWallCollisionBottom() {
        ball.moveTo(new Point(ball.getCenter().getX(), 580)); // 아래쪽 벽 근처 (600 - 20)
        ball.setVelocity(new Vector2D(0, 50)); // 아래쪽으로 이동

        ball.move(0.1); // 충돌 발생

        assertTrue(ball.getVelocity().getY() < 0, "아래쪽 벽 충돌 후 Y 속도가 음수가 되어야 합니다");
    }

    @Test
    public void testInheritance() {
        // BoundedBall이 MovableBall을 상속받는지 확인
        assertTrue(ball instanceof MovableBall, "BoundedBall은 MovableBall을 상속받아야 합니다");
        assertTrue(ball instanceof PaintableBall, "BoundedBall은 PaintableBall을 상속받아야 합니다");
        assertTrue(ball instanceof Ball, "BoundedBall은 Ball을 상속받아야 합니다");

        // 부모 클래스의 메서드 사용 가능한지 확인
        ball.setVelocity(new Vector2D(100, 75));
        Vector2D velocity = ball.getVelocity();
        assertEquals(100, velocity.getX(), 0.001, "상속받은 속도 설정이 작동하지 않습니다");
        assertEquals(75, velocity.getY(), 0.001, "상속받은 속도 설정이 작동하지 않습니다");

        Point oldCenter = ball.getCenter();
        ball.move(0.1);
        Point newCenter = ball.getCenter();
        assertEquals(oldCenter.getX() + 10, newCenter.getX(), 0.001, "상속받은 move 메서드가 작동하지 않습니다");
        assertEquals(oldCenter.getY() + 7.5, newCenter.getY(), 0.001, "상속받은 move 메서드가 작동하지 않습니다");
    }
}
```

### CollisionDetector 클래스 테스트

```java
import javafx.scene.paint.Color;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class CollisionDetectorTest {

    @Test
    public void testWallCollisionDetection() {
        BoundedBall ball = new BoundedBall(new Point(15, 100), 20, Color.RED);

        // 왼쪽 벽 충돌 (x - radius < 0)
        CollisionDetector.WallCollision collision = CollisionDetector.checkWallCollision(ball, 0, 0, 800, 600);
        assertNotNull(collision, "벽 충돌이 감지되지 않았습니다");
        assertEquals(CollisionDetector.Wall.LEFT, collision.getWall(), "잘못된 벽이 감지되었습니다");
        assertEquals(5, collision.getPenetration(), 0.001, "침투 깊이가 잘못 계산되었습니다");

        // 오른쪽 벽 충돌
        ball.moveTo(new Point(785, 100)); // x + radius > 800
        collision = CollisionDetector.checkWallCollision(ball, 0, 0, 800, 600);
        assertEquals(CollisionDetector.Wall.RIGHT, collision.getWall(), "오른쪽 벽 충돌이 감지되지 않았습니다");
        assertEquals(5, collision.getPenetration(), 0.001, "오른쪽 벽 침투 깊이가 잘못되었습니다");

        // 위쪽 벽 충돌
        ball.moveTo(new Point(100, 15)); // y - radius < 0
        collision = CollisionDetector.checkWallCollision(ball, 0, 0, 800, 600);
        assertEquals(CollisionDetector.Wall.TOP, collision.getWall(), "위쪽 벽 충돌이 감지되지 않았습니다");

        // 아래쪽 벽 충돌
        ball.moveTo(new Point(100, 585)); // y + radius > 600
        collision = CollisionDetector.checkWallCollision(ball, 0, 0, 800, 600);
        assertEquals(CollisionDetector.Wall.BOTTOM, collision.getWall(), "아래쪽 벽 충돌이 감지되지 않았습니다");

        // 충돌하지 않는 경우
        ball.moveTo(new Point(400, 300));
        collision = CollisionDetector.checkWallCollision(ball, 0, 0, 800, 600);
        assertNull(collision, "벽과 충돌하지 않는데 충돌이 감지되었습니다");
    }

    // 벽 충돌 후 속도 반전 테스트
    @Test
    public void testWallCollisionResponse() {
        BoundedBall ball = new BoundedBall(new Point(10, 300), 20, Color.RED);
        ball.setVelocity(new Vector2D(-50, 0)); // 왼쪽으로 이동

        // 왼쪽 벽 충돌 감지
        CollisionDetector.WallCollision collision = CollisionDetector.checkWallCollision(ball, 0, 0, 800, 600);
        assertEquals(CollisionDetector.Wall.LEFT, collision.getWall(), "왼쪽 벽 충돌이 감지되지 않았습니다");

        // 벽 충돌 해결
        CollisionDetector.resolveWallCollision(ball, collision);

        // 속도가 반전되었는지 확인
        assertTrue(ball.getVelocity().getX() > 0, "왼쪽 벽 충돌 후 X 속도가 양수가 되어야 합니다");
    }
}
```

### BoundedWorld 클래스 테스트

```java
import javafx.scene.canvas.GraphicsContext;
import javafx.scene.paint.Color;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import static org.junit.jupiter.api.Assertions.*;

public class BoundedWorldTest {

    private BoundedWorld world;

    @BeforeEach
    public void setUp() {
        world = new BoundedWorld(800, 600);
    }

    @Test
    public void testBoundedWorldCreation() {
        assertEquals(800, world.getWidth(), "World 너비가 올바르지 않습니다");
        assertEquals(600, world.getHeight(), "World 높이가 올바르지 않습니다");
    }

    @Test
    public void testWallCollisionHandling() {
        BoundedBall ball = new BoundedBall(new Point(10, 300), 20, Color.RED);
        ball.setVelocity(new Vector2D(-50, 0)); // 왼쪽으로 이동 (벽 충돌 예정)
        world.add(ball);

        world.update(0.1);

        // 벽 충돌 후 속도가 반대 방향으로 바뀌어야 함
        assertTrue(ball.getVelocity().getX() > 0, "벽 충돌 후 X 속도가 양수가 되어야 합니다");

        // 공이 벽 안쪽에 위치해야 함
        assertTrue(ball.getCenter().getX() >= ball.getRadius(), "공이 왼쪽 벽을 벗어났습니다");
    }

    @Test
    public void testBallBallCollisionHandling() {
        BoundedBall ball1 = new BoundedBall(new Point(100, 300), 20, Color.RED);
        BoundedBall ball2 = new BoundedBall(new Point(200, 300), 20, Color.BLUE);

        ball1.setVelocity(new Vector2D(100, 0)); // 오른쪽으로 이동
        ball2.setVelocity(new Vector2D(-50, 0)); // 왼쪽으로 이동 (충돌 예정)

        world.add(ball1);
        world.add(ball2);

        // 충돌 전 총 운동량
        double initialMomentum = ball1.getVelocity().getX() + ball2.getVelocity().getX();

        world.update(0.5); // 충분한 시간으로 충돌 발생

        // 충돌 후 운동량 보존 확인
        double finalMomentum = ball1.getVelocity().getX() + ball2.getVelocity().getX();
        assertEquals(initialMomentum, finalMomentum, 1.0, "운동량이 보존되지 않았습니다");

        // 공들이 분리되어야 함
        assertFalse(CollisionDetector.areColliding(ball1, ball2),
                   "충돌 처리 후에도 공들이 겹쳐있습니다");
    }

    @Test
    public void testMultipleBallCollisions() {
        // 여러 공이 한 번에 충돌하는 상황
        BoundedBall ball1 = new BoundedBall(new Point(100, 300), 15, Color.RED);
        BoundedBall ball2 = new BoundedBall(new Point(200, 300), 15, Color.BLUE);
        BoundedBall ball3 = new BoundedBall(new Point(300, 300), 15, Color.GREEN);

        ball1.setVelocity(new Vector2D(150, 0));
        ball2.setVelocity(new Vector2D(0, 0));
        ball3.setVelocity(new Vector2D(-100, 0));

        world.add(ball1);
        world.add(ball2);
        world.add(ball3);

        // 여러 번 업데이트하여 모든 충돌 처리
        for (int i = 0; i < 10; i++) {
            world.update(0.01);
        }

        // 모든 공이 분리되어야 함
        assertFalse(CollisionDetector.areColliding(ball1, ball2), "공 1과 2가 겹쳐있습니다");
        assertFalse(CollisionDetector.areColliding(ball2, ball3), "공 2와 3이 겹쳐있습니다");
        assertFalse(CollisionDetector.areColliding(ball1, ball3), "공 1과 3이 겹쳐있습니다");
    }

    @Test
    public void testCornerBounce() {
        // 모서리에서의 반사 테스트
        BoundedBall ball = new BoundedBall(new Point(25, 25), 20, Color.YELLOW);
        ball.setVelocity(new Vector2D(-50, -50));
        world.add(ball);

        world.update(0.1);

        // 모서리 충돌 후 두 방향 모두 반사되어야 함
        assertTrue(ball.getVelocity().getX() > 0, "모서리 충돌 후 X 속도가 양수가 되어야 합니다");
        assertTrue(ball.getVelocity().getY() > 0, "모서리 충돌 후 Y 속도가 양수가 되어야 합니다");
    }

    @Test
    public void testInheritance() {
        // BoundedWorld가 MovableWorld를 상속받는지 확인
        assertTrue(world instanceof MovableWorld, "BoundedWorld는 MovableWorld를 상속받아야 합니다");
        assertTrue(world instanceof World, "BoundedWorld는 World를 상속받아야 합니다");

        // 부모 클래스 메서드 사용 가능한지 확인
        BoundedBall ball = new BoundedBall(new Point(100, 100), 20, Color.CYAN);
        world.add(ball);
        assertEquals(1, world.getBallCount(), "상속받은 add 메서드가 작동하지 않습니다");
    }

    @Test
    public void testRender() {
        GraphicsContext gc = Mockito.mock(GraphicsContext.class);
        BoundedBall ball = new BoundedBall(new Point(100, 100), 20, Color.MAGENTA);
        world.add(ball);

        assertDoesNotThrow(() -> {
            world.draw(gc);
        }, "BoundedWorld 렌더링 중 예외가 발생했습니다");
    }
}
```

### WallCollision 내부 클래스 사용 예시

```java
// WallCollision은 CollisionDetector의 내부 클래스입니다
// 사용 예시:

@Test
public void testWallCollisionUsage() {
    BoundedBall ball = new BoundedBall(new Point(10, 100), 20);

    // CollisionDetector를 통해 WallCollision 객체를 얻음
    CollisionDetector.WallCollision collision =
        CollisionDetector.checkWallCollision(ball, 0, 0, 800, 600);

    // 충돌 정보 확인
    if (collision != null) {
        CollisionDetector.Wall wall = collision.getWall();
        double penetration = collision.getPenetration();

        System.out.println("충돌한 벽: " + wall);
        System.out.println("침투 깊이: " + penetration);
    }
}
```

## 다음 장 미리보기

5장에서는 추상 데이터 타입을 다룹니다:
- 추상 클래스의 개념과 활용
- Bounds 추상 클래스 설계와 구현
- Vector 클래스 확장
- 코드 재사용성 향상
- 디자인 패턴 적용

## 추가 학습 자료

- [2D 충돌 감지와 응답](https://www.metanetsoftware.com/technique/tutorialA.html)
- [게임 물리 엔진 기초](https://brm.io/game-physics-for-beginners/)
- [Real-Time Collision Detection](https://realtimecollisiondetection.net/)

## 학습 체크포인트

- [ ] BoundedBall 클래스를 구현했습니다
- [ ] 벽과의 충돌을 처리할 수 있습니다
- [ ] 공 간의 충돌을 감지하고 해결할 수 있습니다
- [ ] 반발 계수의 개념을 이해했습니다
- [ ] 운동량 보존 법칙을 구현했습니다