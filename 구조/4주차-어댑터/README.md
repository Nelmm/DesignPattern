# 어댑터 패턴

> 기존 코드를 클라이언트가 사용하는 인터페이스의 구현체로 박꿔주는 패턴
> 

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/473cadc8-bad2-4f31-9c41-2abe7c74a5c2/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211220%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211220T054742Z&X-Amz-Expires=86400&X-Amz-Signature=1526f4d3d36dadfe051bffa55110f510e30264a87f5d4853b134ed65eed4eba0&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

- `Client`는 Target Interface를 사용
- `Adaptee`를 사용하기 위해 `Target`과 `Adaptee`를 연결할 `Adapter`사용

어댑터 패턴의 핵심은 `adaptee`를 `adapter`를 통해 `target`으로 변환시켜주는 것이다.

> 예제1
> 

```java
public class CType {
    public void getName() {
        System.out.println("C타입 포트");
    }
}
public class EightPin {
    public void getPortName() {
        System.out.println("8핀 포트");
    }
}
public class adapter extends CType {
    private EightPin eightPin;
    public adapter(EightPin eightPin) {
        this.eightPin = eightPin;
    }
    @Override
    public void getName() {
        eightPin.getPortName();
    }
}
public class Main {
    public static void main(String[] args) {
        CType port = new CType();
        CType port2 = new adapter(new EightPin());

        client(port);
        client(port2);
    }
		//클라이언트 코드는 변경되지 않는다.
    public static void client(CType port) {
        port.getName();
    }
}

//결과
C타입 포트
8핀 포트
```

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/fe5ee95b-1f25-4d71-a98a-fdf32c2cb027/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20211220%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211220T054809Z&X-Amz-Expires=86400&X-Amz-Signature=f9960b940637964434ccde3a973017736ea328b2af29a86530244825caa94d56&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

- CType: `Target`
- Adapter: `Adapter`
- EightPin: `Adaptee`

위의 코드를 보면  `client(CType port)` 는 CType을 인자로 받는다. 여기서 만약 8핀으로 수정하고 싶으면 `client()`를 수정해야한다. 

<aside>
👉 그럼 왜 `Adapter`를 사용할까?

현재 `EightPin(Adaptee)`는 직접 작성한 클래스라 **수정이 가능**하지만 만약 라이브러리로 제공되는 클래스라면 **수정이 불가능**할 것이다.
그렇다면 `CType`클래스의 코드를 변경할 수도 있지만 이 경우에도 기존에 `CType`을 많은 곳에서 사용할 수록 수정 작업이 쉽지 않을뿐 아니라 오류 발생율도 증가하게 될 것이다.

</aside>

> 예제2
> 

다음 예제는 `Spring Security`에서 제공하는 `UserDetails`에 관한 코드이다. 

- **UserDetails**
    - username과 password 정보를 알아낼 수 있는 인터페이스
    - `Target`에 해당

```java
public interface UserDetails {
    String getUsername();
    String getPassword();
}
```

- **UserDetailsService**
    - username에 해당하는 UserDetails 유저 정보를 읽어들이는 인터페이스
    - `Target`에 해당

```java
public interface UserDetailsService {
    UserDetails loadUser(String username);
}
```

- **LoginHandler**
    - UserDetails와 UserDetailsService로 로그인을 처리하는 핸들러
    - `Client`에 해당

```java
public class LoginHandler {

    UserDetailsService userDetailsService;

    public LoginHandler(UserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
    }

    public String login(String username, String password) {
        UserDetails userDetails = userDetailsService.loadUser(username);
        if (userDetails.getPassword().equals(password)) {
            return userDetails.getUsername();
        } else {
            throw new IllegalArgumentException();
        }
    }
}
```

**Account**

- 애플리케이션마다 (각 애플리케이션에 맞게) 만드는 일반적인 Account
- security 패키지에서 제공하는 클래스와 다르게 해당 애플리케이션에서만 사용하는 용도의 클래스이다.
- `Adaptee`에 해당

```java
public class Account {
    private String name;
    private String password;
    private String email;
  
  	// getter, setter ...
}
```

