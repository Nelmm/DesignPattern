## 1. 어댑터 패턴(Adapter Pattern) 이란?

어댑터 패턴은 **기존 코드를 클라이언트가 사용하는 인터페이스의 구현체로 바꿔주는 패턴**을 말한다.

일반적으로 어댑터 패턴은 110V 콘센트와 220V콘센트를 변환해주는 것을 예로 많이 든다다. 해당 예시를 개발 관점으로 얘기하자면, 클라이언트가 사용하는 인터페이스가 기존 코드와 다를 때, **기존 코드를 클라이언트 코드와 호환될 수 있게 처리해 주는 것으로 생각**하면 된다.

![adapter](https://user-images.githubusercontent.com/79291114/160358256-30481999-0b13-425b-90e3-ffac32e6f843.PNG)

- `Target` : `Client`가 사용하는 인터페이스
- `Adaptee` : 기존에 사용하던 코드
- `Adapter` : `Client`에서 `Adaptee`를 사용하기 위해 `Target`으로 변환 시켜 줌

결국 핵심은 **Adapter를 이용해 Adaptee를 Target으로 변환**해 준다는 것이다.





## 2. 어댑터 패턴적용 예제

어댑터 패턴의 개념은 이전 글에서 다루었기 때문에 생략했다.

-   이전 글 : [https://dev-youngjun.tistory.com/210](https://dev-youngjun.tistory.com/210)

실무에서는 어떤식으로 어댑터 패턴이 적용되는지 작성했다.

## 1. 실무 요구사항

메일 발송을 위해 **솔루션 회사 A**와 계약하여 발송 프로세스를 개발하여 사용하고 있는데, **솔루션 회사 B**와 계약하여 메일을 발송하고 싶은 요구사항이 있다.

현재까지 **솔루션 회사 A** 인터페이스를 사용한 수많은 클래스와 데이터들을 변경하고 싶지 않고, 현재 서비스 되고 있는 많은 부분을 변경하는 것은 위험하기 때문에, 기존 **솔루션 회사 A** 활용은 그대로 하고 B를 적용하고 싶다.

그래서, 서로 다른 A사와 B사의 인터페이스를 함께 동작시킬 수 있도록 개발했으면 한다.

## 2. Adapter 패턴 적용하기

솔루션 회사 A의 인터페이스는 아래와 같다.

```java
// 솔루션 회사 A 인터페이스
public interface MailSenderA {
    void send(String sendInfo);
}

// 기존에 구현되어있는 메일 발송 로직
public class SolutionA implements MailSenderA {

    @Override
    public void send(String sendInfo) {
        System.out.println("A 솔루션 회사에서 메일 발송 : " + sendInfo);
    }
}
```

당연하겠지만, 솔루션 회사 B는 인터페이스 규격이 회사 A와 다르다.

```java
// 솔루션 회사 B 인터페이스
public interface MailSenderB {
    void sendApi(String sendInfo);
}

// To-Be 메일 발송 로직
public class SolutionB implements MailSenderB {

    @Override
    public void sendApi(String sendInfo) {
        System.out.println("B 솔루션 회사에서 메일 발송 : " + sendInfo);
    }
}
```

여기에 클라이언트 입장에서 솔루션 회사 B를 범용성있게 적용하기 위해 Adapter 패턴을 활용할 수 있다.

![adapter.png](https://user-images.githubusercontent.com/42997924/159971038-0e4cea2f-ebcc-46f7-b44c-441f746443af.png)

```java
// Adapter Class
public class Adapter implements MailSenderA {

    // Adaptee(B회사) 생성자 주입
    private final MailSenderB mailSenderB;

    public Adapter(SolutionB newSolution) {
        this.mailSenderB = newSolution;
    }

    @Override
    public void send(String sendInfo) {
        System.out.print("Using Adapter >>> ");
        // 기존 send를 호출 -> 솔루션 B의 sendApi 호출 연결
        mailSenderB.sendApi(sendInfo);
    }
}
```

클라이언트는 기존에 사용하던 인터페이스 그대로 활용할 수 있게 됐다.

```java
// 클라이언트 코드
public class Client {
    public static void main(String[] args) {

        MailSenderA senderA = new SolutionA();
        senderA.send("기존 발송 메일");

        senderA = new Adapter(new SolutionB());
        senderA.send("TO-BE 발송 메일");
    }
}

/* 실행 결과
A 솔루션 회사에서 메일 발송 : 기존 발송 메일
Using Adapter >>> B 솔루션 회사에서 메일 발송 : TO-BE 발송 메일
*/
```

> 기존에 있는 시스템(솔루션 A 활용)의 레거시 인터페이스를 새로운 인터페이스(솔루션 B)로 교체하는 경우에 기존 코드를 수정하지 않고 재사용성을 높일 수 있다. 새로운 써드파티 라이브러리가 추가되어도 어댑터 패턴은 유용하게 사용된다!