# 팩토리 패턴

# 심플 팩토리 패턴

팩토리 패턴에는 팩토리 메소드 패턴, 추상 팩토리 패턴이있다. 이 두가지 패턴을 알기전에 먼저 심플 팩토리(Simple Factory)패턴에 대해 간략하게 설명하고 가자.

심플 팩토리 패턴은 간단하게 말해서 객체를 생성하는 클래스를 따로 두는 것을 의미한다.

클라이언트에서 휴대폰을 주문하는 코드를 예시로 해보자.

```jsx
//휴대폰을 만드는 공장
public class SimplePhoneFactory {
    public Phone orderPhone(String type) {
        Phone phone = createPhone(type);
        phone.complete();
        return phone;
    }
    private Phone createPhone(String type) {
        return switch (type) {
            case "IPHONE" -> new IPhone();
            case "ANDROID" -> new AndroidPhone();
            default -> null;
        };
    }
}

//휴대폰 인터페이스
public interface Phone {
    void complete();
    void call();
}

//아이폰
public class IPhone implements Phone{
    @Override
    public void complete() {
        System.out.println("아이폰 완성");
    }

    @Override
    public void call() {
        System.out.println("아이폰으로 전화를 한다");
    }
}

//안드로이드 폰
public class AndroidPhone implements Phone{
    @Override
    public void complete() {
        System.out.println("안드로이드폰 완성");
    }

    @Override
    public void call() {
        System.out.println("안드로이드폰으로 전화를 한다");
    }
}

//클라이언트는 휴대폰 공장에 휴대폰을 주문한다.
public class Practice {
    public static void main(String[] args){
        SimplePhoneFactory simplePhoneFactory = new SimplePhoneFactory();
        Phone phone = simplePhoneFactory.orderPhone("ANDROID");
        phone.call();
    }
}

//결과
안드로이드폰 완성
안드로이드폰으로 전화를 한다
```