**AccountService**

- 애플리케이션마다 (각 애플리케이션에 맞게) 만드는 일반적인 AccountService
- security 패키지에서 제공하는 클래스와 다르게 해당 애플리케이션에서만 사용하는 용도의 클래스이다.
- `Adaptee`에 해당

```java
public class AccountService {

    public Account findAccountByUsername(String username) {
        Account account = new Account();
        account.setName(username);
        account.setPassword(username);
        account.setEmail(username);
        return account;
    }

    public void createNewAccount(Account account) {

    }

    public void updateAccount(Account account) {

    }

}
```

`Client` 코드에 해당하는 로그인 기능을 처리해주는 `LoginHandler`는 `UserDatils`와 `UserDetailsService`라는 정해진 규격의 인터페이스를 사용하고 있다. (`Target`에 해당)

우리 애플리케이션의 `Account`와 `AccountService`는 `Adaptee`에 해당한다.

여기에 중간 **어댑터**를 만들어서 현재 `security` 내의 클래스와 상호호환되지 않는 이 두 클래스를 호환시킨 코드는 다음과 같이 만들 수 있다.

### **AccountUserDetailsService**

Adaptee를 사용해서 Target 인터페이스 규약에 맞도록 구현해준다.

1. UserDetailsService 인터페이스를 implements
2. Adaptee에 해당하는 AccountService를 필드로 가지고 사용
3. loadUser()를 Override할 때, AccountService를 사용
4. 이 때, `AccountService`는 `UserDetails`와 상관없는 `Account`를 넘겨주기 때문에 이를 다시 `UserDetails`로 변환해주는 어댑터가 필요

```java
public class AccountUserDetailsService implements UserDetailsService {

    private AccountService accountService;

    public AccountUserDetailsService(AccountService accountService) {
        this.accountService = accountService;
    }

    @Override
    public UserDetails loadUser(String username) {
        return new AccountUserDetails(accountService.findAccountByUsername(username));
    }
}
```

### **AccountUserDetails**

1. UserDetails라는 Target을 Adaptee에 해당하는 Account를 사용해서 구현

```java
public class AccountUserDetails implements UserDetails {

    private Account account;

    public AccountUserDetails(Account account) {
        this.account = account;
    }

    @Override
    public String getUsername() {
        return account.getName();
    }

    @Override
    public String getPassword() {
        return account.getPassword();
    }
}
```

```java
public class App {

    public static void main(String[] args) {
        AccountService accountService = new AccountService();
        UserDetailsService userDetailsService = new AccountUserDetailsService(accountService);
        LoginHandler loginHandler = new LoginHandler(userDetailsService);
        String login = loginHandler.login("solar", "solar");
        System.out.println(login); //solar
    }
}
```

`예제1` 에서와 같이 `예제2` 코드를 보면 기존의 

`UserDetailService`는 `AccountUserDetailService(Adapter)`를 통해 `AccountService(Adaptee)`로 대체됐다.

또한 `AccountUserDetails`(`UserDetails` 의 `Adapter`)를 통해 `UserDetails`대신 `Account(adaptee)`로 대체된 것을 확인할 수 있다.

---

> 장/단점
> 

### 장점

- 기존 코드(Adaptee)를 변경하지 않고 원하는 인터페이스(Target) 구현체를 만들어 재사용할 수 있다.
    - 기존 코드를 변경하지 않고, 확장할 수 있다는 점에서 **OCP(Open Closed Principle)** 원칙에 가까운 패턴이다.
- 기존 코드가 하던 일과 특정 인터페이스 구현체로 변환하는 작업을 각기 다른 클래스로 분리하여 관리할 수 있다.
    - 각각 하던 일에 집중할 수 있기 때문에 **SRP(Single Responsibility Principle)** 원칙에 가까운 패턴이다

### 단점

- 클래스가 많아지고, 구조가 복잡해진다.

---

> 실무 예제
> 

### java.util.Arrays#asList(T...)

- 배열을 리스트로 변환해준다.
- 배열(`Target`) → (`Adapter`)→ 리스트(`Adaptee`)
- `T...` : 가변인자 - 내부적으로는 배열로 넘겨받게 된다.

