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
아주 단순하게 구현하기 위해 map을 통한 데이터 접근을 구현하며 이를 db를 대체한다고 하자. 이런 객체를 controller에서 이용하여 데이터를 조회하는 프로그램이 있다고하자. 그런데 조회할때 같은 과일이름의 조회결과는 캐싱을 추가하려고 한다. 그러면 아래와 같이 코드를 작성할 수 있을 것이다.
```java
@Repository
public class FruitRepository {
    private static final Map<String, Fruit> cachedFruitName = new HashMap<>();
    private static final Map<String, Fruit> fruitList = new HashMap<>(Map.of(
            "orange",new Fruit("orange",10000),
            "strawberry", new Fruit("strawberry", 20000),
            "mango", new Fruit("mango",30000)
    ));

    @Override
    public Fruit getFruitByName(String name) {
        System.out.println("Repository에서 " + name +" 조회");
        cachedFruitName.computeIfAbsent(name, key -> fruitRepository.getFruitByName(name));
        return cachedFruitName.get(name);
    }
}
```
그런데 만약 Repository가 우리가 구현한 객체가 아니라 수정이 불가능하다면 캐싱을 이렇게 직접 적용이 불가능 할 것이다. 이를 프록시를 통해 해결해보자.

```java
public class FruitRepository{
    private static final Map<String, Fruit> fruitList = new HashMap<>(Map.of(
            "orange",new Fruit("orange",10000),
            "strawberry", new Fruit("strawberry", 20000),
            "mango", new Fruit("mango",30000)
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


만일 Repository를 수정할 수 있어 Interface를 구현하도록 만들 수 있다면 아래와 같이 구현을 할 수도 있다.
```java
public interface FruitRepositoryInterface {
    Fruit getFruitByName(String name);
}

public class FruitRepository implements FruitRepositoryInterface{
    private static final Map<String, Fruit> fruitList = new HashMap<>(Map.of(
            "orange",new Fruit("orange",10000),
            "strawberry", new Fruit("strawberry", 20000),
            "mango", new Fruit("mango",30000)
    ));


