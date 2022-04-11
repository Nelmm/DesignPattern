## 1. 이터레이터 패턴(Iterator Pattern)이란?

`이터레이터 패턴`은 **집합 객체 내부 구조를 노출시키지 않고** **순회하는 방법**을 제공하는 패턴

[![https://user-images.githubusercontent.com/79291114/150241467-6dfb6efb-36bb-48e1-82d1-96dcc38fcd93.PNG](https://user-images.githubusercontent.com/79291114/150241467-6dfb6efb-36bb-48e1-82d1-96dcc38fcd93.PNG)](https://user-images.githubusercontent.com/79291114/150241467-6dfb6efb-36bb-48e1-82d1-96dcc38fcd93.PNG)

- `Iterator` : 집합체의 요소들을 순서대로 검색하기 위한 인터페이스
- `ConcreteIterator` : `Iterator` 인터페이스를 구현
- `Aggregate` : 여러 요소들로 이루어져 있는 집합체
- `ConcreteAggregate` : `Aggregate` 인터페이스를 구현





## 2. 코드로 알아보는 이터레이터 패턴

### 2-1. 요구사항

- TOP 상품 영역에 채용 공고 리스트를 출력해야함
- 채용 공고 마감 순으로 정렬 가능

이터레이터 패턴을 적용하기전에 기본적으로 리스트를 출력하는 예제 코드를 보자.

```java
/**
 * 채용 공고 클래스
 **/
public class Recruit {
    String name;
    LocalDateTime startDt;
    LocalDateTime endDt;

    public Recruit(String name, LocalDateTime startDt, LocalDateTime endDt) {
        this.name = name;
        this.startDt = startDt;
        this.endDt = endDt;
    }
		//...Getter Setter
}

public class Main {
    public static void main(String[] args) {
        List<Recruit> recruits = new ArrayList<>();
        recruits.add(new Recruit("채용공고1", LocalDateTime.of(2020, 3,7,0,0),LocalDateTime.of(2020, 3,11,0,0)));
        recruits.add(new Recruit("채용공고2", LocalDateTime.of(2020, 3,7,0,0),LocalDateTime.of(2020, 3,10,0,0)));
        recruits.add(new Recruit("채용공고3", LocalDateTime.of(2020, 3,7,0,0),LocalDateTime.of(2020, 3,12,0,0)));

        //List의 size()와 get()을 이용한 Loop
        for(int i=0; i< recruits.size(); i++) {
            Recruit r = recruits.get(i);
            System.out.printf("%s : %s~%s\n", r.getName(), r.getStartDt().toString(), r.getEndDt().toString());
        }

        System.out.println("===========================");

        //공고 마감순 정렬
        recruits.sort(Comparator.comparing(Recruit::getEndDt));
        //List의 size()와 get()을 이용한 Loop
        for(int i=0; i< recruits.size(); i++) {
            Recruit r = recruits.get(i);
            System.out.printf("%s : %s~%s\n", r.getName(), r.getStartDt().toString(), r.getEndDt().toString());
        }

				//Set을 사용한 루프
				Set<Recruit> set = new HashSet<>();
        for(int i=0; i< set.size(); i++) {
            //컴파일 에러!!!
            Recruit r = set.get(i);
            System.out.printf("%s : %s~%s\n", r.getName(), r.getStartDt().toString(), r.getEndDt().toString());
        }
    }
}

결과
채용공고1 : 2020-03-07T00:00~2020-03-11T00:00
채용공고2 : 2020-03-07T00:00~2020-03-10T00:00
채용공고3 : 2020-03-07T00:00~2020-03-12T00:00
===========================
채용공고2 : 2020-03-07T00:00~2020-03-10T00:00
채용공고1 : 2020-03-07T00:00~2020-03-11T00:00
채용공고3 : 2020-03-07T00:00~2020-03-12T00:00
```

위의 코드에서는 `List`를 이용해 공고 리스트를 출력하고 있다.

그리고 `List`만 사용할 수 있는 루프를 사용하고 있다.(`List.get()`사용)

그래서 `Set` 을 사용하여 똑같은 루프를 사용하면 위 코드처럼 컴파일 에러가 발생한다.

이것의 문제점은 클라이언트에서 `List<Recruit>`라는 구조를 직접 사용한다는 것.

즉, `커플링`이 높다는 뜻이다.

추후 `채용 공고`를 담는 자료구조가 `List`에서 `Set`, `Array`등으로 변경될 경우 **클라이언트 코드의 루프문도 함께 변경되야 하는 문제가 생긴다.**

그렇다면 이제 이터레이터 패턴을 적용해보자.

![Untitled](https://user-images.githubusercontent.com/32676275/157182135-e0612384-48c2-4558-aecf-d23a2b02dab8.png)

```java
/**
 * 채용 공고 클래스
 **/
public class Recruit {
    String name;
    LocalDateTime startDt;
    LocalDateTime endDt;

    public Recruit(String name, LocalDateTime startDt, LocalDateTime endDt) {
        this.name = name;
        this.startDt = startDt;
        this.endDt = endDt;
    }
		//...Getter, Setter
}

/**
 * Iterator 인터페이스
 * ** Iterator **
 */
public interface CustomIterator<T> {
    //순회중 다음 객체 반환
    T getNext();
    //다음 객체가 존재하는지 체크
    Boolean hasNext();
}

/**
 * 리스트 채용공고를 날짜순 Iterator로 변환해주는 CustomIterator 구현체
 */
public class RecentRecruitIteratorForList implements CustomIterator<Recruit> {
    //리스트 채용 공고 Aggregate
    private final List<Recruit> recruits;
    //Iterator 순회중 현재 인덱스
    private Integer curPosition = 0;

    //생성자
    public RecentRecruitIteratorForList(List<Recruit> recruits) {
        this.recruits = recruits;
        //공고 종료 날짜순 정렬
        recruits.sort(Comparator.comparing(Recruit::getEndDt));
    }

    //순회중 다음 채용 공고를 반환
    @Override
    public Recruit getNext() {
        return recruits.get(this.curPosition++);
    }

    //다음 공고가 존재하는지 확인
    @Override
    public Boolean hasNext() {
        return recruits != null && curPosition < recruits.size();
    }
}

/**
 * 배열을 Iterator로 변환해주는 CustomIterator 구현체
 */
public class RecruitIteratorForArray implements CustomIterator<Recruit> {
    //배열 채용 공고 Aggregate
    private final Recruit[] recruits;
    //Iterator 순회중 현재 인덱스
    private Integer curPosition = 0;

    //생성자
    public RecruitIteratorForArray(Recruit[] recruits) {
        this.recruits = recruits;
    }

    //순회중 다음 채용 공고를 반환
    @Override
    public Recruit getNext() {
        return this.recruits[this.curPosition++];
    }

    //다음 공고가 존재하는지 확인
    @Override
    public Boolean hasNext() {
        return this.recruits != null && this.recruits.length > this.curPosition;
    }
}

/**
 * TOP 상품 영역
 * ** Aggregate들을 가진 클래스 **
 */
public class TopArea {
    //List로 받은 채용 공고 Aggregate
    public List<Recruit> listRecruits;
    //배열로 받은 채용 공고 Aggregate
    public Recruit[] arrRecruits;

    //생성자
    public TopArea(List<Recruit> listRecruits, Recruit[] arrRecruits) {
        this.listRecruits = listRecruits;
        this.arrRecruits = arrRecruits;
    }

    //배열로 구성된 채용 공고 Aggregate의 Iterator 생성
    public CustomIterator<Recruit> createIteratorForArray() {
        return new RecruitIteratorForArray(this.arrRecruits);
    }

    //리스트로 구성된 채용 공고 Aggregate의 Iterator 생성
    public CustomIterator<Recruit> createRecentIteratorForList() {
        return new RecentRecruitIteratorForList(this.listRecruits);
    }
}

public class Main {
    public static void main(String[] args) {
        //리스트 채용 공고 생성
        List<Recruit> recruitList = new ArrayList<>();
        recruitList.add(new Recruit("채용공고1", LocalDateTime.of(2020, 3,7,0,0),LocalDateTime.of(2020, 3,11,0,0)));
        recruitList.add(new Recruit("채용공고2", LocalDateTime.of(2020, 3,7,0,0),LocalDateTime.of(2020, 3,10,0,0)));
        recruitList.add(new Recruit("채용공고3", LocalDateTime.of(2020, 3,7,0,0),LocalDateTime.of(2020, 3,12,0,0)));

        //배열 채용 공고 생성
        Recruit[] recruitsArr = new Recruit[3];
        recruitsArr[0] = (new Recruit("채용공고1", LocalDateTime.of(2020, 3,7,0,0),LocalDateTime.of(2020, 3,11,0,0)));
        recruitsArr[1] = (new Recruit("채용공고2", LocalDateTime.of(2020, 3,7,0,0),LocalDateTime.of(2020, 3,10,0,0)));
        recruitsArr[2] = (new Recruit("채용공고3", LocalDateTime.of(2020, 3,7,0,0),LocalDateTime.of(2020, 3,12,0,0)));

        //TOP 영역 객체 생성
        TopArea topArea = new TopArea(recruitList, recruitsArr);

        System.out.println("============");
        //CustomIterator 인자로 전달
        Main.client(topArea.createIteratorForArray());
        System.out.println("============");

        System.out.println("============");
        //CustomIterator 인자로 전달
        Main.client(topArea.createRecentIteratorForList());
        System.out.println("============");
    }

    /**
     * 클라이언트에서 iterator를 전달받아 리스트 출력
     * 
     * 배열, 리스트로 이루어진 채용 공고 모두 
     * CustomIterator로 받아 똑같은 순회 로직으로 출력 가능
     */
    public static void client(CustomIterator<Recruit> iterator) {
        //hasNext()로 체크하면서 순회
        while (iterator.hasNext()) {
            //다음 객체를 받아옴
            Recruit r = iterator.getNext();
            System.out.printf("%s : %s~%s\n", r.getName(), r.getStartDt().toString(), r.getEndDt().toString());
        }
    }
}

결과
============
채용공고1 : 2020-03-07T00:00~2020-03-11T00:00
채용공고2 : 2020-03-07T00:00~2020-03-10T00:00
채용공고3 : 2020-03-07T00:00~2020-03-12T00:00
============
============
채용공고2 : 2020-03-07T00:00~2020-03-10T00:00
채용공고1 : 2020-03-07T00:00~2020-03-11T00:00
채용공고3 : 2020-03-07T00:00~2020-03-12T00:00
============
```

위의 코드를 보면 `TopArea` 생성자로 받은 `List`, `Array` 채용 공고들을 `CustomIterator` 구현체인 `RecentRecruitIteratorForList`, `RecruitIteratorForArray` 에서 변환시켜줌으로써 서로 다른 형태의 채용 공고들을 `CustomIterator`라는 같은 타입으로 얻을 수 있다.

그 결과 `Main.client(CustomIterator<Recruit> iterator)` 에서 똑같은 순회 로직으로 처리가 가능해진다.

이럼으로써 추후 `List → Set`, `Array → Map` 등 **자료구조가 변경되어도 클라이언트 코드는 변경되지 않고** `CustomIterator`**의 구현체만 변경하면 된다.**

즉 `OCP(Open/Closed Principle)`을 만족한다.





## 3. Collection의 구조

아래와 같이 `Collection`은 Iterable을 상속받기 때문에 **Collection 자료구조를 사용하면 Iterator를 사용**할 수 있다.

하지만 **Collection을 상속받지 않는 Map은 자체적으로 Iterator를 사용할 수 없기 때문에 위의 예시와 같이 Map의 Key나 Value를 keySet() 이나 values()를 통해 Collection을 상속받는 Set이나 Collection을 반환 후 Iterator를 사용**할 수 있다.

[![https://user-images.githubusercontent.com/58713853/73814627-f84d6400-4826-11ea-95dc-0de32bd8be6c.PNG](https://user-images.githubusercontent.com/58713853/73814627-f84d6400-4826-11ea-95dc-0de32bd8be6c.PNG)](https://user-images.githubusercontent.com/58713853/73814627-f84d6400-4826-11ea-95dc-0de32bd8be6c.PNG)

[![https://user-images.githubusercontent.com/79291114/115977688-29918b80-a5b5-11eb-85b1-84c72a9c9be9.png](https://user-images.githubusercontent.com/79291114/115977688-29918b80-a5b5-11eb-85b1-84c72a9c9be9.png)](https://user-images.githubusercontent.com/79291114/115977688-29918b80-a5b5-11eb-85b1-84c72a9c9be9.png)





## 4. 이터레이드 패턴의 장단점

### 4-1. 장점

- 집합 객체가 가지고 있는 객체들에 손쉽게 접근할 수 있다.
  - Iterator가 제공하는 인터페이스만 알면되고, 집합 객체의 구조는 알 필요없다.
- 일관된 인터페이스를 사용해 여러 형태의 집합 구조를 순회할 수 있다.
  - **단일책임원칙 -** hasNext()와 getNext()만 호출해도 됨.
  - **OCP -** 기존 코드의 큰 변경 없음 (상황에 따라 변경되기도 함)



### 4-2. 단점

- 클래스가 늘어나고 복잡도가 증가한다.
  - 이터레이터를 만드는 것이 유용한 상황인지 판단할 필요가 있다.
  - 다양한 방법으로 순회하는 방법이 필요하고 내부의 집합 구조가 객체가 변경될 가능성이 있다면, 내부 구조를 클라이언트 쪽에게 숨기는 방법으로 이터레이터 패턴을 적용하는 것이 좋은 방법이 될 수 있다.





## 5. 마치며

이터레이터 패턴은 집합체를 일관적이게 순회할 수 있다는 것이 매력적인 패턴이다. 사용자의 입장에서도 집합체를 굳이 알지 않아도 되기 때문에 더 편하게 사용할 수 있다. 라이브러리 같은 사용자와의 커플링을 낮출 때 사용하면 유용할 것 같다.