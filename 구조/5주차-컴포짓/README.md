# 컴포짓(Composite) 패턴

> 객체들의 관계를 트리 구조로 구성하여 `부분-전체 계층`을 표현하는 패턴으로, 사용자가 `단일 객체`와 `복합 객체` 모두 동일하게 다루도록 함
>

![Untitled](https://user-images.githubusercontent.com/32676275/147181070-50b5031e-a7b5-4e32-96ef-87bc28f19bee.png)

위의 다이어그램의 각각의 역할은 다음과 같다.

- `Component` → 모든 객체(`부분`, `전체`)의 추상 인터페이스
    - 공통적인 기능을 선언
- `Leaf` → `Component` 인터페이스의 `단일 객체 구현체`
- `Composite` → `Component` 인터페이스의 `전체 관리 객체 구현체`
    - `Leaf`와 똑같은 `Component`의 구현체이지만 `Composite`은 `Leaf`들을 관리하는 기능(add, remove)이 있다.


`컴포짓 패턴`은  전체(`composite`)-부분(`Leaf`) 관계(Ex: Directory-File)를 갖는 객체들 사이의 관계 정의할 때 사용
→ 즉 클라이언트에서 `composite` 구현체를 이용해 `Leaf`를 관리하는 것인데 이것을 클라이언트는 `Component`라는 추상 인터페이스로만 기능을 사용하면 된다.

바로 예제를 보자.

```java
//component
public interface Saramin {
    int getWorkerNum();
    int getMaleNum();
    int getFemaleNum();
}

//composite
public class ITLaboratory implements Saramin {
    private List<Saramin> saramins = new ArrayList<>();
    @Override
    public int getWorkerNum() {return saramins.stream().mapToInt(Saramin::getWorkerNum).sum();}
    @Override
    public int getMaleNum() {return saramins.stream().mapToInt(Saramin::getMaleNum).sum();}
    @Override
    public int getFemaleNum() {return saramins.stream().mapToInt(Saramin::getFemaleNum).sum();}
    public void addSaramin(Saramin saramin) {this.saramins.add(saramin);}
}

//composite
public class PlanDepartment implements Saramin {
    private List<Saramin> saramins = new ArrayList<>();
    @Override
    public int getWorkerNum() {return saramins.stream().mapToInt(Saramin::getWorkerNum).sum();}
    @Override
    public int getMaleNum() {return saramins.stream().mapToInt(Saramin::getMaleNum).sum();}
    @Override
    public int getFemaleNum() {return saramins.stream().mapToInt(Saramin::getFemaleNum).sum();}
    public void addSaramin(Saramin saramin) {this.saramins.add(saramin);}
}

//composite의 composite?
public class Department implements Saramin {
    private List<Saramin> saramins = new ArrayList<>();
    @Override
    public int getWorkerNum() {return saramins.stream().mapToInt(Saramin::getWorkerNum).sum();}
    @Override
    public int getMaleNum() {return saramins.stream().mapToInt(Saramin::getMaleNum).sum();}
    @Override
    public int getFemaleNum() {return saramins.stream().mapToInt(Saramin::getFemaleNum).sum();}
    public void addSaramin(Saramin saramin) {this.saramins.add(saramin);}
}

//leaf
@AllArgsConstructor
public class Planner implements Saramin {
    private String sex;
    @Override
    public int getWorkerNum() {return 1;}
    @Override
    public int getMaleNum() {return sex.equals("male") ? 1 : 0;}
    @Override
    public int getFemaleNum() {return sex.equals("female") ? 1 : 0;}
}

//leaf
@AllArgsConstructor
public class Developer implements Saramin {
    private String sex;
    @Override
    public int getWorkerNum() {return 1;}
    @Override
    public int getMaleNum() {return sex.equals("male") ? 1 : 0;}
    @Override
    public int getFemaleNum() {return sex.equals("female") ? 1 : 0;}
}

//client
public class Main {
    public static void main(String[] args) {
        ITLaboratory itLaboratory = new ITLaboratory();
        Developer d1 = new Developer("male");
        Developer d2 = new Developer("female");
        itLaboratory.addSaramin(d1);
        itLaboratory.addSaramin(d2);

        PlanDepartment planDepartment = new PlanDepartment();
        Planner p1 = new Planner("male");
        Planner p2 = new Planner("female");
        planDepartment.addSaramin(p1);
        planDepartment.addSaramin(p2);

        Department department = new Department();
        department.addSaramin(itLaboratory);
        department.addSaramin(planDepartment);

        client(d1);
        client(itLaboratory);
        client(planDepartment);
        client(department);
    }
    public static void client(Saramin saramin) {
        System.out.printf("worker:%d  male:%d  female:%d\n", saramin.getWorkerNum(), saramin.getMaleNum(), saramin.getFemaleNum());
    }
}

//결과
worker:1  male:1  female:0
worker:2  male:1  female:1
worker:2  male:1  female:1
worker:4  male:2  female:2
```

![Untitled](https://user-images.githubusercontent.com/32676275/147181142-5a265a06-7c25-4744-9d71-ed463084e362.png)

`컴포짓 패턴`에 매칭 시키면 다음과 같다.

- `Saramin` → `Component`
- `Department` → `Composite` (어찌보면 `Composite Of Composite`)
- `ITLaboratory` → `Composite`
- `PlanDepartment` → `Composite`
- `Developer` → `Leaf`
- `Planner` → `Leaf`

위의 코드를 해석하면 `부서`와 `직원`은 모두 `Saramin`에 소속되어 있다.
그리고 `직원`은  `개발자` ,`기획자`가 있고 `개발자`는 `IT연구소` 소속이고 `기획자`는 `기획부서` 소속이다.

![Untitled](https://user-images.githubusercontent.com/32676275/147181148-f63c67cb-9fd1-41a8-8221-b16672a65448.png)

클라이언트에서는 `Leaf`, `Composite`를 구별하지 않고 `Saramin`이라는 인터페이스의 추상 기능만으로 사용할 수 있다. 만약 추후 `IT연구소`에 다른 직원군이 생겨도 `Saramin`을 상속하고 구현만 하면 되고 클라이언트 코드는 변경될 필요가 없기 때문에 `OCP`를 만족하게 된다.

## 장점과 단점

### 장점

- 복잡한 트리 구조를 편하게 사용할 수 있습니다.
  - 클라이언트는 Component의 getPrice 메서드만 사용하면 되기 때문

- 다형성과 재귀를 활용할 수 있습니다.
  - 하나의 `getPrice` 메서드가 구현체 마다 다르게 동작하는 `다형성`과 `Leaf 객체`를 찾기 위해 `DFS`와 같은 재귀를 활용하게 됩니다.

- 클라이언트 코드를 변경하지 않고 새로운 구현체를 추가할 수 있습니다.

컴포짓 패턴을 사용함으로써 `OCP(Open-Closed Principle)` 즉, 개방 폐쇄 원칙을 지키면서 프로그래밍을 할 수 있다는 것을 알 수 있습니다.



### 단점

- 트리를 만들야 하기 때문에 (공통된 인터페이스를 정의해야 하기 때문에) 지나치게 일반화 해야 하는 경우가 생길 수 있습니다.
  - 예를 들어, 가격이 존재하지 않는 객체가 있을 수도 있는데, 이 객체는 getPrice가 굳이 필요하지 않지만 **가방에 넣으려면 Component를 상속받아야 하기 때문에 지나친 일반화가 발생하는 경우**라고 할 수 있습니다.


컴포짓 패턴을 적용하다가 억지로 일반화해야하는 경우가 발생한다면, 해당 구조가 컴포짓 패턴으로 구현하는 게 맞는지 다시 한 번 생각해봐야 합니다.

## 컴포짓 패턴을 사용하는 Swing 라이브러리

`스윙(Swing)`은 자바 언어에서 GUI의 구현하기 위해 제공되는 라이브러리입니다. 자바에서 추구하는 WORE(Wirte Once, Run Everywhere)을 구현하기 위해 `JDK 1.2` 버전부터 사용되었습니다.

```java
import javax.swing.*;

public class SwingExample {

    public static void main(String[] args) {
      	// 프레임을 만듬
        JFrame frame = new JFrame();

      	// 텍스트 필드 박스를 만들고 프레임에 추가
        JTextField textField = new JTextField();
        textField.setBounds(200, 200, 200, 40);
        frame.add(textField);

      	// 버튼을 만들고 프레임에 추가
        JButton button = new JButton("click");
        button.setBounds(200, 100, 60, 40);
        button.addActionListener(e -> textField.setText("Hello Swing"));
        frame.add(button);

      	// 프레임 크기 설정 후 보여주기
        frame.setSize(600, 400);
        frame.setLayout(null);
        frame.setVisible(true);
    }
}
```

여기서 `JFrame, JTextField, JButton`은 컴포짓 패턴으로 이루어져 있습니다. 이 3개의 객체는 전부 `Component	`라는 추상 클래스를 상속받고 있습니다.

프레임의 `add` 메서드는 아래 처럼 되어 있는데, 위 예시의 `Bag(Composite)` 같은 `Component`를 상속하는 객체들을 리스트로 가지고 있는 것을 알 수 있습니다.

```java
public Component add(Component comp) {
        addImpl(comp, null, -1);
        return comp;
}

protected void addImpl(Component comp, Object constraints, int index) {
        synchronized (getTreeLock()) {
          
          	.... 생략
              
            // component라는 리스트에 파라미터로 받은 comp를 넣습니다.
            if (index == -1) {
                component.add(comp);
            } else {
                component.add(index, comp);
            }
            
          	.... 생략
              
        }
    }
```





## 컴포짓 패턴의 방식

컴포짓 패턴에서 `Composite` 클래스는 자식들을 관리하기 위한 추가적인 메서드가 필요합니다. 이러한 메서드의 설계 방식에 따라 2가지 형태의 방식으로 나눌 수 있습니다.

![composite-way](https://user-images.githubusercontent.com/79291114/147174785-16d0a493-79eb-4392-8e90-cf3475c51ca6.png)



### 안정성을 추구하는 방식

안정성을 추구하는 방식은 자식을 다루는 `add(), remove()` 와 같은 메소드들은 오직 `Composite` 만 정의되었다. 그로 인해, **Client는 Leaf와 Composite을 다르게 취급**하고 있습니다. 하지만 **Client에서 Leaf객체가 자식을 다루는 메소드를 호출할 수 없기 때문에, 타입에 대한 안정성**을 얻게 됩니다.

*먼저 예시로 들었던, `Bag`과 `Item`을 생각하면 됩니다.*



### 일관성을 추구하는 방식

일관성을 추구하는 방식은 자식을 다루는 메소드들을 `Composite`가 아닌 `Component`에 정의하는 방식입니다. 그로 인해, `Client`는 `Leaf`와 `Composite`를 일관되게 취급할 수 있습니다. 하지만 `Client`는 `Leaf` 객체가 자식을 다루는 메소드를 호출할 수 있기 때문에, 타입의 안정성을 잃게 됩니다.

`Swing`라이브러리가 일관성을 추구하는 방식으로 되어있습니다.



---

참고 : [코딩으로 학습하는 GoF의 디자인 패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)

[컴포지트 패턴(Composite Pattern) :: 마이구미](https://mygumi.tistory.com/343)
