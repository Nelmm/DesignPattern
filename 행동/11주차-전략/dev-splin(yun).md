# 전략(Strategy) 패턴

객체들이 할 수 있는 행위 각각에 대해 클래스(전략)를 생성하고, 유사한 행위들을 캡슐화 하는 인터페이스를 정의하여, **객체의 행위를 동적으로 바꾸고 싶은 경우 직접 행위를 수정하지 않고 전략을 바꿔주기만 함으로써 행위를 유연하게 확장하는 방법**을 말합니다.

간단히 말해서 객체가 할 수 있는 행위들 각각을 전략으로 만들어 놓고, **동적으로 행위의 수정이 필요한 경우 전략을 바꾸는 것만으로 행위의 수정이 가능하도록 만든 패턴**입니다.



## 1. 다이어그램

![strategy](https://user-images.githubusercontent.com/79291114/153302554-bf5b55cc-be05-4aab-b410-93f710f8a2a2.PNG)

- `Context` : 로직(전략)을 수행하는 클래스
  - 여러가지 전략을 수행하기 위하여 `Strategy` 인터페이스를 참조함
- `Strategy` : 다양한 전략들을 하나로 추상화 시킨 인터페이스
- `ConcreteStrategy` : `Startegy` 인터페이스를 구현한 전략 클래스
  - 클라이언트가 직접 사용할 전략 클래스를 정할 수 있음






## 2. 예시 코드

블로그에서 글을 작성하다가 새로운 테마가 나와서 적용하는 상황을 가정하겠습니다.



### 2-1. 기존 코드

#### Data 클래스

```java
public class Data {
    private String name;
    private String cell;
    private String email;

    public Data(String name, String cell, String email) {
        this.name = name;
        this.cell = cell;
        this.email = email;
    }

    public String getName() {
        return name;
    }

    public String getCell() {
        return cell;
    }

    public String getEmail() {
        return email;
    }
}
```



#### Blog 클래스

```java
public class Blog {
    private Data data;
    private Theme theme;

    public Blog(Data data, Theme theme) {
        this.data = data;
        this.theme = theme;
    }

    public void write(String content) {
        theme.apply();
        System.out.printf("이름 : %s\n", data.getName());
        System.out.printf("전화번호 : %s\n", data.getCell());
        System.out.printf("이메일 : %s\n", data.getEmail());
        System.out.printf("내용 : %s\n", content);
    }
}
```



#### Client 클래스

```java
public class Client {
    public static void main(String[] args) {
        Data data = new Data("splin", "010-1234-5678", "dev.splin@gmail.com");
        Blog blog = new Blog(data);

        blog.write("내용1");
    }
}
```

만약 기존 코드에서 블로그 테마를 변경하고 싶다면 `Blog`의 `write` 함수를 바꿔주어야 하는데, 이는 객체 지향 설계 원칙인 `SOLID` 의 `개방 폐쇄 원칙(OCP)`을 위배하게 됩니다.

또한, 시스템이 커져서 **다양한 블로그, 다양한 테마가 생긴다면, 각각의 블로그들의 write 함수를 변경해주어야 하기 때문에 관리 포인트가 늘어나는 단점**이 생깁니다.

따라서 이를 해결하고자 전략 패턴을 사용할 수 있습니다.



### 2-2. 전략 패턴 사용 코드

`Theme 인터페이스(Strategy)`와 `다양한 테마 클래스(ConcreteStrategy)`를 만들어주고 `Blog(Context)`, Client 클래스를 조금만 수정하면 됩니다.



#### Theme 인터페이스

```java
public interface Theme {
   void apply();
}
```



#### 다양한 테마 클래스

```java
public class Basic implements Theme {
    @Override
    public void apply() { System.out.println("===== 기본 테마 ====="); }
}

public class Forest implements Theme {
    @Override
    public void apply() {
        System.out.println("===== 숲 테마 =====");
    }
}

public class Ocean implements Theme {
    @Override
    public void apply() {
        System.out.println("===== 바다 테마 =====");
    }
}
```



#### 전략을 사용하는 2가지 방식

블로그 클래스에서는 기존과 다르게 Theme의 apply 함수를 호출함으로써 다양한 테마(전략)를 받아들일 수 있게 되는데 이 때 아래와 같이 2가지의 방식으로 사용할 수 있습니다.

1. `Blog(Context)`에 `Theme(전략)`을 넣어주는 방식
2. `Theme(전략)`을 사용하는 `write`메서드를 호출할 때 `Theme(전략)`을 정해주는 방식



##### 1. Context에 전략을 넣어주는 방식

`Blog(Context)`에 `theme(전략)`을 넣어줌으로써 해당 블로그는 항상 고정된 테마를 사용하게 됩니다.

```java
public class Blog {
    private Data data;
    private Theme theme;

    public Blog(Data data, Theme theme) {
        this.data = data;
        this.theme = theme;
    }

    public void write(String content) {
        theme.apply();
        System.out.printf("이름 : %s\n", data.getName());
        System.out.printf("전화번호 : %s\n", data.getCell());
        System.out.printf("이메일 : %s\n", data.getEmail());
        System.out.printf("내용 : %s\n", content);
    }
}

public class Client {
    public static void main(String[] args) {
        Data data = new Data("splin", "010-1234-5678", "dev.splin@gmail.com");
        Blog blog = new Blog(data, new Basic());
        blog.write("내용1");
        blog.write("내용2");

        blog.changeTheme(new Forest());
        blog.write("내용2");

        blog.changeTheme(new Ocean());
        blog.write("내용3");
    }
}
```

```java
// 출력 결과
===== 기본 테마 =====
이름 : splin
전화번호 : 010-1234-5678
이메일 : dev.splin@gmail.com
내용 : 내용1
===== 기본 테마 =====
이름 : splin
전화번호 : 010-1234-5678
이메일 : dev.splin@gmail.com
내용 : 내용2
===== 숲 테마 =====
이름 : splin
전화번호 : 010-1234-5678
이메일 : dev.splin@gmail.com
내용 : 내용3
===== 바다 테마 =====
이름 : splin
전화번호 : 010-1234-5678
이메일 : dev.splin@gmail.com
내용 : 내용4
```

이 방식을 사용하면 블로그 내의 글들은 같은 테마를 적용하게 되고 테마를 변경하고 싶다면 블로그 자체의 테마를 변경해야 합니다.



##### 2. 전략을 사용할 메서드를 호출할 때 전략을 정해주는 방식

`Theme(전략)`을 사용하는 `write`메서드를 호출할 때 `Theme(전략)`을 정해줌으로써 글을 쓸 때마다 각각 다른 테마를 적용할 수 있습니다.

```java
public class Blog {
    private Data data;

    public Blog(Data data) {
        this.data = data;
    }

    public void write(String content, Theme theme) {
        theme.apply();
        System.out.printf("이름 : %s\n", data.getName());
        System.out.printf("전화번호 : %s\n", data.getCell());
        System.out.printf("이메일 : %s\n", data.getEmail());
        System.out.printf("내용 : %s\n", content);
    }
}

public class Client {
    public static void main(String[] args) {
        Data data = new Data("splin", "010-1234-5678", "dev.splin@gmail.com");
        Blog blog = new Blog(data);

        blog.write("내용1", new Forest());
        blog.write("내용2", new Ocean());
        blog.write("내용3", new Basic());
        blog.write("내용4", new Ocean());
    }
}
```

```java
// 출력 결과
===== 숲 테마 =====
이름 : splin
전화번호 : 010-1234-5678
이메일 : dev.splin@gmail.com
내용 : 내용1
===== 바다 테마 =====
이름 : splin
전화번호 : 010-1234-5678
이메일 : dev.splin@gmail.com
내용 : 내용2
===== 기본 테마 =====
이름 : splin
전화번호 : 010-1234-5678
이메일 : dev.splin@gmail.com
내용 : 내용3
===== 바다 테마 =====
이름 : splin
전화번호 : 010-1234-5678
이메일 : dev.splin@gmail.com
내용 : 내용4
```

이 방식을 사용하면 블로그 내의 글들은 각각 다른 테마를 적용할 수 있습니다.





## 3. 장단점

전략 패턴을 사용하게 되면 아래와 같은 장단점이 있습니다.



### 장점

- 새로운 전략을 추가하더라도 기존 코드를 변경하지 않습니다.
  - 새로운 전략의 추가는 상속하는 클래스만을 생성해주고 교체해주면 되기 때문
- 상속 대신 위임을 사용할 수 있습니다.
  - 예시 코드에서 **Blog를 상속**해서 다양한 테마를 적용시킬 수 있지만 **Theme 인터페이스를 사용(위임)**하여도 다양한 테마를 적용시킬 수 있습니다.
- 런타임에 동적으로 전략을 변경할 수 있습니다.



### 단점

- 복잡도가 증가합니다.
  - 인터페이스와 클래스가 늘어나기 때문
- 클라이언트 코드가 구체적인 전략을 알아야 합니다.
  - 어떠한 전략이 있는지 알아야 클라이언트가 적용시킬 수 있음





## 4. Command 패턴과 State 패턴과의 비교

공부를 하면서 `전략 패턴, 커맨드 패턴, 상태 패턴`이 비슷하다고 생각했는데, 이 3개 패턴의 다이어그램을 보면서 차이점을 간단히 정리해보도록 하겠습니다.



### 전략 패턴

![strategy](https://user-images.githubusercontent.com/79291114/153312563-a830529c-8f3e-47f8-b047-9c84225f3401.PNG)

여러 알고리즘을 캡슐화하고 상호 교환 가능하게 만드는 패턴

- 여러 알고리즘을 `상호 교환` 한다는 것은 이미 알고리즘이 실행되는 것은 정해져있고 해당 알고리즘만 교체해준다는 의미입니다.
- 다이어그램을 보면, **Client가 ConcreteStrategy를 교체**하는 것을 볼 수 있습니다.
- 즉, **행동이 정해져 있는 상태에서 어떠한 방법으로 수행**할지만 달라집니다.



### 커맨드 패턴

![command](https://user-images.githubusercontent.com/79291114/153312559-5b068040-1e19-493c-a680-499744f56f08.PNG)

요청을 캡슐화하여 호출자와 수신자를 분리하는 패턴

- 호출자와 수신자를 분리한다는 것은 `명령(호출자)`과 `행동(수신자)`을 분리한다는 의미입니다.
- 다이어그램을 보면, **Invoker에 따라서 다양한 Receiver가 호출**되는 것을 알 수 있습니다.
- 즉, **명령에 따라서 다른 행동을 수행**한다는 것입니다.



### 상태 패턴

![state](https://user-images.githubusercontent.com/79291114/153312562-2f70656c-3cd2-4537-8acd-5bfeea7dcfe6.PNG)

객체 내부 상태 변경에 따라 객체의 행동이 달라지는 패턴

- 말 그대로 **객체 내부 상태에 따라서 다른 행동을 수행**한다는 것입니다.
- 다이어그램을 보면, **Client가 changeState로 상태를 변경해줌에 따라 ConcreteState가 바뀌면서 다른 행동**을 하게됩니다.





## 5. 마치며...

전략패턴을 공부하면서 커맨트 패턴, 상태 패턴과 비슷하다고 생각했습니다. 그래서 차이점을 한 번 알아보았는데, 

`패턴을 구분할 때는 생김새로 구분하는 것이 아니라, 목적으로 구분해야 한다.`

라는 문구를 다시 한 번 생각해볼 수 있는 시간이었습니다.



---

참고 : [코딩으로 학습하는 GoF의 디자인 패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)

[https://victorydntmd.tistory.com/292](https://victorydntmd.tistory.com/292)

[https://tecoble.techcourse.co.kr/post/2021-10-04-strategy-command-pattern/](https://tecoble.techcourse.co.kr/post/2021-10-04-strategy-command-pattern/)
