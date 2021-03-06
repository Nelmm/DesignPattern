# 전략 (Strategy) 패턴

여러 알고리듬을 캡슐화하고 상호 교환 가능하게 만드는 패턴.

컨텍스트에서 사용할 알고리즘을 클라이언트가 선택한다.

어떤 일을 수행하는 방법이 여러가지 일 때, 여러 알고리즘을 각각의 개별적인 클래스로 캡슐화하고, 캡슐화된 것을 공통된 인터페이스로 추상화한다. 

로직을 사용하는 곳에서는 추상화된 인터페이스만 사용함으로써 코드는 바뀌지 않지만 사용하는 알고리즘을 바꿔 끼울 수 있도록 해주는 패턴.

* 자바의 `Comparator` 인터페이스가 대표적인 Strategy Interface이다.

  우리가 Comparator 구현체를 제공하는 것이 ConcreteStrategy를 만드는 과정이다.

## 적용할 수 있는 코드

* BluLightRedLight
  * `speed` 값에 따라서 서로 다른 동작을 수행한다.

```java
public class BlueLightRedLight {
    private int speed;
    public BlueLightRedLight(int speed) {
        this.speed = speed;
    }
    public void blueLight() {
        if (speed == 1) {
            System.out.println("무 궁 화    꽃   이");
        } else if (speed == 2) {
            System.out.println("무궁화꽃이");
        } else {
            System.out.println("무광꼬치");
        }
    }
    public void redLight() {
        if (speed == 1) {
            System.out.println("피 었 습 니  다.");
        } else if (speed == 2) {
            System.out.println("피었습니다.");
        } else {
            System.out.println("피어씀다");
        }
    }
}
```

* Client

```java
public class Client {
    public static void main(String[] args) {
        BlueLightRedLight blueLightRedLight = new BlueLightRedLight(3); //속도값 지정
        blueLightRedLight.blueLight();
        blueLightRedLight.redLight();
    }
}
```

```bash
무광꼬치
피어씀다
```

⇒ 특정 파라미터 값에 따라 조건문으로 분기하면서 각각의 다른 행위를 하는 코드가 보인다면 전략패턴을 적용할 수 있다.

그러면 새로운 전략을 추가하더라도 기존 코드를 변경하지 않고 확장할 수 있다. (`OCP 원칙`)


**구조**

