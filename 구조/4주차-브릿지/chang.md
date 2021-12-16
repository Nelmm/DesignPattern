# 브릿지 패턴

추상적인 것과 구체적인 것을 분리하여 연결하는 패턴

하나의 계층 구조일때보다 각기 나누었을때 독립적인 게층 구조로 발전 시킬 수 있다.

GUI ↔ API 연결하는 모습에서 대부분 이런 형태 

기존 방식은 챔피언 인터페이스를 만들어놓고 → 챔피언을 만들고 스킨마다 각각의 메소드를 직접 제정의 했어야함.

브릿지 패턴으로 구현할 경우

챔피언을 정의, 스킨은 인터페이스로 재정의 → 그리고 챔피언, 스킨의 재정의

## 장/단점

장점:

- 추상적인 코드를 구체적인 코드 변경 없이도 독립적으로 확장가능(OCP)
- 추상적인 코드, 구체적 코드 분리 가능.(SRP)
    - 기존코드 재사용, 중복 코드 최소화 → 부가적인 기능

단점;

- 계층 구조가 늘어나서 복잡도 증가

## 어디서 나오나요?

**JDBC** ⇒ Driver: 구체적인것 

JDBC의 나머지 코드들: 추상화쪽.  ⇒ 결국 JDBC의 코드를 많이 변경하지 않아도, 여러가지 DB에 대응 가능. 

SQL을 작성하고 statement를 설정하고, 실행시키는 코드는 크게 변경하지 않음. 

**Slf4j** ⇒ Logger: 추상화. 내부적인 메소드는 그대로 사용하면 되며, 

실제로  내부에 로그 라이브러리만 갈아 끼우면 된다. (log4j2, log4j, logback와 같이 내부를 갈아 끼우면 되는 방식.)

 

![Untitled](/image/slf4j.png)

 

→ 이런식으로 Slf4J를 가장 먼저 두면서 ⇒ 다른 log 라이브러리를 사용하면,  사용성은 유지하되, 다른 새로운 라이브러리로 대체 가능하게 만들 수 있다. 

**Spring MailSender, PlatformTransactionManager** //구체화 된 것. (Implement) 

구체적인 것은 내가 골라서 하면 된다. **JdbcPlatformTransactionManager** → 구체적인 클래스는 이렇게 구현되있다. 

**TransactionTemplate**는 추상화되어있는 것. ⇒ 이 안에 매니저를 넣어서 처리한다. 

 

**Java API** 

List<> interface , Map<> Interface도 브릿지 패턴의 일종 같아보인다.

![Untitled](/image/java-1.png)

![Untitled](/image/java-2.png)

List<> list = new ArrayList<>(); 라던가, LinkedList<>(); 로 갈아끼우고, 내부의 메소드는 동일하게 사용하는 방식이니까 어떤 식으로 보면,

또한 LinkedList의 경우는 Queue Interface에서도 사용 가능하기도 하다.

![Untitled](/image/java-3.png)

![Untitled](/image/java-4.png)

Set 역시 비슷한데, AbstractSet class를 상속하고, Set을 인터페이스로 구현하는 방식으로 되어있다.

![Untitled](/image/java-5.png)

![Untitled](/image/java-6.png)

Map도 비슷하게 HashMap, TreeMap을 통해서 내부 인터페이스의 주요 메소드를 사용할 수 있도록 구현한 것 자체가 이것도 일종의 브릿지 패턴중 하나 아닐까...

<aside>
☝ 참고 hashTable은 다음 형태를 가진다. → 추상화보다는 Dictionary라는 클래스에 상속받고 Map의 형태를 일정 부분가져오는 방식을 택하는 듯..

</aside>

![Untitled](/image/java-7.png)

**Spring Batch** 

Spring Batch에서 사용하는 Reader 기능중 StreamReader는 thread-safe를 위한 interface이고, 결국 멀티 쓰레드를  사용하는 Reader를 이용하기 위해서는, 

![Untitled](/image/java-8.png)

![Untitled](/image/java-9.png)

```java
public interface ItemStream {
	void open(ExecutionContext executionContext) throws ItemStreamException;

	void update(ExecutionContext executionContext) throws ItemStreamException;

	void close() throws ItemStreamException;
}
```

와 같이 복잡하고 헛갈리는 구조를 가진 ItemStream을 구현해야하는데 이것보다 조금더 쉽게 개발할 수 있게 위에 추상화 클래스를 하나 더 개발하여, 그 형태는 무너트리지 않고, Multi-Thread 동기화 개발을 할 수 있도록, `AbstractPagingItemReader` 를 두고 있다.

```java
abstract protected void doReadPage(); //페이징해서 결과 만들어냄.
abstract protected void doJumpToPage(int itemIndex); //혹시 모를 중복을 건너 뛰는 페이징 함수.
```

→ 이거 두개만 구현해도, 멀티쓰레드 방식으로 리더 가능