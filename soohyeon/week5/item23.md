# 아이템 23 태그 달린 클래스보다는 클래스 계층구조를 활용하라

# 태그 달린 클래스

태그 달린 클래스가 대체 뭘까?

> 두 가지 이상의 기능을 갖고 있으며, 그 중 어떤 기능을 제공하는지 표시하는 태그가 달린 클래스를 의미한다.
> 

예시를 통해서 더 자세히 알아보자.

```java
class Figure {
  enum Shape { //두가지 기능
    RECTANGLE, CIRCLE
  };

  final Shape shape; // 태그 필드 

  // 사각형(RECTANGLE)일 때
  double length;
  double width;

  // 원(CIRCLE)일 때
  double radius;

  // 첫번째 기능- 사각형용 생성자
  Figure(double length, double width) {
    shape = Shape.RECTANGLE;
    this.length = length;
    this.width = width;
  }

  // 두번째 기능 - 원용 생성자
  Figure(double radius) {
    shape = Shape.CIRCLE;
    this.radius = radius;
  }

  double area() {
    switch(shape) {
			//각 기능을 switch 문으로 처리
      case RECTANGLE:
        return length * width;
      case CIRCLE:
        return Math.PI * (radius * radius);
      default:
        throw new AssertionError(shape);
    }
  }
}
```

## 단점

코드를 읽어보면 알겠지만 태그 달린 클래스에는 단점이 많다. 

1. 쓸데없는 코드가 많다.
열거 타입선언, 태그 필드, 스위치 문 등 쓸데없는 코드가 많다. 
2. 가독성이 떨어진다.
여러 구현이 한 클래스에 혼재되어있어 가독성이 떨어진다.
3. 메모리 사용량이 많다.
다른 의미를 위한 코드가 항상 같이 실행되므로 메모리를 상대적으로 많이 사용한다.
4. 쓰이지 않는 코드까지 실행한다.
필드들을 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야 한다. 때문에 쓰지 않는 필드까지 초기화하는 불필요한 코드가 늘어난다.
ex) 위에서 원만 사용할 경우에도 사각형을 초기화해야 한다.
5. 문제가 늦게 발견된다.
생성자가 태그 필드를 설정하고 해당 의미에 쓰이는 데이터 필드들을 초기화 하는 데 컴파일러가 할 수 있는 일은 거의 없다. 이 때문에 엉뚱한 필드를 초기화해도 런타임에서야 문제가 드러난다. (어떤문제??- 너무 많은 필드들을 초기화해서 생기는 런타임 문제??)
ex) 새로운 의미를 추가할 때마다 모든 switch 문을 찾아 새 의미를 처리하는 코드를 추가해야 하는데 이 때 하나라도 빠뜨리면 런타임 에러가 발생한다!
6. 의미를 알 수 없다.
인스턴스의 타입만으로는 현재 나타나는 의미를 알 수 있는 방법이 없다. 

<aside>
💡 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적이다.

</aside>

이 문제들은 다 클래스 계층 구조를 활용하는 서브 타이핑으로 해결이 가능하다. 

# 클래스 계층구조를 활용하는 subtyping

## 태그 달린 클래스를 클래스 계층 구조로 변경하는 법

1. 계층 구조의 root가 될 추상 클래스를 정의한다. 
태그 값에 따라 동작이 달라지는 메서드들은 루트 클래스의 추상 메서드로 선언한다.
주어진 예시 코드에서는 area가 루트 클래스의 추상 메서드가 된다. 
2. 태그 값에 상관없이 동작이 일정한 메서드들은 루트 클래스에 일반 메서드로  추가한다. 
Figure 클래스에서는 태그값에 상관없는 메서드가 하나도 없어 그대로 냅둔다. 
그 후, 모든 하위 클래스에서 공통으로 사용ㅇ하는 데이터 필드들도 전부 루트 클래스로 올린다. 
→ 그 결과 루트 클래스에는 추상 메서드인 area 하나만 남게 된다. 
3. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다. 
figure 클래스에서 확장한 원 클래스와 사각형 클래스를 만들면 된다. 
4. 루트 클래스가 정의한 추상 메서드를 각자의 의미에 맞게 구현한다. 

```java
abstract class Figure {
	abstract double radius; //1. 추상 메서드
}

//3. 구체 클래스
class Circle extends Figure {
  final dobule radius;
  Circle(double radius) {
    this.radius = radius;
  }
  @Override
//4. 각자의 의미에 맞게 구현
  double area() {
    return Math.PI * (radius * radius);
  }
}

class Rectangle extends Figure {
  final double length;
  final double width;
  Rectangle(double length, double width) {
    this.length = length;
    this.width = width;
  }
  @Override
  double area() {
    return length * width;
  }
}
	
```

### 장점

1. 쓸데없는 코드가 모두 사라졌다.
2. 살아남은 필드들이 모두 final이다. 
3. 각 클래스의 생성자가 모든 필드들을 남김없이 초기화하고 추상 메서드를 모두 구현했는지 컴파일러가 확인해준다. 
태그 달린 클래스에서 switch문을 빼먹고 필요없는 필드를 초기화해주는 문제가 해결된다. 
4. 루트 클래스의 코드를 건드리지 않고도 독립적으로 계층구조를 확장하고 함께 사용할 수 있다. 
이 덕에 타입 사이의 자연스로운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일 타임 타입 검사 능력을 높여준다는 장점도 있다. 

예를 들어 위 코드에서 정사각형 클래스를 추가한다고 해보자.

이 경우, 계층 구조를 활용하여 Rectangle을 확장하여 루트 클래스를 건드리지 않고 정사각형 클래스를 만들 수 있다. 

```java
class Square extends Rectangle{
	Square(double side) {
		super(side, side);
	}
}
```

## 결론

<aside>
💡 태그 달린 클래스는 지양하자.
태그 필드가 등장한다면 이를 계층 구조를 이용한 방법으로 바꾸는 방식으로 생각해보자.

</aside>
