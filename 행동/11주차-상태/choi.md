# 상태 패턴

> 객체 내부 상태 변경에 따라 객체의 행동이 달라지는 패턴
>

![Untitled](https://user-images.githubusercontent.com/32676275/153326385-ff50910e-dcd3-4d74-aa73-1ead795348e6.png)

상태에 특화된 행동들을 **분리해 낼 수 있으며**, 새로운 행동을 추가하더라도 **다른 행동에 영향을 주지 않는다**.

`ConcreteState`가 `Context`객체를 가지고 있고 각각의 `ConcreteState`에 따라 `Context`를 어떻게 호출 할 지 결정한다.



예제 코드 요구사항

- 유저는 채용 공고에 지원할 수 있다.
- 채용 공고가 진행중일 때 입사 지원을 할 수 있다.
- 공고에 2명이 지원하면 해당 공고에는 더이상 다른 유저는 지원할 수 없다.
- 채용 공고가 대기중이거나 끝난 상태일 때 유저는 해당 공고에 지원을 취소할 수 없다.

위 요구사항을 만족하는 코드를 작성하면 다음과 같다.

```java
@Getter
public class User {
    private String name;

    public User(String name) {
        this.name = name;
    }
}

@Getter
public class Recruit {
    public enum State {
        WAIT, PROGRESS, END, FULL
    }

    private static final int MAX_NUM = 2;
    private State state;
    private final Set<com.company.state.after.User> users = new HashSet<>();

    public Recruit(State state) {
        this.state = state;
    }

    public void applyRecruit(User user) {
        if (state == State.WAIT || state == State.END || state == State.FULL) {
            System.out.println("해당 채용공고에 지원할 수 없습니다.");
            return;
        }
        users.add(user);
        if (users.size() == MAX_NUM) {
            this.state = State.FULL;
        }
    }

    public void cancelRecruit(User user) {
        if (state == State.WAIT) {
            System.out.println("아직 공고 모집이 시작되지 않았습니다.");
        } else if (state == State.END) {
            System.out.println("이미 종료된 채용 공고입니다.");
        }
        this.users.remove(user);
        if (state == State.FULL) {
            state = State.PROGRESS;
        }
    }

    public void printUser() {
        System.out.println("---------------");
        System.out.println("입사 지원 유저 목록");
        System.out.println("---------------");
        users.forEach(u -> System.out.println(u.getName()));
    }
}

public class Main {
    public static void main(String[] args) {
        com.company.state.after.User user1 = new com.company.state.after.User("a");
        com.company.state.after.User user2 = new com.company.state.after.User("b");
        com.company.state.after.User user3 = new User("c");

        com.company.state.after.Recruit recruit1 = new com.company.state.after.Recruit(new Wait());
        recruit1.applyRecruit(user1);

        System.out.println("================");

        com.company.state.after.Recruit recruit2 = new com.company.state.after.Recruit(new Progress());
        recruit2.applyRecruit(user1);
        recruit2.applyRecruit(user2);
        recruit2.applyRecruit(user3);

        recruit2.printUser();

        recruit2.cancelRecruit(user1);
        recruit2.applyRecruit(user3);
        recruit2.printUser();

        System.out.println("================");

        com.company.state.after.Recruit recruit3 = new Recruit(new End());
        recruit3.applyRecruit(user1);

    }
}

결과:
아직 공고 모집이 시작되지 않았습니다.
================
a의 입사지원이 완료됐습니다.
b의 입사지원이 완료됐습니다.
채용 공고 지원수가 꽉 찼습니다.
---------------
입사 지원 유저 목록
---------------
a
b
a의 입사지원이 취소됐습니다.
c의 입사지원이 완료됐습니다.
---------------
입사 지원 유저 목록
---------------
c
b
================
이미 종료된 채용 공고입니다.
```

위 코드를 보면 채용공고에 상태에 따라 입사지원, 취소가 분기처리 되고 있는 것을 볼 수 있다.

하지만 분기 처리 코드 인해 코드가 복잡해지고 있다. 그리고 만약 상태값들이 더 늘어나게 된다면 그만큼 분기처리도 더욱 많아지게 될 것이다.

이러한 문제를 해결하기 위해 상태 패턴을 사용할 수 있다.

아래는 상태 패턴을 적용한 코드이다.

```java
//Context
@Getter
public class Recruit{
    private final Set<User> users = new HashSet<>();
		//State를 가지고 있다.
    private RecruitState recruitState;

    public Recruit(RecruitState recruitState) {
        this.recruitState = recruitState;
        this.recruitState.setRecruit(this);
    }

    public void applyRecruit(User user) {
        recruitState.addUser(user);
    }

    public void cancelRecruit(User user) {
        recruitState.removeUser(user);
    }

    public void changeState(RecruitState recruitState) {
        this.recruitState = recruitState;
    }

    public void printUser() {
        System.out.println("---------------");
        System.out.println("입사 지원 유저 목록");
        System.out.println("---------------");
        users.forEach(u -> System.out.println(u.getName()));
    }
}

//채용 공고에 지원할 유저 클래스
@Getter
public class User {
    private String name;

    public User(String name) {
        this.name = name;
    }
}

//State Interface
public interface RecruitState {
    void setRecruit(Recruit recruit);
    //공고 지원
    void addUser(User user);
    //공고 취소
    void removeUser(User user);
}

//Concrete State
public class Wait implements RecruitState{
    private Recruit recruit;
    @Override
    public void setRecruit(Recruit recruit) {
        this.recruit = recruit;
    }
    @Override
    public void addUser(User user) {
        System.out.println("아직 공고 모집이 시작되지 않았습니다.");
    }
    @Override
    public void removeUser(User user) {
        System.out.println("아직 공고 모집이 시작되지 않았습니다.");
    }
}

//Concrete State
public class Progress implements RecruitState{
    private Recruit recruit;
    private static final int MAX_NUM = 2;
    public Progress() {}
    public Progress(Recruit recruit) {this.recruit = recruit;}
    @Override
    public void setRecruit(Recruit recruit) {
        this.recruit = recruit;
    }
    @Override
    public void addUser(User user) {
        recruit.getUsers().add(user);
        System.out.println(user.getName() + "의 입사지원이 완료됐습니다.");
        if(recruit.getUsers().size() == MAX_NUM) {
            this.recruit.changeState(new Full(recruit));
        }
    }
    @Override
    public void removeUser(User user) {
        recruit.getUsers().remove(user);
        System.out.println(user.getName() + "의 입사지원이 취소됐습니다.");
    }
}

//Concrete State
public class End implements RecruitState {
    private Recruit recruit;
    public End() {}
    public End(Recruit recruit) {this.recruit = recruit;}
    @Override
    public void setRecruit(Recruit recruit) {
        this.recruit = recruit;
    }
    @Override
    public void addUser(User user) {
        System.out.println("이미 종료된 채용 공고입니다.");
    }
    @Override
    public void removeUser(User user) {
        System.out.println("이미 종료된 채용 공고입니다.");
    }
}

//Concrete State
public class Full implements RecruitState{
    Recruit recruit;
    public Full() {}
    public Full(Recruit recruit) {
        this.recruit = recruit;
    }
    @Override
    public void setRecruit(Recruit recruit) {
        this.recruit = recruit;
    }
    @Override
    public void addUser(User user) {
        System.out.println("채용 공고 지원수가 꽉 찼습니다.");
    }
    @Override
    public void removeUser(User user) {
        recruit.getUsers().remove(user);
        System.out.println(user.getName() + "의 입사지원이 취소됐습니다.");
        this.recruit.changeState(new Progress(recruit));
    }
}

//Client
public class Main {
    public static void main(String[] args) {
        User user1 = new User("a");
        User user2 = new User("b");
        User user3 = new User("c");

        Recruit recruit1 = new Recruit(new Wait());
        recruit1.applyRecruit(user1);

        System.out.println("================");

        Recruit recruit2 = new Recruit(new Progress());
        recruit2.applyRecruit(user1);
        recruit2.applyRecruit(user2);
        recruit2.applyRecruit(user3);

        recruit2.printUser();

        recruit2.cancelRecruit(user1);
        recruit2.applyRecruit(user3);
        recruit2.printUser();

        System.out.println("================");

        Recruit recruit3 = new Recruit(new End());
        recruit3.applyRecruit(user1);
    }
}

결과ㅣ
아직 공고 모집이 시작되지 않았습니다.
================
a의 입사지원이 완료됐습니다.
b의 입사지원이 완료됐습니다.
채용 공고 지원수가 꽉 찼습니다.
---------------
입사 지원 유저 목록
---------------
a
b
a의 입사지원이 취소됐습니다.
c의 입사지원이 완료됐습니다.
---------------
입사 지원 유저 목록
---------------
c
b
================
이미 종료된 채용 공고입니다.
```

![Untitled](https://user-images.githubusercontent.com/32676275/153326418-6fe4c82d-f8c3-430f-a2ec-ae22fedaed48.png)

---

### 장/단점

> 장점
>
- 상태에 따른 동작을 개별 클래스로 옮겨서 관리할 수 있다.
- 기존의 특정 상태에 따른 동작을 변경하지 않고 새로운 상태에 다른 동작을 추가할 수 있다.
- 코드 복잡도를 줄일 수 있다.

> 단점
>
- 복잡도가 증가한다.
(on, off 단 두가지 정도의 상태만 있지만 상태 패턴을 적용하면 상태 개수에 비해 복잡해질 수 있다.)