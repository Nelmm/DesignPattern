# 이터레이터(Iterator) 패턴

**이터레이터 패턴은 집합 객체 내부 구조를 노출시키지 않고 순회하는 방법을 제공하는 패턴**입니다.



## 다이어그램

![iterator](https://user-images.githubusercontent.com/79291114/150241467-6dfb6efb-36bb-48e1-82d1-96dcc38fcd93.PNG)

- `Iterator` : 집합체의 요소들을 순서대로 검색하기 위한 인터페이스
- `ConcreteIterator` : `Iterator` 인터페이스를 구현
- `Aggregate` : 여러 요소들로 이루어져 있는 집합체
- `ConcreteAggregate` : `Aggregate` 인터페이스를 구현



## 예시 코드

`메뉴판 제작자(클라이언트)`가 `라멘`과 `김치찌개`를 파는 가게의 메뉴판을 만든다고 가정하겠습니다.



### 이터레이터 적용 전

### Menu 추상 클래스

매뉴 추상 클래스입니다. 가격과 이름을 가지고 있습니다.

```java
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
```



### 라멘, 김치찌개 클래스

라멘, 김치찌개 모두 메뉴를 상속받습니다.

```java
public class Ramen extends Menu{
    Ramen(String name, int price) {
        super(name, price);
    }
}

public class KimchiStew extends Menu{
    KimchiStew(String name, int price) {
        super(name, price);
    }
}
```



### 라멘, 김치찌개 식당 클래스

라멘가게는 많은 메뉴의 라멘을 판매하기 때문에 데이터를 `HashMap`으로 관리합니다.

김치찌개가게는 적은 종류의 김치찌개만을 판매하기 때문에 배열로 관리합니다.

```java
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
```



### 메뉴판 제작 클래스

두 가게는 **다른 자료구조를 이용하기 때문에 각 식당에 대한 메뉴판 제작 함수를 별도로 만들어주어야 합니다.**

```java
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

해당 함수에서 각 가게의 자료구조를 전부 알고있어야 하기 때문에 **종속성이 높아져, 만약 각 가게의 자료구조가 변경된다면 클라이언트 코드까지 바꿔주어야 합니다.**





## 이터레이터 적용 후

김치찌개가게의 자료구조는 **배열이기 때문에 Iterator 정의가 필요**합니다.

반면, 라멘가게의 자료구조는 `Map`인데, 이 때 **key나 value를 keySet() 이나 values()로 Collection이나 Collection을 상속받는 Set으로 반환**할 수 있기 때문에 **keySet()이나 values()의 iterator()를 호출**할 수 있어, 따로 Iterator를 정의하지 않아도 됩니다.

> Collection은 Iterator(정확히는 Iterable)를 상속받습니다.



### 김치찌개 이터레이터

`Iterator<KimchiStew>`를 상속받는 김치찌개 이터레이터는 생성 시 김치찌개 자료구조를 받고 **Iterator의 hasNext()와 next()를 Override**하게 됩니다.

```java
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
```



### 라멘, 김치찌개 클래스

각 클래스에서는 **기존의 getMenus() 대신 createIterator()로 각 클래스에 해당하는 Iterator를 생성**합니다.

```java
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
```



### 메뉴판 제작 클래스

메뉴판 제작 클래스의 각 메뉴를 만들어주는 함수는

```java
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

이터레이터를 사용하게 되면 **클라이언트에서 각 가게의 자료구조를 알 필요 없이, 각 가게의 이터레이터만을 이용해 메뉴판을 제작할 수 있어 종속성이 낮아집니다.**





## Collection

아래와 같이 Collection은 Iterable을 상속받기 때문에 **Collection 자료구조를 사용하면 Iterator를 사용**할 수 있습니다.

하지만 **Collection을 상속받지 않는 Map은 자체적으로 Iterator를 사용할 수 없기 때문에 위의 예시와 같이 Map의 Key나 Value를 keySet() 이나 values()를 통해 Collection을 상속받는 Set이나 Collection을 반환 후 Iterator를 사용**할 수 있습니다.

![컬렉션](https://user-images.githubusercontent.com/58713853/73814627-f84d6400-4826-11ea-95dc-0de32bd8be6c.PNG)

---

<img src="https://user-images.githubusercontent.com/79291114/115977688-29918b80-a5b5-11eb-85b1-84c72a9c9be9.png" alt="Collection" style="zoom: 50%;" />





---

참고 : [코딩으로 학습하는 GoF의 디자인 패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)

[https://lktprogrammer.tistory.com/40](https://lktprogrammer.tistory.com/40)

[https://gdtbgl93.tistory.com/143](https://gdtbgl93.tistory.com/143)