### **HandlerAdapter**

- 핸들러 : 요청을 처리하고 응답을 반환

우리가 작성하는 다양한 형태의 핸들러 코드를 스프링 MVC가 실행할 수 있는 형태로 변환해주는 어댑터용 인터페이스.

```java
public class AdapterInSpring {
    public static void main(String[] args) {
        DispatcherServlet dispatcherServlet = new DispatcherServlet();
        HandlerAdapter handlerAdapter = new RequestMappingHandlerAdapter();
    }
}
```

- 가장 많이 사용하는 형태의 핸들러

```java
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "hi";
    }
}
```

- `doDispatch()` 코드 일부

```java
// 해당 핸들러를 처리할 수 있는 HandlerAdapter를 찾아온다.
// Determine handler adapter for the current request.
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

// ..(생략)

// 핸들러를 찾아오면 요청을 처리한다. 처리결과로 model and view를 반환한다.
// Actually invoke the handler.
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

- `getHandlerAdapter()`

핸들러는 다양한 형태이기 때문에 `Object` 타입으로 받아온다.

핸들러를 처리할 수 있는 `HandlerAdapter`를 찾아서 반환한다.

```java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
   if (this.handlerAdapters != null) {
      for (HandlerAdapter adapter : this.handlerAdapters) {
         if (adapter.supports(handler)) {
            return adapter;
         }
      }
   }
   throw new ServletException("No adapter for handler [" + handler +
         "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
}
```

어떤 핸들러를 사용하느냐에 따라 각기 다른 핸들러 어댑터를 사용하게 된다.

핸들러 어댑터는 간단한 인터페이스만 구현해주면 된다.

```java
public interface HandlerAdapter {

   /**    * Given a handler instance, return whether or not this {@code HandlerAdapter}
    * can support it. Typical HandlerAdapters will base the decision on the handler
    * type. HandlerAdapters will usually only support one handler type each.
    * <p>A typical implementation:
    * <p>{@code    * return (handler instanceof MyHandler);    * }
    * @param handler the handler object to check
    * @return whether or not this object can use the given handler
    */boolean supports(Object handler);

   /**    * Use the given handler to handle this request.
    * The workflow that is required may vary widely.
    * @param request current HTTP request
    * @param response current HTTP response
    * @param handler the handler to use. This object must have previously been passed
    * to the {@code supports} method of this interface, which must have
    * returned {@code true}.
    * @throws Exception in case of errors
    * @return a ModelAndView object with the name of the view and the required
    * model data, or {@code null} if the request has been handled directly
    */@Nullable
   ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

