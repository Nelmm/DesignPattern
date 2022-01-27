# 옵저버 패턴
특정 `객체`에서 일어나는 행동,상태변화 등을 이를 관찰 중인 객체들에게 알리는 패턴. 일종의 pub/sub 구조

하나의 발행자에 대해 이를 구독하고 있는 구독자는 여러 형태의 구독자들이 존재할 수 있으며, 이를 모두 의존할 수 없고 미리 알지 못할 수 있기 때문에 인터페이스와 같이 추상적인 계층을 이용하거나 동일한 형태의 프로퍼티/메서드를 이용해 통신해야 한다.


## 코드
```java
//Subject
public class ChatServer {
    private Map<String, List<Subscriber>> subscribers = new HashMap<>();

    public void register(String subject, Subscriber subscriber) {
        if (this.subscribers.containsKey(subject)) {
            this.subscribers.get(subject).add(subscriber);
        } else {
            List<Subscriber> list = new ArrayList<>();
            list.add(subscriber);
            this.subscribers.put(subject,list);
        }
    }

    public void unregister(String subject, Subscriber subscriber) {
        if (this.subscribers.containsKey(subject)) {
            this.subscribers.get(subject).remove(subscriber);
        }
    }

    public void sendMessage(User user, String subject, String message) {
        if (this.subscribers.containsKey(subject)) {
            String userMsg = "[" + subject + "] " + user.getName() + " : " + message;
            this.subscribers.get(subject).forEach(s -> s.notifyMessage(userMsg));
        }
    }
}

//Observer
public interface Subscriber {
    void notifyMessage(String msg);
}

//Concrete Observer
public class User implements Subscriber{
    private String name;

    public User(String name) {
        this.name = name;
    }

    @Override
    public void notifyMessage(String msg) {
        System.out.println(msg);
    }

    public String getName() {
        return name;
    }
}

public class Client {
    public static void main(String[] args) {
        ChatServer chatServer = new ChatServer();
        User user1 = new User("user1");
        User user2 = new User("user2");

        chatServer.register("채팅방 1",user1);
        chatServer.register("채팅방 1",user2);

         chatServer.register("채팅방 2",user1);

         chatServer.sendMessage(user1,"채팅방 1", "안녕하세요!");
         chatServer.sendMessage(user2,"채팅방 1", "반가워요");

         chatServer.sendMessage(user2,"채팅방 2", "나 혼자인가?...");
    }
}
```

<br>

## 장점/단점
### 장점
1. OCP
2. 런타임에 객체간의 관계 설정 가능

### 단점
1. 구독자에게 보내는 알림의 순서는 보장되지 못한다.
1. 런타임에 구독자들을 추가/삭제 해줌으로써 추가 비용이 발생
2. 명시적으로 객체를 해지해주지 않으면, GC가 되지 않아 메모리 leak이 발생한다.

<br>

## 다른패턴과 차이점
- Mediator : 연관되어있는 서로 다른 객체간의 종속성을 제거하는 것이 목적이며, 연관되어있는 객체가 Mediator에 의존하여 상호의존적이다.  Observer는 객체간의 연결을 `동적`으로 `단반향 연결` 하는 것이 목적이다.

- 책임 연쇄 패턴 : 옵저버패턴을 이용하면 객체의 값 변화에 따른 이벤트로 다른 객체의 값(상태)가 변화가 됨으로써 책임연쇄 패턴의 원인이 될 수 있다.

<br>

## 추가

### Strong Reference
일반적인 참조 방법으로 아래와 같이 새로운 인스턴스를 선언하면 강하게 연결되어 GC에 collect가 되지 않음
```java
User user = new User();
```

하지만 Caching 하거나 데이터 보관을 위해 Map과 같은 자료구조에 저장을하게되면 내부에 존재하는 `widget객체`들은 strong refrence를 갖게되어 내부 데이터들이 GC의 대상이 되지 못해 메모리 누수가 발생한다.

### Weak Reference
WeakReference Wrapper 객체를 이용하면 GC가 Collect하는 과정중에 reachability를 판단하는데 힌트를 얻을 수 있다. 도달상태가 약해지면 객체는 다음 GC때 반드시 제거된다.

아래와 같이 WeakReference 클래스를 이용하여 객체를 생성하면 get()으로 객체를 얻어 올 수 있는데, 어느샌가 GC의 대상이 될 수 있기 때문에 null이 반환될 수도 있다.
```java
WeakReference<User> weakUser = new WeakReference<>(new User("user1"));
User user = weakUser.get();
```

WeakReference에도 세세하게 들어가면 세가지로 분류가 될 수 있다.

