## 1. 메멘토 패턴(Memento Pattern)이란?

메멘토 패턴은 **캡슐화를 유지하면서 객체 내부 상태를 외부에 저장하는 패턴**

[![memento](https://user-images.githubusercontent.com/79291114/151184142-6a60b443-3ba4-4eea-855a-981aaa82d9fd.PNG)](https://user-images.githubusercontent.com/79291114/151184142-6a60b443-3ba4-4eea-855a-981aaa82d9fd.PNG)

- `Originator` : 객체의 정보를 가지고 있는 오리지널 객체
  - 제 3자가 객체 내부 정보를 알지 못하게 하기 위하여 **Memento를 만들 수 있는 메서드가 필요**함
- `Memento` : 오리지널 객체의 정보를 담는 불변 객체
- `CareTaker` : 오리지널이 반환한 **Memento를 가지고 있다가 필요할 때 Memento의 값을 Originator에게 전달**함





## 2. 메멘토 패턴 적용 예제

### 2-1. 요구사항

이력서 작성 App 예제

App은 이력서 작성/수정을 할 수 있고, 수정 시 페이지를 이탈하면 이전에 저장했던 이력서로 복원이 가능하도록 설계한다.

### 2-2. 샘플 코드

1. Memento → 원본 Resume 객체 정보를 보관하는 Snapshot
2. Resume (Originator) → 이력서 클래스 원본
3. ResumeCareTaker (CareTaker) → 메멘토를 관리할 CareTaker
4. App (클라이언트) → 이력서 작성/수정 가능

```java
// 이력서 Entity
@Getter
public class Resume {
    private final String name;
    private String job;
    private String description;

    // 생성자
    public Resume(String name, String job, String description) {
        this.name = name;
        this.job = job;
        this.description = description;
    }

    // Memento(Snapshot) 생성
    public Memento createMemento() {
        return new Memento(this.job, this.description);
    }

    // Memento(Snapshot) 복원
    public void restore(Memento ResumeSnapshot) {
        this.job = ResumeSnapshot.getJob();
        this.description = ResumeSnapshot.getDescription();
    }

    // 현재 Resume 상태 보기
    public void view() {
        System.out.println("name = " + this.name);
        System.out.printf("    job [%s] description : %s  ", job, description);
        System.out.println();
    }

    // Resume 데이터 변경
    public void changeData(String job, String description) {
        this.job = job;
        this.description = description;
    }

}
 
@Getter
// Snapshot 개념의 Memento
public class Memento {
    private final String job;
    private final String description;

    public Memento(String job, String description) {
        this.job = job;
        this.description = description;
    }
}

public class ResumeCareTaker {
    // Memento 관리를 위한 stack
    Stack<Memento> ResumeSnapshots = new Stack<>();

    // 특정 시점의 Memento Push
    public void push(Memento ResumeSnapshot)  {
        ResumeSnapshots.push(ResumeSnapshot);
    }

    // 복원을 위한 Memento 객체 반환
    public Memento pop() {
        return ResumeSnapshots.pop();
    }
}

public class App {
    // 시나리오 1
    public static void main(String[] args) {
        System.out.println("================ 이력서 작성 ================");
        // 이력서 작성
        Resume resume = new Resume("내 이력서 1", "개발자", "백엔드 잘합니다.");
        // Snapshot 바로 생성
        Memento snapshot = resume.createMemento();
        // CareTaker 생성
        ResumeCareTaker careTaker = new ResumeCareTaker();
        // 현재 이력서 상태를 CareTaker Stack 저장
        careTaker.push(snapshot);
        // 이력서 상태 확인
        resume.view();

        System.out.println("================ 이력서 변경 ================");
        // 이력서 변경
        resume.changeData("프로그래머", "백엔드 합니다.");
        resume.view();

        System.out.println("============= 이력서 변경 취소 (복원) =============");
        // 페이지 이탈 - 이전 데이터 복원
        resume.restore(careTaker.pop());
        resume.view();
    }
}
```

**결과**

```shell
================ 이력서 작성 ================
name = 내 이력서 1
    job [개발자] description : 백엔드 잘합니다.  
================ 이력서 변경 ================
name = 내 이력서 1
    job [프로그래머] description : 백엔드 합니다.  
============= 이력서 변경 취소 (복원) =============
name = 내 이력서 1
    job [개발자] description : 백엔드 잘합니다.  
```





## 3. 장점과 단점

### 3-1. 장점

- 캡슐화를 지키면서 상태 객체 상태 스냅샷을 만들 수 있음
  - `스냅샷` : 객체의 특정 시점의 데이터를 저장해 놓는 것
- 객체 상태 저장하고 또는 복원하는 역할을 CareTaker에게 위임할 수 있음
- 객체 상태가 바뀌어도 클라이언트 코드는 변경되지 않음

### 3-2. 단점

- 많은 정보를 저장하는 Mementor를 자주 생성하는 경우 메모리 사용량에 많은 영향을 줄 수 있음





## 4. 마치며.

메멘토 패턴은 객체의 특정 시점 상태를 저장할 수 있는데, 이 때, 객체와 거의 동일한 메멘토 객체를 만들기 때문에 객체의 크기가 커질수록 메모리 사용량에 영향을 줄 확률이 높아지므로 상황에 따라 적절히 사용할 수 있어야 한다.