![strategy_01](https://user-images.githubusercontent.com/42997924/154192846-bff0a3a2-9dc3-4ec4-b8f5-08e1881bbaca.png)

* Context

    * 원래 로직을 수행하는 클래스 (전체 동작)

    * Strategy **인터페이스**를 참조한다.

      구현체가 아닌 **인터페이스**를 참조해야 여러 전략을 바꿔 끼울 수 있는 상호교환이 가능한 상태가 된다.

* Strategy 인터페이스

    * Context의 모든 로직 중에서 **상황에 따라 달라지는 전략을 Strategy 인터페이스로 추출**한다.

* ConcreteStrategy

    * Strategy 인터페이스의 구현체
    * 각각의 알고리즘을 해당 Strategy내에 구현해놓는다.

* Client가 직접 사용할 ConcreteStrategy를 선택해서 Context를 만들 때 넣어줄 수 있다.

    * 방법1. 생성자에 넘겨줄 수 있다.
    * 방법2. 특정 오퍼레이션을 수행할 때, 그 때마다 다른 ConcreteStrategy를 넘겨줄 수 있다.

![strategy_02](https://user-images.githubusercontent.com/42997924/153322766-7fc9f467-76b6-4a77-a5af-f047ad3d4a0c.png)

* BlueLightRedLight : Context
* Speed : Strategy 인터페이스
* Fastest, Normal, Faster : ConcreteStrategy - 각각의 전략
* Client가 Fastest, Normal, Faster 중 하나를 선택해서 BlueLightRedLight를 실행한다.

## 전략 패턴 적용

### 1. Strategy Interface 정의

* 전략들의 공통 기능인 `bluLight()`와 `redLight()`를 가지는 인터페이스를 정의한다.

```java
public interface Speed {
    void blueLight();
    void redLight();
}
```

### 2. Context 정의

* 구체적인 타입에 의존하지 않고, Strategy Interface를 가지고 동작한다.
* `Speed` 인터페이스로 기능을 수행한다.
* 구현체는 필드를 통해서 주입받는다.

####  방법1. 생성자를 통해서 전략을 주입받는다.

```java
public class BlueLightRedLight {
  
   private Speed speed;
  
   public BlueLightRedLight(Speed speed) {
     this.speed = speed;
   }
    public void blueLight() {
        speed.blueLight();
    }
    public void redLight() {
        speed.redLight();
    }
}
```



#### 방법2. 오퍼레이션을 수행할 때, ConcreteStrategy를 주입받는다.

```java
public class BlueLightRedLight {
    public void blueLight(Speed speed) {
        speed.blueLight();
    }
    public void redLight(Speed speed) {
        speed.redLight();
    }
}
```

### 3. ConcreteStrategy 구현

Strategy Interface를 구현해서 각각의 전략마다의 구체적인 동작을 작성한다.

* Normal

  ```java
  public class Normal implements Speed {
      @Override
      public void blueLight() {
          System.out.println("무 궁 화    꽃   이");
      }
  
      @Override
      public void redLight() {
          System.out.println("피 었 습 니  다.");
      }
  }
  ```

* Faster

  ```java
  public class Faster implements Speed {
      @Override
      public void blueLight() {
          System.out.println("무궁화꽃이");
      }
  
      @Override
      public void redLight() {
          System.out.println("피었습니다.");
      }
  }
  ```

* Fastest

  ```java
  public class Fastest implements Speed{
      @Override
      public void blueLight() {
          System.out.println("무광꼬치");
      }
  
      @Override
      public void redLight() {
          System.out.println("피어씀다.");
      }
  }
  ```



### 4. Client는 구체적인 전략을 선택해서 Context를 실행한다.

어떠한 전략으로 게임을 진행할 것인지 지정해서 BlueLightRedLight 오퍼레이션을 실행한다.

####  방법1. 생성자를 통해서 전략을 주입받는 경우

* `BlueLightRedLight` 생성 시, 구체적인 전략을 생성해서 주입해준다.

```java
public class Client {
    public static void main(String[] args) {
        BlueLightRedLight game = new BlueLightRedLight(new Normal()); //생성시 전략을 주입한다.
        game.blueLight();
        game.redLight();
    }
}
```

```tex
무 궁 화    꽃   이
피 었 습 니  다.
```


#### 방법2. 오퍼레이션을 수행할 때, ConcreteStrategy를 주입받는 경우

* `blueLight()` 또는 `redLight()` 오퍼레이션을 실행할 때마다 다른 전략을 선택해서 주입해준다.

```java
public class Client {
    public static void main(String[] args) {
        BlueLightRedLight game = new BlueLightRedLight();
        game.blueLight(new Normal());
        game.redLight(new Fastest());
      
        // 익명 내부 클래스로 새로운 전략 추가
        // Context 코드 변경이 일어나지 않는다.
        game.blueLight(new Speed() {
            @Override
            public void blueLight() {
                System.out.println("blue light");
            }
            @Override
            public void redLight() {
                System.out.println("red light");
            }
        });
    }
}
```

```tex
무 궁 화    꽃   이
피어씀다.
blue light
red light
```

⇒ 새로운 전략 ConcreteStrategy를 추가해도 `Context`에 해당하는 BlueLightRedLight 코드에 변경이 일어나지 않는다.


## 장점

* 새로운 전략을 추가하더라도 기존 코드를 변경하지 않는다.

  ⇒ `OCP(개방 폐쇄 원칙)` 객체지향 원칙을 따른다.

* 상속 대신 위임을 사용할 수 있다.

    * 상속으로도 가능하지만, 상속 자체의 단점을 가지기 때문에 위임을 사용함으로서 이를 피할 수 있다.
    * 상속의 단점
        * 하나 밖에 상속받을 수 없다.
        * 상속을 강요하면 다른 상속이 필요할 때 받을 수 없게 된다.
        * 상위 클래스가 변경되면 하위 클래스가 전부 영향을 받는다.
    * 위임을 사용하면 `Context`의 코드가 변경되더라도 `Strategy` 코드는 변경의 영향을 받지 않는다. 그래서 일반적으로 상속보다 위임을 선호한다.

* 런타임에 전략을 변경할 수 있다.

* 전략이 한번만 사용되고 재사용되지 않는다면 익명클래스로 구현해 넣는 방식을 사용할 수 있다. 여러 클래스를 만들 필요가 없어진다.

## 단점

* 복잡도가 증가한다.
* 클라이언트 코드가 구체적인 전략을 알아야 한다. (구체적인 전략에 의존성이 생긴다.)

## 실무 사용 예

* 자바
    * Comparator

* 스프링
    * ApplicationContext
    * PlatformTransactionManager
    * ...
    

### 1. 자바 - Comparator

* 컬렉션에서 정렬 시, `sort()`에 정렬기준이 되는 `Comparator` 구현체를 넣어주면 된다.
* `Comparator`는 정렬 기준을 `compare()`의 반환값으로 판단한다.
* 컬렉션에서 사용하는 `Integer`의 경우 자바에서 기본으로 지원하는 데이터 타입이기 때문에 어떻게 계산해서 정렬하면 되는지 알고 있다. 따라서 직접 Compartor를 구현하지 않고, 그냥 호출해서 오름차순으로 정렬해준다.
* `Comparator`는 Java 8 버전부터 static 메소드로 기본 정렬 기준을 지원해준다.
    * `Comparator.reverseOrder()` : 역순정렬
    * `Comparator.naturalOrder()`

```java
public class StrategyInJava {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>();
        numbers.add(10);
        numbers.add(5);
        System.out.println(numbers); // [10, 5]
           
       // 숫자 크기 순으로 정렬하도록 Comparator를 구현해서 sort()에 넣어준다.
        Collections.sort(numbers, new Comparator<Integer>() {
            @Override
            public int compare(final Integer o1, final Integer o2) {
                return o1 - o2;
            }
        });
      
        System.out.println(numbers); // [5, 10]
        Collections.sort(numbers, Comparator.naturalOrder());
        System.out.println(numbers); // [5, 10]
    }
}
```



### 2. 스프링 - ApplicationContext

스프링이 지원해주는 인터페이스의 경우 거의 대부분 전략패턴이다.

대표적으로 `ApplicationContext`가 있다.

어떻게 설정파일을 읽어서 `ApplicationContext`를 만들지 여러 전략들이 있다.

* `ClassPathXmlApplicationContext` : ClassPath에서 XML을 읽어서 빈 설정파일로 사용
* `FileSystemXmlApplicationContext` : FileSystem 기준으로 XML을 찾아서 빈 설정파일로 사용
* `AnnotationConfigApplicationContext` : 자바 애노테이션 설정파일을 사용

각각이 서로 다른 전략인 것이다. 전략에 따라 구체적인 ConcreteStrategy에 해당하는 클래스가 나뉘어져 있다.

* 빈 설정 파일로부터 파싱해서 `BeanDefinitionParser`을 만들어야 한다. 다양한 전략이 존재한다.

  new BeanDefinitionParser → 자동완성에서 확인할 수 있다.

* `PlatformTransactionManager` : 스프링에서 `@Transaction` 사용할 때, 애노테이션에 있는 정보를 읽어서 실제 트랜잭션 처리를 이 인터페이스를 구현한 구현체들이 해준다.

  각각의 특정 기술마다 사용하는 API가 다르고, 트랜잭션 처리 로직도 다르다.

  각각의 알고리즘을 구체적인 클래스에 캡슐화해놓고 인터페이스 기반으로 코딩해서 사용한다.

  각각의 구체적인 전략은 빈으로 등록되게 된다.

* `CacheManager` : 캐시 기능 제공

  * EhCache, JCache, NoCache 등

```java
public class StrategyInSpring {
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext();
        ApplicationContext applicationContext1 = new FileSystemXmlApplicationContext();
        ApplicationContext applicationContext2 = new AnnotationConfigApplicationContext();
        
        BeanDefinitionParser parser;
        
        PlatformTransactionManager platformTransactionManager;
        
        CacheManager cacheManager;
    }
}
```

![strategy_03](https://user-images.githubusercontent.com/42997924/153322768-6ce21f57-a224-49f2-9ece-87dd54b2b04e.png)

> Spring에서 기술이 달라짐에 따라 인터페이스는 같지만, 다른 Bean을 등록하는 것은 대부분 전략 패턴이라고 보면 됨 
