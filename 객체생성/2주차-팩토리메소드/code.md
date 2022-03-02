### 예제 상황
회원가입 서비스를 만들고자 할때 회원종류를  일반 회원과 기업회원을 구분하여 회원가입 시키고자 함.

```java
//회원의 경우 컬럼(속성)은 달라질 수 있으나 동일한 행동을 수행하는 부분을 인터페이로 분리
public interface Member {
    public void validateIdentity() throws Exception;                     //회원의 속성 validate
    public void register() throws Exception;            //회원가입 
    public String toString();                           //String으로 직렬화 메서드
}

//기업 회원
public class CompanyMember implements Member{
    private String csn;     //사번
    private String name;    //기업명
    private boolean isPaid; //유료서비스 결제 여부

    public CompanyMember(String csn, String name, boolean isPaid) {
        this.csn = csn;
        this.name = name;
        this.isPaid = isPaid;
    }

    @Override
    public void validateIdentity() throws Exception{
        System.out.println("기업 회원 property validate");
    }

    @Override
    public void register() throws InterruptedException {
        System.out.println("기업 회원 회원가입 로직 수행 ...");
        Thread.sleep(1);                                        //회원가입 로직 수행중임을 가정하기 위해 1초 sleep
        System.out.println("기업 회원 회원가입 완료");
    }

    @Override
    public String toString() {
        return "CompanyMember{" +
                "csn='" + csn + '\'' +
                ", name='" + name + '\'' +
                ", isPaid=" + isPaid +
                '}';
    }
}


//개인 회원
public class PersonalMember implements Member{
    private String name;        //이름
    private String phoneNumber; //전화번호
    private String email;       //이메일

    public PersonalMember(String name, String phoneNumber, String email) {
        this.name = name;
        this.phoneNumber = phoneNumber;
        this.email = email;
    }

    @Override
    public void validateIdentity() throws Exception{
        System.out.println("일반 회원 property validate");
    }

    @Override
    public void register() throws InterruptedException {
        System.out.println("일반 회원 회원가입 로직 수행");
        Thread.sleep(1);                                     //회원가입 로직 수행중임을 가정하기 위해 1초 sleep
        System.out.println("일반 회원 회원가입");
    }

    @Override
    public String toString() {
        return "PersonalMember{" +
                "name='" + name + '\'' +
                ", phoneNumber='" + phoneNumber + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}


//인증 서비스
//Controller에서 직접 사용자를 생성하고 로직을 수행하지 않고 회원가입의 로직을 다른 계층에게 위임할 수 있도록 서비스 계층 추가
public class AuthService {
    //... 로그인, 로그아웃 등 메서드들 생략

    //회원가입 메서드
    public Member signUp(String type, Map<String,Object> userData) throws Exception {
        Member member = null;

        //type을 input으로 회원을 구분하여 생성
        switch (type){
            case "p":
                member = new PersonalMember((String)userData.get("name"),(String)userData.get("phoneNumber"),(String)userData.get("email"));
                break;
            case "c":
                member = new CompanyMember((String)userData.get("csn"),(String)userData.get("name"),false);
                break;
            default:
                throw new Exception();      //지원하지 않은 type이면 exception
        }

        member.validateIdentity();  //회원가입 전 회원 속성 validate 로직 수행
        member.register();          //회원가입 로직 수행
        return member;  
    }
}

//Client
public class Controller {
    //빠른 메서드 실행을 위해 main메서드에서 수행
    public static void main(String[] args) throws Exception {
        AuthService authService = new AuthService();                //Service 생성
        Member member = authService.signUp("p", Map.of("name","홍길동","phoneNumber","010-1111-1111","email","example@example.com"));       //SignUp 수행
        System.out.println(member);    //회원정보 String으로 변환후 출력
    }
}

//Console Print
일반 회원 property validate
일반 회원 회원가입 로직 수행
일반 회원 회원가입
PersonalMember{name='홍길동', phoneNumber='010-1111-1111', email='example@example.com'}
```
위와 같이 회원별로 나누어 개인 회원에게 회원가입등의 행동을 책임지게 해고 Service 계층을두어 switch문으로 특정 회원을 생성하여 로직을 수행함으로써 개발을 진행했다. 

이렇게 개발을 완료했으나 추후에 기능을 추가함에 있어 세가지 `문제점`에 직면하게 되었다.

1. AuthService의 signUp의 메서드와 같이 특정 type을 통해 회원을 생성하는 로직이 다른 메서드에도 필요하다면 해당 switch문을 반복해야 한다는 점.
2. 새로운 회원종류를 추가해야한다면 모든 switch 문의 코드를 수정해야 한다는 점.
3. AuthService가 Member의 구체적인 클래스들과 밀접하게 연결되어 많은 종속성을 가진다는 점


<br>

위 문제를 해결하기 위해 Factory클래스를 만들어 해당 클래스에서 객체 생성 로직을 담당하도록 옮겨보자.

