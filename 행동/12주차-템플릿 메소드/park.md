# 템플릿 메소드 (Template method) 패턴

알고리즘 구조를 서브 클래스가 확장할 수 있도록 템플릿으로 제공하는 방법.

추상 클래스는 템플릿을 제공하고 하위 클래스는 구체적인 알고리즘을 제공한다.

![image-템플릿_메소드_01](https://user-images.githubusercontent.com/42997924/154322588-83209f4a-5841-476e-8260-5514bb8962c6.png)

- 구조 
  - AbstractClass 하나와 이를 구현한 하나 이상의 ConcreteClass가 존재한다. 
  - AbstractClass에 템플릿 메소드 역할을 하는 메소드가 있어야 한다. 
  - 템플릿 메소드는 알고리즘의 구조를 표현한 메소드라고 볼 수 있다.

- 데이터를 읽고 → 처리 → 출력 등의 일련의 과정인 알고리즘 구조가 있을 때,
  구조를 템플릿으로 제공한다.
  - 그 중 구체적인 방법(데이터를 가져오는 방법, 처리 방법, 출력 방법 등)을 이 템플릿을 상속받는 서브 클래스가 구체적으로 구현할 수 있도록 하는 패턴이다.

- 상속을 사용한다.

## 코드로 알아보는 템플릿 메소드 패턴이 필요한 상황

- Client

FileProcessor로 number.txt 파일(한 줄에 하나의 숫자가 적혀있는 파일)을 읽어서 모든 숫자의 합을 구해주는 `process()`를 통해 결과값을 받아와서 출력

```java
public class Client {
    public static void main(String[] args) {
        FileProcessor fileProcessor = new FileProcessor("number.txt");
        int result = fileProcessor.process();
        System.out.println(result);
    }
}
```

- FileProcessor

  `process()` : 파일의 라인에 있는 모든 숫자를 합한 결과를 반환

```java
public class FileProcessor {
    private String path;
    public FileProcessor(String path) {
        this.path = path;
    }
    public int process() {
        // Try-with-resources 자원해제
        try(BufferedReader reader = new BufferedReader(new FileReader(path))) {
            int result = 0;
            String line = null;
            while((line = reader.readLine()) != null) {
                // 차이점 Integer.valueOf(n) : Integer return
                result += Integer.parseInt(line);
            }
            return result;
        } catch (IOException e) {
            throw new IllegalArgumentException(path + "에 해당하는 파일이 없습니다.", e);
        }
        // finally 블록에서 처리해줄 필요 없음
    }
}
```

#### 참고

**parseInt()**

```java
public static int parseInt(String s) throws NumberFormatException {
    return parseInt(s,10);
}
```

> parseInt() 메소드는 전달받은 인자를 파싱하고 기본데이터 타입인 int 로 반환한다.

**valueOf()**

```java
public static Integer valueOf(String s, int radix) throws NumberFormatException {
    return Integer.valueOf(parseInt(s,radix));
}

public static Integer valueOf(int i) {
	final int offset = 128;
	if (i >= -128 && i <= 127) { // must cache
		return IntegerCache.cache[i + offset];
	}
	return new Integer(i);
}
```

valueOf() 메소드는 String 을 parseInt() 메소드로 전달하고 이 메소드는 실제로 String 을 변환하는 작업 후 int 데이터를 반환한다. 

그 다음  valueOf() 메소드가 호출되며, 반환 받은 int 타입이 이 메소드로 전달된다. 

이 메소드 내에는 -128 부터 127 범위의 Integer pool을 사용하여 전달한 int 가 `캐쉬 범위` 내에 포함되어 있으면 풀에서 Integer 를 꺼내와 반환하며, 캐쉬 범위에 포함되지 않을 경우에는 `새로운 객체를 생성`한다. 

---

### 문제 상황

> 📢 모든 숫자를 더하지 않고 곱하거나 빼고 싶어요.

앞서 FileProcessor와 같이 MultiplyFileProcessor를 구현할 수도 있다. 

```java
public class MultiplyFileProcessor {
    /*생략*/
    public int process() {
        try(BufferedReader reader = new BufferedReader(new FileReader(path))) {
            /*생략*/    
            while((line = reader.readLine()) != null) {
                result *= Integer.parseInt(line); // To-Be
            }
            return result;
        } 
        /*생략*/
    }
}
```

하지만, 대부분의 코드가 기존의 FileProcessor와 중복된다.

템플릿 메소드 패턴을 적용하면 개선이 가능하다.

## 패턴 적용하기

![image-템플릿_메소드_02](https://user-images.githubusercontent.com/42997924/154322599-bd8b839a-3858-453b-92b4-17ba6e6d2838.png)

### 0. 예제 분석과 구조 잡기

- 한 라인씩 읽고, 처리하고, 반환 (3가지 역할)
- 이 중 특정 단계(step)를 하위 클래스에서 재정의하도록 해당 부분을 `추상화`할 수 있도록 **추상 메서드로 분리**해서 템플릿 메서드 내에서 호출해주는 방식으로 변경

ConcreteClass에서는 Abstract Method만 구현하면 된다.

- 상위 클래스 : 전체적인 흐름을 추상화 (템플릿 메소드)
- 하위 클래스 : 구체적인 로직을 구현

> 하위 클래스에서 제공한 코드가 상위 클래스에 정의한 코드의 흐름 그대로 호출되기 때문에 이러한 방식의 구조를 일종의 IoC(Inversion of Control - 제어권 역전)라고 볼 수 있다.

### 1. 템플릿 메소드 정의

FileProcessor의 `process()`가 알고리즘을 담고 있는 템플릿 메소드가 된다.

요구사항에 따른 템플릿에서 변경을 허용하고 싶은 로직 : `result += Integer.parseInt(line);`

이 부분을 어떻게 메소드로 추출할지 고민이 될 때, Intellij의 `Extract Method` 기능을 사용하면 좋다!

```java
// 추상 클래스로 선언
public abstract class FileProcessor { 
    private String path;
    public FileProcessor(String path) {
        this.path = path;
    }
    public final int process() {
        try(BufferedReader reader = new BufferedReader(new FileReader(path))) {
            int result = 0;
            String line = null;
            while((line = reader.readLine()) != null) {
                // 메소드로 추출
                result = getResult(result, Integer.parseInt(line)); 
            }
            return result;
        } catch (IOException e) {
            throw new IllegalArgumentException(path + "에 해당하는 파일이 없습니다.", e);
        }
    }
    // 하위 클래스에서 접근할 수 있어야 하므로 protected
    protected abstract int getResult(int result, int number);
}
```

### 2. 추상 메소드를 구현한 ConcreteClass 구현

구현체에서는 자신이 신경써야하는 로직만 담게 된다.

- Plus

  ```java
  public class Plus extends FileProcessor {
    
  	 public Plus(String path) {
       super(path);
     }
    
      @Override
      public int getResult(int result, int number) {
          return result += number;
      }
  }
  ```

- Multiply

  ```java
  public class Multiply extends FileProcessor {
    
  	 public Plus(String path) {
       super(path);
     }
    
      @Override
      public int getResult(int result, int number) {
          return result *= number;
      }
  }
  ```

- Client

    - Plus와 Multiply를 호출해서 사용하면 된다.

  ```java
  public class Client {
      public static void main(String[] args) {
          // FileProcessor fileProcessor = new Plus("number.txt");
  	      FileProcessor fileProcessor = new Multiply("number.txt");
          int result = fileProcessor.process();
          System.out.println(result);
      }
  }
  ```

## 장점

- 템플릿 코드를 재사용하고 중복 코드를 줄일 수 있다.
- 템플릿 코드를 변경하지 않고 상속을 받아서 구체적인 알고리즘만 변경할 수 있다.
  - OCP, SRP 👍 

## 단점

- 리스코프 치환 원칙(LSP)을 위반할 수 있음
  - 예제에서 만약 FileProcessor를 상속받은 클래스에서 `process()`를 Override해서 예기치 못한 동작을 수행하게 만들 가능성 있음
    - 이런 위험을 없애기 위해서 `process()` 메소드의 접근 제한자를 `final`로 해서 막을 수도 있다.
    - 그러나 process() 내부에 템플릿 메서드에서 구현하고자 하는(예제의 getResult) step에 해당하는 Operation은 특정 의도가 있는 메소드 인데, 상속은 언제든지 그 의도를 깨트릴 수 있는 가능성이 있다.
- 알고리즘 구조가 복잡할 수록 템플릿을 유지하기 어려워진다.
    - 하위 클래스에서 알고리즘을 변경할 수 있도록 허용하는 부분이 많아질수록 템플릿 메소드 코드가 복잡해줄 수 있다.
    
> 리스코프 치환 원칙    
> 상속 구조에서 상위 클래스 타입으로 사용하는 코드에서 (상위 타입이 아닌) 상위 타입을 상속받은 임의의 클래스 중 어떠한 것으로 바꾸더라도 코드가 의도했던 대로 동작해야한다.  
> 한마디로, 상속을 받은 자식 클래스가 부모가 원래 가지고 있던 오퍼레이션 의도를 그대로 수행해야 한다.


## 실무 사용 예

- 자바
    - HttpServlet
- 스프링
    - 템플릿 메소드 패턴
        - Configuration
    - 템플릿 콜백 패턴
        - JdbcTemplate
        - RestTemplate
        - ...

### HttpServlet

- HttpServlet을 상속받아서 필요로 하는 메소드만 Override한다.
- doGet() : GET 요청이 들어왔을 때 해야할 일을 작성
- 이 코드는 코드 스스로가 로직을 실행하는 것이 아니라 `서블릿 컨테이너 엔진`이 서블릿을 (요청이 처음들어오면) 초기화하고, 만들어 놓은 인스턴스를 요청이 들어오면 요청의 메소드(GET, POST..)를 보고, 이 때 해당하는 요청에 따라 doGet, doPost에 정의한 메소드를 실행한다.

```java
public class MyHello extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}
```

> IoC(Inversion of Control)  
> 코드를 제어하는 제어권이 본인 스스로에게 없고 외부(서블릿 컨테이너 엔진)에 있는 것

### Configuration

- 스프링 설정 파일을 만들 때, 보통 특정 클래스를 상속 받아서 만든다. 상속받는 클래스가 제공하는 일부 메소드를 구현한다.
- 이렇게 구현한 부분이 어딘가 특정 객체를 구성할 때 쓰이게 된다.

- 예제

  `WebSecurityConfigurerAdapter`의 `init()`에서 시큐리티 필터에 필요한 구성을 만들게 된다. 필터 체인 중 일부 메소드를 커스터 마이징한 것이다.

  `init()`이라는 템플릿 메소드의 일부 알고리즘을 `configure()`에서 재정의했다고 볼 수 있다.

```java
public class TemplateInSpring {
    @Configuration
    class SecurityConfig extends WebSecurityConfigurerAdapter {
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.authorizeRequests().anyRequest().permitAll();
        }
    }
}
```

### JdbcTemplate (템플릿-콜백 패턴)

- `JdbcTemplate`의 `execute()`를 통해서 쿼리를 실행한다.

```java
public class TemplateInSpring {
    public static void main(String[] args) {
        // JdbcTemplate
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.execute("insert 쿼리");
        // RestTemplate
        RestTemplate restTemplate = new RestTemplate();
        HttpHeaders headers = new HttpHeaders();
        headers.setAccept(Arrays.asList(MediaType.APPLICATION_JSON));
        headers.set("X-COM-PERSIST", "NO");
        headers.set("X-COM-LOCATION", "USA");
        HttpEntity<String> entity = new HttpEntity<String>(headers);
        ResponseEntity<String> responseEntity = restTemplate
                .exchange("http://localhost:8080/users", HttpMethod.GET, entity, String.class);
    }
}
```

- `execute()`를 확인해보면 Callback으로 감싸서 넘겨준다.

![image-템플릿_메소드_03](https://user-images.githubusercontent.com/42997924/154322605-34037d30-b8b1-45b6-9b58-1c77f6139e7f.png)

아래 호출되는 `execute()`를 확인해보면 템플릿 메소드가 정의되어있다.

![image-템플릿_메소드_04](https://user-images.githubusercontent.com/42997924/154322611-1779aa8c-184e-4adf-ab2d-ebbd4c089227.png)

- 상황에 따라 `execute()` , `query()` 등등의 템플릿을 가져다 사용하면 된다.
- 템플릿이 복잡해지면 하나로 관리하기 어려워지기 때문에 이처럼 템플릿을 나누는 것도 하나의 방법이 된다.

### RestTemplate

- RestAPI 클라이언트 코드로서 HTTP 요청을 보내주는 편의성 메소드를 제공해준다.

```java
public class TemplateInSpring {
    public static void main(String[] args) {
        // RestTemplate
        RestTemplate restTemplate = new RestTemplate();
        HttpHeaders headers = new HttpHeaders();
        headers.setAccept(Arrays.asList(MediaType.APPLICATION_JSON));
        headers.set("X-COM-PERSIST", "NO");
        headers.set("X-COM-LOCATION", "USA");
        HttpEntity<String> entity = new HttpEntity<String>(headers);
        ResponseEntity<String> responseEntity = restTemplate
                .exchange("http://localhost:8080/users", HttpMethod.GET, entity, String.class);
    }
}
```

- `exchange()` 확인 → `execute()` 실행 → `doExcute()` 실행

![image-템플릿_메소드_05](https://user-images.githubusercontent.com/42997924/154322619-f25ffe98-14fb-446e-8383-6396164bcdc4.png)

- `doExcute()` 확인

  Request를 만들고, 보내고, 응답 받아서 파상해서 리턴해주고 예외처리해주는 일련의 오퍼레이션이 구현되어있다.

![image-템플릿_메소드_06](https://user-images.githubusercontent.com/42997924/154322620-aaa1b729-cb37-4c38-9c3e-12c4ed40bb1e.png)

---

# 템플릿 콜백(Template-Callback) 패턴

> GoF에서 정의하지 않은 패턴  
> But. 스프링에서 많이 사용되는 패턴

콜백으로 상속 대신 위임을 사용하는 템플릿 패턴

상속 대신 **<u>익명 내부 클래스</u>** 또는 **<u>람다 표현식</u>** 을 활용할 수 있다.

![image-템플릿_메소드_07](https://user-images.githubusercontent.com/42997924/154322595-bf172e36-bbdd-43dd-a9f6-b8880aeacc96.png)

- 콜백이라는 인터페이스를 사용
- 콜백 인터페이스 : 전략 패턴처럼 전략을 제공!

**템플릿 메소드 패턴과의 차이점**

- 템플릿 메소드 패턴 : 계산하는 로직을 추상 메서드로 분리
- 템플릿 콜백 패턴 : 계산 로직을 담고 있는 메소드 하나를 콜백이라는 인터페이스에 담아둔다.

**전략 패턴과 다른 점**

- 전략 패턴은 여러 개의 메소드를 가지고 있을 수 있음
- 콜백은 무조건 하나의 메소드
  - 만약 오퍼레이터가 여러개 필요하다면 인터페이스를 여러개 만들어야 함

## 장점

- 상속을 사용하지 않아도 된다.
- 전략 패턴처럼 위임을 사용할 수 있다.
- 콜백을 구현하는 방법
    1. ConcreteCallback 새로운 구현체를 만든다.
    2. 구현체를 만들지 않고, **<u>익명 내부 클래스</u>** 또는 **<u>람다 표현식</u>** 을 활용할 수 있다.
      - 좀 더 코드가 간결해진다!


## 패턴 적용하기

### 1. 콜백 역할을 하는 인터페이스 정의

- 하나의 추상 메소드를 담고 있다.

```java
public interface Operator {
    abstract int getResult(int result, int number);
}
```

### 2. 콜백 인터페이스를 받아서 사용

- 추상메서드가 없으므로 추상 클래스일 필요가 없어진다.
- 계산 로직을 콜백 함수를 호출을 통해서 수행한다.

```java
// Abstract 선언할 필요 없음
public class FileProcessor {
    private String path;
    public FileProcessor(String path) {
        this.path = path;
    }
    // 콜백을 파라미터로 받음
    public final int process(Operator operator) { 
        try(BufferedReader reader = new BufferedReader(new FileReader(path))) {
            int result = 0;
            String line = null;
            while((line = reader.readLine()) != null) {
                result = operator.getResult(result, Integer.parseInt(line)); 
            }
            return result;
        } catch (IOException e) {
            throw new IllegalArgumentException(path + "에 해당하는 파일이 없습니다.", e);
        }
    }
}
```

> `Plus` 와 `Multiply` 클래스가 필요없어진다.

### 3. Client에서 사용

- 익명 내부 클래스로 구현

```java
public class Client {
    public static void main(String[] args) {
        FileProcessor fileProcessor = new FileProcessor("number.txt");
        int result = fileProcessor.process(new Operator() {
          
          @Override
          public int getResult(int result, int number) {
            return result += number;
          }
        });
        System.out.println(result);
    }
}
```

- 람다로 구현

```java
public class Client {
    public static void main(String[] args) {
        FileProcessor fileProcessor = new FileProcessor("number.txt");
        int result = fileProcessor.process((result1, number) -> result1 += number);
        System.out.println(result);
    }
}
```

- Operator 구현체 정의

  > 만약 Operator 구현체를 다른 곳에서도 사용한다면 이를 클래스로 만들어주면 된다.

```java
public class Plus implements Operator {
    @Override
    public int getResult(int result, int number) {
        return result += number;
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        FileProcessor fileProcessor = new FileProcessor("number.txt");
        int result = fileProcessor.process(new Plus());
        System.out.println(result);
    }
}
```

> 정리  
> 
> 개발을 하다보면 대부분의 로직과 코드는 같은데 중간에 분기 조건이나 log 만 다른 경우를 종종 보곤하는데, 템플릿 메소드 패턴이나 템플릿 콜백 패턴으로 이를 개선해보면 좋을 것 같다.
