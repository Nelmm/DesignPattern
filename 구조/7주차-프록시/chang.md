# 프록시 패턴

특정 객체에 대한 접근을 제어하거나 기능을 추가 할 수 있는 패턴

대리인 패턴 → 접근 제어, 비용이 큰 경우 초기화 지연, 로깅, 캐싱

## 프록시 패턴의 설계 절차

1. interface로 메소드 설계 

2. RealSubject를 인터페이스를 받아 implements로 구현. 

3. Proxy 역시 인터페이스 받아서 구현함. 

4. RealSubject를 주입받아서, 리얼 서비스 자체를 변수로 받아서 처리하기

부가적인 일을 Proxy가 대신해서 처리할 수 있다. (메소드 직전에 앞뒤로 로깅한다던가... Real Object가 주입이 되었는지 확인도 가능함)

## 장단점

장점

- 기존 코드 변경없이 새로운 기능 추가 가능
- 기존 코드가 해야하는 일만 유지 가능.
- 기능 추가 및 초기화 지연 등으로 다양하게 활용 가능.

단점 

- 코드 복잡도 증가

## 어디서 쓰는가?

### 자바

다이나믹 프록시, 리플렉션.

invocationHandler을 통해서 invoke 메소드를 통해서 클래스 로드하고, 클래스를 받아서, 프록시 방식으로 적용해준다.

### 스프링

**AOP**

Aspect를 적용하면, → 내부적으로 다이나믹 프록시를 사용해서 동적으로 프록시 빈을 만듬. 프록시 빈을 인터페이스로 

CGLibray를 통해 가짜객체로 감싸고 사용하게 됨.  ⇒ 진짜 객체를 사용해서 덮어 씌우는 방식으로 이용되어진다. 

`@Transactional`, `@Cachable`과 같은 것들이 모두 AOP를 통해서 구현되어진 방식이다. 

## JPA

JPA도 Proxy방식으로 구성되어있는데, 개인적으로 JPA에서 `@NoArgsConstructor` 방식으로 Entity를 구성해야하는 이유가 상당히 궁금했었는데 그 이유가 이 프록시 패턴을 사용하면 위에서 말했었던, 여러가지 이점들을 누릴 수 있기 때문에 사용했던 것이라고 생각했고, 좀 찾아봤다.

JPA에서 이 프록시를 사용하는 방식은 다음과 같은데, 

![](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/31_proxy.PNG?raw=true)

DB 조회 전 MemberProxy에서 일단 기본적으로 비어있는 기본 생성자를 구성해서 구현하는 방식을 취하되, 초기화 요청이 오는 경우 이 생성한 기본 생성자를 받고, 그 안에 넣은 후 이 맴버를 실제로 구현하는 방식이라고 생각하는데... 

[java - Why does JPA require a no-arg constructor for domain objects? - Stack Overflow](https://stackoverflow.com/questions/2808747/why-does-jpa-require-a-no-arg-constructor-for-domain-objects)

그리고 생각해보면 지연 로딩(Lazy Loading)을 위해서 다른 Entity를 가져오기 위해서는 엔티티를 일단은 찾기전에 가져오긴해야 하므로, 기본 생성자를 통해 엔티티를 가져오는데, 

JPA에서는 대략 이런 방식으로 Entity를 가져오는 듯하다. 

```java
    public PojoInstantiator(
            Class mappedClass,
            ReflectionOptimizer.InstantiationOptimizer optimizer,
            boolean embeddedIdentifier) {
        this.mappedClass = mappedClass;
        this.optimizer = optimizer;
        this.embeddedIdentifier = embeddedIdentifier;
        this.isAbstract = ReflectHelper.isAbstractClass( mappedClass );

        try {
            constructor = ReflectHelper.getDefaultConstructor(mappedClass);
        }
```

위처럼  MappedClass에서 리플랙션핼퍼를 통해서 기본 생성자를 불러와서 생성자를 가져오게 되는데, 만약에 MappedClass가 기본 생성자가 없다면 아예 만들어지지 않고...

```java
else if ( constructor == null ) {
			throw new InstantiationException( "No default constructor for entity: ", mappedClass );
		}
```

`InstantiationException`를 던지게 된다. 

즉, 기본 생성자가 없으면 Proxy 방식으로 Entity를 기본으로 생성할 수 없다.



물론, 문제는 JPA에서 기본 생성자를 만드는 이유는 반드시 Entity를 생성하기 위해서 기본 생성자를 가지고 있어야하는 것까지는 맞는 것 같은데, 

그것이 JPA Proxy 패턴을 사용하기 위해서 반드시 기본 생성자를 사용해야하는 건지에 대해서는 명확하게 말할 수 는 없을 것 같다.

그냥 단순하게, Proxy 패턴에 리플렉션을 쓰고 그 리플렉션에서 생성자를 가져오는 조건 자체가 Default Constructor를 통해서 구현하는 것이니까 이게 Proxy 패턴을 사용했기 때문에,  기본 생성자를 쓰는 것이 프록시 패턴을 사용하는 건가라고 말할 수 있는건지 확답을 못내리겠음...

[JPA Entity의 기본 생성자 관련 예외 정리하기](https://wbluke.tistory.com/6)



## 추가 궁금증

다이나믹 리플렉션과 CGLibrary 프록시 방식은 뭐가 다른가?  

## 다이나믹 리플렉션

```java
Object proxy = Proxy.newProxyInstance(
   ClassLoader, 
   Class<?>[],
   InvocationHandler
);
```

다이나믹 리플렉션은  Class를 받아서 처리하지만, Interface기반의 Proxy extend하지 않음.

특정 Class가 interface를 없으면 Proxy 패턴을 구현할 수 없다. 

## CGLibrary 리플렉션

```java
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(Target.class); 
enhancer.setCallback(MethodInterceptor);

Object proxy = enchancer.create(); // proxy 생성
```

리플렉션은 Class를 받아서, Extends를 통해서 프록시 객체를 만듬. 

단점이 몇개 존재했지만.. 1. Enhancer라는 외부 라이브러리 참조해야한다는 점. 2. 디폴트 생성자가 무조건 있어야했음. 3. 생성자가 두번 호출됨. 

-> 스프링 버전업을 하면서 Enhancer는 Spring 프로젝트에 편입, 디폴트 생성자도 필요없어지고, 생성자 두번 호출되는 이슈도 해결되면서 Spring Boot에서는 기본 Proxy  생성 방식으로 차용함.



[JDK Dynamic Proxy와 CGLIB의 차이점은 무엇일까? | Moon`s Development Blog](https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html)

[Spring AOP의 원리 - CGlib vs Dynamic Proxy — 천천히 올바르게](https://huisam.tistory.com/entry/springAOP)