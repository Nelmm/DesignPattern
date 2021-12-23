# 컴포짓 패턴

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

> 장점
>
- 클라이언트가 `Leaf`, `Composite` 구분할 필요 없이 사용할 수 있다.
- OCP를 만족하면서 확장할 수 있다.

> 단점
>
- 지나친 공통화로 인한 객체간 구분이 힘들 수 있다.
