# 비지터 패턴

기존 코드를 변경하지 않고 새로운 기능을 추가하는 방법.

![스크린샷 2022-02-16 오후 7.02.10.png](%E1%84%87%E1%85%B5%E1%84%8C%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%91%E1%85%A2%E1%84%90%207fa2b/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-02-16_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_7.02.10.png)

## 문제점

구체적 클래스가 존재하고, 구체적인 클래스는 여러 가지의 Client에 따라서 구체적 클래스의 메소드의 아웃풋이 변경되야하는 상황임. 이럴때의 문제는 그 구체적인 클래스에 각 클라이언트에 따른 `if-else`문이 작성되야하는 상황이 생길 수 있음. 그러면 client가 늘어나면 늘어날수록 계속 `if-else` 가 중첩되야하는데, 이러면 클라이언트가 늘어나는것에 따라서 코드 수정을 불가피하게 계속 해야하는데, 이러면 가독성이 떨어지게 된다. 

![https://refactoring.guru/images/patterns/diagrams/visitor/problem2-en.png?id=f53c592d755890f5d027](https://refactoring.guru/images/patterns/diagrams/visitor/problem2-en.png?id=f53c592d755890f5d027)

지도파일에서 각 장소의 카테고리별 class가 존재했는데 XML으로 각 장소의 정보를 파일을 내보내야하는 메소드를 만들어야하는데, 각 장소의 클래스에다가 모두 XML export하는 method를 구현하려면 모두 메소드를 집어넣야하는 상황이 생기는 경우와 같은 상황을 해결하기 위해서 나타났다.

## 해결 방식

1. visitor 인터페이스를 구성하고 ⇒ 인터페이스에 그 구체적 클래스를 파라미터를 사용하는 메소드 구현.
2. 구체적 클래스를 순회하면서 각 경우에 대해서 if else로 메소드 사용해야하는 구조로 구현

하지만.. 그런 경우 위의 문제점을 명확하게 해결하지는 않기 때문에, **더블 디스패치** 사용시...

1. 구체적 클래스에 visitor를 파라미터로 사용하는 공통 method 구현
2. 구체적 visitor생성후 거기에  구체적 클래스를 넣어서 작동시키게하면 그대로 작동함.

### 더블 디스 패치란?

오버로드된 메소드와 함께 동적 바인딩을 사용하는 방식

원래 방식대로 하면 Shape가 우선되서 실제 Shape interface의 메소드인 exporting Shape가 나온다.

```java
class Exporter{ //visitor로만 구현하면.. 
    method export(s: Shape)
        print("Exporting shape")
    method export(d: Dot)
      print("Exporting dot")
}

foreach (Node node in graph) 
//인터페이스로 찾는거다보니까 구체적인 class를 일일히 대조해야함.
    if (node instanceof City)
        exportVisitor.doForCity((City) node)
    if (node instanceof Industry)
        exportVisitor.doForIndustry((Industry) node)
    // ...
}
```

그 대신 더블 디스패치를 이용하면...

```java
public interface Visitor {
    String visitDot(Dot dot);
}

public class XMLExportVisitor implements Visitor {

    public String export(Shape... args) {
        for (Shape shape : args) {
            //여기서는 this가 visitor임. 
            shape.accept(this)
        }
    }

    public String visitDot(Dot d) {
            return "<dot>" + "\n"
    }
}

public interface Shape {
    String accept(Visitor visitor);
}

public class Dot implements Shape {
    private int id;

        @Override
    public String accept(Visitor visitor) {
                //여기서는 this가 Shape임.
        return visitor.visitDot(this);
    }
}
```

굳이 타입채킹을 하지 않아도 compling시에 코드가 스스로 무슨 타입인지 파악해서 자동적으로 코드 매핑해줘서 for문 if-else문등의 문제점을 해결해준다. 

## 장-단점

### 장점

- 코드변경 없이도, 기능 추가 가능.
- 추가 기능을 한쪽에 다 몰아 둘수 있다.

### 단점

- 엘리먼트의 삭제시, 많은 변경 필요 → OCP, SRP 원칙이 깨질 가능성이 커짐

## 다른 패턴과의 관계성

- visitor 패턴은 command 패턴의 강력한 버전임. 이객체가 서로 다른 클래스의 다양한 객체를 실행시킬수 있음.
- 전체 composite 트리를 실행하는데 visitor를 사용할 수 있음.
- iterator와 함께 복잡한 데이터구조를 순회하는데 visitor패턴을 사용할 수 있는데, 모든 클래스가 다 다르더라도 그안의 있는 파일들을 실행하면서,

## 어디서 사용하는지?

- 자바
  - fileVisitor, SimpleFileVisitor
  - AnnotationValueVisitor
  - ElementVisitor
- 스프링
  - BeanDefinitionVisitor

출처: 

[https://refactoring.guru/design-patterns/visitor](https://refactoring.guru/design-patterns/visitor)

[https://refactoring.guru/design-patterns/visitor/java/example](https://refactoring.guru/design-patterns/visitor/java/example#example-0--shapes-CompoundShape-java)