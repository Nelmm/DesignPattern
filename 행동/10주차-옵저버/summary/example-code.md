# 옵저버 패턴

**채팅 서비스**를 만들 때 다음과 같은 `폴링(polling)` 방식을 적용한다해보자

- 유저가 특정 `토픽`에 `메세지`를 전달후 저장
- 유저가 주기적으로 서버에 원하는 `토픽`의 `전체 메세지`를 요청

위 방식의 예제 코드를 보자.

```java
//채팅 서버 클래스
public class ChatServer {
    //특정 토픽의 메세지 리스트
    private Map<String, List<String>> messages = new HashMap<>();

    //토픽에 메세지 추가
    public void addMessage(String topic, String message) {
        //토픽 존재시
        if (this.messages.containsKey(topic)) {
            //토픽에 메세지 추가
            this.messages.get(topic).add(message);
        } else {
            List<String> messageList = new ArrayList<>();
            //메세지 추가
            messageList.add(message);
            //토픽 추가
            this.messages.put(topic, messageList);
        }
    }

    //토픽 메세지 얻기
    public List<String> getMessage(String topic) {
        return this.messages.get(topic);
    }
}
```

```java
//채팅을 이용하는 클라이언트 클래스
public class Client {
    private final ChatServer chatServer;
    private final String name;

    //채팅 서버와 Client 이름 받기
    public Client(ChatServer chatServer, String name) {
        this.chatServer = chatServer;
        this.name = name;
    }

    //토픽에 메세지 전송
    public void sendMessage(String topic, String message) {
        chatServer.addMessage(topic, message);
    }

    //해당 토픽 메세지 출력
    public void printMessage(String topic) {
        List<String > messages = this.getMessage(topic);
        if(Objects.isNull(messages)) return;
				//메세지 출력
        System.out.printf("======== 이름: %s   토픽: %s =======\n", this.name, topic);
        messages.forEach(System.out::println);
    }

    //해당 토픽 메세지 얻기
    private List<String> getMessage(String topic) {
        return this.chatServer.getMessage(topic);
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        //채팅 서버 생성
        ChatServer chatServer = new ChatServer();
        //클라이언트 생성
        Client client1 = new Client(chatServer, "홍길동");
        Client client2 = new Client(chatServer, "아무개");

        //사람인이라는 토픽에 메세지 전송
        client1.sendMessage("사람인", "안녕하세요");
        client2.sendMessage("사람인", "반갑습니다");
        //클라이언트가 서버로부터 메세지 출력 호출
        client1.printMessage("사람인");
        client2.printMessage("사람인");

        //점핏이라는 토픽에 메세지 전송
        client1.sendMessage("점핏", "잘부탁드립니다");
        //클라이언트가 서버로부터 메세지 출력 호출
        client2.printMessage("점핏");
    }
}

결과
======== 이름: 홍길동   토픽: 사람인 =======
안녕하세요
반갑습니다
======== 이름: 아무개   토픽: 사람인 =======
안녕하세요
반갑습니다
======== 이름: 아무개   토픽: 점핏 =======
잘부탁드립니다
```

예제에서 `클라이언트`는 메세지를 출력하고 싶을 때마다 `채팅 서버`에 요청을 보낸다.

이 방식의 문제는 전달된 메세지는 없지만 `클라이언트`가 주기적으로 서버에 메세지를 요청하면 불필요한 작업이 늘어나 리소스가 낭비된다는 점이다.

`옵저버 패턴`을 사용하면 메세지 전달을 감지하여 `클라이언트`가 불필요한 작업을 없앨 수 있다.

패턴을 적용한 예제를 보자

**요구사항**은 다음과 같다

- 유저는 특정 `토픽`을 구독 가능
- 유저가 구독한 `토픽`에 `메세지`를 전송시 `토픽`을 구독한 다른 유저는 `메세지` 확인 가능