    public Fruit getFruitByName(String name) {
        System.out.println("Repository에서 과일 조회");
        return fruitList.getOrDefault(name,null);
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
- 지연 초기화 : 시스템 리소스를 많이 차지하는 서비스 객체가 존재할때 앱이 시작되는 시점에 객체를 생성하는 것이 아니라 실제로 사용하는 시점에 사용하도록하여 성능향상을 꾀하고자 할때
- 접근 제어 : 특정 클라이언트만 서비스 객체를 사용할 수 있도록 하려는 경우
- 로깅 요청 : 실제 서비스 객체에 요청을 전달하기 전에 제어할 수 있다는 점을 이용하여 요청을 로깅하려고 하는 경우
- 캐싱 : 서비스 로직을 통한 반환값이 일정하며 처리시간이 긴 경우에 결과를 캐시하고 캐시의 수명주기를 관리하고자 하는 경우
- 자원 해제 : File, datasource등 자원을 해제하지 않으면 많은 리소스를 잡아먹는 객체의 경우 자원해제를 사용자가 아닌 프록시객체에게 위임하고자 하는 경우

<br><br>

## 3. 다른 패턴들과 비교
- Adapter : 랩핑된 객체와 다른 인터페이스를 제공하지만, 프록시는 동일한 인터페이스를 제공한다.
- Facade : 객체를 버퍼링하고 자체적으로 초기화한다는 점은 비슷하지만 프록시는 해당 서비스 객체와 동일한 인터페이스를 갖는다.
- Decorator : 구조가 매우 비슷하며 특정 작업을 다른 객체에게 위임하는 점은 비슷하나 프록시는 객체 자체적으로 서비스 객체의 수명주기, 행동을 관리하지만 데코레이터는 행동의 제어가 클라이언트에게 있다.
    
    데코레이터는 `런타임`에 `기능을 추가`하는 것이 목적, Proxy는 `컴파일 타임`에 `행동을 제어` 하는 것이 목적

<br><br><br>


## 4. Spring 프로젝트에 Proxy를 통해 서비스를 확장해보자!
```java
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
}
```
위와 같은 controller-service-repository 계층의 앱이 존재할때 각 계층의 log를 깊이별로 tracing하는 기능을 추가하고자 할때 아래와 같이 추가할 수 있다.

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
2021-12-30 10:12:08.205  INFO 16704 --- [nio-8080-exec-1] com.example.springex.proxy.LogTrace      : [8b8df55b] |   |-->FruitService.getFruit()
2021-12-30 10:12:08.205  INFO 16704 --- [nio-8080-exec-1] com.example.springex.proxy.LogTrace      : [8b8df55b] |   |<--FruitService.getFruit() time=0ms
2021-12-30 10:12:08.205  INFO 16704 --- [nio-8080-exec-1] com.example.springex.proxy.LogTrace      : [8b8df55b] |<--FruitService.getFruit() time=0ms
2021-12-30 10:12:08.208  INFO 16704 --- [nio-8080-exec-1] com.example.springex.proxy.LogTrace      : [8b8df55b] FruitsController.getFruitLogging() time=4ms
```

하지만 이는 로깅을 적용하고 싶은 빈마다 LogTrace를 주입시켜주고 비즈니스로직을 try로 감싸 기능을 추가해주어야 한다. 굉장히 보일러플레이트도 늘어날뿐 아니라 단일책임원칙에도 위배되고 있다. 

Proxy를 시작으로 AOP까지 확장해가며 서비스를 확장시켜보자.

<br>

### 1. Concreate Proxy
```java
@RequestMapping
@RequiredArgsConstructor
public class FruitController {
    private final FruitService fruitService;

    @GetMapping("/v2/fruit")
    public ResponseEntity<Fruit> getFruitLogging(@RequestParam String name){
        return ResponseEntity.ok(fruitService.getFruit(name));
    }
}
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

@RequiredArgsConstructor
public class FruitService {
    private final FruitRepository fruitRepository;

    public Fruit getFruit(String name) {
        return fruitRepository.getFruitByName(name);
    }
}
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

@Bean
public FruitController fruitController(LogTrace logTrace) {
    FruitController controller = new FruitController(fruitService(logTrace));
    return new FruitControllerProxy(controller,logTrace);
}

@Bean
public FruitService fruitService(LogTrace logTrace) {
    FruitService service = new FruitService(fruitRepository(logTrace));
    return new FruitServiceProxy(service,logTrace);
}

@Bean
public FruitRepository fruitRepository(LogTrace logTrace) {
    FruitRepository repository = new FruitRepository();
    return new FruitRepositoryProxy(repository,logTrace);
}
```

<br>

### 2. Interface Proxy
```java
@RequestMapping
@ResponseBody
public interface FruitController {
    @GetMapping("/v1/fruit")
    ResponseEntity<Fruit> getFruit();

    @GetMapping("/v2/fruit")
    ResponseEntity<Fruit> getFruitLogging(@RequestParam String name);
}

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

<br>

### 3. DynamicProxy
```java
public class LogTraceBasicHandler implements InvocationHandler {

    private final Object target;
    private final LogTrace logTrace;

    public LogTraceBasicHandler(Object target, LogTrace logTrace) {
        this.target = target;
        this.logTrace = logTrace;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

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

#### 패턴 매칭

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
        String methodName = method.getName();
        if (!PatternMatchUtils.simpleMatch(patterns, methodName)) {
            return method.invoke(target, args);
        }

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

<br>

### 4. ProxyFactory
여기서부터는 AOP를 이용한 방법이다. spring에서 제공하는 aop api를 이용해보자.

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
        FruitController orderController = new FruitControllerImpl(orderService(logTrace));
        ProxyFactory factory = new ProxyFactory(orderController);
        factory.addAdvisor(getAdvisor(logTrace));
        FruitController proxy = (FruitController) factory.getProxy();
        log.info("ProxyFactory proxy={}, target={}", proxy.getClass(), orderController.getClass());
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
        //pointcut
        NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
        pointcut.setMappedNames("getFruit*");
        //advice
        LogTraceAdvice advice = new LogTraceAdvice(logTrace);
        return new DefaultPointcutAdvisor(pointcut, advice);
    }
}
```

<br>

### 5. postProcessor
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

### 6. AutoProxy
여기서부터는 pointcut을 표현식을 통해서 적용하는 패턴으로 `implementation 'org.springframework.boot:spring-boot-starter-aop'`를 추가해주어야 한다.

```java
@Configuration
public class AutoProxyConfig {
//    @Bean
    public Advisor advisor1(LogTrace logTrace) {
        //pointcut
        NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
        pointcut.setMappedNames("getFruit*");
        //advice
        LogTraceAdvice advice = new LogTraceAdvice(logTrace);
        return new DefaultPointcutAdvisor(pointcut, advice);
    }

//    @Bean
    public Advisor advisor2(LogTrace logTrace) {
        //pointcut
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* com.example.springex.proxy._6_autoProxy..*(..))");
        //advice
        LogTraceAdvice advice = new LogTraceAdvice(logTrace);
        return new DefaultPointcutAdvisor(pointcut, advice);
    }

    @Bean
    public Advisor advisor3(LogTrace logTrace) {
        //pointcut
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* com.example.springex.proxy._6_autoProxy..*(..)) && !execution(* com.example.springex.proxy._6_autoProxy.FruitController.getFruit(..))");

        //advice
        LogTraceAdvice advice = new LogTraceAdvice(logTrace);
        return new DefaultPointcutAdvisor(pointcut, advice);
    }
}
```

<br>

### 7. AOP (pointcut expression)
```java
@Slf4j
@Aspect
public class LogTraceAspect {
    private final LogTrace logTrace;

    public LogTraceAspect(LogTrace logTrace) {
        this.logTrace = logTrace;
    }

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

<br>

### 8. AOP (annotation)
```java
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

    @Log
    @GetMapping("/v2/fruit")
    public ResponseEntity<Fruit> getFruitLogging(@RequestParam String name){
        return ResponseEntity.ok(fruitService.getFruit(name));
    }
}

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

<br><br><br>

### Reference
- [https://refactoring.guru/design-patterns/facade](https://refactoring.guru/design-patterns/facade)
- [인프런 디자인 패턴 강의](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)
- [인프런 스프링 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard)