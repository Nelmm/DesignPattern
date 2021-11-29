# 싱글톤

## 목적
1. 인스턴스를 오직 한개만 만들기 위함
2. 한개만 만든 객체에 대해 글로벌하게 접근이 가능해야함


## 구현 방법

- Eager Initialization (사전 초기화) : 클래스 로딩시에 인스턴스를 생성하는 방법 ( 클래스 내부에서 인스턴스를 생성)

  인스턴스를 실제로 사용하지 않는다몀 불필요한 연산과 메모리 낭비

    ```java
    public class Settings{
        private static final Settings INSTANCE = new Settings();

        private Settings(){}

        public static Settings getInstance() {
            return INSTANCE;
        }
    }

    //byte code
    private final static Lstudy/Settings; INSTANCE

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
  

- Lazy Initialization (지연 초기화) : 인스턴스를 실제로 사용할 시점에서 인스턴스를 생성하는 방법으로 인스턴스를 실제로 사용하지 않는다면 메모리와 연산량을 아낄 수 있으나 이중 객체 생성 문제 발생할 가능성이 높다.

    ```java
    public class Settings{
        private static Settings instance;

        private Settings(){}

        public static Settings getInstance() {
            if(instance == null){
                instance = new Settings();
            }
            return instance;
        }
    }
    //
    public class study/Settings {

        // compiled from: Settings.java

        // access flags 0xA
        private static Lstudy/Settings; instance

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
            GETSTATIC study/Settings.instance : Lstudy/Settings;
            IFNONNULL L1
        L2
            LINENUMBER 11 L2
            NEW study/Settings
            DUP
            INVOKESPECIAL study/Settings.<init> ()V
            PUTSTATIC study/Settings.instance : Lstudy/Settings;
        L1
            LINENUMBER 13 L1
        FRAME SAME
            GETSTATIC study/Settings.instance : Lstudy/Settings;
            ARETURN
            MAXSTACK = 2
            MAXLOCALS = 0
        }
    ```

    여러개의 스레드가 getInstance를 거의 동시에 접근하여 객체를 얻는 상황이 발생한다고 한다면, instance가 null 인지 검사하는 부분에서 조건이 걸리지 않아 여러개의 객체를 생성하는 문제가 발생할 수 있다. 이를 해결하기 위해서는 동시에 접근을 하지못하도록 lock을 걸어야 하기 때문에 synchronized 키워드를 사용하면 확정으로 한개만 생성할 수 있다.  

    ```java
    public class Settings{
        private static Settings instance;

        private Settings(){}

        public static synchronized Settings getInstance() {
            if(instance == null){
                instance = new Settings();
            }
            return instance;
        }
    }

    //byte code
    public class study/Settings {
        // compiled from: Settings.java

        // access flags 0xA
        private static Lstudy/Settings; instance

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

        // access flags 0x29
        public static synchronized getInstance()Lstudy/Settings;
        L0
            LINENUMBER 10 L0
            GETSTATIC study/Settings.instance : Lstudy/Settings;
            IFNONNULL L1
        L2
            LINENUMBER 11 L2
            NEW study/Settings
            DUP
            INVOKESPECIAL study/Settings.<init> ()V
            PUTSTATIC study/Settings.instance : Lstudy/Settings;
        L1
            LINENUMBER 13 L1
        FRAME SAME
            GETSTATIC study/Settings.instance : Lstudy/Settings;
            ARETURN
            MAXSTACK = 2
            MAXLOCALS = 0
        }
    ```
    하지만 lock을 걸고 회수하는 것도 모두 자원을 소비하는 것이기 때문에 성능상 불이익이 있을 수 있다.

    - 최적화 
        - Double Check Locking : 스레드를 학부생때 배운내용으로 동기화 과정에서 lock의 범위는 최소한으로 하는 것이 좋다고 배웠었다. lock의 범위가 넓어질 수록 락이 걸릴 필요가 없는 경우도 포함되어 스레드가 대기하는 부분이 많아지고 그만큼 비용이 들기 때문이다.
  
            ```java
            public class Settings{
                private static volatile Settings instance;

                private Settings(){}

                public static Settings getInstance() {
                    if(instance == null){
                        synchronized ( Settings.class ){
                            if(instance == null){
                                instance = new Settings();
                            }
                        }
                    }
                    return instance;
                }
            }

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

                // access flags 0x9
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
            
            위처럼 instance가 생성되었는지를 두번 체크함으로써 lock의 범위를 줄여 필요없는 동기화 과정을 줄일 수 있다.

            Settings를 `LDC`하는 것을 볼 수 있는데 이는 `런타임 상수풀에 값을 push(LDC)`하는 작업이다. 후에 모니터객체와 값을 비교하기위해 클래스를 상수값으로 푸시하는 것을 볼 수 있다.

            이때 instance를 volatile keyword를 사용하지 않을 경우, double-checked locking에서 문제가 발생한다. 
            <br>두 개의 thread T1, T2가 있다고 가정해보자. T1이 instance를 생성한 후에 synchronized block를 벗어나고 T2가 synchronized block에 들어가서 null 체크를 하는 시점에서 working memory와 main memory 사이의 동기화가 이뤄져있지 않을 수 있다. 즉, T1이 생성한 instance가 main memory에 존재하지 않거나 main memory에는 있지만 T2의 working memory에는 없을 경우 T2 또한 instance를 생성하게 된다.

            ```java
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
            ```
            동기화 블럭을 사용시 괄호 안의 객체를 전달받고 있으며 이객체를 통해 동기화를 진행한다. 이를 `모니터 객체`라고 부른다.  그래서 여러 스레드에서 이 모니터 객체에 접근하고 이를 통해 동기화를 수행하기 때문에 컴파일타임에 새로 변수에 할당하는 것을 볼 수 있고 바이트 코드를 봐도 DUP을 통해 스택의 최상위에 있는 값(여기서는 Settings.class)를 복제하여 ASTORE하는 것을 볼 수 있다.

        


        - static inner 클래스 : static inner 클래스를 활용하면 위의 double check locking처럼 getInstance가 호출되는 타이밍에 inner cl
  
            ```java
                public class Settings{
                private static Settings instance;

                private Settings(){}

                private static class SettingsHolder {
                    private static final Settings INSTANCE = new Settings();
                }

                public static synchronized Settings getInstance() {
                    return SettingsHolder.INSTANCE;
                }
            }

            //bytecode
            public class study/Settings {

                // compiled from: Settings4.java
                NESTMEMBER study/Settings$SettingsHolder
                // access flags 0xA
                private static INNERCLASS study/Settings$SettingsHolder study/Settings SettingsHolder

                // access flags 0x2
                private <init>()V
                L0
                    LINENUMBER 8 L0
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
                    LINENUMBER 15 L0
                    GETSTATIC study/Settings$SettingsHolder.INSTANCE : Lstudy/Settings;
                    ARETURN
                    MAXSTACK = 1
                    MAXLOCALS = 0
            }
            ```

            이를 `Initialization-on-demand holder idiom`라고 하마며 요청시 초기화하여 lazy-load를 하는 패턴. JVM이 Settings 클래스를 load하여 초기화할때 중첩(내부) 클래스는 최초 실행 시점에 load한다는 점을 이용한 것.


