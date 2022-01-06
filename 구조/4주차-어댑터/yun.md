# 어댑터(Adapter) 패턴

어댑터 패턴은 기존 코드를 클라이언트가 사용하는 인터페이스의 구현체로 바꿔주는 패턴을 말합니다.



## 1. 정의

일반적으로 어댑터 패턴은 110V 콘센트와 220V콘센트를 변환해주는 것을 예로 많이 드는데,

프로그래밍적으로 얘기하자면 클라이언트가 사용하는 인터페이스가 기존 코드와 다를 때, 기존 코드를 클라이언트 코드와 호환될 수 있게 처리해 주는 것으로 생각하면 됩니다.



## 2. 예시

![adapter](https://user-images.githubusercontent.com/79291114/145826232-fe549766-c31e-4a31-a009-65a30cef8c1d.png)

그림을 보게되면, 클라이언트는 `Target` 인터페이스를 사용하고 있습니다. 이 때, 기존의 코드는 `Adaptee`에 해당하게 되는데, 이 Adaptee를 `Adapter`를 이용해 `Target` 인터페이스로 매핑시켜 줍니다.



### 2.1 코드

#### 클라이언트 코드

```java
// 유저 상세 정보
public interface UserDetails {

    String getUsername();

    String getPassword();

}

// 유저 상세 서비스
public interface UserDetailsService {

    UserDetails loadUser(String username);

}

// 로그인 핸들러
public class LoginHandler {

    UserDetailsService userDetailsService;

    public LoginHandler(UserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
    }

    // login(password 일치 판단)
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

`LoginHandler`의 `login` 메서드는 파라미터로 받은 username을 `UserDetailsService`를 이용해 `UserDetails`를 불러와 UserDetails의 **password와 login 메서드로 받은 password가 일치하는지 판단**하는 메서드 입니다.



#### 기존 코드

```java
// 계정 정보
public class Account {

    private String name;

    private String password;

    private String email;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

}

// 계정 서비스
public class AccountService {

    // username으로 계정 정보 찾기
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

기존의 계정 관련 코드 입니다. 보시다시피 클라이언트 중복되는 부분이 있긴 하지만 같다고 할 수는 없습니다.

이 기존 코드를 어떻게 클라이언트 코드와 호환되게 만들 수 있을까요?? (*hint : 인터페이스 상속을 이용*)



#### 어댑터 패턴 적용

```java
// 로그인 핸들러
public class LoginHandler {

    UserDetailsService userDetailsService;

    public LoginHandler(UserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
    }

    // login(password 일치 판단)
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

아까 위에서 `LoginHandler`는 `UserDetailsService`와 `UserDetails`을 사용하고 있는데, 이 두 Class는 인터페이스로 되어 있기 때문에 **해당 인터페이스들을 상속을 이용해 클라이언트 코드와 기존 코드를 호환되게 만들 수 있습니다.**

---

```java
// UserDetails을 상속받음
public class  AccountUserDetails implements UserDetails {

    // 계정 정보를 가지고 있음
    private Account account;

    public AccountUserDetails(Account account) {
        this.account = account;
    }

    //Override한 메서드에서 알맞는 계정 정보를 반환
    @Override
    public String getUsername() {
        return account.getName();
    }

    @Override
    public String getPassword() {
        return account.getPassword();
    }
}

// UserDetailsService를 상속받음
public class AccountUserDetailsService implements UserDetailsService {

    // 계정 서비스를 가지고 있음
    private AccountService accountService;

    public AccountUserDetailsService(AccountService accountService) {
        this.accountService = accountService;
    }

    //Override한 메서드에서 알맞는 계정 서비스 메서드를 가져와 반환
    // AccountUserDetails가 UserDetails를 상속받기 때문에 이런 식으로 사용 가능
    @Override
    public UserDetails loadUser(String username) {
        return new AccountUserDetails(accountService.findAccountByUsername(username));
    }
}
```

```java
public static void main(String[] args) {
        AccountService accountService = new AccountService();
        UserDetailsService userDetailsService = new AccountUserDetailsService(accountService);
        LoginHandler loginHandler = new LoginHandler(userDetailsService);
        String login = loginHandler.login("keesun", "keesun");
        System.out.println(login);
    }
```

이렇게 새로운 클래스를 만들어 인터페이스 상속을 이용하면 어댑터 패턴으로 기존의 코드를 수정하지 않고도 서로 다른 두 객체를 호환되게 만들 수 있습니다.

> 만약 기존 코드를 수정할 수 있다면, `Account`에 바로 `UserDetails`를 상속해주고 `AccountService`에 `UserDetailsService`을 상속해서 복잡도를 줄여 편하게 사용할 수 있지만
>
> `SRP(Single Responsibility Principle)` 즉, 단일 책임 원칙에서 보자면 **User 쪽에서는 User와 관련된 로직이 들어갈 수 있기 때문**에 Class를 나누는게 조금 더 객체지향에 알맞는 프로그래밍이라고 할 수 있습니다. (하지만 실용적인 선택을 해야하는 경우도 있기 때문에 잘 판단해야 합니다.)





## 3. 장점과 단점

### 장점

- **기존 코드를 변경하지 않고 원하는 인터페이스 구현체를 만들어 재사용**할 수 있습니다.
  - 기존의 `UserDetails, UserDetailsService`를 변경하지 않고 호환될 수 있게 만들어주었기 때문에 `개방 폐쇄 원칙(OCP, Open-Closed Principle)` 을 지키면서 프로그래밍할 수 있습니다.
- 기존 코드가 하던 일과 **특정 인터페이스 구현체로 변환하는 작업을 각기 다른 클래스로 분리하여 관리**할 수 있습니다.
  - 변환하는 작업 자체를 `AccountUserDetailsService`로 분리해서 하기 때문에 `단일 책임 원칙(SRP, Single Responsibility Principle)` 을 지키면서 프로그래밍할 수 있습니다.



### 단점

- 다른 디자인 패턴들과 마찬가지로 새 클래스가 생겨 **복잡도가 증가**할 수 있습니다.
  - 때문에 경우에 따라서 기존 코드가 해당 인터페이스를 구현하도록 수정하는 것이 좋은 선택이 될 수도 있습니다.





## 4. 자바와 스프링에서의 어댑터 패턴

우리가 사용하는 자바, 스프링에서 사용되는 어댑터 패턴을 알아보겠습니다.



### 자바

```java
// collections
List<String> strings = Arrays.asList("a", "b", "c"); // 배열을 리스트로 변환
Enumeration<String> enumeration = Collections.enumeration(strings); // Collections(여기에서는 List)를 Enumeration으로 변환
ArrayList<String> list = Collections.list(enumeration); // Enumeration을 list로 변환

// io
try(InputStream is = new FileInputStream("input.txt"); // 파일로 부터 바이트를 입력 받음
    InputStreamReader isr = new InputStreamReader(is); // 바이트를 문자로 변환
    BufferedReader reader = new BufferedReader(isr)) { // 문자를 버퍼링
    while(reader.ready()) {
        System.out.println(reader.readLine());
    }
} catch (IOException e) {
    throw new RuntimeException(e);
}
```

> **Enumeration** : Collection에서 자주 쓰이며 배열에서 반복문을 이용하여 데이터를 출력하는 것과 같이 반복문을 통해 데이터를 한 번에 출력할 수 있도록 도와주는 클래스, `Iterator`도 비슷한 기능을 하는데, 가능하면 `Iterator` 사용을 권장
>
> **FileInputStream** : `InputStream`을 상속받았으며, 파일로 부터 바이트로 입력받아, 바이트 단위로 출력할 수 있는 클래스
>
> **InputStreamReader** : 바이트 스트림을 문자 스트림으로 변환 해주는 클래스, 바이트를 읽고 지정된 문자 집합을 사용하여 문자로 디코딩
>
> **BufferedReader** : 문자 입력 스트림에서 텍스트를 읽고 문자, 배열 및 행을 효율적으로 읽을 수 있도록 문자를 버퍼링하는 클래스

생성자나 파라미터로 받은 타입을 **다른 타입으로 반환해주거나 다른 방식으로 사용할 수 있게 해주기 때문**에 `어댑터 패턴`이 적용됐다고 볼 수 있습니다.

**참고 링크**

- [Enumeration과 Iterator의 차이](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=seektruthyb&logNo=150114747262)



### 스프링

스프링에서는 스프링 MVC에서 사용하는 `HandlerAdapter`가 어댑터 패턴을 사용하고 있다고 볼 수 있습니다. `HandlerAdapter`가 다양한 핸들러를 받아 `ModelAndView`를 반환하기 때문입니다. 즉, 스프링에서 아래와 같이 어댑터에 해당하는 인터페이스를 제공해 주는 것 입니다.

```java
public interface HandlerAdapter {
    boolean supports(Object var1);

    @Nullable
    // Object에 다양한 핸들러가 올 수 있음
    ModelAndView handle(HttpServletRequest var1, HttpServletResponse var2, Object var3) throws Exception;

    long getLastModified(HttpServletRequest var1, Object var2);
}
```



---

참고 : [코딩으로 학습하는 GoF의 디자인 패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)