![Untitled](https://user-images.githubusercontent.com/32676275/158595711-7b63d418-90ae-44dd-b9d3-3c2ea4262f23.png)

```java
/**
 * 구독자 인터페이스
 * ** Observer **
 */
public interface Subscriber {
		//전달받은 메세지 처리 메소드
    void handleMessage(String message);
}
```

```java
/**
 * Subscriber 인터페이스 구현체
 * ** ConcreteObserver **
 */
public class User implements Subscriber{
    //유저 이름
    private final String name;
    
    //생성자
    public User(String name) {
        this.name = name;
    }
    
    //메세지 처리
    @Override
    public void handleMessage(String message) {
        System.out.printf("%s --- %s 수신\n",message,this.name);
    }
    //Getter
}
```

```java
/**
 * 채팅 서버 클래스
 * ** Subject **
 */
public class ChatServer {
    //각 토픽에 구독자들을 저장
    public Map<String, List<Subscriber>> subscribers = new HashMap<>();

    //구독 등록
    public void register(String topic, Subscriber subscriber) {
        //토픽 존재시 구독자 추가
        if(this.subscribers.containsKey(topic)) {
            this.subscribers.get(topic).add(subscriber);
            return;
        }
        //존재하지 않을시 토픽 생성후 추가
        List<Subscriber> newSubscribers = new ArrayList<>();
        newSubscribers.add(subscriber);
        this.subscribers.put(topic, newSubscribers);
    }

    //구독 취소
    public void unregister(String topic, Subscriber subscriber) {
        //토픽 존재시 토픽의 구독자 제거
        if(this.subscribers.containsKey(topic)) {
            this.subscribers.get(topic).remove(subscriber);
        }
    }

    //토픽의 구독자들에게 메세지 전송
    public void sendMessage(User user, String topic, String message) {
        // 해당 토픽이 존재시
        if (this.subscribers.containsKey(topic)) {
            String userMessage = user.getName() + ": " + message;
            // 토픽 구독자들에게 메세지 전달
            this.subscribers.get(topic).forEach(s -> s.handleMessage(userMessage));
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        ChatServer chatServer = new ChatServer();
        User user1 = new User("홍길동");
        User user2 = new User("아무개");

        System.out.println("==========각 토픽에 구독자가 1명일 때==========");
        chatServer.register("개발", user1);   //구독
        chatServer.register("기획", user2);   //구독
        chatServer.sendMessage(user1, "개발", "개발 토픽에 메세지 전달");   //메시지전송
        chatServer.sendMessage(user2, "기획", "기획 토픽에 메세지 전달");   //메시지전송

        System.out.println("==========구독자 추가후 메세지 전달==========");
        chatServer.register("기획", user1);   //구독
        chatServer.sendMessage(user1, "기획", "기획 구독후 메세지 전달");   //메시지전송

        System.out.println("==========구독 취소후 메세지 전달==========");
        chatServer.unregister("기획", user2);  //구독 취소
        chatServer.sendMessage(user2, "기획", "기획 구독 취소후 메세지 전달");  //메시지 전송
    }
}

결과
==========각 토픽에 구독자가 1명일 때==========
홍길동: 개발 토픽에 메세지 전달 --- 홍길동 수신
아무개: 기획 토픽에 메세지 전달 --- 아무개 수신
==========구독자 추가후 메세지 전달==========
홍길동: 기획 구독후 메세지 전달 --- 아무개 수신
홍길동: 기획 구독후 메세지 전달 --- 홍길동 수신
==========구독 취소후 메세지 전달==========
아무개: 기획 구독 취소후 메세지 전달 --- 홍길동 수신
```

`옵저버 패턴`을 적용하면 `ChatServer(Subject)` 에서 `User(Observer)` 리스트를 가지고 있는 구조가 된다.

`ChatServer.sendMessage()`메소드 호출시 `User(Observer).handleMessage()` 메소드를 호출함으로써   **유저는 메세지가 전달되는 시점에 처리할 수 있는 구조가 된다.**