# 이터레이터(Iterator) 패턴

> `이터레이터 패턴`은 **집합 객체 내부 구조를 노출시키지 않고** **순회하는 방법**을 제공하는 패턴
>

![https://user-images.githubusercontent.com/79291114/150241467-6dfb6efb-36bb-48e1-82d1-96dcc38fcd93.PNG](https://user-images.githubusercontent.com/79291114/150241467-6dfb6efb-36bb-48e1-82d1-96dcc38fcd93.PNG)

- `Iterator` : 집합체의 요소들을 순서대로 검색하기 위한 인터페이스
- `ConcreteIterator` : `Iterator` 인터페이스를 구현
- `Aggregate` : 여러 요소들로 이루어져 있는 집합체
- `ConcreteAggregate` : `Aggregate` 인터페이스를 구현

---

## 예제1

요구사항은 다음과 같다.

- 게시판에 글을 적을 수 있다
- 게시글을 **최근글**으로 정렬해서 볼 수 있다

> `이터레이터 패턴` 적용 전 코드
>

```java
//게시글 클래스
@Getter
@Setter
public class Post {

    private String title;

    private LocalDateTime createdDateTime;

    public Post(String title) {
        this.title = title;
        this.createdDateTime = LocalDateTime.now();
    }
}

//게시판 클래스
@Getter
@Setter
public class Board {
    List<Post> posts = new ArrayList<>();

    public void addPost(String content) {
        this.posts.add(new Post(content));
    }
}

//클라이언트
public class Client {
    public static void main(String[] args) {
        Board board = new Board();
        board.addPost("디자인 패턴 게임");
        board.addPost("선생님, 저랑 디자인 패턴 하나 학습하시겠습니까?");
        board.addPost("지금 이 자리에 계신 여러분들은 모두 디자인 패턴을 학습하고 계신 분들입니다.");
        // TODO 들어간 순서대로 순회하기
        List<Post> posts = board.getPosts();
        for (int i = 0 ; i < posts.size() ; i++) {
            Post post = posts.get(i);
            System.out.println(post.getTitle());
        }
        // TODO 가장 최신 글 먼저 순회하기
        Collections.sort(posts, (p1, p2) ->
            p2.getCreatedDateTime().compareTo(p1.getCreatedDateTime()));
        for (int i = 0 ; i < posts.size() ; i++) {
            Post post = posts.get(i);
            System.out.println(post.getTitle());
        }
    }
}

// 실행 결과
디자인 패턴 게임
선생님, 저랑 디자인 패턴 하나 학습하시겠습니까?
지금 이 자리에 계신 여러분들은 모두 디자인 패턴을 학습하고 계신 분들입니다.
선생님, 저랑 디자인 패턴 하나 학습하시겠습니까?
지금 이 자리에 계신 여러분들은 모두 디자인 패턴을 학습하고 계신 분들입니다.
디자인 패턴 게임
```

- 첫번째 for loop를 보면 List 컬렉션의 순서대로 순회해서 작성한 글 순서대로 정렬된다.
- 두번째 for loop는 최신 글 먼저 순회한다.

> 문제점
>
- `Board`에 들어간 `Post`를 순회할 때, `Board`가 어떠한 구조로 이루어져 있는지를 `Client`가 알게된다. →`메뉴판 제작자(클라이언트)`가 `라멘`과 `김치찌개`를 파는 가게의 메뉴판을 만든다고 가정하겠습니다.`메뉴판 제작자(클라이언트)`가 `라멘`과 `김치찌개`를 파는 가게의 메뉴판을 만든다고 가정하겠습니다. `Board`에 `Post`가 `List`로 담겨있다.
- `Board`가 자료구조를 `List`에서 `Set`이나 `Array`로 바꾸게 되면 `Client` 코드도 변경에 영향을 받게 된다. → `List`인 경우에만 쓰일 수 있음

> `이터레이터 패턴` 적용 후 코드
>

먼저 간단하게 `이터레이터 패턴`을 적용한 순회 코드를 보도록 하자

![https://user-images.githubusercontent.com/42997924/150263614-8a1e7536-6d73-42da-bc59-c90071a6b6ce.png](https://user-images.githubusercontent.com/42997924/150263614-8a1e7536-6d73-42da-bc59-c90071a6b6ce.png)

```java
List<Post> posts = board.getPosts(); // List<Post> posts = new ArrayList<>();
Iterator<Post> iterator = posts.iterator();
```

![https://user-images.githubusercontent.com/42997924/150263487-e4a9b3ef-722d-4059-8243-d632a54a68e4.png](https://user-images.githubusercontent.com/42997924/150263487-e4a9b3ef-722d-4059-8243-d632a54a68e4.png)

자바에서는 `Iterator`인터페이스를 기본적으로 제공해주고 있다. 그리고 `ArrayList`, `LinkedList`등 `List`구현체에는 `Iterator`가 구현되어있다.

이제 UML다이어그램에 대입해보면 `List`는 *`Aggregate Interface`*,  `Iterator`는 `Iterator Interface`가 된다.

`posts`의 실제 구현체인 `ArrayList`가 *`ConcreteAggregate`*, 그리고 `Iterator`에 실질적으로 넘어올 수 있는 여러 타입들이 `ConcreteIterator`가 된다.

출력해서 확인해보면 `ArrayList`의 `Iterator`가 `ConcreteIterator`이다.

클라이언트 코드는 다음과 같다.

```java
public class Client {
    public static void main(String[] args) {
        Board board = new Board();
        board.addPost("디자인 패턴 게임");
        board.addPost("선생님, 저랑 디자인 패턴 하나 학습하시겠습니까?");
        board.addPost("지금 이 자리에 계신 여러분들은 모두 디자인 패턴을 학습하고 계신 분들입니다.");

        // TODO 들어간 순서대로 순회하기
        Iterator<Post> iterator = board.getPosts().iterator();
        while(iterator.hasNext()) {
          System.out.println(iterator.next().getTitle());
        }
    }
}
```

위 코드를 보면 클라이언트는 이제 `Iterator` 객체를 가지고 `hasNext()`, `next()` 메소드만 사용해 순회를 할 수 있게 되었다. 좀 더 클라이언트가 쉽게 사용할 수 있게 하고 쉽다면 아래처럼 `Iterator`를 반환해주는 메소드를 만들어 줄 수 있다.

```java
public class Board {
    List<Post> posts = new ArrayList<>();
    
		public List<Post> getPosts() {
        return posts;
    }
    public void addPost(String content) {
        this.posts.add(new Post(content));
    }
    // Iterator 제공
    public Iterator<Post> getDefaultIterator() {
        return posts.iterator();
    }
}
```

이제 요구사항인 `최근 글` 순서대로 정렬해 순회하는 코드를 보자

```java
public class RecentPostIterator implements Iterator<Post> {
    private Iterator<Post> internalIterator;

    public RecentPostIterator(List<Post> posts) { //Board로 받아도 된다. (선택사항)
      //최신 글 목록이 먼저 오도록 정렬
        Collections.sort(posts, (p1, p2) -> 
        	p2.getCreatedDateTime().compareTo(p1.getCreatedDateTime()));
        this.internalIterator = posts.iterator();
    }
    @Override
    public boolean hasNext() {
        return this.internalIterator.hasNext(); //위임
    }
    @Override
    public Post next() {
        return this.internalIterator.next(); //위임
    }
}

public class Board {
    List<Post> posts = new ArrayList<>();

    public List<Post> getPosts() {
        return posts;
    }
    public void addPost(String content) {
        this.posts.add(new Post(content));
    }
    // Iterator 반환
    public Iterator<Post> getRecentPostIterator() {
    	// this로 Board를 넘겨줘도 되고, posts를 넘겨줘도 됨
        return new RecentPostIterator(this.posts); 
    }
}

public class Client {
    public static void main(String[] args) {
        Board board = new Board();
        board.addPost("디자인 패턴 게임");
        board.addPost("선생님, 저랑 디자인 패턴 하나 학습하시겠습니까?");
        board.addPost("지금 이 자리에 계신 여러분들은 모두 디자인 패턴을 학습하고 계신 분들입니다.");

        // TODO 가장 최신 글 먼저 순회하기
        Iterator<Post> recentPostIterator = board.getRecentPostIterator(); //Iterator 받아와서 사용
        while(recentPostIterator.hasNext()) {
            System.out.println(recentPostIterator.next().getTitle());
        }
    }
}
```

위처럼 최근 글로 정렬할 `Iterator구현체(ConcretIterator)` 인 `RecentPostIterator` 를 구현해 `Board`에서 반환해주면 사용자는 이 `Iterator`가 어떻게 정렬됐는지 알 필요 없이 `hasNext()`, `next()`만으로 최근 글로 정렬된 `List`를 순회할 수 있다.

---

## 예제2

`메뉴판 제작자(클라이언트)`가 `라멘`과 `김치찌개`를 파는 가게의 메뉴판을 만든다고 가정하자.

> `이터레이터 패턴` 적용 전 코드
>

```java
//메뉴 추상 클래스, 가격과 이름을 가짐
public abstract class Menu {
    private int price;
    private String name;

    Menu(String name, int price) {
        this.name = name;
        this.price = price;
    }

    public int getPrice() {
        return price;
    }

    public String getName() {
        return name;
    }
}

//라면 클래스
public class Ramen extends Menu{
    Ramen(String name, int price) {
        super(name, price);
    }
}

//김치찌개
public class KimchiStew extends Menu{
    KimchiStew(String name, int price) {
        super(name, price);
    }
}

//라면 식당
//라면 메뉴들을 HashMap으로 관리
public class RamenStore {
    private final HashMap<Integer, Ramen> menu = new HashMap<>();

    public RamenStore() {
        for (int i = 1; i <= 50; i++) {
            menu.put(i, new Ramen("라멘" + i, 7000 + i * 100));
        }
    }

    public HashMap<Integer, Ramen> getMenu() {
        return menu;
    }
}

//김치 찌개 식당
//김치 찌개 메뉴들을 배열로 관리
public class KimchiStewStore {
    private final KimchiStew[] menu = new KimchiStew[4];

    public KimchiStewStore() {
        menu[0] = new KimchiStew("햄 김치찌개", 7000);
        menu[1] = new KimchiStew("참치 김치찌개", 8000);
        menu[2] = new KimchiStew("삼겹살 김치찌개", 9000);
        menu[3] = new KimchiStew("오겹살 김치찌개", 10000);
    }

    public KimchiStew[] getMenu() {
        return menu;
    }
}

//메뉴판 제작 클라이언트 클래스
//두 식당의 메뉴는 각각 HashMap, 배열로 관리하기에 각 식당마다 메뉴판 함수가 다름
public class MenuBoardCreator {
    public static void main(String[] args) {
        MenuBoardCreator menuBoardCreator = new MenuBoardCreator();

        System.out.println(menuBoardCreator.makeKimchiStewMenuBoard());
        System.out.println(menuBoardCreator.makeRamenMenuBoard());
    }

    private String makeRamenMenuBoard() {
        String menuBoard = "============ 메뉴판 ============\n";

        RamenStore ramenStore = new RamenStore();
        HashMap<Integer, Ramen> menu = ramenStore.getMenu();

        for (Map.Entry<Integer, Ramen> entry : menu.entrySet()) {
            Integer key = entry.getKey();
            Ramen value = entry.getValue();

            menuBoard += "" + key + "." + value.getName() + " : " + value.getPrice() + "\n";
        }

        menuBoard += "===============================";

        return menuBoard;
    }

    private String makeKimchiStewMenuBoard() {
        String menuBoard = "============ 메뉴판 ============\n";

        KimchiStewStore kimchiStewStore = new KimchiStewStore();
        KimchiStew[] menu = kimchiStewStore.getMenu();

        for (int i = 0; i < menu.length ; i++) {
            menuBoard += "" + (i+1) + "." + menu[i].getName() + " : " + menu[i].getPrice() + "\n";
        }

        menuBoard += "===============================";

        return menuBoard;
    }
}

// 출력 결과
============ 메뉴판 ============
1.햄 김치찌개 : 7000
2.참치 김치찌개 : 8000
3.삼겹살 김치찌개 : 9000
4.오겹살 김치찌개 : 10000
===============================
============ 메뉴판 ============
1.라멘1 : 7100
2.라멘2 : 7200
3.라멘3 : 7300
...
...
48.라멘48 : 11800
49.라멘49 : 11900
50.라멘50 : 12000
===============================
```

위 처럼 `식당`마다 `메뉴`관리 방식이 달라 `클라이언트 코드`에서  `메뉴 생성 메소드`를 `식당`마다 따로 정의해준다. 즉 `클라이언트`와의 `종속성`이 높아지게 된다.

> `이터레이터 패턴` 적용 후 코드
>

```java
//김치찌개 이터레이터
public class KimchiStewIterator implements Iterator<KimchiStew> {
    private KimchiStew[] menus;
    private int curPosition = 0;

    public KimchiStewIterator(KimchiStew[] menus) {
        super();
        this.menus = menus;
    }

    @Override
    public boolean hasNext() {
        return curPosition < menus.length && menus[curPosition] != null;
    }

    @Override
    public KimchiStew next() {
        KimchiStew menu = menus[curPosition];
        curPosition++;

        return menu;
    }
}

public class RamenStore {
    ...
        
//    public HashMap<Integer, Ramen> getMenus() {
//        return menus;
//    }
    
    public Iterator<Ramen> createIterator() {
        return menus.values().iterator();
    }
}

public class KimchiStewStore {
    ...

//    public KimchiStew[] getMenus() {
//        return menus;
//    }

    public Iterator<KimchiStew> createIterator() {
        return new KimchiStewIterator(menus);
    }
}

public class MenuBoardCreator {
    public static void main(String[] args) {
        MenuBoardCreator menuBoardCreator = new MenuBoardCreator();

        System.out.println(menuBoardCreator.makeKimchiStewMenuBoard());
        System.out.println(menuBoardCreator.makeRamenMenuBoard());
    }

    private String makeRamenMenuBoard() {
        String menuBoard = "============ 메뉴판 ============\n";

        RamenStore ramenStore = new RamenStore();
        Iterator<Ramen> menu = ramenStore.createIterator();
        int i = 1;

        // 기존의 방식과 다르게 hasnext()와 next()를 이용해 순회
        while(menu.hasNext()) {
            Ramen ramen = menu.next();

            menuBoard += "" + i + "." + ramen.getName() + " : " + ramen.getPrice() + "\n";
            ++i;
        }

        menuBoard += "===============================";

        return menuBoard;
    }

    private String makeKimchiStewMenuBoard() {
        String menuBoard = "============ 메뉴판 ============\n";

        KimchiStewStore kimchiStewStore = new KimchiStewStore();
        Iterator<KimchiStew> menu = kimchiStewStore.createIterator();
        int i = 1;

        // 기존의 방식과 다르게 hasnext()와 next()를 이용해 순회
        while(menu.hasNext()) {
            KimchiStew kimchiStew = menu.next();

            menuBoard += "" + i + "." + kimchiStew.getName() + " : " + kimchiStew.getPrice() + "\n";
            ++i;
        }

        menuBoard += "===============================";

        return menuBoard;
    }
}

// 출력 결과(동일)
============ 메뉴판 ============
1.햄 김치찌개 : 7000
2.참치 김치찌개 : 8000
3.삼겹살 김치찌개 : 9000
4.오겹살 김치찌개 : 10000
===============================
============ 메뉴판 ============
1.라멘1 : 7100
2.라멘2 : 7200
3.라멘3 : 7300
...
...
48.라멘48 : 11800
49.라멘49 : 11900
50.라멘50 : 12000
===============================
```

이터레이터를 사용하게 되면 클라이언트에서 각 가게의 자료구조를 알 필요 없이, 각 가게의 이터레이터만을 이용해 메뉴판을 제작할 수 있어 종속성이 낮아집니다.

위 클라이언트 코드는 이터레이터 패턴 적용전 코드와 비교하기 위해 메소드 두개를 통해 구현했지만 만약  각 식당이 하나의 `Store`를 상속받는다면 `Store`를 순회하면서 `Iterator`를 호출 시키는 방식으로 더 간단한 코드를 만들 수 있을 것이다.

---

## Collection의 구조

아래와 같이 Collection은 Iterable을 상속받기 때문에 **Collection 자료구조를 사용하면 Iterator를 사용**할 수 있습니다.

하지만 **Collection을 상속받지 않는 Map은 자체적으로 Iterator를 사용할 수 없기 때문에 위의 예시와 같이 Map의 Key나 Value를 keySet() 이나 values()를 통해 Collection을 상속받는 Set이나 Collection을 반환 후 Iterator를 사용**할 수 있습니다.

![https://user-images.githubusercontent.com/58713853/73814627-f84d6400-4826-11ea-95dc-0de32bd8be6c.PNG](https://user-images.githubusercontent.com/58713853/73814627-f84d6400-4826-11ea-95dc-0de32bd8be6c.PNG)

![https://user-images.githubusercontent.com/79291114/115977688-29918b80-a5b5-11eb-85b1-84c72a9c9be9.png](https://user-images.githubusercontent.com/79291114/115977688-29918b80-a5b5-11eb-85b1-84c72a9c9be9.png)

---

## 이터레이드 패턴의 장단점

> 장점
>
- 집합 객체가 가지고 있는 객체들에 손쉽게 접근할 수 있다.
    - Iterator가 제공하는 인터페이스만 알면되고, 집합 객체의 구조는 알 필요없다.
- 일관된 인터페이스를 사용해 여러 형태의 집합 구조를 순회할 수 있다.
    - **단일책임원칙 -** hasNext()와 getNext()만 호출해도 됨.
    - **OCP -** 기존 코드의 큰 변경 없음 (상황에 따라 변경되기도 함)

> 단점
>
- 클래스가 늘어나고 복잡도가 증가한다.
    - 이터레이터를 만드는 것이 유용한 상황인지 판단할 필요가 있다.
    - 다양한 방법으로 순회하는 방법이 필요하고 내부의 집합 구조가 객체가 변경될 가능성이 있다면, 내부 구조를 클라이언트 쪽에게 숨기는 방법으로 이터레이터 패턴을 적용하는 것이 좋은 방법이 될 수 있다.

---

## Java, Spring 적용 사례

> 자바
>
- java.util.Enumeration과 java.util.Iterator
- Java StAX (Streaming API for XML)의 Iterator 기반 API
    - XmlEventReader, XmlEventWriter

> 스프링
>
- CompositeIterator

### ****자바 - java.util.Enumeration****

- Iterator가 만들어지기 전 java 1.0부터 있었던 API
- hasMoreElements(), nextElement()
- Iterator로 거의 대체되었다.
- java 9 에서는 Iterator로 변환해주는 코드가 추가되었다.

### ****자바 - java.util.Iterator****

- hasNext(), next(), remove(), forEachRemainint()
    - **remove()** : Iterator에서 next()로 받았던 해당 엘리먼트를 삭제해준다.
        - 모든 이터레이터에서 다 지원하는 것은 아니다.
            - → UnsupportedOperationException()을 던지는 경우가 많다.
        - 보통 이 기능은 Concurrent Modification 동시에 다발적으로 같은 오퍼레이션을 수행해도 안전한 컬렉션에서 제공한다.
    - **forEachRemainint()**
        - 순회를 좀 더 쉽게 해주도록 도와준다.
        - Consumer 라는 Functional Interface를 파라미터로 받아서 컬렉션을 순회하면서 다 적용해준다.

```java
public class IteratorInJava {

    public static void main(String[] args) throws FileNotFoundException, XMLStreamException {
        Enumeration enumeration;
        Iterator iterator;

        Board board = new Board();
        board.addPost("디자인 패턴 게임");
        board.addPost("선생님, 저랑 디자인 패턴 하나 학습하시겠습니까?");
        board.addPost("지금 이 자리에 계신 여러분들은 모두 디자인 패턴을 학습하고 계신 분들입니다.");
      
        // forEachRemaining()
        // 단순히 소모하는 Functional Interface
        board.getPosts().iterator().forEachRemaining(p -> System.out.println(p.getTitle()));
        
        // 위 코드와 동일
        Iterator<Post> iterator1 = board.getPosts().iterator();
        while (iterator1.hasNext()) {
          Post next = iterator1.next();
          System.out.println(next.getTitle());
        }
    }
}
```

### ****자바 - Java StAX (Streaming API for XML)의 Iterator 기반 API****

- 자바가 제공해주는 라이브러리
- XML을 만들거나 읽을 때 사용할 수 있다.

콘솔 기반의 API, 이터레이터 기반의 API를 제공

- **이터레이터 기반의 API**
    - XMl Element 당 **이벤트**가 지나가면서 캡쳐한다. 해당 영역을 지나가면서 그 영역을 표현하는 XMLEvent라는 인스턴스가 새롭게 만들어진다.
- **콘솔 기반의 API**
    - 커서 기반의 API - 하나의 인스턴스가 Element를 지나가면서 안의 내용들이 갱신되는 방식이다.

콘솔 기반의 API가 메모리 리소스를 덜 사용하므로 더 효율적이다.
하지만, 일반적으로 이터레이터 기반의 API를 사용하는 것을 권장한다.
만들어진 인스턴스를 재사용하거나 변경하는 등의 유연한 처리를 하기 위해서는 (안의 상태가 변경되는 것보다) Immutable하게 하나하나 제 각각의 XML Element를 표현하는 이터레이터 기반의 API가 다루기 용이하다.

**이터레이터 기반의 API 코드 예제**

- XmlEventReader, XmlEventWriter

```java
// TODO Streaming API for XML(StAX), 이터레이터 기반의 API - XML 읽어오는 코드
XMLInputFactory xmlInputFactory = XMLInputFactory.newInstance();
XMLEventReader reader = xmlInputFactory.createXMLEventReader(new FileInputStream("Book.xml"));

while (reader.hasNext()) {
    XMLEvent nextEvent = reader.nextEvent();
    if (nextEvent.isStartElement()) {
        StartElement startElement = nextEvent.asStartElement();
        QName name = startElement.getName();
        if (name.getLocalPart().equals("book")) {
            Attribute title = startElement.getAttributeByName(new QName("title"));
            System.out.println(title.getValue());
        }
    }
}

<?xml version="1.0" encoding="UTF-8"?>
<books>
    <book title="오징어 게임"/>
    <book title="숨바꼭질"/>
    <book title="우리집에 왜 왔니"/>
</books>
```

- Book.xml의 title만 읽어들인다.
- hasNext(), nextEvent() XMl 엘리먼트를 순회한다.
    - → isStartElement() 시작하는 태그만
    - → nextEvent.asStartElement().getName().getLocalPart().equals("book") 태그 이름이 "book"인 것만
    - → 그 중 "title" 속성값만 출력

**참고**SAX (Simple API for XML)와 다른 것임. SAX는 XML을 읽기만 가능하다.

### ****스프링 - CompositeIterator****

- 기존의 Interator에 기능만 하나 추가한 것이다.
- add()
    - 여러 Iterator들을 Composite(조합)해서 사용할 수 있다.

```java
public class IteratorInSpring {

    public static void main(String[] args) {
        CompositeIterator iterator;
    }
}
```

CompositeIterator는 컴포짓 패턴과 겹치는 내용이 있어서 비교하면 좋을 것 같다.