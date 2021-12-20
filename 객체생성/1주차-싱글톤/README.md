# 싱글톤
> **인스턴스를 오직 한 개만 만들어서 제공하는 클래스**가 필요한 경우에 사용하는 패턴

![img](https://user-images.githubusercontent.com/42997924/143546595-d548b627-e85d-4fed-be93-19ff66eafaa6.png)

즉, 클래스가 최초 한번만 메로리를 할당받고 그 메모리에 인스턴스를 만들어 사용하는 디자인 패턴으로 생성자가 여러 차례 호출되더라도 실제 생성되는 인스턴스는 기존에 생성된 인스턴스이다.

new 를 통해서 생성자를 만들어지면 같지 않은 경우가 생길수 있기 때문에 생성자를 private으로 제한하고 static 메서드를 통해서 생성하도록 해야한다.


<br><br>

## 1. 사용 이유

시스템 런타임, 환경 세팅 관련 정보 등 인스턴스가 여러 개일 때 문제가 발생하는 경우 등이 있는데 싱글톤 패턴을 사용함으로써 가져갈 수 있는 이점은 다음과 같다.

1.  **메모리, 속도 측면** : 객체의 인스턴스를 재사용하기 때문(고정된 메모리 영역을 사용)
2.  **데이터 공유가 쉬움** : 기존 인스턴스가 전역으로 사용되기 때문
3.  인스턴스가 **한 개만 존재하는 것을 보장**하고 싶은 경우

<br><br>

## 2. 구현
### 이른 초기화(Eager Initialization)
```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        return INSTANCE;
    }
}
// access flags 0x2
    private <init>()V
    L0
        LINENUMBER 7 L0
        ALOAD 0
        INVOKESPECIAL java/lang/Object.<init> ()V
        RETURN
    L1
        LOCALVARIABLE this Lstudy/Settings; L0 L1 0
        MAXSTACK = 1
        MAXLOCALS = 1

    // access flags 0x9
    public static getInstance()Lstudy/Settings;
    L0
        LINENUMBER 10 L0
        GETSTATIC study/Settings.INSTANCE : Lstudy/Settings;
        ARETURN
        MAXSTACK = 1
        MAXLOCALS = 0

    // access flags 0x8
    static <clinit>()V
    L0
        LINENUMBER 5 L0
        NEW study/Settings
        DUP
        INVOKESPECIAL study/Settings.<init> ()V
        PUTSTATIC study/Settings.INSTANCE : Lstudy/Settings;
        RETURN
        MAXSTACK = 2
        MAXLOCALS = 0
```
static 키워드를 통해 클래스 로더가 초기화하는 시점에 **정적 바인딩(Static Binding)**을 통해 해당 인스턴스를 메모리에 등록하기 때문에 **Thread-safe** 하다.

static <clinit>() 메서드를 통해 인스턴스를 초기화 하는 것을 볼 수 있다. (PUTSTATIC)

#### 장점 : Thread safe
#### 단점 : 미리 만들어두기 때문에 실제 해당 인스턴스를 사용하지 않으면 메모리 측면에서 손해


<br>

### 늦은 초기화(Lazy Initialization)
```java
public class Settings {
	private static Settings instance;

	private Settings() {}

	private static Setiings getInstance(){
		if(instance == null){
			instance = new Settings();
		}
		return instance;
	}
}
```
인스턴스를 **실제 사용하는 시점에서 생성**하는 방법 - **동적 바인딩(Dynamic Binding)**

-  이른 초기화 방법보다 메모리 측면에서 효율적
-  아래 **getInstance( )** 는 멀티 스레드 환경에서는 안전하지 않다.

```bash
Thread A : if(INSTANCE == null) 수행 결과 true
Thread B : if(INSTANCE == null) 수행 결과 true

Thread A : INSTANCE = new Singleton() 수행으로 인스턴스1 생성
Thread B : INSTANCE = new Singleton() 수행으로 인스턴스2 생성
```
만약 두 Thread 가 거의 동시에 해당 인스턴스에 접근 시하면 인스턴스가 생성되어 있지 않는 것으로 보고 중복으로 생성할 수 있기 때문이다.

#### 장점 : 사용 시점에 인스턴스를 생성하여 메모리를 효율적으로 사용
#### 단점 : Thread Safe 하지 않음

<br>

### 늦은 초기화, 동기화 처리(Lazy Initialization with synchronized)
```java
public class Singleton {
    private static Singleton instance;
    private Singleton() { }

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
위에서 살펴본 Lazy Initialization의 멀티 스레드 문제는 **Synchronized** 키워드를 사용하여 동기화 처리를 통해 해결할 수 있다.

단점은 getInstance( )를 호출 시에 해당 인스턴스의 생성 여부와 상관 없이 **동기화** 블록을 거쳐야 한다는 점이다.

기본적으로 동기화라는 과정이 락(Lock)을 거는 메커니즘을 사용하기 때문에 성능이 떨어질 수 밖에 없다.

#### 장점 : 메모리 효율적으로 사용, Thread Safe
#### 단점 : 인스턴스 생성 여부와 상관없는 동기화(Lock) 때문에 성능이 떨어짐

<br>

### 늦은 초기화, DCL(Lazy Initialization. Double Checked Locking)
```java
public class Singleton {
    private volatile static Singleton instance;
    private Singleton() { }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if(instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}


```
위 동기화 블럭 방식을 개선한 방식으로, **먼저 인스턴스의 생성 여부를 확인**하는 방법이 있다. 이는 스레드와 관련된 내용으로 동기화 과정에서 lock의 범위는 최소한으로 하는 것이 대기하는 부분이 적어지고 그만큼 비용이 적게 들기 때문이다.

인스턴스가 생성되지 않은 경우에 동기화 처리를 하기 때문에 **효율적으로 동기화** 블록을 만들 수 있다.

이 경우에는 **volatile** 키워드를 사용해야 DCL이 정상적으로 동작할 수 있다.

멀티스레딩을 쓰더라도 instance 변수가 Sigleton 인스턴스로 초기화 되는 과정이 올바르게 진행되기 때문이다.

```java
//compile된 메서드
public static Settings getInstance() {
    if (instance == null) {
        Class var0 = Settings.class;
        synchronized(Settings.class) {
            if (instance == null) {
                instance = new Settings();
            }
        }
    }

    return instance;
}
// access flags 0x9
//byte code
public class study/Settings {
    // compiled from: Settings.java

    // access flags 0x4A
    private static volatile Lstudy/Settings; instance

    // access flags 0x2
    private <init>()V
    L0
        LINENUMBER 7 L0
        ALOAD 0
        INVOKESPECIAL java/lang/Object.<init> ()V
        RETURN
    L1
        LOCALVARIABLE this Lstudy/Settings; L0 L1 0
        MAXSTACK = 1
        MAXLOCALS = 1
    public static getInstance()Lstudy/Settings;
        TRYCATCHBLOCK L0 L1 L2 null
        TRYCATCHBLOCK L2 L3 L2 null

    L4
        LINENUMBER 10 L4
        GETSTATIC study/Settings.instance : Lstudy/Settings;
        IFNONNULL L5
    L6
        LINENUMBER 11 L6
        LDC Lstudy/Settings;.class
        DUP
        ASTORE 0
        MONITORENTER
    L0
        LINENUMBER 12 L0
        GETSTATIC study/Settings.instance : Lstudy/Settings;
        IFNONNULL L7
    L8
        LINENUMBER 13 L8
        NEW study/Settings
        DUP
        INVOKESPECIAL study/Settings.<init> ()V
        PUTSTATIC study/Settings.instance : Lstudy/Settings;
    L7
        LINENUMBER 15 L7
    FRAME APPEND [java/lang/Object]
        ALOAD 0
        MONITOREXIT
    L1
        GOTO L5
    L2
    FRAME SAME1 java/lang/Throwable
        ASTORE 1
        ALOAD 0
        MONITOREXIT
    L3
        ALOAD 1
        ATHROW
    L5
        LINENUMBER 17 L5
    FRAME CHOP 1
        GETSTATIC study/Settings.instance : Lstudy/Settings;
        ARETURN
        MAXSTACK = 2
        MAXLOCALS = 2
}
```
컴파일된 메서드를 보면 못보던 Class타입 var0이라는 변수를 선언한 것을 볼 수 있다.

동기화 블럭을 사용하면 괄호 안의 객체를 전달받고 있으며 이객체를 통해 동기화를 진행한다. 이를 `모니터 객체`라고 부른다.  그래서 여러 스레드에서 이 모니터 객체에 접근하고 이를 통해 동기화를 수행하기 때문에 컴파일타임에 새로 변수에 할당하는 것을 볼 수 있고 바이트 코드를 봐도 DUP을 통해 스택의 최상위에 있는 값(여기서는 Settings.class)를 복제하여 ASTORE하는 것을 볼 수 있다.

바이트 코드를 봐보면 Settings를 `LDC`하는 것을 볼 수 있는데 이는 `런타임 상수풀에 값을 push(LDC)`하는 작업이다. 후에 `모니터객체`와 값을 비교하기위해 클래스를 상수값으로 푸시하는 것을 볼 수 있다.

    

#### 장점
- 메모리 효율적으로 사용
- Thread Safe
- 인스턴스 생성 여부 검사 (Lock 이슈 해결)

#### 단점
- 비동기화된 Resource 필드에 의존하게 되어 위에서 알아본 것 처럼 변수의 최신 값이나 원자성을 보장해줘야한다.

#### volatile
volatile 변수를 사용하고 있지 않는 멀티 스레드 어플리케이션에서는 작업(Task)을 수행하는 동안 **성능 향상을 위해 Main Memory 에서 읽은 변수 값을 CPU Cache 에 저장**하게 된다. 만약에 멀티 스레드 환경에서 스레드가 변수 값을 읽어올 때 각각의 CPU Cache 에 저장된 값이 다르기 때문에 **변수 값 불일치 문제가 발생**하게 되는데, volatile 키워드가 이런 문제를 해결할 수 있다.

즉, volatile 변수는 **Main Memory 에 값을 저장하고 읽어오기 때문에(read and write)** 변수 값 불일치 문제가 생기지 않는다.

1. 하나의 스레드는 read and write 하며, 나머지 스레드는 read 만 하는 경우 변수의 최신 값을 보장한다.
2. 여러개의 스레드가 write 하는 상황이라면 동기화 블럭(synchronized) 을 지정해서 원자성(atomic) 을 보장해야 한다.

#### 자바의 동기화
1. 암묵적인 동기화 : synchronized keyword

    JVM의 BytecodeInterpreter에서 InterpreterRuntime의 MONITORENTER / MONITOREXIT 메서드 를 호출하여 구현. 

    명시적인 동기화와 비교하면 암묵적인 동기화는 `biased lock`이라는 기법을 사용하여 경쟁상태가 아닌 스레드의 성능 향상 효과가 있다. 한마디로 동일 스레드가 연속적으로 cirtical section에 접근하는 경우에 실제 lock이 아닌 흉내만 내는 lock을 통해 atomic operation을 수행하지 않아 수행속도를 향상 시킬 수 있다.

    - 메서드 동기화 : BYTECODE  Interpreter가 메서드를 수행하는 시저메 동기화 여부를 판별
    - 블럭 동기화 : 컴파일러에 의해 MONITORENTER / MONITOREXIT 로 변환


2. 명시적인 동기화 : concurrent.locks.lock

    HOTSPOT에서 바로 NATIVE INTRINSIC FUNCTION 호출

<br>

### LazyHolder - 늦은 초기화, Static Inner class사용
```java
public class Singleton {
    private Singleton() { }

    private static class SingletonHolder {
        public static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singlton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```
이를 `Initialization-on-demand holder idiom`라고 하마며 요청시 초기화하여 lazy-load를 하는 패턴. JVM이 Settings 클래스를 load하여 초기화할때 중첩(내부) 클래스는 최초 실행 시점에 load한다는 점을 이용한 것.

여기서 getInstance가 호출될 때 **SingletonHolder 클래스**가 호출이 되면 실제 인스턴스가 만들어지기 때문에 성능 이슈가 없다.

Singleton 클래스의 getInstance() 메서드에서 SingletonHolder.INSTANCE를 참조하는 순간 Class가 로딩되며 초기화가 진행된다. Class를 로딩하고 초기화하는 시점은 **thread-safe**를 보장하기 때문에 **volatile이나 synchronized 같은 키워드가 없어도 된다.**


<br>

### 늦은 초기화, Enum 사용

Enum 인스턴스의 생성은 기본적으로 Thread-safe 하기 때문에 스레드 관련 코드를 사용하지 않아도 되기 때문에 간편해진다.

```java
public enum Singleton {
    INSTANCE;
}
```

Enum을 사용하는 방식의 장점은 위에서 언급한 리플랙션, 직렬화와 역직렬화의 상황을 방지할 수 있다는 것이다.

다만,  이 경우에는 상속을 사용할 수 없다. 또한, Context의존성이 있는 환경에서는 싱글턴의 초기화 과정에 Context라는 의존성이 끼어들 가능성이 있는 단점이 있다.

#### 내부 생성방식
```java
public enum TestEnum {
	WHITE, BLACK, RED, YELLOW, BLUE;
}

//디컴파일
public final class TestEnum extends Enum
{

    private TestEnum(String s, int i)
    {
        super(s, i);
    }

    public static TestEnum[] values()
    {
        TestEnum atestenum[];
        int i;
        TestEnum atestenum1[];
        System.arraycopy(atestenum = ENUM$VALUES, 0, atestenum1 = new TestEnum[i = atestenum.length], 0, i);
        return atestenum1;
    }

    public static TestEnum valueOf(String s)
    {
        return (TestEnum)Enum.valueOf(Enums/TestEnum, s);
    }

    public static final TestEnum WHITE;
    public static final TestEnum BLACK;
    public static final TestEnum RED;
    public static final TestEnum YELLOW;
    public static final TestEnum BLUE;
    private static final TestEnum ENUM$VALUES[];

    static 
    {
        WHITE = new TestEnum("WHITE", 0);
        BLACK = new TestEnum("BLACK", 1);
        RED = new TestEnum("RED", 2);
        YELLOW = new TestEnum("YELLOW", 3);
        BLUE = new TestEnum("BLUE", 4);
        ENUM$VALUES = (new TestEnum[] {
            WHITE, BLACK, RED, YELLOW, BLUE
        });
    }
}
```
Enum을 컴파일러가 생성하는 것을 보면 private 생성자만을 가지며 각 필드 값들은 static final로 내부에서 미리 생성해 할당하여 배열로 이를 보관하고 있는 것을 볼 수 있다. 이 때문에 싱글톤이 유지가 된다.

<br><br>

## 3. 싱글톤이 깨지는 경우
### 리플렉션
```java
public class Application {
    public static void main(String[] args) throws NoSuchMethodExceotion, 
                                                  InvocationTargetException,
                                                  InstantiationExcetpion {
        Singleton singleton = Singleton.getInstance();
    
        Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
        constructor.setAccessible(true);
        Singleton singleton2 = constructor.newInstance();
    
        System.out.println(singleton == singletons2);    //false
    }
}
```
생성자를 private으로 제한을 해놓았어도 reflection을 통해 접근하여 생성을 하게 되면 결국 완전하게 방어를 할 수 없다.

<br>

### Serialize / DeSerialize
```java
public class Study {
    public static void main(String[] args) throws IOException,ClassNotFoundException {
        Settings settings = Settings.getInstance();
        Settings settings1 = null;

        try (ObjectOutput out = new ObjectOutputStream(new FileOutputStream("settings.obj"))){
            out.writeObject(settings);
        }

        try (ObjectInput in = new ObjectInputStream(new FileInputStream("settings.obj"))){
            settings1 = (Settings) in.readObject();
        }

        System.out.println(settings == settings1);      //false
    }
}
```
Serializable을 구현한 객체라면 Stream으로 직렬화/역직렬화 할 수 있는데 이때 역직렬화 하는 과정에서 기본이 new 키워드를 통해 객체를 생성하기 때문에 싱글톤이 깨질 수 있다.
```java
public class Settings implements Serializable {
    private Settings() { }

    private static class SettingsHolder {
        private static final Settings INSTANCE = new Settings();
    }

    public static Settings getInstance(){
        return SettingsHolder.INSTANCE;
    }
    
    @Serial
    protected Object readResolve() {
        return getInstance();
    }
}
```
물론, 위와 같이 readResolve 메서드를 구현하면 역직렬화의 방법을 재정의 하여 싱글톤을 유지 할 수 있다.

<br><br>

## 4. 사용 예
### java.lang.Runtime

Runtime이라는 자바가 제공하고 있는 라이브러리를 사용하는 경우

```java
Runtime runtime = Runtime.getRuntime();
```

new 생성자를 통해 생성할 수 없다.

<br>

### 스프링에서의 싱글톤 스코프

특정 정의된 빈을 가지고 ApplicationContext를 만들면 항상 같은 type의 빈이 나오게 된다.

```
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(Singleton.class);
```

이경우 싱글톤 스코프라고 말하는데 엄밀히 말해서는 싱글톤 패턴과는 다르다고 한다.

ApplicationContext내부에서 유일한 인스턴스로서 관리가 되는 것일 뿐이기 때문이다.

<br><br>

## Reference

-   [인프런 - 코딩으로 학습하는 GoF의 디자인 패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)
-   [webdevtechblog : 싱글턴 패턴](https://webdevtechblog.com/%EC%8B%B1%EA%B8%80%ED%84%B4-%ED%8C%A8%ED%84%B4-singleton-pattern-db75ed29c36)
-   [https://javaplant.tistory.com/21](https://javaplant.tistory.com/21)
-   [https://www.infoworld.com/article/2074979/double-checked-locking--clever--but-broken.html](https://www.infoworld.com/article/2074979/double-checked-locking--clever--but-broken.html)
-   [https://01010011.blog/2017/01/20/java-synchronization-internal/](https://01010011.blog/2017/01/20/java-synchronization-internal/)