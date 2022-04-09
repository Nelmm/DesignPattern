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

결과
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