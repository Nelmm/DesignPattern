Real Object에 대해서 실제로 필요할 때 instance가 생성되고 실제 작업이 진행될 수 있도록 하기 위해 적용되는 패턴

## Proxy 예
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

### 3. Interface Proxy
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

### 4. ProxyFactory
AOP를 이용한 방법
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
// implementation 'org.springframework.boot:spring-boot-starter-aop'

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

## 차이점
- Decorator : `런타임`에 `기능을 추가`하는 것이 목적, Proxy는 `컴파일 타임`에 `행동을 제어` 하는 것이 목적