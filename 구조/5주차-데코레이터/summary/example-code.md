# 데코레이터 패턴 적용 예제

데코레이터 패턴의 개념은 이전 글에서 다루었기 때문에 생략했다.

-   이전 글 : [https://dev-youngjun.tistory.com/213](https://dev-youngjun.tistory.com/213)

이번 글은 실무에서는 어떤식으로 데코레이터 패턴이 적용되는지 작성했다.

## 1. 실무 요구사항

채팅 서비스를 도입하는데, 광고 메시지는 보내지 않았으면 좋겠고 욕설은 필터링이 되었으면 좋겠다.

이를 데코레이터 패턴으로 구현해보자.

## 2. Decorator 패턴 적용하기


![decorator.png](https://user-images.githubusercontent.com/42997924/159971013-8f0009d2-b9f5-46e6-b97f-158e16e0b3a5.png)

클라이언트(채팅 입력 주체)는 sendMessage를 하는 목적이 중요하고, 코드는 변경되지 않아야 한다.

```java
// 채팅 입력 클라이언트(변경x)
public class Client {
    private final ChatService chatService;
    public Client(ChatService chatService) {
        this.chatService = chatService;
    }
    // 채팅 메시지를 발송
    public void sendMessage(String message) {
        chatService.sendMessage(message);
    }
}
```

기본 컴포넌트(ChatService)와 기본 컴포넌트 구현체(DefaultChatService)는 다음과 같다.

```java
// Component 인터페이스
public interface ChatService {
    void sendMessage(String message);
}

// ConcreteComponent : 기본 메시지 발송
public class DefaultChatService implements ChatService {
    // 단 하나의 컴포넌트(ChatService)를 포함
    @Override
    public void sendMessage(String message) {
        System.out.println("send message : " + message);
    }
}
```

여기에 자주 추가/삭제될 수 있는 필터링 기능을 Decorator로 구현하면 된다.

```java
// Decorator : 기능 추가 후 Component 호출
public class ChatDecorator implements ChatService {
    private final ChatService chatService;
    public ChatDecorator(ChatService chatService) {
        this.chatService = chatService;
    }
    @Override
    public void sendMessage(String message) {
        chatService.sendMessage(message);
    }
}

// 광고 필터링 Decorator
public class AdFilterChatDecorator extends ChatDecorator {
    public AdFilterChatDecorator(ChatService chatService) {
        super(chatService);
    }

    @Override
    public void sendMessage(String message) {
        // 광고 필터링 기능 추가
        if (isNotAd(message) ) {
            super.sendMessage(message);
        }
    }

    private boolean isNotAd(String message) {
        return !message.contains("광고");
    }
}

// 욕설 필터링 Decorator
public class AbuseFilterChatDecorator extends ChatDecorator {
    public AbuseFilterChatDecorator(ChatService chatService) {
        super(chatService);
    }

    @Override
    public void sendMessage(String message) {
        // 욕설 필터링 기능 추가
        super.sendMessage(abuseFilter(message));
    }

    private String abuseFilter(String message) {
        return message.replace("똥개야","이쁜진돗개순종아" );
    }
}
```

이제 채팅App에서는 런타임 시 동적으로 플래그 값에 따라 필터를 추가할 수 있다!

```java
// 채팅 App
public class ChatApp {
    private static final boolean enabledAdFilter = true;
    private static final boolean enabledAbuseFilter = true;

    public static void main(String[] args) {
        // 기본 채팅 서비스 생성
        ChatService chatService = new DefaultChatService();
        // 런타임 시 플래그 값에 따라 필터를 추가
        if (enabledAdFilter) { // 광고 제거 필터
            chatService = new AdFilterChatDecorator(chatService);
        }
        if (enabledAbuseFilter) { // 욕설 치환 필터
            chatService = new AbuseFilterChatDecorator(chatService);
        }

        Client client = new Client(chatService);
        client.sendMessage("안녕하세요!");
        client.sendMessage("(광고) 돈이 복사가 된다고?");
        client.sendMessage("똥개야 답장해줘");
    }
}

/* 실행 결과
send message = 안녕하세요!
send message = 이쁜진돗개순종아 답장해줘
 */
```