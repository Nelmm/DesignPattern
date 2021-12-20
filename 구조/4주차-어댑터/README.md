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