   /**    * Same contract as for HttpServlet's {@code getLastModified} method.
    * Can simply return -1 if there's no support in the handler class.
    * @param request current HTTP request
    * @param handler the handler to use
    * @return the lastModified value for the given handler
    * @see javax.servlet.http.HttpServlet#getLastModified
    * @see org.springframework.web.servlet.mvc.LastModified#getLastModified
    */long getLastModified(HttpServletRequest request, Object handler);
```

핸들러 어댑터는 요청을 처리하는 방법을 구현해주면 된다.

그 중 가장 많이 사용하는 것이 `RequestMappingHandlerAdapter`이다.

원한다면 직접만들어서 구현할 수 있다.

스프링은 `Adapter`에 해당하는 인터페이스를 제공해주는 것이다.

- `HttpServletRequest`와 `HttpServletResponse`를 받아서 `ModelAndView`를 반환해주는 어댑터에 대한 인터페이스를 정의한 것이 `HandlerAdapter`이다.

> 왜 이런 어댑터 인터페이스가 필요했을까?
> 

다양한 형태의 핸들러가 있고, 각기 다른 형태에 따라 각각 다르게 처리해야하기 때문에, 다르게 처리해야하는 모든 핸들러가 스프링MVC에 들어있고, 다양한 형태의 핸들러를 다 지원할 수 있게(확장에 열려있게) 해주기 위해 스프링 MVC가 고안해놓은 인터페이스이다.

> 어댑터 패턴을 이해한 후에 SpringMVC의 `DispatcherServlet` 구현 코드를 읽어보고 `doDispatch`의 동작 원리를 공부해보면 좋을 것 같다!
> 

> 레거시 시스템을 원하는 인터페이스로 사용 가능하게 할 수 있고, 어댑터 객체에서 적절히 구현 후 적용한다면 단순한 `wrapping` 이상의 효과를 볼 수 있을 것 같다.
> 

### 추가로 설명하자면

선언한 컨트롤러 클래스내의 메서드를 실행하여 응답을 하는데 이때 반환 값이 `String`/`ModelAndView` 로 다양한 결과값을 반환할 수 있습니다. 하지만 `Servlet`에서는 무조건 `ModelAndView`를 받아서 이를 사용하도록 구현되었기 때문에 두 사이를 연결하기 위해 `HandlerAdpater`가 필요합니다.

대표적인 `HandlerAdpater`로 `RequestMappingHandlerAdpater` / `HttpRequestHadlerAdapter` / `SimpleControllerAdapter`가 존재고 이외에도 `HandlerFucntionAdapter`, `SimpleServletHandlerAdpater`가 존재합니다.

### 1. **RequestMappingHandlerAdpater**

첫번째로 **`RequestMappingHandlerAdpater`**는 `AbstractHandlerMethodAdpater`를 상속한 구현체로 `@Annotation`방식의 핸들러에 사용이 됩니다.

```java
protected ModelAndView handleInternal(HttpServletRequest request, HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {
    this.checkRequest(request);
    ModelAndView mav;
    if (this.synchronizeOnSession) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            Object mutex = WebUtils.getSessionMutex(session);
            synchronized(mutex) {
                mav = this.invokeHandlerMethod(request, response, handlerMethod);
            }
        } else {
            mav = this.invokeHandlerMethod(request, response, handlerMethod);
        }
    } else {
        mav = this.invokeHandlerMethod(request, response, handlerMethod);
    }

    if (!response.containsHeader("Cache-Control")) {
        if (this.getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
                this.applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
        } else {
            this.prepareResponse(response);
        }
    }

    return mav;
}

@Nullable
protected ModelAndView invokeHandlerMethod(HttpServletRequest request, HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {
    ServletWebRequest webRequest = new ServletWebRequest(request, response);

    ModelAndView var15;
    try {
        //생략...

        var15 = this.getModelAndView(mavContainer, modelFactory, webRequest);
    } finally {
        webRequest.requestCompleted();
    }

    return var15;
}
```

`invokeHandlerMethod()`를 통해 컨트롤러의 메서드를 실행하고 `getModelAndView()`메서드를 통해 `ModelAndView`로 바꿔 반환합니다.

추가적으로 핸들러에 대해 설명하자면 아래에 설명할 다른 핸들러는 하나의 컨트롤러가 하나의 `url`에 만 맵핑되지만, 이 방식은 하나의 컨트롤러에서 여러 `url`과 맵핑될 수 있습니다.

### 2. HttpRequestHandlerAdapter

두번째로 `HttpRequestHandlerAdapter`는 `HttpRequestHandler`라는 인터페이스를 구현한 컨트롤러(핸들러)를 위한 `Adapter`로 `Model/view`를 사용하지 않는 `Http`기반의 `low level`서비스를 개발할때 사용할 수 있습니다.

```java
@Nullable
public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    ((HttpRequestHandler)handler).handleRequest(request, response);
    return null;
}
```

별도의 `Model/View`를 사용하지 않기 때문에 `null`을 반환하는 것을 볼 수 있습니다.

### 3. **SimpleControllerHandlerAdpter**

`Controller`라는 인터페이스를 구현한 컨트롤러(핸들러)를 위한 `Adapter`로 어노테이션방식의 컨트롤러가 등장하기 전의 가장 기본적인 컨트롤러 타입입니다.

```java
@Nullable
public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    return ((Controller)handler).handleRequest(request, response);
}
```

`Controller` 인터페이스의 `handleRequest()`가 `ModelAndView`를 반환하기에 그대로 실행하여 반환하는 것을 볼 수 있습니다.