#### +) 자바의 동기화
1. 암묵적인 동기화 : synchronized keyword

    JVM의 BytecodeInterpreter에서 InterpreterRuntime의 MONITORENTER / MONITOREXIT 메서드 를 호출하여 구현. 

    명시적인 동기화와 비교하면 암묵적인 동기화는 `biased lock`이라는 기법을 사용하여 경쟁상태가 아닌 스레드의 성능 향상 효과가 있다. 한마디로 동일 스레드가 연속적으로 cirtical section에 접근하는 경우에 실제 lock이 아닌 흉내만 내는 lock을 통해 atomic operation을 수행하지 않아 수행속도를 향상 시킬 수 있다.

    - 메서드 동기화 : BYTECODE  Interpreter가 메서드를 수행하는 시저메 동기화 여부를 판별
    - 블럭 동기화 : 컴파일러에 의해 MONITORENTER / MONITOREXIT 로 변환


2. 명시적인 동기화 : concurrent.locks.lock

    HOTSPOT에서 바로 NATIVE INTRINSIC FUNCTION 호출

## 싱글톤이 깨지는 경우
객체를 사용하는쪽에서 이상하게 사용하면 깨질 수 있다.

### 1. Reflection
```java
public class Study {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Settings settings = Settings.getInstance();

        Constructor<Settings> constructor = Settings.class.getDeclaredConstructor();
        constructor.setAccessible(true);
        Settings settings1 = constructor.newInstance();

        System.out.println(settings == settings1); //false
    }
}
```
생성자를 private로 막아놔도 리플레션을 통해 접근을 하게 되면 싱글톤이 깨질 수 있다.

### 2. Serialize / DeSerialize
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



### Reference
-----

https://01010011.blog/2017/01/20/java-synchronization-internal/