### Simple Factory
```java
//Service내의 객체 생성 로직 부분을 Factory로 분리
public class MemberFactory {

    //Member 생성 메서드
    public Member createMember(String type, Map<String,Object> userData) throws Exception {
        Member member = null;
        switch (type){
            case "p":
                member = new PersonalMember((String)userData.get("name"),(String)userData.get("phoneNumber"),(String)userData.get("email"));
                break;
            case "c":
                member = new CompanyMember((String)userData.get("csn"),(String)userData.get("name"),false);
                break;
            default:
                throw new Exception();
        }

        member.validateIdentity();  //회원가입 전 회원 속성 validate 로직 수행
        return member;
    }
}

//인증 서비스
public class AuthService {
    private MemberFactory memberFactory;     //MemberFactory 생성자 주입

    public AuthService(MemberFactory memberFactory) {
        this.memberFactory = memberFactory;
    }

    public Member signUp(String type, Map<String,Object> userData) throws Exception {
        Member member = memberFactory.createMember(type,userData);      //Factory 통해 Member 생성
        member.register();  
        return member;
    }
}

//컨트롤러
public class Controller {
    public static void main(String[] args) throws Exception {
        AuthService authService = new AuthService(new MemberFactory());     //기존 코드에서 MemberFactory를 주입해주도록 수정
        Member member = authService.signUp("p", Map.of("name","홍길동","phoneNumber","010-1111-1111","email","example@example.com"));
        System.out.println(member);
    }
}
```
객체 생성로직을 Factory로 분리하여 공통 로직을 한곳에 모았기 때문에 여러 메서드를 찾아 돌아다니면서 case를 추가해주어야 하는 문제점은 해결이 되었다. 하지만, 회원의 종류가 추가될때 Factory의 클래스가 변경이 되어야 하는 문제점은 그대로이다. 

이처럼 단순히 Factory를 추가해주는 방법은 UML 다이어그램과 설계 형태가 다르듯이 Factory Method 패턴이 아니라 단순한 Factory 패턴이다. 

이를 Factory Method패턴을 이용해서 리팩토링해보자.

```java
//Product
//기존에 존재하던 Member 인터페이스는 UML 다이어그램의 Product 역할
public interface Member {
    public void validateIdentity();
    public void register() throws Exception;
    public String toString();
}

//Member의 구현체로 Concrete Product 역할
public class PersonalMember implements Member{
    private String name;
    private String phoneNumber;
    private String email;

    public PersonalMember(String name, String phoneNumber, String email) {
        this.name = name;
        this.phoneNumber = phoneNumber;
        this.email = email;
    }

    @Override
    public void validateIdentity() {
        System.out.println("일반 회원 property validate");
    }

    @Override
    public void register() throws InterruptedException {
        System.out.println("일반 회원 회원가입 로직 수행");
        Thread.sleep(1);
        System.out.println("일반 회원 회원가입");
    }

    @Override
    public String toString() {
        return "PersonalMember{" +
                "name='" + name + '\'' +
                ", phoneNumber='" + phoneNumber + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}

//Member의 구현체로 Concrete Product 역할
public class CompanyMember implements Member{
    private String csn;
    private String name;
    private boolean isPaid;

    public CompanyMember(String csn, String name, boolean isPaid) {
        this.csn = csn;
        this.name = name;
        this.isPaid = isPaid;
    }

    @Override
    public void validateIdentity() {
        System.out.println("기업 회원 property validate");
    }

    @Override
    public void register() throws InterruptedException {
        System.out.println("기업 회원 회원가입 로직 수행 ...");
        Thread.sleep(1);
        System.out.println("기업 회원 회원가입 완료");
    }

    @Override
    public String toString() {
        return "CompanyMember{" +
                "csn='" + csn + '\'' +
                ", name='" + name + '\'' +
                ", isPaid=" + isPaid +
                '}';
    }
}

//생성했던 Factory를 수정하여 회원객체 생성은 하위클래스에게 위임하고 validate와 같은 공통 로직을 수행후에 최종적으로 Member를 생성하도록 수정
//UML다이어그램의 Creator역할
public abstract class MemberFactory {
    public Member createMember(Map<String,Object> userData) throws Exception{
        Member member = createFromMap(userData);    //하위 객체에게 생성 위임
        member.validateIdentity();                  //공통 로직 수행
        return member;                              //최종 객체 반환
    }

    protected abstract Member createFromMap(Map<String,Object> userData);   //Factory Method
                                                                            //하위객체에게 이 메서드를 통해 생성위임
}

//Concrete Creator
//일반회원 생성 Factory
public class PersonalMemberFactory extends MemberFactory{
    protected Member createFromMap(Map<String, Object> userData) {
        return new PersonalMember((String)userData.get("name"),(String)userData.get("phoneNumber"),(String)userData.get("email"));
    }
}

//Concrete Creator
//기업회원 생성 Factory
public class CompanyMemberFactory extends MemberFactory{
    protected Member createFromMap(Map<String, Object> userData) {
        return  new CompanyMember((String)userData.get("csn"),(String)userData.get("name"),false);
    }
}

//인증 Service 계층
//상세 객체 생성역할을 Factory에게 위임했으므로 signUp에서 type input 제거
public class AuthService {
    private MemberFactory memberFactory;

    public AuthService(MemberFactory memberFactory) {
        this.memberFactory = memberFactory;
    }

    //기존의 메서드에서 type 매개인자 제거
    public Member signUp(Map<String,Object> userData) throws Exception {
        Member member = memberFactory.createMember(userData);  
        member.register();
        return member;
    }
}

//Client
public class Controller {
    public static void main(String[] args) throws Exception {
        AuthService authService = new AuthService(new PersonalMemberFactory()); //Service를 생성할때 ConcreteFactory를 주입해줌으로써 특정 Member구현체 생성 

        Member member = authService.signUp(Map.of("name","홍길동","phoneNumber","010-1111-1111","email","example@example.com"));
        System.out.println(member);
    }
}
```
이제 새로운 회원종류인 `ManageMentMember`를 추가하고자 한다면 `ManageMentMember` 클래스를 추가해주고 MemberFactory를 상속하여 ManageMentMember를 반환하는 Factory를 추가해주면 기존에 존재하는 `코드의 수정없이` 확장이 가능해졌다.