![Untitled](https://user-images.githubusercontent.com/32676275/145397381-a9d10e26-3d83-49c6-868a-51e64fe897ef.png)

위 코드를 보면 SimplePhoneFactory에서 switch를 통해 클라이언트가 주문할 휴대폰의 객체를 createPhone()를 통해 **직접** 객체를 생성하는 것을 볼 수 있다.

이게 끝이다.

심플 팩토리는 단순히 객체를 만드는 작업을 하나에 팩토리 클래스에 모아두는 것이다.

그렇다면 이제 팩토리 메소드 패턴을 알아보자.

---

# 팩토리 메소드 패턴

팩토리 메소드 패턴의 정의는 다음과 같다.

> 객체를 생성하기 위한 인터페이스를 정의하는 과정에서
어떤 클래스의 인스턴스를 만들지는 **서브클래스에서 결정**
즉, **클래스의 인스턴스를 만드는 일을 서브클래스에게 맡기는 것**
>

다시 말하면 위에서 봤던 Simple Factory Pattern에서 createPhone() 부분에서 Factory에서 직접 객체를 만드는 것을 Factory를 상속한 서브클래스에서 객체를 만들게끔 하는 것이 팩토리 메소드 패턴이라 보면된다. 위의 코드에 팩토리 메소드 패턴을 적용해보자.

```jsx
public interface PhoneFactory {
    default Phone orderPhone() {
        Phone phone = createPhone();
        phone.complete();
        return phone;
    }
    Phone createPhone();
}

public class IPhoneFactory implements PhoneFactory {
    @Override
    public Phone createPhone() {
        return new IPhone();
    }
}

public class AndroidPhoneFactory implements PhoneFactory {
    @Override
    public Phone createPhone() {
        return new AndroidPhone();
    }
}

public interface Phone {
    void complete();
    void call();
}
public class IPhone implements Phone{
    @Override
    public void complete() { System.out.println("아이폰 완성"); }
    @Override
    public void call() { System.out.println("아이폰으로 전화를 한다"); }
}
public class AndroidPhone implements Phone{
    @Override
    public void complete() {System.out.println("안드로이드폰 완성");}
    @Override
    public void call() {System.out.println("안드로이드폰으로 전화를 한다");}
}

public class Practice {
    public static void main(String[] args){
        IPhoneFactory iPhoneFactory = new IPhoneFactory();
        Phone phone7 = iPhoneFactory.orderPhone();
        phone7.call();

        AndroidPhoneFactory androidPhoneFactory = new AndroidPhoneFactory();
        Phone phone2 = androidPhoneFactory.orderPhone();
        phone2.call();
    }
}

//결과
아이폰 완성
아이폰으로 전화를 한다
안드로이드폰 완성
안드로이드폰으로 전화를 한다
```

위 코드를 봤을 때 심플 팩토리 패턴에서 달라진것은 먼저 SimplePhoneFactory를 인터페이스화 한다. 그리고 구현체로 IPhoneFactory, AndroidPhoneFactory를 만들고createPhone()을 각자의 Factory에서 Phone 객체 생성 구현체를 만든다.

이렇게 하면 팩토리 메소드 패턴 정의대로 **서브클래스(IPhoneFactory,AndroidPhoneFactory)에서 어떤 객체를 생성할지 결정할 수 있게 된것이다.**

이렇게 구조를 잡는다면 추후 아이폰, 안드로이드폰의 종류가 여러개로 늘어난다 해도 **Phone 구현체 클래스의 생성과 createPhone()의 분기처리만 해줌으로써 확장이 가능한 구조가 된다.**

즉 **클래스 만들 때 확장은 가능하게 하되, 한번 만들면 추후에 수정할 필요 없게 만들라는 원칙인** OCP (**개방 폐쇄의 원칙 : Open Close Principle)를 따르게 된다.**

다음은 이제 추상 팩토리 패턴에 대해 알아보자.

---

# 추상 팩토리 패턴

> 구체적인 클래스에 의존하지 않고 서로 연관되거나 의존적인 객체들의 조합을 만드는 인터페이스를 제공하는 패턴
>

먼저 위에서 본 메소드 팩토리 패턴에서는 PhoneFactory의 구현체 IPhoneFactory, AndroidPhoneFactory가 각각 IPhone객체 AndroidPhone 객체 하나씩을 생성하게끔 작성했다.

추상 팩토리 패턴은 이것을 한번더 감싸서 하나의 Factory에서 여러개의 제품군(Product)조합을 생성할 수 있게 해주는 패턴이다.

추상 팩토리 패턴 예제를 위해 아이폰에는 **IOS**를 안드로이드폰에는 **Google OS**를 설치하는 요구사항을 넣어보자.

```java
///
public interface PhoneFactoryOfFactory {
    PhoneFactory requestPhone(String company);
}
public class DefaultPhoneFactoryOfFactory implements PhoneFactoryOfFactory{
    @Override
    public PhoneFactory requestPhone(String company) {
        switch (company) {
            case "IPHONE":
                return new IPhoneFactory();
            case "ANDROID":
                return new AndroidPhoneFactory();
        }
        throw new IllegalArgumentException();
    }
}
///

///
public interface PhoneFactory {
    Phone createPhone();
    OS createOS();
}
public class IPhoneFactory implements PhoneFactory{
    @Override
    public Phone createPhone() {
        OS os = createOS();
        os.installOS();
        return new IPhone();
    }
    @Override
    public OS createOS() {
        return new IOS();
    }
}
public class AndroidPhoneFactory implements PhoneFactory{
    @Override
    public Phone createPhone() {
        OS os = createOS();
        os.installOS();
        return new AndroidPhone();
    }
    @Override
    public OS createOS() {
        return new GoogleOS();
    }
}
///

///
public interface OS {
    void installOS();
}
public class IOS implements OS {
    @Override
    public void installOS() {
        System.out.println("IOS 설치");
    }
}
public class GoogleOS implements OS {
    @Override
    public void installOS() {
        System.out.println("구글OS 설치");
    }
}
///

///
public interface Phone {
    public void call();
    public void playGame();
}
public class IPhone implements Phone{
    @Override
    public void call() {
        System.out.println("아이폰으로 전화하다");
    }

    @Override
    public void playGame() {
        System.out.println("아이폰으로 게임하다");
    }
}
public class AndroidPhone implements Phone{
    @Override
    public void call() {
        System.out.println("안드로이드로 전화하다");
    }

    @Override
    public void playGame() {
        System.out.println("안드로이드로 게임하다");
    }
}
///

public class Main {
    public static void main(String[] args) {
        PhoneFactoryOfFactory phoneFactoryOfFactory = new DefaultPhoneFactoryOfFactory();
        PhoneFactory iphoneFactory= phoneFactoryOfFactory.requestPhone("IPHONE");   //아이폰을 산다.
        Phone iphone = iphoneFactory.createPhone();
        iphone.call();
        iphone.playGame();

        PhoneFactory androidPhoneFactory = phoneFactoryOfFactory.requestPhone("ANDROID");   //안드로이드폰을 산다.
        Phone androidPhone = androidPhoneFactory.createPhone();
        androidPhone.call();
        androidPhone.playGame();
    }
}

//결과
IOS 설치
아이폰으로 전화하다
아이폰으로 게임하다
구글OS 설치
안드로이드로 전화하다
안드로이드로 게임하다
```

![Untitled 1](https://user-images.githubusercontent.com/32676275/145397444-b0b14a52-5232-478f-93ce-83f924a20b9a.png)

코드를 보면 PhoneFactory에서 Phone과 OS를 생성할 수 있는데 IPhoneFactory는 IPhone과 IOS조합으로 객체를 생성할 수 있고, AndroidPhoneFactory는 AndroidPhone과 GoogleOS조합으로 객체를 생성할 수 있다.

즉, PhoneFactory는 Phone과 OS를 조합하는 구현체를 제공한다.

그래서 추상 팩토리 패턴의 장점은 다음과 같다.

- 구체적인 클래스를 사용자로부터 분리할 수 있다.
  → 사용자가 사용할 때는 정의된 인터페이스에 정의된 추상 메소드를 사용만 하면 된다.
- 제품군을 쉽게 대체할 수 있다.
  → **내가 만약 IPhone을 생성하지 블랙베리 폰을 생성하고 싶다면 블랙베리 Factory를 구현후 IPhoneFactory를 블랙베리 Factory로 변경만 해주면 된다.**

단점은 다음과 같다.

- 새로운 종류의 제품을 제공하기 어렵다
  → 만약 PhoneFactory에 createBattery()라는 추상 메소드가 추가된다면 PhoneFactory의 모든 서브 구현체를 다시 수정해야한다.

---