#### 예제
```java
public class WeakReference {
    public static class Referred {
        protected void finalize() {
            System.out.println("객체 소멸");
        }
    }

    public static void collect() throws InterruptedException {
        System.out.println("garbage collection 요청");
        System.gc();
        Thread.sleep(5000);
    }

    public static void main(String args[]) throws InterruptedException {
        System.out.println("Weak Reference 생성");
        Referred strong = new Referred();
        java.lang.ref.WeakReference<Referred> soft = new java.lang.ref.WeakReference<Referred>(strong);

        WeakReference.collect();
        System.out.println("reference 제거");
        strong = null;
        WeakReference.collect();
        System.out.println("Done");
    }
}
//print
Soft Reference 생성
garbage collection 요청
//5초후
reference 제거
garbage collection 요청
객체 소멸
//5초후
Done
```


### soft reference
남아있는 메모리양에 따라 다음 GC때 제거할지 결정.

캐싱을 위해 Soft Reference를 사용할 수 있으나 오히려 독이 될 수 있다. JVM의 메모리에 GC 한계점 까지 계속 객체들이 살아있는상태로 유지되기 때문에 더 잦은 GC를 발생시킬 수 있다고 한다.
```java
SoftReference<User> softReference = new SoftReference<>(new User("user1"));
User user = softReference.get();
```

#### 예제
```java
public class SoftReference {
    public static class Referred {
        protected void finalize() {
            System.out.println("객체 소멸");
        }
    }

    public static void collect() throws InterruptedException {
        System.out.println("garbage collection 요청");
        System.gc();
        Thread.sleep(5000);
    }

    public static void main(String args[]) throws InterruptedException {
        System.out.println("Soft Reference 생성");
        Referred strong = new Referred();
        java.lang.ref.SoftReference<Referred> soft = new java.lang.ref.SoftReference<Referred>(strong);

        SoftReference.collect();
        System.out.println("reference 제거");
        strong = null;
        SoftReference.collect();
        try {
            Thread.sleep(10000);
            System.out.println("메모리 소비 시작");
            List<SoftReference> heap = new ArrayList<SoftReference>(100000);
            while (true) {
                heap.add(new SoftReference());
            }
        } catch (OutOfMemoryError e) {
            System.out.println("메모리 초과");
        }
        System.out.println("Done");
    }
}

//print
Soft Reference 생성
garbage collection 요청
//5초후
reference 제거
garbage collection 요청
//15초후
메모리 소비 시작
객체 소멸
메모리 초과
Done
```

#### phantom reference
get()사용시에 항상 null을 반환하고 해당 객체가 죽었는지 살아있는지 판단하기 위해 사용되고 Reference Queue 사용이 필수적이다.

```java
public class Phantom {
    public static class Referred {
    }

    public static void collect() throws InterruptedException {
        System.out.println("GC 요청");
        System.gc();
        Thread.sleep(5000);
    }

    public static void main(String args[]) throws InterruptedException {
        System.out.println("phantom references 생성");
        ReferenceQueue dead = new ReferenceQueue();
        Map<Reference, String> cleanUpMap = new HashMap<Reference, String>();
        Referred strong = new Referred();
        PhantomReference<Referred> phantom = new PhantomReference(strong, dead);

        System.out.println("phantom 객체 : " + phantom.get());

        cleanUpMap.put(phantom, phantom.getClass().getName() + " 제거");

        Phantom.collect();

        System.out.println("Reference Queue에서 enqueue요청");
        Reference reference = dead.poll();
        if (reference != null) {
            System.out.println(cleanUpMap.remove(reference));
        } else {
            System.out.println("메모리가 해제된 객체가 없습니다.");
        }

        System.out.println("인스턴스 null로 초기화");
        strong = null;
        Phantom.collect();

        System.out.println("Reference Queue에서 enqueue요청");
        reference = dead.poll();
        if (reference != null) {
            System.out.println(cleanUpMap.remove(reference));
        } else {
            System.out.println("메모리가 해제된 객체가 없습니다.");
        }
        System.out.println("Done");
    }
}
//
phantom references 생성
phantom 객체 : null
GC 요청
//5초후
Reference Queue에서 enqueue요청
메모리가 해제된 객체가 없습니다.
인스턴스 null로 초기화
GC 요청
//5초후
Reference Queue에서 enqueue요청
java.lang.ref.PhantomReference 제거
Done
```
Phantom Reference는 거의 사용되지 않지만 다음과 같은 상황에 사용될 수 있다.

1. 명확하게 메모리에서 제거되었는지 확인이 필요한 경우. 
2. finalize()함수에서 객체를 메모리해제하고 작업을 마치려는 시점에 다시 strong reference를 갖도록 코딩이 되어있으면 GC동작에 있어 심각한 지연을 유발하는데, phantom reference는 메모리가 해제된 상태에서만 ReferenceQueue에서 enqueue 되기 때문에 finalize에서 재부활 할 수 없다.

Phantom Reference를 사용하면 객체가 확실히 제거된 것을 확인한 후에 특정 작업을 수행하도록 코드를 작성 가능하다.


<br><br><br>

### Reference

https://tourspace.tistory.com/37