예제 코드 요구사항

- 유저는 채용 공고에 지원할 수 있다.
- 채용 공고가 진행중일 때 입사 지원을 할 수 있다.
- 공고에 2명이 지원하면 해당 공고에는 더이상 다른 유저는 지원할 수 없다.
- 채용 공고가 대기중이거나 끝난 상태일 때 유저는 해당 공고에 지원을 취소할 수 없다.

위 요구사항을 만족하는 코드를 작성하면 다음과 같다.

### 상태패턴 적용 전
```java
@Getter
@RequiredArgsConstructor
public class User {
    private final String name;
}

@Getter
public class Recruit {
    public enum State {
        WAIT, PROGRESS, END, FULL
    }

    private static final int MAX_NUM = 2;
    private final Set<User> users = new HashSet<>();
    private State state;
    private final String name;

    public Recruit(String name) {
        this.name = name;
        System.out.println("========공고 게시(모집전)=====");
        this.state = State.WAIT;
    }

    public void applyRecruit(User user) {
        System.out.print(user.getName() + " 님의 공고 지원 결과 : ");
        if (state == State.WAIT || state == State.END || state == State.FULL) {
            System.out.println("해당 채용공고에 지원할 수 없습니다.");
            return;
        }

        this.users.add(user);
        if (users.size() == MAX_NUM) {
            this.state = State.FULL;
        }
        System.out.println("지원 성공하였습니다.");
    }

    public void cancelRecruit(User user) {
        System.out.print(user.getName() + " 님의 지원 취소 결과 : ");
        if (state == State.WAIT) {
            System.out.println("아직 공고 모집이 시작되지 않았습니다.");
            return;
        } else if (state == State.END) {
            System.out.println("이미 종료된 채용 공고입니다.");
            return;
        }else if(!this.users.remove(user)){
            System.out.println("지원한 공고가 아닙니다.");
            return;
        }

        if (state == State.FULL) {
            state = State.PROGRESS;
        }
        System.out.println("지원 취소 되었습니다.");
    }

    public void closeRecruit() {
        System.out.println("========공고 모집 종료=====");
        this.state = State.END;
    }

    public void startRecruit() {
        System.out.println("========공고 모집 시작=====");
        this.state = State.PROGRESS;
    }

    public void printUsers() {
        System.out.println("====입사 지원 유저 목록====");
        System.out.println(users.stream().map(User::getName).toList().toString());
    }
}

public class Client {
    public static void main(String[] args) {
        User user1 = new User("고길동");
        User user2 = new User("고둘리");
        User user3 = new User("또치");

        //공고 게시(모집전)
        Recruit recruit = new Recruit("사람인 공고");
        recruit.applyRecruit(user1);
        System.out.println();

        //공고 모집 시작
        recruit.startRecruit();
        recruit.applyRecruit(user1);
        recruit.applyRecruit(user2);
        recruit.applyRecruit(user3);
        System.out.println();
        
        recruit.printUsers();
        System.out.println();

        //공고 한명 취소후 다른사람이 지원
        recruit.cancelRecruit(user1);
        recruit.applyRecruit(user3);

        System.out.println();
        recruit.printUsers();
    }
}

결과:
========공고 게시(모집전)=====
고길동 님의 공고 지원 결과 : 해당 채용공고에 지원할 수 없습니다.

========공고 모집 시작=====
고길동 님의 공고 지원 결과 : 지원 성공하였습니다.
고둘리 님의 공고 지원 결과 : 지원 성공하였습니다.
또치 님의 공고 지원 결과 : 해당 채용공고에 지원할 수 없습니다.

====입사 지원 유저 목록====
[고길동, 고둘리]

고길동 님의 지원 취소 결과 : 지원 취소 되었습니다.
또치 님의 공고 지원 결과 : 지원 성공하였습니다.

====입사 지원 유저 목록====
[고둘리, 또치]
```

채용공고에 상태를 Enum으로 정의해놓고 해당 상태에 따라 입사지원, 취소가 분기처리 되고 있는 것을 볼 수 있다.

