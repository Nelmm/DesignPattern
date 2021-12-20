# [생성 패턴] 프로토 타입 (Prototype Parttern)

> 정의
>

**원본 객체를 새로운 객체로 복사**하여 사용하는 패턴이라 할 수 있다

프로토 타입 패턴은 딱히 패턴에 대한 방법이 있지 않고 정의 그대로 원본 객체를 복사하는 것을 의미한다고 할 수 있다.

> 자바 프로토타입 패턴 예제
>

자바에서는 객체를 복제할 수 있게끔 **clone()**를 지원한다.

clone()을 사용할 때는 Cloneable을 implements 해야하는데 그 이유에 대해서는 밑에 정리해놓았다.

바로 예제를 보자

```java
@Getter
public class Test implements Cloneable{
    private String name;
    private TestChild testParent;
    @Override
    public Test clone() throws CloneNotSupportedException {
        return (Test) super.clone();
    }
    public Test(String name, TestChild testParent) {
        this.name = name;
        this.testParent = testParent;
    }
}
@Getter
public class TestDeepCopy implements Cloneable{
    private String name;
    private TestChild testChild;

    public TestDeepCopy(String name, TestChild testChild) {
        this.name = name;
        this.testChild = testChild;
    }
    @Override
    public TestDeepCopy clone() throws CloneNotSupportedException {
        TestChild testChild = new TestChild(this.testChild.getName());
        return new TestDeepCopy(this.name, testChild, integer, anInt);
    }
}
@Getter
public class TestChild {
    private String name;
    public TestChild(String name) {
        this.name = name;
    }
}

public class App {
    public static void main(String[] args) throws CloneNotSupportedException {
        //**************** super.clone() 얕은 복사 ***********//
				TestChild testChild = new TestChild("testchild");
        Test test = new Test("test", testChild);
        Test testClone = test.clone();
        //같은 객체가 아님
        System.out.println(test != testClone);
        //값 복사
        System.out.println(Objects.equals(test.getName(), testClone.getName()));
        //객체 얕은 복사
        System.out.println(test.getTestParent() == testClone.getTestParent());
        //************************************//

        //**************** 깊은 복사 ***********//
        TestChild testChild1 = new TestChild("testchild1");
        TestDeepCopy testDeepCopy = new TestDeepCopy("testDeepCopy", testChild1);
        TestDeepCopy testDeepCopy1 = testDeepCopy.clone();
        //복사된 객체는 다름
        System.out.println(testDeepCopy != testDeepCopy1);
        //클래스 내부의 객체도 깊은 복사
        System.out.println(testDeepCopy.getTestChild() != testDeepCopy1.getTestChild());
        //**************** 깊은 복사 ***********//
    }
}

//결과
true
true
true

true
true
```

위 코드를 봤을 때 super.clone()을 수행하면 다음과 같은 특징을 볼 수 있다.

- **복사된 객체는 기존 객체와 다름(주소값 다름)**
- **값은 그대로 복사**
- **객체 안에 있는 TestChild 객체는 얕은 복사 즉, 주소값이 같다**

clone()을 직접 재정의한 경우 TestChild객체를 새로 생성해 복사하여 보내주므로 복사된 객체와 내부 TestChild객체도 모두 다른 객체라는 것을 확인할 수 있다.

> 장점
>
- 복잡한 객체를 만드는 과정을 숨길 수 있다.
- 기존 객체를 복제하는 과정이 새 인스턴스를 만드는 것보다는 비용적측면에서 효율적임.
- 추상적 타입 리턴 가능.

> 단점
>
- 복제한 객체를 만드는 과정 자체가 복잡해질 수 있음

---

> 마커 인터페이스
>

위에서 clone() 사용할 때 Cloneable을 상속해야한다 했다. 그 이유를 설명해보려한다.

**clone()은 Object Class에 정의되어있다.** Object는 모든 Class의 부모이기 때문에 아래와 같이 호출 할 수 있다.

```java
public class Test{
    public String str;
    public Test copy() throws CloneNotSupportedException {
        return (Test) super.clone();
    }
}

public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        Test a = new Test();
        Test b = a.copy();
        System.out.println(b.str);
    }
}

결과
java.lang.CloneNotSupportedException 발생
```

하지만 있는 그대로 호출을 할 시 **CloneNotSupportedException**이 발생한다.

Object의 clone() 부분을 살펴보자.

