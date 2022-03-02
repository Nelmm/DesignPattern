>  정리 : [영준](https://github.com/jun108059)

# 프록시

어떤 객체에 대한 대리자를 통해 접근을 제어하여 객체의 생성을 지연초기화 하거나, 캐싱, 객체의 행동 전후에 행동제어를 수행할 수 있는 패턴

<br>

## 1. 코드

```java
public class FruitRepository{
    private static final Map<String, Fruit> fruitList = new HashMap<>(Map.of(
            "orange",new Fruit("orange",10000),
            "strawberry", new Fruit("strawberry", 20000),
            "mango", new Fruit("mango",30000)
    ));


    public Fruit getFruitByName(String name) {
        System.out.println("Repository에서 " + name +" 조회");
        return fruitList.getOrDefault(name,null);
    }
}

@Controller
@RequestMapping("/ex")
@RequiredArgsConstructor
public class FruitController {
    private final FruitRepository fruitRepository;

    @GetMapping("/fruit")
    public ResponseEntity<Fruit> getFruit(@RequestParam String name) {
        return ResponseEntity.ok(fruitRepository.getFruitByName(name));
    }
}
```

아주 단순하게 구현하기 위해 map을 통한 데이터 접근을 구현하며 이는 db를 대체한다고 하자. 

이런 객체를 controller에서 이용하여 데이터를 조회하는 프로그램이다. 

그런데 조회할 때 같은 과일 이름의 조회 결과는 캐싱을 추가하려고 한다. 

그러면 아래와 같이 코드를 작성할 수 있을 것이다.

```java
@Repository
public class FruitRepository {
    private static final Map<String, Fruit> cachedFruitName = new HashMap<>();
    private static final Map<String, Fruit> fruitList = new HashMap<>(Map.of(
            "orange", new Fruit("orange", 10000),
            "strawberry", new Fruit("strawberry", 20000),
            "mango", new Fruit("mango", 30000)
    ));

    @Override
    public Fruit getFruitByName(String name) {
        System.out.println("Repository에서 " + name +" 조회");
        cachedFruitName.computeIfAbsent(name, key -> fruitRepository.getFruitByName(name));
        return cachedFruitName.get(name);
    }
}
```

그런데 만약 Repository가 우리가 구현한 객체가 아닌 경우, 수정이 불가능하다면 지금과 같이 캐싱을 직접 적용하는 것이 불가능하다. 

이를 프록시를 통해 해결해보자.

```java
public class FruitRepository{
    private static final Map<String, Fruit> fruitList = new HashMap<>(Map.of(
            "orange", new Fruit("orange", 10000),
            "strawberry", new Fruit("strawberry", 20000),
            "mango", new Fruit("mango", 30000)
    ));
    
    public Fruit getFruitByName(String name) {
        System.out.println("Repository에서 과일 조회");
        return fruitList.getOrDefault(name,null);
    }
}

@Repository
@RequiredArgsConstructor
public class FruitRepositoryProxy extends FruitRepository {
    private static final Map<String, Fruit> cachedFruitName = new HashMap<>();
    private final FruitRepository fruitRepository;

    @Override
    public Fruit getFruitByName(String name) {
        cachedFruitName.computeIfAbsent(name, key -> fruitRepository.getFruitByName(name));
        return cachedFruitName.get(name);
    }
}

@Controller
@RequestMapping("/ex")
@RequiredArgsConstructor
public class FruitController {
    private final FruitRepository fruitRepository;

    @GetMapping("/fruit")
    public ResponseEntity<Fruit> getFruit(@RequestParam String name) {
        return ResponseEntity.ok(fruitRepository.getFruitByName(name));
    }
}
```

자식클래스는 부모클래스로 업캐스팅이 될 수 있다는 점을 이용하여 위와 같이 프록시를 구현할 수 있다.

만일 Repository를 수정할 수 있어서 Interface를 구현하도록 만들 수 있다면 아래와 같이 구현할 수도 있다.

```java
public interface FruitRepositoryInterface {
    Fruit getFruitByName(String name);
}

public class FruitRepository implements FruitRepositoryInterface{
    private static final Map<String, Fruit> fruitList = new HashMap<>(Map.of(
            "orange", new Fruit("orange", 10000),
            "strawberry", new Fruit("strawberry", 20000),
            "mango", new Fruit("mango", 30000)
    ));


    public Fruit getFruitByName(String name) {
        System.out.println("Repository에서 과일 조회");
        return fruitList.getOrDefault(name, null);
    }
}
@Repository
public class FruitRepositoryProxy implements FruitRepositoryInterface {
    private static final Map<String, Fruit> cachedFruitName = new HashMap<>();
    private final FruitRepositoryInterface fruitRepository;

    public FruitRepositoryProxy() {
        this.fruitRepository = new FruitRepository();
    }

    @Override
    public Fruit getFruitByName(String name) {
        cachedFruitName.computeIfAbsent(name, key -> fruitRepository.getFruitByName(name));
        return cachedFruitName.get(name);
    }
}

@Controller
@RequestMapping("/ex")
@RequiredArgsConstructor
public class FruitController {
    private final FruitRepositoryInterface fruitRepository;

    @GetMapping("/fruit")
    public ResponseEntity<Fruit> getFruit(@RequestParam String name) {
        return ResponseEntity.ok(fruitRepository.getFruitByName(name));
    }
}
```

<br><br>

## 2. 적용할 수 있는 곳

- 지연 초기화 : 시스템 리소스를 많이 차지하는 서비스 객체가 존재할 때 앱이 시작되는 시점에 객체를 생성하는 것이 아니라 실제로 사용하는 시점에 사용하도록 하여 성능향상을 꾀하고자 할 때
- 접근 제어 : 특정 클라이언트만 서비스 객체를 사용할 수 있도록 하려는 경우
- 로깅 요청 : 실제 서비스 객체에 요청을 전달하기 전에 제어할 수 있다는 점을 이용하여 요청을 로깅하려고 하는 경우
- 캐싱 : 서비스 로직을 통한 반환 값이 일정하며 처리시간이 긴 경우에 결과를 캐시하고 캐시의 수명주기를 관리하고자 하는 경우
- 자원 해제 : File, datasource등 자원을 해제하지 않으면 많은 리소스를 잡아먹는 객체의 경우 자원해제를 사용자가 아닌 프록시객체에게 위임하고자 하는 경우

<br><br>

## 3. 다른 패턴들과 비교

- Adapter : 랩핑된 객체와 다른 인터페이스를 제공하지만, 프록시는 동일한 인터페이스를 제공한다.
- Facade : 객체를 버퍼링하고 자체적으로 초기화한다는 점은 비슷하지만 프록시는 해당 서비스 객체와 동일한 인터페이스를 갖는다.
- Decorator : 구조가 매우 비슷하며 특정 작업을 다른 객체에게 위임하는 점은 비슷하나 프록시는 객체 자체적으로 서비스 객체의 수명주기, 행동을 관리하지만 데코레이터는 행동의 제어가 클라이언트에게 있다.
  데코레이터는 `런타임`에 `기능을 추가`하는 것이 목적, Proxy는 `컴파일 타임`에 `행동을 제어` 하는 것이 목적

<br><br><br>

## 4. Spring 프로젝트에 프록시를 통해 서비스를 확장해보자!

```java
// Entity
public class Fruit {
    private String name;
    private int price;

    public Fruit(String name, int price) {
        this.name = name;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public int getPrice() {
        return price;
    }
}

@Controller
@RequestMapping
@RequiredArgsConstructor
public class FruitsController {

    private final FruitService fruitService;
    private final LogTrace trace;

    @GetMapping("/v1/fruit")
    public ResponseEntity<Fruit> getFruit(@RequestParam String name){
        return ResponseEntity.ok(fruitService.getFruit(name));
    }
}

@Service
public class FruitService {
    private final FruitRepsitory fruitRepsitory;

    public FruitService(FruitRepsitory fruitRepsitory) {
        this.fruitRepsitory = fruitRepsitory;
    }

    public Fruit getFruit(String name) {
        return fruitRepsitory.getFruitByName(name);
    }
}

@Repository
public class FruitRepsitory {
    private static final Map<String,Fruit> fruitList = new HashMap<>(Map.of(
            "orange",new Fruit("orange",10000),
            "strawberry", new Fruit("strawberry", 20000),
            "mango", new Fruit("mango",30000)
    ));


    public Fruit getFruitByName(String name) {
        return fruitList.getOrDefault(name,null);
    }
}
```

```java
//새로 추가할 기능의 객체
@Slf4j
public class LogTrace {

    private static final String START_PREFIX = "-->";
    private static final String COMPLETE_PREFIX = "<--";
    private static final String EX_PREFIX = "<X-";

    private final ThreadLocal<TraceId> traceIdHolder = new ThreadLocal<>();

    public TraceStatus begin(String message) {
        syncTraceId();
        TraceId traceId = traceIdHolder.get();
        Long startTimeMs = System.currentTimeMillis();
        log.info("[{}] {}{}", traceId.getId(), addSpace(START_PREFIX, traceId.getLevel()), message);

        return new TraceStatus(traceId, startTimeMs, message);
    }

    public void end(TraceStatus status) {
        complete(status, null);
    }

    public void exception(TraceStatus status, Exception e) {
        complete(status, e);
    }

    private void complete(TraceStatus status, Exception e) {
        Long stopTimeMs = System.currentTimeMillis();
        long resultTimeMs = stopTimeMs - status.getStartTimeMs();
        TraceId traceId = status.getTraceId();
        if (e == null) {
            log.info("[{}] {}{} time={}ms", traceId.getId(), addSpace(COMPLETE_PREFIX, traceId.getLevel()), status.getMessage(), resultTimeMs);
        } else {
            log.info("[{}] {}{} time={}ms ex={}", traceId.getId(), addSpace(EX_PREFIX, traceId.getLevel()), status.getMessage(), resultTimeMs, e.toString());
        }

        releaseTraceId();
    }

    private void syncTraceId() {
        TraceId traceId = traceIdHolder.get();
        if (traceId == null) {
            traceIdHolder.set(new TraceId());
        } else {
            traceIdHolder.set(traceId.createNextId());
        }
    }

    private void releaseTraceId() {
        TraceId traceId = traceIdHolder.get();
        if (traceId.isFirstLevel()) {
            traceIdHolder.remove(); //destroy
        } else {
            traceIdHolder.set(traceId.createPreviousId());
        }
    }

    private static String addSpace(String prefix, int level) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < level; i++) {
            sb.append( (i == level - 1) ? "|" + prefix : "|   ");
        }
        return sb.toString();
    }

    public class TraceId {

        private String id;
        private int level;

        public TraceId() {
            this.id = createId();
            this.level = 0;
        }

        private TraceId(String id, int level) {
            this.id = id;
            this.level = level;
        }

        private String createId() {
            return UUID.randomUUID().toString().substring(0, 8);
        }

        public TraceId createNextId() {
            return new TraceId(id, level + 1);
        }

        public TraceId createPreviousId() {
            return new TraceId(id, level - 1);
        }

        public boolean isFirstLevel() {
            return level == 0;
        }

        public String getId() {
            return id;
        }

        public int getLevel() {
            return level;
        }
    }

    public class TraceStatus {

        private TraceId traceId;
        private Long startTimeMs;
        private String message;

        public TraceStatus(TraceId traceId, Long startTimeMs, String message) {
            this.traceId = traceId;
            this.startTimeMs = startTimeMs;
            this.message = message;
        }

        public Long getStartTimeMs() {
            return startTimeMs;
        }

        public String getMessage() {
            return message;
        }

        public TraceId getTraceId() {
            return traceId;
        }
    }
}때
```

위와 같은 controller-service-repository 계층의 앱이 존재하고 각 계층의 log를 깊이별로 tracing하는 기능을 LogTrace객체를 통하여 추가하고자 할 때 일반적인 방법으로는 아래와 같이 추가할 수 있다.

```java
@Controller
@RequestMapping
@RequiredArgsConstructor
public class FruitsController {

    private final FruitService fruitService;
    private final LogTrace trace;

    @GetMapping("/v2/fruit")
    public ResponseEntity<Fruit> getFruitLogging(@RequestParam String name){
        LogTrace.TraceStatus status = null;
        try {
            status = trace.begin("FruitsController.getFruitLogging()");
            Fruit fruit = fruitService.getFruit(name);
            trace.end(status);
            return ResponseEntity.ok(fruit);
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }
    }
}

@Service
@RequiredArgsConstructor
public class FruitService {
    private final FruitRepsitory fruitRepsitory;
    private final LogTrace trace;

    public Fruit getFruit(String name) {
        LogTrace.TraceStatus status = null;
        try {
            status = trace.begin("FruitService.getFruit()");
            Fruit fruit = fruitRepository.getFruitByName(name);
            trace.end(status);
            return fruit;
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }
    }
}

@Repository
@RequiredArgsConstructor
public class FruitRepository {
    private final LogTrace trace;

    private static final Map<String,Fruit> fruitList = new HashMap<>(Map.of(
            "orange",new Fruit("orange",10000),
            "strawberry", new Fruit("strawberry", 20000),
            "mango", new Fruit("mango",30000)
    ));


    public Fruit getFruitByName(String name) {
        LogTrace.TraceStatus status = null;
        try {
            status = trace.begin("FruitRepository.getFruitByName()");
            Fruit fruit = fruitList.getOrDefault(name,null);
            trace.end(status);
            return fruit;
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }
    }
}

//출력
2021-12-30 10:12:08.204  INFO 16704 --- [nio-8080-exec-1] com.example.springex.proxy.LogTrace      : [8b8df55b] FruitsController.getFruitLogging()
2021-12-30 10:12:08.205  INFO 16704 --- [nio-8080-exec-1] com.example.springex.proxy.LogTrace      : [8b8df55b] |-->FruitService.getFruit()
2021-12-30 10:12:08.205  INFO 16704 --- [nio-8080-exec-1] com.example.springex.proxy.LogTrace      : [8b8df55b] |   |-->FruitRepository.getFruit()
2021-12-30 10:12:08.205  INFO 16704 --- [nio-8080-exec-1] com.example.springex.proxy.LogTrace      : [8b8df55b] |   |<--FruitRepository.getFruit() time=0ms
2021-12-30 10:12:08.205  INFO 16704 --- [nio-8080-exec-1] com.example.springex.proxy.LogTrace      : [8b8df55b] |<--FruitService.getFruit() time=0ms
2021-12-30 10:12:08.208  INFO 16704 --- [nio-8080-exec-1] com.example.springex.proxy.LogTrace      : [8b8df55b] FruitsController.getFruitLogging() time=4ms
```

하지만 이는 로깅을 적용하고 싶은 빈마다 LogTrace를 주입시켜주고 비즈니스로직을 try로 감싸 기능을 추가해주어야 한다. 굉장히 보일러플레이트도 늘어날뿐 아니라 단일책임원칙에도 위배되고 있다.

Proxy를 시작으로 AOP까지 확장해가며 서비스를 확장시켜보자.

<br>

### 1. Concreate Proxy

자식 클래스는 부모클래스로 업캐스팅이 가능하다는 점을 이용하여 상속을 통해서 프록시를 구현한 방법.

```java
//컨트롤러
@RequestMapping
@RequiredArgsConstructor
public class FruitController {
    private final FruitService fruitService;

    @GetMapping("/v2/fruit")
    public ResponseEntity<Fruit> getFruitLogging(@RequestParam String name){
        return ResponseEntity.ok(fruitService.getFruit(name));
    }
}

//상속을 통해 컨트롤러 프록시객체 생성
public class FruitControllerProxy extends FruitController {
    private final FruitController target;
    private final LogTrace trace;

    public FruitControllerProxy(FruitController target, LogTrace trace) {
        super(null);
        this.target = target;
        this.trace = trace;
    }

    @Override
    public ResponseEntity<Fruit> getFruitLogging(@RequestParam String name){
        LogTrace.TraceStatus status = null;
        try {
            status = trace.begin("FruitsController.getFruitLogging()");
            ResponseEntity<Fruit> result = target.getFruitLogging(name);
            trace.end(status);
            return result;
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }
    }
}

//service
@RequiredArgsConstructor
public class FruitService {
    private final FruitRepository fruitRepository;

    public Fruit getFruit(String name) {
        return fruitRepository.getFruitByName(name);
    }
}

//Service proxy 객체
public class FruitServiceProxy extends FruitService{
    private final FruitService target;
    private final LogTrace trace;

    public FruitServiceProxy( FruitService target, LogTrace trace) {
        super(null);
        this.target = target;
        this.trace = trace;
    }

    @Override
    public Fruit getFruit(String name) {
        LogTrace.TraceStatus status = null;
        try {
            status = trace.begin("FruitService.getFruit()");
            Fruit fruit = target.getFruit(name);
            trace.end(status);
            return fruit;
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }
    }
}

//Repository
@RequiredArgsConstructor
public class FruitRepository {
    private static final Map<String,Fruit> fruitList = new HashMap<>(Map.of(
            "orange",new Fruit("orange",10000),
            "strawberry", new Fruit("strawberry", 20000),
            "mango", new Fruit("mango",30000)
    ));


    public Fruit getFruitByName(String name) {
        return fruitList.getOrDefault(name,null);
    }
}

//Repository Proxy
public class FruitRepositoryProxy extends FruitRepository {
    private final FruitRepository target;
    private final LogTrace trace;

    public FruitRepositoryProxy(FruitRepository target,LogTrace trace) {
        this.target = target;
        this.trace = trace;
    }

    @Override
    public Fruit getFruitByName(String name) {
        LogTrace.TraceStatus status = null;
        try {
            status = trace.begin("FruitService.getFruit()");
            Fruit fruit = target.getFruitByName(name);
            trace.end(status);
            return fruit;
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }
    }
}

//Bean등록을 위한 Config 클래스
@Configuration
public class AppConfig {
    @Bean
    public FruitController fruitController(LogTrace logTrace) {
        //Controller객체 생성
        FruitController controller = new FruitController(fruitService(logTrace));

        //위에서 만든 Controller를 주입하여 Proxy 컨트롤러를 빈으로 등록
        return new FruitControllerProxy(controller,logTrace);
    }

    @Bean
    public FruitService fruitService(LogTrace logTrace) {
        //Service객체 생성
        FruitService service = new FruitService(fruitRepository(logTrace));

        //Service Proxy객체 빈등록
        return new FruitServiceProxy(service,logTrace);
    }

    @Bean
    public FruitRepository fruitRepository(LogTrace logTrace) {
        FruitRepository repository = new FruitRepository();
        return new FruitRepositoryProxy(repository,logTrace);
    }
}
```

이는 LogTrace기능을 추가하려는 Bean마다 직접 Proxy객체를 생성해주어야 한다는 번거로움이 존재한다.

<br>

### 2. Interface Proxy

프록시 패턴과 같이 인터페이스를 통하여 프록시 객체를 생성하는 방법으로 보통 컨트롤러는 인터페이스를 구현하지 않지만 예제를 위하여 컨트롤러도 인터페이스를 구현.

```java
//Controller 인터페이스
@RequestMapping     //해당 어노테이션이 존재해야 컨트롤러로 인지
@ResponseBody
public interface FruitController {
    @GetMapping("/v1/fruit")
    ResponseEntity<Fruit> getFruit();

    @GetMapping("/v2/fruit")
    ResponseEntity<Fruit> getFruitLogging(@RequestParam String name);
}

//구현체
@RequiredArgsConstructor
public class FruitControllerImpl {
    private final FruitService fruitService;

    @Override
    public ResponseEntity<Fruit> getFruit(@RequestParam String name){
        return ResponseEntity.ok(fruitService.getFruit(name));
    }

    @Override
    public ResponseEntity<Fruit> getFruitLogging(@RequestParam String name){
        return ResponseEntity.ok(fruitService.getFruit(name));
    }
}

//Serivce 인터페이스
public interface FruitService {
    Fruit getFruit(String name);
}

@RequiredArgsConstructor
public class FruitServiceImpl {
    private final FruitRepository fruitRepository;

    public Fruit getFruit(String name) {
        return fruitRepository.getFruitByName(name);
    }
}

//Repository 인터페이스
public interface FruitRepository {
    Fruit getFruitByName(String name);
}

@RequiredArgsConstructor
public class FruitRepositoryImpl {
    private static final Map<String,Fruit> fruitList = new HashMap<>(Map.of(
            "orange",new Fruit("orange",10000),
            "strawberry", new Fruit("strawberry", 20000),
            "mango", new Fruit("mango",30000)
    ));


    public Fruit getFruitByName(String name) {
        return fruitList.getOrDefault(name,null);
    }
}


//위에서 정의한 인터페이스를 구현하여 Proxy 객체 생성
public class FruitControllerProxy implements FruitController {
    private final FruitController target;
    private final LogTrace trace;

    public FruitControllerProxy(FruitController target, LogTrace trace) {
        this.target = target;
        this.trace = trace;
    }

    @Override
    public ResponseEntity<Fruit> getFruit(@RequestParam String name){
        return target.getFruit(name);
    }

    @Override
    public ResponseEntity<Fruit> getFruitLogging(@RequestParam String name){
        LogTrace.TraceStatus status = null;
        try {
            status = trace.begin("FruitsController.getFruitLogging()");
            ResponseEntity<Fruit> result = target.getFruitLogging(name);
            trace.end(status);
            return result;
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }
    }
}
public class FruitServiceProxy implements FruitService{
    private final FruitService target;
    private final LogTrace trace;

    public FruitServiceProxy( FruitService target, LogTrace trace) {
        this.target = target;
        this.trace = trace;
    }

    @Override
    public Fruit getFruit(String name) {
        LogTrace.TraceStatus status = null;
        try {
            status = trace.begin("FruitService.getFruit()");
            Fruit fruit = target.getFruit(name);
            trace.end(status);
            return fruit;
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }
    }
}

public class FruitRepositoryProxy implements FruitRepository {
    private final FruitRepository target;
    private final LogTrace trace;

    public FruitRepositoryProxy(FruitRepository target, LogTrace trace) {
        this.target = target;
        this.trace = trace;
    }

    @Override
    public Fruit getFruitByName(String name) {
        LogTrace.TraceStatus status = null;
        try {
            status = trace.begin("FruitService.getFruit()");
            Fruit fruit = target.getFruitByName(name);
            trace.end(status);
            return fruit;
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }
    }
}

//Bean으로 구현체가 아닌 Proxy객체 등록을 위한 Config
@Configuration
public class InterfaceProxyConfig {
    @Bean
    public FruitController fruitController(LogTrace logTrace) {
        FruitController controllerImpl = new FruitControllerImpl(fruitService(logTrace));
        return new FruitControllerProxy(controllerImpl, logTrace);
    }

    @Bean
    public FruitService fruitService(LogTrace logTrace) {
        FruitService serviceImpl = new FruitServiceImpl(fruitRepository(logTrace));
        return new FruitServiceProxy(serviceImpl, logTrace);
    }

    @Bean
    public FruitRepository fruitRepository(LogTrace logTrace) {
        FruitRepository repositoryImpl = new FruitRepositoryImpl();
        return new FruitRepositoryProxy(repositoryImpl, logTrace);
    }
}
```

이도 1번의 방법과 마찬가지로 Proxy객체를 하나하나 직접 생성해주어야 하며, 인터페이스가 존재하지 않는 경우라면 억지로 인터페이스를 생성해야하는 문제점이 존재한다.

<br>

### 3. DynamicProxy

Java에서 제공하는 기능으로 동적으로 프록시를 생성할 수 있는 방법이며, DynamicProxy는 인터페이스가 존재해야지만 프록시를 생성할 수 있다. `Proxy`의 `newInstance()`메서드를 통해서 프록시 객체를 정의할 필요없이 동적으로 프록시객체를 생성할 수 있다.

newInstance()의 두번째 인자로 인터페이스들을, 세번째 인자로 `InvocationHandler`를 구현한 Handler를 주입해줌으로써 생성이 가능하다.

```java
//Dynamic Proxy를 위한 InvocationHandler 구현체
public class LogTraceBasicHandler implements InvocationHandler {

    private final Object target;
    private final LogTrace logTrace;

    public LogTraceBasicHandler(Object target, LogTrace logTrace) {
        this.target = target;
        this.logTrace = logTrace;
    }

    //해당 메서드를 구현함으로써 프록시 기능을 수행
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        LogTrace.TraceStatus status = null;
        try {
            //이전에는 각 계층마다 Proxy를 정의해주었기 때문에 log message에 필요한 클래스이름을 하드코딩했지만 여기서부터는 동적으로 Proxy를 만들기 때문에 동적으로 클래스이름을 얻어와 log내용 작성
            String message = method.getDeclaringClass().getSimpleName() + "." +
                    method.getName() + "()";
            status = logTrace.begin(message);

            //실제 로직 호출
            Object result = method.invoke(target, args);

            logTrace.end(status);
            return result;
        } catch (Exception e) {
            logTrace.exception(status, e);
            throw e;
        }
    }
}

//DynaicProxy를 통해 생성된 프록시객체를 Bean으로 등록
@Configuration
public class DynamicProxyBasicConfig {

    @Bean
    public FruitController fruitController(LogTrace logTrace) {
        FruitController fruitController = new FruitControllerImpl(fruitService(logTrace));
        return (FruitController) Proxy.newProxyInstance(FruitController.class.getClassLoader(),
                new Class[]{FruitController.class},
                new LogTraceBasicHandler(fruitController, logTrace));
    }

    @Bean
    public FruitService fruitService(LogTrace logTrace) {
        FruitService fruitService = new FruitServiceImpl((fruitRepository(logTrace)));
        return (FruitService) Proxy.newProxyInstance(FruitService.class.getClassLoader(),
                new Class[]{FruitService.class},
                new LogTraceBasicHandler(fruitService, logTrace));
    }

    @Bean
    public FruitRepository fruitRepository(LogTrace logTrace) {
        FruitRepository orderRepository = new FruitRepositoryImpl();

        return (FruitRepository) Proxy.newProxyInstance(FruitRepository.class.getClassLoader(),
                new Class[]{FruitRepository.class},
                new LogTraceBasicHandler(orderRepository, logTrace));
    }
}

```

Proxy.newProxyInstance()를 통해 Proxy 객체를 런타임에 생성할 수 있는데, 이때 인자로 InvocateionHandler를 넘겨주어 ClassLoader의 객체가 load될때 해당 핸들러로 행동을 제어할 수 있다.

DynamicProxy를 구현하기 위해 InvocationHandler를 구현하여 사용하고 있는데 이때 invoke()메서드 내부에서 실제 로직를 시작하기 위한 매서드로 `method.invoke()`를 수행한다.

이 invoke()메서드는 내부적으로 Reflection을 사용하여 동작하고 Spring framework에서 AOP를 위해 default로 사용되는 방법이다.

#### 패턴 매칭

위의 방법은 해당 클래스내부의 모든 메서드라면 logtrace기능이 수행되게 되는데 같은 클래스안에서 특정메서드에만 수행하게 하고 싶을 수 있을 것이다. 이때 메서드 이름을 `pattern`을 통해 제한하고자 할 때 아래와 같이 구현할 수도 있다.

```java
public class LogTraceFilterHandler implements InvocationHandler {

    private final Object target;
    private final LogTrace logTrace;
    private final String[] patterns;

    public LogTraceFilterHandler(Object target, LogTrace logTrace, String[] patterns) {
        this.target = target;
        this.logTrace = logTrace;
        this.patterns = patterns;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        //메서드 이름 필터
        //메서드 이름이 패턴에 일치하지 않는다면 바로 메서드 수행
        String methodName = method.getName();
        if (!PatternMatchUtils.simpleMatch(patterns, methodName)) {
            return method.invoke(target, args);
        }

        //패턴이 일지한다면 logtrace기능 추가하여 메서드 수행
        LogTrace.TraceStatus status = null;
        try {
            String message = method.getDeclaringClass().getSimpleName() + "." +
                    method.getName() + "()";
            status = logTrace.begin(message);

            //로직 호출
            Object result = method.invoke(target, args);
            logTrace.end(status);
            return result;
        } catch (Exception e) {
            logTrace.exception(status, e);
            throw e;
        }
    }
}

@Configuration
public class DynamicProxyFilterConfig {

    private static final String[] PATTERNS = {"getFruit*", "*Logging"};

    @Bean
    public FruitController fruitController(LogTrace logTrace) {
        FruitController fruitController = new FruitController(fruitService(logTrace));
        return (FruitController) Proxy.newProxyInstance(FruitController.class.getClassLoader(),
                new Class[]{FruitController.class},
                new LogTraceFilterHandler(fruitController, logTrace, PATTERNS));
    }

    @Bean
    public FruitService fruitService(LogTrace logTrace) {
        FruitService fruitService = new FruitService(fruitRepository(logTrace));
        return (FruitService) Proxy.newProxyInstance(FruitService.class.getClassLoader(),
                new Class[]{FruitService.class},
                new LogTraceFilterHandler(fruitService, logTrace, PATTERNS));
    }

    @Bean
    public FruitRepository fruitRepository(LogTrace logTrace) {
        FruitRepository orderRepository = new FruitRepository();

        return (FruitRepository) Proxy.newProxyInstance(FruitRepository.class.getClassLoader(),
                new Class[]{FruitRepository.class},
                new LogTraceFilterHandler(orderRepository, logTrace, PATTERNS));
    }
}
```

패턴매칭방법으로 spring AOP와 비슷한 기능을 흉내낼 수 있고, 1/2의 방법과 달리 이제는 Bean들마다 프록시 객체를 따로 정의해주지 않아도 되게 성능이 향상되었다!!

하지만, Bean의 인터페이스가 존재하지 않다면 이 방법은 사용이 불가능하다는 단점이 존재한다. 이를 개선해보자.

<br>

### 4. ProxyFactory

여기서부터는 AOP를 이용한 방법이며 `CGLIB`를 이용한 방법이다.

CGLIB를 통해 프록시를 생성하고자 할 때 `MethodInterceptor`를 구현한 클래스를 가지고 프록시를 생성할 수 있다.

CGLIB는 DynamicProxy와 다르게 인터페이스가 아닌 상속을 통해 프록시를 생성하는 방법이며 spring boot에서 AOP 기능을 제공하는데 default로 사용되는 방식이기 때문에 spring boot를 이용하고 있다면 별도의 라이브러리를 추가할 필요가 없다.

```java
@RequiredArgsConstructor
public class LogTraceAdvice implements MethodInterceptor {
    private final LogTrace logTrace;


    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        LogTrace.TraceStatus status = null;
        try {
            Method method = invocation.getMethod();
            String message = method.getDeclaringClass().getSimpleName() + "." +
                    method.getName() + "()";
            status = logTrace.begin(message);

            //로직 호출
            Object result = invocation.proceed();

            logTrace.end(status);
            return result;
        } catch (Exception e) {
            logTrace.exception(status, e);
            throw e;
        }
    }
}

@Slf4j
@Configuration
public class ProxyFactoryConfig {
    @Bean
    public FruitController orderController(LogTrace logTrace) {
        //프록시를 적용할 실제 객체 생성
        FruitController orderController = new FruitControllerImpl(orderService(logTrace));

        //ProxyFactory 생성
        ProxyFactory factory = new ProxyFactory(orderController);

        //Advisor를 ProxyFactory에 추가
        factory.addAdvisor(getAdvisor(logTrace));
        FruitController proxy = (FruitController) factory.getProxy();
        log.info("ProxyFactory proxy={}, target={}", proxy.getClass(), orderController.getClass());

        //생성한 Proxy를 빈으로 등록
        return proxy;
    }

    @Bean
    public FruitService orderService(LogTrace logTrace) {
        FruitService orderService = new FruitServiceImpl(orderRepository(logTrace));
        ProxyFactory factory = new ProxyFactory(orderService);
        factory.addAdvisor(getAdvisor(logTrace));
        FruitService proxy = (FruitService) factory.getProxy();
        log.info("ProxyFactory proxy={}, target={}", proxy.getClass(), orderService.getClass());
        return proxy;
    }

    @Bean
    public FruitRepository orderRepository(LogTrace logTrace) {
        FruitRepository orderRepository = new FruitRepositoryImpl();
        ProxyFactory factory = new ProxyFactory(orderRepository);
        factory.addAdvisor(getAdvisor(logTrace));
        FruitRepository proxy = (FruitRepository) factory.getProxy();
        log.info("ProxyFactory proxy={}, target={}", proxy.getClass(), orderRepository.getClass());
        return proxy;
    }

    private Advisor getAdvisor(LogTrace logTrace) {
        //Logtrace기능이 적용될 pointcut 정의
        NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
        pointcut.setMappedNames("getFruit*");

        //실제 LogTrace기능이 적용되는 로직이 정의된 advice
        LogTraceAdvice advice = new LogTraceAdvice(logTrace);
        return new DefaultPointcutAdvisor(pointcut, advice);
    }
}
```

3번의 방법에서 개선이되어 이제는 인터페이스가 존재하지 않아도 동적으로 프록시객체를 만들 수 있게 되었다. 하지만, 여전히 불편하다!

프록시 객체를 빈으로 등록하는 로직을 직접 빈마다 작성을 해주어야 한다. 이를 개선해보자.

<br>

### 5. postProcessor

Bean을 생성하고 컨테이너에 등록하기 전에 후처리기(PostProcessor)를 통해 특정 작업을 수행할 수 있다.(기능 추가, 제어 등)

이를 위해 `BeanPostProcessor`를 구현하여 후처리기를 정의할 수 있다. 이 후처리기에 4번방법인 `ProxyFactory`를 추가하여 특정패키지,메서드라면 동적으로 Proxy객체를 만들어 해당 Proxy를 빈으로 등록 할 수 있다.

```java
@Slf4j
@RequiredArgsConstructor
public class PackageLogTracePostProcessor implements BeanPostProcessor {

    private final String basePackage;
    private final Advisor advisor;

    public PackageLogTracePostProcessor(String basePackage, Advisor advisor) {
        this.basePackage = basePackage;
        this.advisor = advisor;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        log.info("param beanName={} bean={}", beanName, bean.getClass());

        //프록시 적용 대상 여부 체크
        //프록시 적용 대상이 아니면 원본을 그대로 진행
        String packageName = bean.getClass().getPackageName();
        if (!packageName.startsWith(basePackage)) {
            return bean;
        }

        //프록시 대상이면 프록시를 만들어서 반환
        ProxyFactory proxyFactory = new ProxyFactory(bean);
        proxyFactory.addAdvisor(advisor);

        Object proxy = proxyFactory.getProxy();
        log.info("create proxy: target={} proxy={}", bean.getClass(), proxy.getClass());
        return proxy;
    }
}

//빈을 생성하고 등록할 때 이용할 후처리기(postProcessor)등록
@Slf4j
@Configuration
public class BeanPostProcessorConfig {
    @Bean
    public PackageLogTracePostProcessor logTracePostProcessor(LogTrace logTrace) {
        return new PackageLogTracePostProcessor("com.example.springex", getAdvisor(logTrace));
    }

    private Advisor getAdvisor(LogTrace logTrace) {
        //pointcut
        NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
        pointcut.setMappedNames("getFruit*");

        //advice
        LogTraceAdvice advice = new LogTraceAdvice(logTrace);
        return new DefaultPointcutAdvisor(pointcut, advice);
    }
}
```

이 방법을 통해 빈이 생성되고 컨테이너에 등록되기 전에 실행되는 후처리기를 통해 프록시객체를 생성하고 빈을 등록하는 과정이 줄어들었다.

하지만, 기능을 추가하고자하는 메서드를 필터링하는 구문이 setMappedNames()로 특정 클래스/클래스내의 특정 메서드는 제외와 같이 디테일한 경우는 필터링이 힘들다. 이를 해결하여 확장해보자.

### 6. AutoProxy

여기서부터는 pointcut을 표현식을 통해서 적용하는 패턴으로 `implementation 'org.springframework.boot:spring-boot-starter-aop'`를 추가해주어야 한다.

AspectJExpressionPointcut을 이용해 표현식으로 pointcut을 정의할 수 있는데 이름에 AspectJ가 들어간다고 해서 AspectJ가 사용되는 것은 아니다.

여기서부터는 위의 방법들처럼 프록시객체를 만들어 빈으로 등록을 하는 것이 아니라 실제 객체를 빈으로 등록하고 프록시 기능이 수행되는 메서드라면 그때 런타임에 프록시객체를 만들어 수행하는 방법.

```java
@Configuration
public class AutoProxyConfig {

    //기존의 수행방식
    @Bean
    public Advisor advisor1(LogTrace logTrace) {
        //pointcut
        NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
        pointcut.setMappedNames("getFruit*");
        //advice
        LogTraceAdvice advice = new LogTraceAdvice(logTrace);
        return new DefaultPointcutAdvisor(pointcut, advice);
    }

    //포인트컷 표현식 이용
    @Bean
    public Advisor advisor2(LogTrace logTrace) {
        //pointcut
        //spring-aop에서 제공하는 AspectJExpressionPointcut를 이용하면 특정문법으로 point cut을 정의할 수 있다.
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* com.example.springex.proxy._6_autoProxy..*(..))");
        //advice
        LogTraceAdvice advice = new LogTraceAdvice(logTrace);
        return new DefaultPointcutAdvisor(pointcut, advice);
    }

    @Bean
    public Advisor advisor3(LogTrace logTrace) {
        //pointcut
        //표현식에도 &&, || , ! 와 같은 연산자 사용가능
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* com.example.springex.proxy._6_autoProxy..*(..)) && !execution(* com.example.springex.proxy._6_autoProxy.FruitController.getFruit(..))");

        //advice
        LogTraceAdvice advice = new LogTraceAdvice(logTrace);
        return new DefaultPointcutAdvisor(pointcut, advice);
    }
}
```

표현식을 통해 디테일하게 필터링을 수행할 수 있게 되었으며, 생성한 advisor를 직접 빈 후처리기에 등록하지 않아도 런타임에 스프링이 등록한 advisor를 이용해 프록시객체를 생성해준다.

<br>

### 7. AOP 어노테이션 (표현식으로 빈 등록)

지금까지는 `MethodInterceptor`의 구현체를 통한 Advise를 가지고 프록시를 수행했었다. 이는 내부적으로 MethodInvocation 인터페이스의 구현체(CglibMethodInvocation/Reflection...)를 가지고 메서드를 수행했다.

로직이 담겨있는 빈이 별도로 존재하고 이가 수행되는 지점을 다른 클래스인 Configuration에서 정의했었는데, 이를 합쳐 실제 로직이 정의된 부분에서 수행되는지점을 정의하도록 수정해보자.

```java
@Slf4j
@Aspect
@Component
public class LogTraceAspect {
    private final LogTrace logTrace;

    public LogTraceAspect(LogTrace logTrace) {
        this.logTrace = logTrace;
    }

    //정의한 형식에 맞는 메서드라면 execute를 수행하여 제어
    @Around("execution(* com.example.springex.proxy._7_aop.app..*(..)) && !execution(* com.example.springex.proxy._7_aop.app.FruitController.getFruit(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        LogTrace.TraceStatus status = null;
        try {
            String message = joinPoint.getSignature().toShortString();
            status = logTrace.begin(message);

            //로직 호출
            Object result = joinPoint.proceed();

            logTrace.end(status);
            return result;
        } catch (Exception e) {
            logTrace.exception(status, e);
            throw e;
        }
    }
}
```

`@Aspect` 어노테이션을 클래스에 붙여줌으로써, 해당 클래스는 advise와 pointcut이 정의된 Advisor라고 명시해줄 수 있다.

이렇게 코드를 수정함으로써, 흐름을 제어하는 곳과 이것을 사용하는 곳이 분리되지 않게 되어 역할을 더 명확히 명시해 줄 수 있게 되었다.

<br>

### 8. AOP 어노테이션 (custom annotation을 통한 등록)

7의 경우도 보면 아직 불편하다! 패키지가 많아질 수록, 제외하는 메서드가 많아질 수록 표현식이 길고 복잡해지게 되어 관리가 힘들어진다.

`@Transactional` 과 같이 어노테이션을 통하여 AOP기능을 적욯해보자.

```java
//Custom 어노테이션 생성
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD,ElementType.TYPE})
public @interface Log {
}

@Controller
@RequiredArgsConstructor
public class FruitController {
    private final FruitService fruitService;

    @GetMapping("/v1/fruit")
    public ResponseEntity<Fruit> getFruit(@RequestParam String name){
        return ResponseEntity.ok(fruitService.getFruit(name));
    }

    //LogTrace기능을 이 메서드에만 적용
    @Log
    @GetMapping("/v2/fruit")
    public ResponseEntity<Fruit> getFruitLogging(@RequestParam String name){
        return ResponseEntity.ok(fruitService.getFruit(name));
    }
}

//Service는 모든 메서드에 LogTrace기능 적용
@Log
@Service
@RequiredArgsConstructor
public class FruitService {
    private final FruitRepository fruitRepository;

    public Fruit getFruit(String name) {
        return fruitRepository.getFruitByName(name);
    }
}

@Log
@Repository
@RequiredArgsConstructor
public class FruitRepository {
    private static final Map<String, Fruit> fruitList = new HashMap<>(Map.of(
            "orange",new Fruit("orange",10000),
            "strawberry", new Fruit("strawberry", 20000),
            "mango", new Fruit("mango",30000)
    ));

    public Fruit getFruitByName(String name) {
        return fruitList.getOrDefault(name,null);
    }
}
@Slf4j
@Aspect
public class LogTraceAspect {
    private final LogTrace logTrace;

    public LogTraceAspect(LogTrace logTrace) {
        this.logTrace = logTrace;
    }

    //@annotaion()는 클래스단위에 붙은 경우만 check -> method에 붙었다면 필터링 불가
    //@Within()을 통해 메서드단위도 필터링
    @Around("@annotation(Log) || @within(Log)")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        LogTrace.TraceStatus status = null;
        try {
            String message = joinPoint.getSignature().toShortString();
            status = logTrace.begin(message);

            //로직 호출
            Object result = joinPoint.proceed();

            logTrace.end(status);
            return result;
        } catch (Exception e) {
            logTrace.exception(status, e);
            throw e;
        }
    }
}
@Configuration
public class AopConfig {
    @Bean
    public LogTraceAspect logTraceAspect(LogTrace logTrace) {
        return new LogTraceAspect(logTrace);
    }
}
```

단순한 Proxy를 시작으로 springAOP까지 총 8번의 과정을 통해 기능을 확장시켜보았다.

<br>

### 9. 주의사항

```java
@Controller
@RequiredArgsConstructor
public class FruitController {
    private final FruitService fruitService;

    @GetMapping("/v1/fruit")
    public ResponseEntity<Fruit> getFruit(@RequestParam String name){
        return ResponseEntity.ok(fruitService.getFruit(name));
    }

    @GetMapping("/v2/fruit")
    public ResponseEntity<Fruit> getFruitLogging(@RequestParam String name){
        return ResponseEntity.ok(fruitService.getFruit(name));
    }

    @GetMapping("/v3/fruit")
    private ResponseEntity<Fruit> getF(@RequestParam String name) {
        return ResponseEntity.ok(fruitService.getFruit(name));
    }
}
GET http://localhost:8080/v3/fruit?name=orange 호출시 200 응답
```

만일, AOP가 적용되지 않는 클래스라면 따로 spring 이 프록시를 만들어 사용하지 않기 때문에 내부 메서드가 private이어도 상관없이 메서드 호출이 정상적으로 수행된다.

```java
@Controller
@RequiredArgsConstructor
public class FruitController {
    private final FruitService fruitService;
    Method[] a = this.getClass().getDeclaredMethods();

    @GetMapping("/v1/fruit")
    public ResponseEntity<Fruit> getFruit(@RequestParam String name){
        return ResponseEntity.ok(fruitService.getFruit(name));
    }

    @Log
    @GetMapping("/v2/fruit")
    public ResponseEntity<Fruit> getFruitLogging(@RequestParam String name){
        return ResponseEntity.ok(fruitService.getFruit(name));
    }

    @GetMapping("/v3/fruit")
    private ResponseEntity<Fruit> getF(@RequestParam String name) {
        return ResponseEntity.ok(fruitService.getFruit(name));
    }
}

//
GET http://localhost:8080/v3/fruit?name=orange 호출시 500에러
FruitService 가 null
```

하지만 AOP가 적용되는 클래스(메서드 한개라도 pointcut에 해당한다면 적용)는 spring boot는 런타임에 cglib를 이용하여 프록시 객체를 만들어 호출하게 된다.

이때 cglib의 특성이 Enhancer를 통해 구현객체를 상속하여 프록시 객체를 만들어낸다는 점이다. AOP가 지정된 메서드가 public이라면 Enhancer를 통해 객체를 만들고 실제 메서드 호출타이밍에 AdvisedInterceptor가 가로채서 MethodInvokation을 통해 `MethodProxy`가 호출된다. 최종적으로 MethodProxy의 invoke메서드에서 FastClass의 invoke메서드로 실제 로직이 실행된다. (이때 처음에 생성된 Enhancer 객체는 연관된 의존성이 하나도 주입이 되지 않은 형태이다. Enhancer특성상 동적으로 의존성을 주입해주기 힘들어 보이기 때문에 그런것이 아닐까 싶다..)

FastClass는 cglib에 있는 클래스로 MethodProxy가 생성될때 init메서드를 통해 FastClass가 생성된다. (FastClass도 일종의 프록시로 우리가 실제로 등록한 빈을 주입받아 생성되기 때문에 실제 빈을 통해 수행된다.(정상적으로 의존성주입이 된 빈)

하지만 private이라면 Enhancer를 통해 프록시 객체를 만들고 이를 호출하는 타이밍에 메서드 정보를 알지 못하기 때문에 Interceptor가 가로채지 못하고 그대로 Enhancer객체로 프록시가 수행된다. (하지만 위에서 말했듯이 Enhancer는 의존성주입이 되지 않은 객체로 service가 null이 되어 null을 참조하려고해 null pointException이 발생한다.)

<br>

상속을 통해서 프록시 객체를 생성하기 때문에 private은 상속하지 못한다 했는데 어떻게 private 메서드가 호출이 된 것일까?

우리는 인터페이스나 클래스의 계층관계를 정의할 때 아래와같이 사용할 수 있다는 것을 알고 있다.

```java
ExInterface ex = new ExInterfaceImpl();
ExSuperClass ex2 = new ExChildClass();
```

결국은 Super클래스로 업캐스팅을 수행하여 해당 obj로 invoke를 수행하기 때문에 private메서드가 수행될 수 있는 것이다. 한마디로, private메서드는 aop기능이 추가된 코드가 아닌 우리가 정의한 로직으로 수행이 된다.

<br>

그렇다면 의존성주입된 객체를 사용안하면 Proxy기능은 작동할까? 에러는 여전히 발생할까?

```java
@Custom
@GetMapping("/v3")
private ResponseEntity<String> get3(){
    Method[] c = this.getClass().getDeclaredMethods();
    return ResponseEntity.ok("1");
}
```

마찬가지로 Enhancer로 동작하며 null point exception없이 정상적으로 응답하지만 AOP기능은 동작하지 않는다.

또한 c의 값을 살펴보면 get3()의 정보는 없는 것을 볼 수 있는데, 이 부분이 재미있는 포인트이다. get3()를 통해 c가 생성되었지만 현재 클래스의 메서드에는 get3()의 정보가 없다.

이유는 위에서 말했듯이 this는 Enhancer로 생성된 객체이기 때문에 get3()가 정의 되지 않아 c의 값에 없는 것이고 호출될 수 있었던 것은 부모클래스의 get3()를 통해 호출이 될 수 있었던 것이다.

<br>

<br>

#### 정상적인 경우의 logtrace

![Fast](/구조/7주차-프록시/image/fastClass.PNG)
최종적으로 `FruitController$$FastClassBySpringCGLIB$$8f3c710b@6334`에 param으로 FruitController 주입하여 메서드호출하는 방식으로 프록시 객체 생성

아래의 MethodFroxy의 invoke메서드를 통해 실제 빈 호출

```java
//MethodProxy 중...

public Object invoke(Object obj, Object[] args) throws Throwable {
		try {
			init(); //FastClass get
			FastClassInfo fci = fastClassInfo;
			return fci.f1.invoke(fci.i1, obj, args);
		}
        //catch문...
}

private void init() {
    if (fastClassInfo == null) {  //fastClassInfo는 volataile로 여러 스레드에서 접근가능
        synchronized (initLock) {
            if (fastClassInfo == null) {
                CreateInfo ci = createInfo;

                FastClassInfo fci = new FastClassInfo();
                fci.f1 = helper(ci, ci.c1);     //FastClass 생성
                fci.f2 = helper(ci, ci.c2);
                fci.i1 = fci.f1.getIndex(sig1);
                fci.i2 = fci.f2.getIndex(sig2);
                fastClassInfo = fci;
                createInfo = null;
            }
        }
    }
}

private static FastClass helper(CreateInfo ci, Class type) {
		FastClass.Generator g = new FastClass.Generator();
		g.setType(type);
		// SPRING PATCH BEGIN
		g.setContextClass(type);
		// SPRING PATCH END
		g.setClassLoader(ci.c2.getClassLoader());
		g.setNamingPolicy(ci.namingPolicy);
		g.setStrategy(ci.strategy);
		g.setAttemptLoad(ci.attemptLoad);
		return g.create();
	}
```

#### private 경우 logtrace

![Enhancer](/구조/7주차-프록시/image/enhancer.PNG)
Interceptor에 걸리지 못해 그대로 Enhancer의 프록시 객체로 로직을 수행하여 의존성주입이 이루어지지 않음.

이는 트랜잭션, AOP, Secruity,Async 에서도 발생할 수 있는 예외상황으로 알아두면 좋을 것 같다.

<br><br><br>

### Reference

- [https://refactoring.guru/design-patterns/facade](https://refactoring.guru/design-patterns/facade)
- [인프런 디자인 패턴 강의](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)
- [인프런 스프링 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard)