하지만 분기 처리 코드 인해 코드가 한눈에 들어오지 않고 만약 상태값들이 더 늘어나게 된다면 그만큼 분기처리도 더욱 많아져 더 가독성이 악화 될 것이다.

이러한 문제를 해결하기 위해 상태 패턴을 사용할 수 있다.

아래는 상태 패턴을 적용한 코드이다.

### 상태패턴 적용 후
```java
@Getter
@RequiredArgsConstructor
public class User {
    private final String name;
}

//Context
@Getter
public class Recruit{
    private final String name;  //공고 이름
    private final Set<User> users = new HashSet<>();    //공고 지원자들
    private RecruitState recruitState;  //공고 상태

    public Recruit(String name) {
        this.name = name;
        System.out.println("========" + name +" 공고 게시(모집전)=====");
        this.recruitState = new Wait(this);
    }

    //공고 지원
    public void applyRecruit(User user) {
        System.out.print(user.getName() + " 님의 지원 결과 : ");
        recruitState.addUser(user); //공고 지원에 대한 세부 로직은 상태에게 위임
    }

    //공고 취소
    public void cancelRecruit(User user) {
        System.out.print(user.getName() + " 님의 지원 취소 결과 : ");
        recruitState.removeUser(user);  //공고 취소에 대한 세부 로직은 상태에게 위임
    }

    public void changeState(RecruitState recruitState) {
        this.recruitState = recruitState;   //상태 변경
    }

    public void printUsers() {
        System.out.println("====입사 지원 유저 목록====");
        System.out.println(users.stream().map(User::getName).toList().toString());
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
        void addUser(User user);    //공고 지원
        void removeUser(User user); //공고 취소
}


//Concrete State
public class Wait implements RecruitState{
    private Recruit recruit;

    public Wait(Recruit recruit) {this.recruit = recruit;}

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
            this.recruit.changeState(new Full(recruit));    //가득 찼다면 마감상태로 변경
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

    public Full(Recruit recruit) {this.recruit = recruit;}

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
        this.recruit.changeState(new Progress(recruit));        //모집중 상태로 변경
    }
}

public class Client {
    public static void main(String[] args) {
        User user1 = new User("고길동");
        User user2 = new User("고둘리");
        User user3 = new User("또치");

        //공고 게시(모집전)
        Recruit recruit = new Recruit("사람인");
        recruit.applyRecruit(user1);
        System.out.println();

        //공고 모집 시작
        System.out.println("==== 공고 모집 시작 ====");
        recruit.changeState(new Progress(recruit)); //모집중 상태로 변경
        recruit.applyRecruit(user1);
        recruit.applyRecruit(user2);
        recruit.applyRecruit(user3);
        System.out.println();

        recruit.printUsers();
        System.out.println();

        //공고 한명 취소후 다른사람이 지원
        recruit.cancelRecruit(user1);
        recruit.applyRecruit(user3);
        
        System.out.println();
        recruit.printUsers();
    }
}

결과 : 
========사람인 공고 게시(모집전)=====
고길동 님의 지원 결과 : 아직 공고 모집이 시작되지 않았습니다.

==== 공고 모집 시작 ====
고길동 님의 지원 결과 : 고길동의 입사지원이 완료됐습니다.
고둘리 님의 지원 결과 : 고둘리의 입사지원이 완료됐습니다.
또치 님의 지원 결과 : 채용 공고 지원수가 꽉 찼습니다.

====입사 지원 유저 목록====
[고둘리, 고길동]

고길동 님의 지원 취소 결과 : 고길동의 입사지원이 취소됐습니다.
또치 님의 지원 결과 : 또치의 입사지원이 완료됐습니다.

====입사 지원 유저 목록====
[또치, 고둘리]
```

![Untitled](https://user-images.githubusercontent.com/32676275/153326418-6fe4c82d-f8c3-430f-a2ec-ae22fedaed48.png)

---

UML 다이어그램대로 상태 패턴을 적용하여 Recruit이라는 객체 내에 존재하던 비즈니스 로직을 각 State에게 위임하여 확장에 유연하도록 개선해보았다. 