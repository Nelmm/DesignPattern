# 플라이웨이트(Flyweight) 패턴

플라이웨이트는 복싱에서의 플라이웨이트 체급(제일 가벼운 체급)에서 이름을 가져왔습니다. 플라이웨이트 패턴은 **많은 인스턴스를 생성하는 애플리케이션에서 사용할 수 있는 패턴** 중 하나입니다.



## 1. 정의

플라이웨이트 패턴은 **자주 변하는 객체와 자주 변하지 않는 객체를 분리해서 자주 변하지 않는 객체를 재사용하므로써 최적화에 도움을 주는 패턴**입니다.





## 2. 예시

색깔과 폰트를 가지고 있는 문자 객체를 만들어 보겠습니다.



### 2-1. 플라이웨이트패턴 적용 전 코드

플라이웨이트 패턴 적용 전 코드는 아래와 같습니다.



#### Character 클래스

문자 값과 색깔, 폰트 이름/사이즈를 가지는 문자 객체입니다.

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



#### 클라이언트

코드를 실행 시킬 수 있는 클라이언트 코드 입니다. 똑같은 색깔, 폰트, 사이즈로 문자를 만듭니다.

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



### 2-2. 플라이웨이트패턴 적용 후 코드

플라이웨이트 패턴은 `자주 변하는 속성(외적인 속성, extrinsit)`과 `변하지 않는 속성(내적인 속성, intrinsit)`을 분리하고 재사용하여 메모리 사용을 줄일 수 있습니다.

위의 코드에서 색깔, 폰트, 폰트사이즈가 고정되어 있습니다. **색깔은 변할 수 있는 가능성이 다분하지만 폰트와 폰트사이즈는 대부분의 애플리케이션에서 통일시키기 때문에 변하지 않는 속성이라고 볼 수 있습니다.**

그렇기 때문에 아래의 그림에서 `Flyweight`는 폰트 이름과 폰트 사이즈를 가지는 `Font`객체가 될 것이며, 이 **Font 객체를 재사용할 수 있게 가지고 있는(캐싱) FlyweightFactory가 필요**합니다.

![Flyweight](https://user-images.githubusercontent.com/79291114/147515638-353e930d-713d-46db-a1c6-c7b348c9aa70.PNG)



#### Character 클래스

마찬가지로 문자 값과 색깔, 폰트 이름/사이즈를 가지는 문자 객체이지만, 폰트 이름/사이즈 대신에 폰트 객체를 가지게 됩니다.

Font 객체를 비교해보기 위해 Font getter를 만들었습니다.

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
    
    public Font getFont() {
        return font;
    }
}
```



#### Font 클래스

famliy(폰트 이름)과 사이즈를 가지고 있는 Font 객체 입니다. 

이 **Font클래스는 변하지 않는 속성이기 때문에 값 초기화 후 값을 변경할 수 없게 family와 size를 final로 선언**하고 **getter만 생성**해야 합니다. 또, **상속도 막아야하므로 class에도 final**을 붙여줍니다.

```java
// 상속 불가
public final class Font {

    // 속성에 final을 붙여 변경할 수 없게 함
    final String family;

    final int size;

    public Font(String family, int size) {
        this.family = family;
        this.size = size;
    }

    // getter만 만듦
    public String getFamily() {
        return family;
    }

    public int getSize() {
        return size;
    }
}
```



#### FontFactory 클래스

변하지 않는 Font 객체를 가지고 있다가(캐싱) 해당 String 키에 해당하는 Font 객체를 반환해주기 때문에 메모리를 절약할 수 있습니다.

기존의 예제에서 아래와 같이 변경을 해봤습니다.

- **Factory** 이기 때문에 `final class`로 상속을 막음
- `Inner Class(FontFactoryHolder)`를 이용하여 **Factory를 Singleton으로 관리**
- **캐싱하고 있는 Map은 변경 불가능해야하기 때문**에 `final`로 선언
- Font 객체를 가지고 올 때 **String Key의 형식에 통일성**이 있어야 하기 때문에 `Pattern` 체크

```java
public final class FontFactory {
    private final Map<String, Font> cache = new HashMap<>();
    private final String pattern = "^\\w+:\\d+";

    private FontFactory() {}

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



#### 클라이언트

코드를 실행 시킬 수 있는 클라이언트 코드 입니다. 기존과의 차이점은 `가방 안에 들어있는 아이템들의 가격을 구하는 메서드`와 `아이템의 가격을 구하는 메서드`를 따로 구분하지 않고 `Component의 getPrice만 호출`해주면 된다는 것입니다.

```java
public class Client {

    public static void main(String[] args) {
        // FontFactory 생성
        FontFactory fontFactory = FontFactory.getInstance();
        
        // Character 객체를 생성할 때 같은 폰트를 가져옴
        Character c1 = new Character('h', "white", fontFactory.getFont("nanum:12"));
        Character c2 = new Character('e', "white", fontFactory.getFont("nanum:12"));
        Character c3 = new Character('l', "white", fontFactory.getFont("nanum:12"));

        System.out.println(c1.getFont() == c2.getFont()); // true
    }
}
```



#### 구조

![FlyweightStruct](https://user-images.githubusercontent.com/79291114/147536438-481efd42-c7a7-4176-8ccd-5fab6b0b6439.PNG)





## 3. 장점과 단점

### 장점

- 애플리케이션에서 사용하는 메모리를 줄일 수 있습니다.
  - 자주 사용하는 속성들을 사용할 때마다 만들지 않고 **캐싱해두기 때문에 메모리를 줄일 수 있습니다.**



### 단점

- 코드의 복잡도가 증가합니다.
  - 간단하게 바로 만들어서 사용하지 않고 **캐싱하는 과정의 코드가 필요하기 때문에 복잡도가 증가**합니다.





## 4. 플라이웨이트 패턴을 사용하는 Integer.valueOf()

**Integer** 클래스의 `valueOf`의 설명을 보면

> Returns an `Integer` instance representing the specified `int` value. If a new `Integer` instance is not required, this method should generally be used in preference to the constructor [`Integer(int)`](https://docs.oracle.com/javase/8/docs/api/java/lang/Integer.html#Integer-int-), as this method is likely to yield significantly better space and time performance by caching frequently requested values. **This method will always cache values in the range -128 to 127**, inclusive, and may cache other values outside of this range.

라고 되어 있는데, `This method will always cache values in the range -128 to 127` 문장에서 캐싱, 즉 플라이웨이트 패턴을 사용하는 것을 알 수 있습니다.



---

참고 : [코딩으로 학습하는 GoF의 디자인 패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)

