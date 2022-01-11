# 플라이웨이트 패턴

객체를 가볍게 만들어 메모리 사용을 줄이는 패턴

공통으로 사용하는 클래스(Flyweight)를 생성하는 팩토리 클래스(FlyweightFactory)를 만들어, 인스턴스를 최초 1개만 생성하고 공유하여 `재사용`할 수 있도록 하는 구조 패턴이다.

자주 변하는 속성(= 외적인 속성, extrinsit)과 변하지 않는 속성(= 내적인 속성, intrinsit)을 분리하고 재사용하여 메모리 사용을 줄인다.

![1](https://user-images.githubusercontent.com/42997924/148320539-34a7d748-3a56-4e52-8152-010a98f6a209.png)

<br><br>

## 1. 구조

- **Flyweight** : 공유에 사용할 클래스 (또는 API)
- **FlyweightFactory** : Flyweight 인스턴스를 생성 또는 공유 (공장 역할)
- **Client** : Flyweight 인스턴스를 필요로 하는 클라이언트

<br><br>

## 2. 적용할 수 있는 상황

-   공통적인 인스턴스를 많이 생성하는 로직이 포함된 경우
-   자주 변하지 않는 속성을 재사용할 수 있는 경우

<br><br>

## 3. 코드로 알아보는 플라이웨이트 패턴 필요한 상황

클라이언트 요구사항은 다음과 같다.

> 1. 편집기에 글을 쓰고 싶어요.  
> 2. 편집기니까 한 글자마다 폰트와 글씨 색, 크기를 바꿀 수 있으면 좋겠어요.

### 3-1. 편집기에 글을 쓰고 싶어요

글을 쓰게 되면 각 글자 하나마다 문자, 색상, 폰트이름, 폰트사이즈 4개의 속성이 필요하다고 도메인을 정해보자.

해당 속성을 갖고 있는 클래스(Character)를 다음과 같이 작성할 수 있다.

**Character 클래스**

글자 하나를 표현한 도메인

```java
public class Character {

    private char value;

    private String color;

    private String fontFamily;

    private int fontSize;

    public Character(char value, String color, String fontFamily, int fontSize) {
        this.value = value;
        this.color = color;
        this.fontFamily = fontFamily;
        this.fontSize = fontSize;
    }
}
```

**Client**

편집기에 글을 입력하는 클라이언트로 글자 하나마다 `Character` 인스턴스를 생성한다.

```java
public class Client {

    public static void main(String[] args) {
        Character c1 = new Character('h', "white", "Nanum", 12);
        Character c2 = new Character('e', "white", "Nanum", 12);
        Character c3 = new Character('l', "white", "Nanum", 12);
        Character c4 = new Character('l', "white", "Nanum", 12);
        Character c5 = new Character('o', "white", "Nanum", 12);
    }
}
```

편집기에 글을 쓰면 한글자 한글자 모두 **new** 생성자를 통해 객체를 생성하게 되는데, 메모리를 아끼기 위해 자주 변하는 속성(공유불가), 자주 변하지 않는 속성(공유)을 **분류**해보자.

1.  **문자**와 **색** : 자주 변하는 속성
    1.  **색**은 ENUM 타입으로 선언해주어도 좋을 것 같다.
2.  **글자 폰트**와 **크기** : 자주 변하지 않는 속성

**Font** 라는 클래스(Flyweight)로 만들어서 개선해보자.

<br>

### 3-2. 플라이웨이트 패턴 적용

Character 인스턴스가 모든 데이터를 따로 저장하지 않고, 공유할 수 있는 부분을 공유해서 메모리를 아껴보자.

자주 변하지 않아서 공유할 수 있는 속성으로 분류했던 **글자 폰트**와 **글자 크기** 속성을 Font 클래스로 선언했다.

**Font**

```java
public final class Font {

    final String family;

    final int size;

    public Font(String family, int size) {
        this.family = family;
        this.size = size;
    }

    public String getFamily() {
        return family;
    }

    public int getSize() {
        return size;
    }
}
```
공유하는 객체이기 때문에 불변으로 선언 **(final)** 하고생성자로만 선언 후 getter로만 접근하도록 수정하자.


**Character**
Font를 field로 갖도록 수정했다.

```java
public class Character {

    private char value;

    private String color;

    private Font font;

    public Character(char value, String color, Font font) {
        this.value = value;
        this.color = color;
        this.font = font;
    }
}
```

**FontFactory**
Font 생성을 관리해주는 팩토리는 다음과 같다.

```java
public class FontFactory {

    private Map<String, Font> cache = new HashMap<>();
    private final String pattern = "^\\w+:\\d+";

    rivate FontFactory() {}

    private static class FontFactoryHolder {
        public static final FontFactory INSTANCE = new FontFactory();
    }

    public static FontFactory getInstance() {
        return FontFactoryHolder.INSTANCE;
    }

    /**
     * 캐싱된 Map에서 Font 객체를 가져옴
     * @param font
     * @return
     */
    public Font getFont(String font) {
        if (!Pattern.matches(this.pattern, font)) {
            throw new IllegalArgumentException();
        }

        if (cache.containsKey(font)) {
            return cache.get(font);
        } else {
            String[] split = font.split(":");
            Font newFont = new Font(split[0], Integer.parseInt(split[1]));
            cache.put(font, newFont);
            return newFont;
        }
    }
}
```
FontFactory의 getFont() 메소드를 통해 Font 생성을 하게 되면, HashMap에 데이터가 있는지 먼저 검사하게 된다.

기존에 저장해둔 Font 인스턴스가 있다면, 새로 생성하지 않고 기존 인스턴스를 반환해준다.

**Client**

```java
    public static void main(String[] args) {
        FontFactory fontFactory = new FontFactory();
        Character c1 = new Character('h', "white", fontFactory.getFont("nanum:12"));
        Character c2 = new Character('e', "white", fontFactory.getFont("nanum:12"));
        Character c3 = new Character('l', "white", fontFactory.getFont("nanum:12"));
    }
```
공유된 Font를 가져올 수 있다.

![2](https://user-images.githubusercontent.com/42997924/148320561-86adab1f-adfc-43fe-8f5f-91b2b123d48f.png)

<br><br>

## 4. 장점과 단점

장점
- 많은 객체를 만들때 성능을 향상시킬수 있다.
- 많은 객체를 만들때 메모리를 줄일수 있다.

단점
- 특정 인스턴스를 다르게 처리하는게 힘들다.
- 코드의 복잡도가 증가합니다.
  - 간단하게 바로 만들어서 사용하지 않고 **캐싱하는 과정의 코드가 필요하기 때문에 복잡도가 증가**합니다.

<br><br>

## 5. 싱글톤 패턴과의 차이점

싱글톤 패턴은 클래스 자체가 오직 1개의 인스턴스만 허용하지만 플라이웨이트 패턴은 싱글톤이 아닌 클래스를 팩토리에서 제어한다.

한마디로, 인스턴스 생성의 제한을 어디서 제어하느냐에 차이.

<br><br>

## 6. Java에서 사용된 플라이웨이트 패턴

도메인이나 로직에서 하나의 객체를 계속 사용될 때 활용되는 경우가 많아서 Java나 Spring 코드에서 많이 찾아볼 수는 없다.

### 6-1. Java의 Integer

-128 ~ 127 까지의 값은 캐싱을 해두고 있기 때문에 == 비교는 이외 데이터에서 다르다고 뜬다.

![3](https://user-images.githubusercontent.com/42997924/148320577-13f4dd57-439a-4dc1-b748-a9fe1310cab3.png)

```java
Integer i1 = Integer.valueOf(12000);
Integer i2 = Integer.valueOf(12000);
System.out.println(i1 == i2); // false
System.out.println(i1.equals(i2)); // true
```