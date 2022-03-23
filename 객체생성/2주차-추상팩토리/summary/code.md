## 1. 추상 팩토리 패턴(Abstract Factory Pattern)이란? 

**서로 관련있는 여러 객체를 만들어주는 인터페이스**

구체적으로 어떤 클래스의 인스턴스를(concrete product)를 사용하는지 감출 수 있다.

![추상메소드패턴_01](https://user-images.githubusercontent.com/42997924/157094871-885c6458-f36b-4268-b009-076ef202776e.png)

팩토리를 추상화된 형태(인터페이스, abstract 등)

구체적인 펙토리에서 구체적인 인스턴스를 만드는 것은 팩토리 메소드 패턴과 굉장히 유사하다.

**추상 팩토리 패턴의 목적?**

- `클라이언트 코드(팩토리에서 인스턴스를 생성해서 사용하는 코드)`를 인터페이스 기반으로 활용할 수 있도록 제공
- 즉, **인스턴스를 생성하는 팩토리를 추상화 시켜서 다양하게 활용**한다는 것

> 팩토리 쪽에만 집중하면 팩토리 메소드 패턴과 유사하지만, 클라이언트에서 추상 팩토리를 어떻게 사용하는지에 대한 관점으로 접근해야 목적과 차이점을 이해하기 쉽다.





## 2. 코드로 알아보는 추상 팩토리 패턴

요구사항을 구현한 코드와 이를 개선하며 추상 팩토리 패턴이 필요한 경우를 알아보자.



### 2-1. 여러가지 맞춤 메일 생성 시스템

사람인과 같은 채용사이트에서 사용자 맞춤 추천 메일을 보내는 메일 생성 시스템이 있다고 하자.

메일을 받을 사용자를 추출하고, 메일 내용을 채워서 메일을 발송하는 시스템이다. 주요 흐름을 아래와 같다.

> 1. 발송 대상자 추출  
> 2. 맞춤 메일 내용 생성  
> 3. 메일 발송

맞춤/추천 메일은 여러 케이스로 생성될 수 있다.

- 사용자 검색 데이터 기반 메일 (검색 추천 메일)

- 사용자 정보 기반 메일 (개인 맞춤 메일)

여러가지 케이스의 메일을 생성하더라도 메일 생성 시스템 코드(클라이언트) 변경은 최소화하는 방향으로 구성하고 싶은 상황이다.



### 2-2. 추상화 할 수 있는 기능 찾기 (템플릿 메소드 패턴 적용)

추천 메일 종류가 다른 경우, '발송 대상자 모수' 와 '메일 내용' 이 다를 것이다.

이 중 `메일 내용`을 만들어주는 MailTemplate 객체의 필요한 기능을 정리하면 다음과 같다.

1.  발송자 / 수신자 설정
2.  메일 내용 추가
3.  하단 Footer 추가

기능 중  `2번 메일 내용 추가` 를 제외하면 다른 기능은 동일하다.

먼저 **템플릿 메소드 패턴을 적용**해서 코드의 중복을 줄여보자.

![추상메소드패턴_02](https://user-images.githubusercontent.com/42997924/157094890-222dc7fe-4111-4d1a-8dde-4c0c514cd33f.png)

```java
// 메일 템플릿 생성 추상 클래스
public abstract class MailTemplate {
    public Map<String, String> template = new HashMap<>();

    /**
     * 발신자 Setting
     * @param sender 수신자
     */
    public void setSender(String sender) {
        template.put("sender", sender);
    }

    /**
     * 수신자 Setting
     * @param receiver 발신자
     */
    public void setReceiver(String receiver) {
        template.put("receiver", receiver);
    }

    /**
     * 하단 Footer Setting
     * @param footer 하단 내용
     */
    public void setFooter(String footer) {
        template.put("footer", footer);
    }

    /**
     * 메일 내용 Setting
     * @param content 메일 내용
     */
    protected abstract void setContent(String content);
}

// 개인 검색 데이터 기반
public class SearchMailTemplate extends MailTemplate {
    @Override
    protected void setContent(String content) {
        System.out.println("Set Content For Search Mail");
    }
}

// 경력, 직무 기반
public class AvatarMailTemplate extends MailTemplate {
    @Override
    protected void setContent(String content) {
        System.out.println("Set Content For Avatar Mail");
    }
}
```

#### 2-2-1. 문제점

현재 코드에서 문제점은 메일 생성 시스템의 클라이언트 입장에서, 특정 메일의 객체가 각각 필요하다.

예를 들어 `AvatarMailTemplate, AvatarMailValidUser` 같은 구체적인 객체가 필요하다는 의미이다.

최초에 구성한 디자인과는 다르게 클라이언트 코드 변경이 불가피하기 때문에 확장성이 좋은 코드로 바꿔보자.



### 2-3. 확장성 있는 코드로 변경 (팩토리 메소드 패턴 적용)


![추상메소드패턴_03](https://user-images.githubusercontent.com/42997924/157094892-cd401a7c-51ed-4681-b117-3e6d50cf42e0.png)


문제가 있던 `객체 생성`을 서브 클래스로 분리하고 캡슐화하여 더 확장성 있는 코드로 변경할 수 있다.

```java
// Enum 타입으로 Mail 종류 선언
public enum MailType { AVATAR, SEARCH }

// 메일 템플릿 펙토리
public class MailTemplateFactory {

    /**
     * Mail Type에 따라 Search, Avatar 메일 템플릿 객체 생성
     *
     * @param mailType 메일 종류
     * @return MailTemplate 메일 템플릿
     */
    public static MailTemplate createMailTemplate(MailType mailType) {
        MailTemplate mailTemplate = null;

        switch (mailType) {
            case AVATAR:
                mailTemplate = new AvatarMailTemplate();
                break;
            case SEARCH:
                mailTemplate = new SearchMailTemplate();
                break;
        }
        return mailTemplate;
    }
}

// 클라이언트 호출 코드
public class Client {
    public static void main(String[] args) {
        // 팩토리 메소드 호출
        ValidUser avatarValidUser = ValidUserFactory.createValidUser(MailType.AVATAR);
        MailTemplate avatarMailTemplate = MailTemplateFactory.createMailTemplate(MailType.AVATAR);

        // 대상자 추출
        Set<Long> validUser = avatarValidUser.getValidUser();
        
        // 메일 내용 Setting
        avatarMailTemplate.setContent("신입/경력 기반 개인 맞춤 메일 내용");
        String sender = avatarMailTemplate.template.get("sender");
        String receiver = avatarMailTemplate.template.get("receiver");
        // ..생략
        
        // 대상자에게 메일 템플릿 설정하여 메일 발송 로직 추가
    }
}
```

**ValidUserFactory는 동일**하기 때문에 생략했다.

실행해보면 Avatar 메일 타입으로 수행되는 것을 확인할 수 있다.

```bash
// 실행 결과
Valid User For Avatar Data
Set Content For Avatar Mail
```

#### 2-3-1. 문제점

1. 여전히 클라이언트 코드의 수정이 필요하다.

-   팩토리 메소드를 만들어서 코드 수정이 줄어들었지만, 다른 타입의 Mail을 발송하려면 모두 수정해야한다.

```java
public class Client {
    public static void main(String[] args) {
        // AS-IS
        ValidUser avatarValidUser = ValidUserFactory.createValidUser(MailType.AVATAR);
        
        // TO-BE
        ValidUser searchValidUser = ValidUserFactory.createValidUser(MailType.SEARCH);
        
        // ...
    }
}
```

2. 새로운 타입의 메일이 추가되는 경우

-   커뮤니티 질문 기반의 맞춤 메일 타입이 추가된다면 선언했던 모든 Factory 클래스에 Type을 추가해야한다.

```java
public class MailTemplateFactory {

    public static MailTemplate createMailTemplate(MailType mailType) {
        MailTemplate mailTemplate = null;

        switch (mailType) {
            case AVATAR:
                mailTemplate = new AvatarMailTemplate();
                break;
            case SEARCH:
                mailTemplate = new SearchMailTemplate();
                break;
            // 새로운 타입을 추가
            case POSTING:
                mailTemplate = new PostingMailTemplate();
                break;
        }
        return mailTemplate;
    }
}
```

> 결과적으로 팩토리 메소드 패턴을 이용한 객체 생성은 여러 객체를 일관성 있는 방식으로 생성하는 경우, 많은 코드 변경이 발생한다.



### 2-4. 관련 객체를 일관성 있게 생성 (추상 팩토리 패턴 적용)

관련성이 있는 여러 종류의 객체를 생성할 때 각각 별도의 Factory 클래스를 사용하는 대신, 관련 객체들을 일관성 있게 생성하는 Factory 클래스를  정의해보자.

![추상메소드패턴_04](https://user-images.githubusercontent.com/42997924/157094895-5ad5747b-e697-49fd-b7cd-2d13da7858ac.png)

-    이전에 정의했던 `ValidUserFactory, MailTemplateFactory` 클래스와 같이 `기능 단위`로 Factory 클래스를 정의하지 않고, `SearchMailFactory, AvatarMailFactory` 클래스와 같이 **실제 발송할 메일 타입 단위**로 Factory 클래스를 정의했다.

```java
// 추상 대상자, 추상 메일 템플릿을 생성하는 추상 팩토리 클래스
public abstract class MailFactory {
    public abstract Set<Long> createValidUser();
    public abstract MailTemplate createMailTemplate();
}


// 개인 정보 맞춤 메일 생성 팩토리 클래스
public class AvatarMailFactory extends MailFactory {
    @Override
    public Set<Long> createValidUser() {
        return new AvatarMailValidUser().validUserList;
    }

    @Override
    public MailTemplate createMailTemplate() {
        return new AvatarMailTemplate();
    }
}

// 검색 기반 맞춤 메일 생성 팩토리 클래스
public class SearchMailFactory extends MailFactory {
    @Override
    public Set<Long> createValidUser() {
        return new SearchMailValidUser().validUserList;
    }

    @Override
    public MailTemplate createMailTemplate() {
        return new SearchMailTemplate();
    }
}

public class Client {
    public static void main(String[] args) {
        MailFactory mailFactory;
        // 메일 타입을 런타임에 받음
        String mailType = args[0];

        if (mailType.equals("AVATAR")) {
            mailFactory = new AvatarMailFactory();
        } else {
            mailFactory = new SearchMailFactory();
        }

        // 해당하는 mail 타입에 따른 대상자, 메일 템플릿 생성
        Set<Long> validUser = mailFactory.createValidUser();
        MailTemplate mailTemplate = mailFactory.createMailTemplate();

        // (추가) 발송하는 로직 ...
    }
}
```

-   클라이언트 코드에서 런타임에 인자로 받는 `MailType`에 따라 Mail 생성 팩토리를 생성한다.
    -   다른 메일 타입으로 변경되어도 클라이언트 코드를 변경할 필요가 없다! (첫번째 문제 해결)
-   메일 타입 별 Factory 클래스로 정의하여, 새로운 메일 타입을 기존 코드의 변경 없이 적용할 수 있다.
    -   새로운 메일 타입 Factory를 선언하고 해당하는 대상자추출, 메일템플릿 클래스만 생성하면 확장이 가능하다.

![추상메소드패턴_05](https://user-images.githubusercontent.com/42997924/157094901-5a72faa3-25d9-4a68-af9f-3916ae604fdf.png)

> 추상 팩토리 패턴은 관련성이 있는 여러 객체를 일관적으로 생성하는 다양한 case가 생길 가능성이 있다면, Client와 객체간의 결합을 피할 수 있고, SRP와 OCP를 지킬 수 있어 유용하게 사용될 것 같다.





## 3. 팩토리 메소드 패턴과의 비교

추상 팩토리 패턴은 언뜻보면 팩토리 메소드 패턴과 비슷하다고 느껴질 수 있다. 두 패턴을 간략하게 설명해 보자면 아래와 같다.

- `팩토리 메소드 패턴` :  구체적인 인스턴스를 만드는 과정을 Factory로 감춰서 만드는 방식임. 즉, **구현하는 방법에 초점**을 둠
  - 팩토리를 구현하는 방법 :arrow_right: 구체적인 객체 생성 과정을 하위 또는 구체적 클래스에 옮기는게 목적
- `추상 팩토리 패턴` : **팩토리를 사용하는 방법에 초점**을 둠
  - 여러 객체를 구체적인 클래스에 의존적이지 않게 해주는게 목적



### 구체적인 비교

1. Factory 클래스에서 객체에 대한 생성을 지원하는 범위
   - `팩토리 메소드 패턴`
     - 한 팩토리당 한 종류 ( create 메서드가 Factory 클래스에 1개) :arrow_right: 어떤 객체를 만들지만 결정하면 됨
     - 한 개의 메서드로 여러 개의 객체를 만듦
   - `추상 팩토리 패턴`
     - 한 팩토리에서 서로 연관된 여러 종류 모두 지원( `create...()` 메서드가 팩토리 클래스에 여러 개 존재)
       - createA :arrow_right: A를 만듦, createB :arrow_right: B를 만듦 (선택지 자체에 구체적으로 만들 대상이 있음, 선택지 자체를 만들어서 줌)
     - 구상 클래스에 의존하지 않고 여러 개의 관련된 객체를 하나의 팩토리로 묶음
2. 팩토리 메서드에서 만드는 객체의 종류
   - `팩토리 메소드 패턴`의 팩토리 메소드
     - **인자에 따라 객체의 종류**가 결정 :arrow_right: 인자 A :arrow_right: A클래스 인스턴스 생성
   - `추상 팩토리 패턴 패턴`의 팩토리 메서드
     - **인자에 따라 관련된 객체들을 생성하는 팩토리의 종류**가 결정됨 :arrow_right: 인자 A :arrow_right: A Factory를 골라서 만듦
3. 결합도를 낮추는 대상
   - `팩토리 메서드 패턴`
     - **ConcreteProduct와 Client 간의 결합도를 낮출 때** 사용 :arrow_right: 구체적인 객체를 Client가 몰라도 됨
   - `추상 팩토리 패턴`
     - **ConcreteFactory와 Client간의 결합도를 낮출 때** 사용 :arrow_right: 특정한 대상에 관련된 것들을 하나의 팩토리로 묶어서 처리하는데, 특정한 대상에 관련된 해당 팩토리를 Client에서 몰라도 됨
4. 포커스
   - `팩토리 메서드 패턴`
     - **메서드(Factory Method) 레벨**에서 포커스를 맞춤
     - **클라이언트의 ConcreteProduct 인스턴스 생성 및 구성에 대한 책임**을 덜어줌
   - `추상 팩토리 패턴`
     - **클래스(Abstract Factory) 레벨**에서 포커스를 맞춤
     - **각 Product들이 다른 클래스와 함께 사용될 때의 제약사항**을 강제할 수 있다.
     - 단, **새로운 ConcreteFactory를 추가할 때 많은 작업**이 필요하다.
5. 메서드와 오브젝트
   - `팩토리 메서드 패턴`은 **단일 Method**이다
   - `추상 팩토리 패턴`은 **Object**이다. (**팩토리 오브젝트** 생성)
6. Inhritance(상속), Composition(구성)
   - `팩토리 메서드 패턴`
     - **상속**을 사용하여 객체의 **인스턴스 생성에 대해서는 서브클래스에 의존**
     - 지역 레퍼런스 없이 바로 **하위 클래스의 함수를 호출하여 객체를 만듦**
   - `추상 팩토리 패턴`
     - 지역 레퍼런스를 두어 , **외부로부터 Factory 객체를 DI 받아서 위임.**



---

**Reference**

-   [인프런 : 코딩으로 학습하는 GoF의 디자인 패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)
-   [heejeong Kwon :  추상 팩토리 패턴이란](https://gmlwjd9405.github.io/2018/08/08/abstract-factory-pattern.html)

- [https://dev-youngjun.tistory.com/230](https://dev-youngjun.tistory.com/230)