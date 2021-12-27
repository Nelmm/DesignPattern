## 1. 데코레이터 패턴(Decorator Pattern)이란? 

> 기존 코드를 변경하지 않고 부가 기능을 동적으로(유연하게) 추가하는 패턴

상속이 아닌 위임을 사용해서 보다 유연하게(**런타임에**) 부가 기능을 추가하는 것도 가능하다.
기능 확장이 필요할 때 상속대신 사용할 수 있는 패턴.

![1](https://user-images.githubusercontent.com/42997924/147181146-0404abec-a5ae-4fdf-9edf-506696d813b6.png)

### 1-1. 구조

-   **Component (Interface)**
    -   ConcreteComponent와 Decorator가 이를 구현하고 있다. 같은 오퍼레이션을 가지고 있다.
-   **Decorator**
    -   컴포짓 패턴의 Composite과 비슷해보이지만 차이점이 있다.
    -   Decorator는 단 하나의 `wrappee`라고 하는 Component 타입의 인스턴스 타입을 가지고 있다.
    -   자신이 감싸고 있는 하나의 Component를 (오퍼레이션 내부에서) 호출하면서 호출 전/후로 부가적인 로직을 추가할 수 있다.

### 1-2. 언제 활용될까

-   상황에 따라 다양한 기능이 빈번하게 추가/삭제되는 경우
-   객체의 결합을 통해 기능이 생성될 수 있는 경우

## 2. 코드로 알아보는 데코레이터 패턴 필요한 상황

> 1. 댓글을 달고 싶어요.  
> 2. 댓글에 공백이 없으면 좋겠어요.  
> 3. 광고성 댓글은 필터링 됐으면 좋겠어요.  
> 4. 공백도 없고 필터링도 됐으면 좋겠어요.  
> 5. 욕설을 제거됐으면 좋겠어요.  
> 6. 공백 +  광고 필터링 + 욕설 필터링이 됐으면 좋겠어요.  
>    ...

클라이언트는 결국 댓글(Comment)을 추가하는 행동을 하지만, **요구사항이 빈번하게 변경되는 상황**이다.

### 2-1. 댓글을 달고 싶어요.

**Client 코드**

-   댓글을 다는 메소드를 호출한다. 변경되지 않았으면 하는 코드 이다.

```
public class Client {
    
    private CommentService commentService;
    public Client(CommentService commentService) {
        this.commentService = commentService;
    }
    
    private void writeComment(String comment) {
        commentService.addComment(comment);
    }
    
}
```

**ClientService 코드**

-    댓글을 다는 기능(메소드)을 제공한다.

```
public class CommentService {
    public void addComment(String comment) {
        System.out.println(comment);
    }
}
```

**Main 코드**

```
public class CommentApp {
	Client client = new Client(new CommentService());
	client.writeComment("디자인패턴 스터디");
	client.writeComment("보는게 하는거 보다 재밌을 수가 없지...");
	client.writeComment("https://github.com/jun108059");
}
```

### 2-2. 댓글에 공백이 없었으면 좋겠어요.

> CommentService를 확장해서 trim 기능을 추가로 제공해주는 별도의 서비스인 TrimmingCommentService를 만들어 사용한다.

**TrimmingCommentService 코드**

-   CommentService 상속받아서 확장
-   addComent()하기 전에 trim을 해주도록 기능 추가

```
public class TrimmingCommentService extends CommentService {
    @Override
    public void addComment(String comment) {
        super.addComment(trim(comment));
    }
    private String trim(String comment) {
        return comment.replace("...", "");
    }
}
```

-   Client에서 사용
    -   `new TrimmingCommentService()` 구현체를 넣어줘서 사용한다.

```
public class CommentApp {
	Client client = new Client(new TrimmingCommentService());
	client.writeComment("디자인패턴 스터디");
	client.writeComment("보는게 하는거 보다 재밌을 수가 없지..."); // 공백 제거
	client.writeComment("https://github.com/jun108059");
}
```

기존 코드 변경없이 새로운 기능을 추가할 수 있다.

하지만 이 방식은 **상속을 사용해서 컴파일 타임에 고정적으로 의존성이 정해진다.** (유연하지 않다.)

### 2-3. 광고성 댓글은 필터링 됐으면 좋겠어요.  

> 광고 필터링을 하는 CommentService를 또 만들 수 있다.

**SpamFilteringCommentService 코드**

-   comment에 http 문자가 들어가 있으면 스팸처리하여 출력하지 않는다.

```
public class SpamFilteringCommentService extends CommentService {
    @Override
    public void addComment(String comment) {
        boolean isSpam = isSpam(comment);
        if (!isSpam) {
            super.addComment(comment);
        }
    }
    private boolean isSpam(String comment) {
        return comment.contains("http");
    }
}
```

-   `new SpamFilteringCommentService()` 구현체를 넣어줘서 사용한다.

```
public class CommentApp {
	Client client = new Client(new CommentService());
	client.writeComment("디자인패턴 스터디");
	client.writeComment("보는게 하는거 보다 재밌을 수가 없지...");
	client.writeComment("https://github.com/jun108059");
}
```

### 2-4. 공백도 없고 필터링도 됐으면 좋겠어요.

>  두개의 기능을 모두 담은 새로운 Service 클래스를 생성해야 한다.

이 때 부터 단일 상속의 문제를 느끼게 된다.

단일 상속만 되기 때문에, 두 가지 기능을 제공하는 TrimAndSpamFilteringCommentService를 또 하나 만들게 된다.

**클라이언트 코드는 바뀌지 않지만 상속만으로 확장해나가기 불편한 구조이다.**

요구사항이 추가돼서 경우에 따라서 동적으로 필터링을 적용하거나 안하거나 해야한다면?

`enabledSpamFilter`, `enabledTrimmin` 플래그 설정 값에 따라 동적으로 필터링서비스를 골라서 적용하고 싶다.

```
public class Client {
    private CommentService commentService;
        private boolean enabledSpamFilter;

      private boolean enabledTrimmin;
  //..생략
}
```

이 문제를 데코레이터 패턴으로 해결할 수 있다.

## 3. 데코레이터 패턴 적용

-   **CommentService**
    -   Component (Interface)
-   **DefaultCommentService**
    -   ConcreteComponent
    -   기존에 CommentService에서 구체적으로 하던 작업을 여기에 구현
-   **CommentDecorator**
    -   Decorator
    -   SpamFilteringCommentService와 TrimmingCommentService를 추상화 시킨 데코레이터
-   **SpamFilteringCommentDecorator**, **TrimmingCommentDecorator**
    -   ConcreteDecorator
    -   SpamFilteringCommentService와 TrimmingCommentService에서 하던 작업을 여기에 구현


![2](https://user-images.githubusercontent.com/42997924/147181151-f7a4ba8c-de4f-4645-97b4-b74d8dab0d34.png)

### 3-1. Component (Interface) 정의 - CommentService

```
public interface CommentService {
    void addComment(String comment);
}
```

### 3-2. ConcreteComponent 구현 - DefaultCommentService

-   기존에 CommentService에서 구체적으로 하던 작업을 여기에 구현

```
public class DefaultCommentService implements CommentService {
    @Override
    public void addComment(String comment) {
        System.out.println(comment);
    }
}
```

### 3-3. Decorator 구현

-   동일한 `CommentService` 타입이어야 한다.
-   딱 하나의 `wrappee`라고 하는 하나의 Component 타입의 인스턴스 타입을 가지고 있다.
-   ⇒ `CommentService` 하나를 가지고 있는 형태이다.
-   **그냥 가지고 있는 Component를 호출해주기만 하면 된다. (데코레이터의 역할 끝)**

```
public class CommentDecorator implements CommentService {
    private CommentService commentService;
    public CommentDecorator(CommentService commentService) {
        this.commentService = commentService;
    }
    @Override
    public void addComment(String comment) {
        commentService.addComment(comment);
    }
}
```

### 3-4. Trim 기능을 제공하는 Decorator 구현 - TrimmingCommentDecorator

-   `CommentDecorator` 를 구현
-   `addComment()`를 오버라이드 하는데, 이 때 부가적인 trim 기능을 추가한다.

```
public class TrimmingCommentDecorator extends CommentDecorator {
    public TrimmingCommentDecorator(CommentService commentService) {
        super(commentService);
    }
    @Override
    public void addComment(String comment) {
        super.addComment(trim(comment)); //trim 기능 추가
    }
    private String trim(String comment) {
        return comment.replace("...", "");
    }
}
```

### 3-5. Spam Filter 기능을 제공하는 Decorator 구현 - SpamFilteringCommentDecorator

-   `CommentDecorator` 를 구현
-   `addComment()`를 오버라이드 하는데, 이 때 부가적인 스팸 필터 기능을 추가한다.

```
public class SpamFilteringCommentDecorator extends CommentDecorator {
    public SpamFilteringCommentDecorator(CommentService commentService) {
        super(commentService);
    }
    @Override
    public void addComment(String comment) {
        if (isNotSpam(comment)) { // 스팸 필터 기능 추가
            super.addComment(comment);
        }
    }
    private boolean isNotSpam(String comment) {
        return !comment.contains("http");
    }
}
```

CommentDecorator(`Decorator`)에 들어올 수 있는 것은 DefaultCommentService(`ConcreteComponent`)여도 되고, SpamFilteringCommentDecorator, TrimmingCommentDecorator인 `ConcreteDecorator` 여도 된다.

`CommentService` 타입이기만 하면 된다.

이런 유연성을 가지게하기 위해서 Decorator와 ConcreteComponent가 같은 인터페이스를 구현하도록 한 것이다.

### 3-6. Client

-   인터페이스 타입인 CommentService를 사용하면 된다.

```
public class Client {
    private CommentService commentService;
    public Client(CommentService commentService) {
        this.commentService = commentService;
    }
    public void writeComment(String comment) {
        commentService.addComment(comment);
    }
}
```

### 3-7. Application 코드

-   Client가 사용할 구체적인 CommentService 구현체는 런타임시에 동적으로 바뀔 수 있게 된다.
-   SpamFilter도 사용하고 Trim도 사용한다고 하면, Decorator가 Decorator를 감싸는 구조가 된다.
-   즉, 두 기능 모두 수행하게 된다.

```
public class App {
    private static boolean enabledSpamFilter = true;
    private static boolean enabledTrimming = true;
    public static void main(String[] args) {
        CommentService commentService = new DefaultCommentService();
        // 런타임시에 플래그 값에 따라 commentService가 결정된다.
        if (enabledSpamFilter) {
            commentService = new SpamFilteringCommentDecorator(commentService);
        }
        if (enabledTrimming) {
            commentService = new TrimmingCommentDecorator(commentService);
        }
        Client client = new Client(commentService);
        client.writeComment("디자인패턴 스터디");
        client.writeComment("보는게 하는거 보다 재밌을 수가 없지...");
        client.writeComment("https://github.com/jun108059");
    }
}
```

⇒ 또 다른 기능을 추가할 때 상속을 사용한다면 또 다른 상속 클래스를 만들어야 했지만, 이제는 Decorator가 Decorator를 감쌀 수 있는 구조이기 때문에 유연하게 사용할 수 있다.

> main 코드가 늘어나는 것이 아닐까?  
> yes. 이 부분의 코드는 객체를 동적으로 조합해서 전달해주는 부분이다.

스프링부트를 쓰고 있다면 자바 메소드를 통해 빈을 정의할 수 있다. 이 코드에 application.properties에 설정한 값에 따라 각기 다른 빈을 만들어서 전달해주도록 하면 된다.

[다른 예제](/구조/5주차-데코레이터/hong.md)

## 4. 장단점

### 4-1. 장점

-   새로운 클래스를 만들지 않고 기존 기능을 조합할 수 있다.
    -   데코데리터는 자신이 해야하는 일만 하고, 이를 조합하는 것은 사용하는 측에서 정한다.
    -   ⇒ `SRP(단일 책임 원칙)` 객체지향 원칙을 따른다.
    -   Component 코드를 수정하지 않고, Client 코드도 수정하지 않으면서 기능을 확장할 수 있다.
    -   ⇒ `OCP(개방 폐쇄 원칙)` 객체지향 원칙을 따른다.
-   컴파일 타임이 아닌 런타임에 동적으로 기능을 변경할 수 있다.
-   Client가 구현체가 아닌 인터페이스를 사용한다.
-   ⇒ `DI(의존 역전 원칙)` 객체지향 원칙을 따른다.

### 4-2. 단점

-   데코레이터를 조합하는 코드가 복잡할 수 있다.

## 5. 실무 사용 예

-   자바
    -   InputStream, OutputStream, Reader, Writer의 생성자를 활용한 랩퍼
    -   java.util.Collections가 제공하는 메소드들 활용한 랩퍼
    -   javax.servlet.http.HttpServletRequest/ResponseWrapper
-   스프링
    -   ServerHttpRequestDecorator

### 5-1. InputStream, OutputStream, Reader, Writer의 생성자를 활용한 랩퍼

-   어댑터 패턴에서 다뤘던 예시와 같다.
-   어댑터 패턴의 목적 : 한 인터페이스를 다른 인터페이스로 변환한다.
-   패턴의 목적에 따라 어댑터 패턴이라 볼 수도 있고 데코레이터 패턴이라 볼 수도 있다.
-   하나의 타입을 다른 타입이 감싸는 구조이다.
-   InputStream → InputStreamReader → BufferedReader
-   이 과정에서 부가기능이 추가된다. 점점 더 로우 레벨을 고차원 수준의 API로 다룰 수 있도록 추가된다.

```
public class AdapterInJava {
    public static void main(String[] args) {
        // io
        try(InputStream is = new FileInputStream("input.txt");
            InputStreamReader isr = new InputStreamReader(is);
            BufferedReader reader = new BufferedReader(isr)) {
            while(reader.ready()) {
                System.out.println(reader.readLine());
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

### 5-2. java.util.Collections가 제공하는 메소드들 활용한 랩퍼

-   `checkedXXX()` : 기존 컬렉션 인스턴스를 부가적인 기능을 추가해서 다른 타입으로 변환해주는 메소드
    -   타입 체크를 제공해준다.
    -   해당 컬렉션의 오퍼레이션을 실행할 때, 컬렉션에 들어가는 인스턴스들의 타입을 확인한다.
-   `synchronizedXXX()` : 컬렉션에 여러 오퍼레이션이 들어올 때, 동시에 접근하지 못하도록 동기화 처리하는 메소드
-   `unmodifiableXXX()` : 컬렉션을 불변 객체로 취급하는 오퍼레이션
    -   wrapper을 이용해서 기능을 변경한 것이다.

```
public class DecoratorInJava {
    public static void main(String[] args) {
        // Collections가 제공하는 데코레이터 메소드
        ArrayList list = new ArrayList<>(); //어떠한 타입도 들어갈 수 있는 컬렉션
        list.add(new Book());
      // list 컬렉션에서 Book 클래스에 해당하는 타입만 선별해서 books에 담는다.
        List books = Collections.checkedList(list, Book.class);
      list.add(new Item()); //-> 가능
//        books.add(new Item()); //-> 불가능. ClassCastException 발생. 타입이 맞지 않아서
        List unmodifiableList = Collections.unmodifiableList(list);
        list.add(new Item()); //-> 가능. mutable하기 때문에
        unmodifiableList.add(new Book()); //-> 불가능. 불변객체이기 때문에
    }
    private static class Book {
    }
    private static class Item {
    }
}
```

### 5-3. javax.servlet.http.HttpServletRequest/ResponseWrapper

서블릿에서 제공해주는 Wrapper로 일종의 데코레이터 패턴이라고 볼 수 있다.

-   HttpServletRequest를 확장해서 HttpServletRequestWrapper가 제공하는 기능을 오버라이딩해서 부가적인 기능을 추가할 수 있다.
-   ex) HTTP 요청 메시지 본문을 다르게 처리해서 본다. 캐싱한다. 로깅을 남긴다. 의심스러운 요청 확인 등등의 작업을 해야할 때, 이런 wrapper를 만들어서 사용할 수 있다.
-   wrapper을 만들어서 HttpServletRequest를 담고, filter를 거치도록 하면, 항상 이 wrapper을 거쳐서 요청이 처리된다.

```
public static void main(String[] args) {
  // 서블릿 요청 또는 응답 랩퍼
  HttpServletRequestWrapper requestWrapper;
  HttpServletResponseWrapper responseWrapper;
}
```

![3](https://user-images.githubusercontent.com/42997924/147181157-3abb8a72-beae-4537-9c29-47a90715fe68.png)

### 5-4. ServerHttpRequestDecorator

-   BeanDefinitionDecorator
    -   빈 설정 데코레이터
    -   직접사용할 일이 드물다. (스프링의 인프라쪽)
-   ServerHttpRequestDecorator / ServerHttpResponseDecorator
    -   웹플럭스 HTTP 요청 /응답 데코레이터
    -   Webflux를 쓰면 사용하게될 수도 있다.
    -   ServerHttpRequest와 ServerHttpResponse를 커스터마이징할 수 있는 인터페이스이다.
    -   이 데코레이터를 상속받는 클래스를 만들어서WebFilter를 거쳐가는 모든 요청이 이 데코레이터의 하위 클래스를 거쳐가게 된다.
    -   리액티브의 WebFilter를 상속해서 만든 Filter를 거쳐가는 요청인 ServerHttpRequest를 (데코레이터에) 담아준다.



## 다른 패턴들과 비교

- 어댑터 : 기능을 확장한다는 점에서 비슷하다고 할 수 있으나 어댑터는 기존 인터페이스와는 다른 인터페이스를 반환하고, 데코레이터는 인터페이스를 변경하지 않고 기능이 향상된 인터페이스를 제공한다.
- 컴포짓 : 합성을 이용한 패턴이라는 점에서는 비슷하나 컴포짓은 관련된 기능을 가지고 있는 객체들을 하나의 추상층으로 묶어 결과를 집계(요약) 한다면, 데코레이터는 해당 객체에 책임을 추가한다.

  

  ![Composite](/객체생성/3주차-빌더/image/abstractFactory-architecture.png)