```java
public class Object {
	/**
     * Creates and returns a copy of this object.  The precise meaning
     * of "copy" may depend on the class of the object. The general
     * intent is that, for any object {@code x}, the expression:
     * <blockquote>
     * <pre>
     * x.clone() != x</pre></blockquote>
     * will be true, and that the expression:
     * <blockquote>
     * <pre>
     * x.clone().getClass() == x.getClass()</pre></blockquote>
     * will be {@code true}, but these are not absolute requirements.
     * While it is typically the case that:
     * <blockquote>
     * <pre>
     * x.clone().equals(x)</pre></blockquote>
     * will be {@code true}, this is not an absolute requirement.
     * <p>
     * By convention, the returned object should be obtained by calling
     * {@code super.clone}.  If a class and all of its superclasses (except
     * {@code Object}) obey this convention, it will be the case that
     * {@code x.clone().getClass() == x.getClass()}.
     * <p>
     * By convention, the object returned by this method should be independent
     * of this object (which is being cloned).  To achieve this independence,
     * it may be necessary to modify one or more fields of the object returned
     * by {@code super.clone} before returning it.  Typically, this means
     * copying any mutable objects that comprise the internal "deep structure"
     * of the object being cloned and replacing the references to these
     * objects with references to the copies.  If a class contains only
     * primitive fields or references to immutable objects, then it is usually
     * the case that no fields in the object returned by {@code super.clone}
     * need to be modified.
     * <p>
     * The method {@code clone} for class {@code Object} performs a
     * specific cloning operation. **First, if the class of this object does
     * not implement the interface {@code Cloneable}, then a
     * {@code CloneNotSupportedException} is thrown.** Note that all arrays
     * are considered to implement the interface {@code Cloneable} and that
     * the return type of the {@code cl**one} met**hod of an array type {@code T[]}
     * is {@code T[]} where T is any reference or primitive type.
     * Otherwise, this method creates a new instance of the class of this
     * object and initializes all its fields with exactly the contents of
     * the corresponding fields of this object, as if by assignment; the
     * contents of the fields are not themselves cloned. Thus, this method
     * performs a "shallow copy" of this object, not a "deep copy" operation.
     * <p>
     * The class {@code Object} does not itself implement the interface
     * {@code Cloneable}, so calling the {@code clone} method on an object
     * whose class is {@code Object} will result in throwing an
     * exception at run time.
     *
     * @return     a clone of this instance.
     * @throws  CloneNotSupportedException  if the object's class does not
     *               support the {@code Cloneable} interface. Subclasses
     *               that override the {@code clone} method can also
     *               throw this exception to indicate that an instance cannot
     *               be cloned.
     * @see java.lang.Cloneable
     */
    @HotSpotIntrinsicCandidate
    protected native Object clone() throws CloneNotSupportedException;
}
```

중간 구문에 보면 Cloneable을 implements하지 않으면 CloneNotSupportedException을 던진다는 문구가 있다. 즉 Class에 Cloneable을 상속해야 clone()을 사용할 수 있다.

그럼 Cloneable Interface를 봐보자.

```java
public interface Cloneable {
}
```

위처럼 Interface에 Method가 정의되어 있지 않다.

그럼 이걸 상속해서 어디다 쓴다는 걸까?

이러한 아무것도 정의되지 않은 Interface를 상속함으로써 해당 Class가 **특정  속성을 가졌다는 것을 표시**해주는 용도로만 사용하는 것을 **마커 인터페이스(Marker Interface)라고 한다.**

```java
public interface Marker {
}

public class MarkerImpl implements Marker {
}

public class NotMarker {
}

public class Main {

    public static void main(String[] args) throws Exception {
	    MarkerImpl marker = new MarkerImpl();
        NotMarker notMarker = new NotMarker();
        //타입 분리
        markerType(marker);
        markerType(notMarker);
        
        //컴파일단에서 에러 검사 가능
        compileCheck(marker);
        compileCheck(notMarker);    //컴파일 시점에 에러 발생
    }
    //타입 분리
    //오브젝트로 받은 경우 컴파일단에서 에러 검사 불가
    public static void markerType(Object obj) throws Exception {
        if(obj instanceof Marker) {
            System.out.println("marker 맞음");
        }
        throw new Exception("마커 아님");
    }

    //컴파일단에서 에러 검사 가능
    public static void compileCheck(Marker marker) {
        System.out.println("마커가 매개변수로 옴");
    }
}

//결과
marker 맞음
Exception 마커 아님
마커가 매개변수로 옴
compile error
```

마커 인터페이스를 사용하면 인스턴스를 구별하는 것이 가능해지고, 컴파일단에서 에러를 검사할 수 있다.

하지만 인터페이스에 아무것도 구현안하고 사용하다 추후 Method를 추가하는 순간 상속한 모든 클래스에 Override를 해줘야하기 때문에 확장면에서는 유연하지 않다.

---

비슷한 기능으로 마커 어노테이션(Marker Annotation)이 있다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MarkerAnnotation {
}

@MarkerAnnotation
public class MarkerAnnotationClass {
}

public class Main {
    public static void main(String[] args) {
        MarkerAnnotationClass markerAnnotationClass = new MarkerAnnotationClass();
        String str = "";
        annotationTest(markerAnnotationClass);
        annotationTest(str);
    }
    public static void annotationTest(Object obj) {
        MarkerAnnotation markerAnnotation = obj.getClass().getAnnotation(MarkerAnnotation.class);
        if(markerAnnotation != null) {
            System.out.println("markerAnnotation을 가지고 있음");
        } else {
            System.out.println("markerAnnotation이 없음");
        }
    }
}

결과
markerAnnotation을 가지고 있음
markerAnnotation이 없음
```

이 경우 마커 인터페이스와 비슷하게 사용가능하나 마커 어노테이션의 경우 컴파일단에서 에러를 찾을 수 없고 런타임에서 에러를 알 수 있다.

하지만 어노테이션같은 경우 많은 정보를 추후에 추가 할 수 있어 **확장에 있어서 유연함을 가지고 있다.**