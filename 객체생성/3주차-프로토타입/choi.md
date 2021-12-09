# 프로토타입 패턴

> 프로토타입 패턴은 원본 객체를 새로운 객체로 복사하여 사용하는 패턴이라 할 수 있다.
>

자바에서는 Cloneable이라는 인터페이스를 통해 구현할 수 있다.

```java
@Getter
public class Test implements Cloneable{
    private String name;
    private  int anInt;
    private Integer integer;
    private TestChild testParent;
    @Override
    public Test clone() throws CloneNotSupportedException {
        return (Test) super.clone();
    }
    public Test(String name, TestChild testParent, int anInt, Integer integer) {
        this.name = name;
        this.anInt = anInt;
        this.integer = integer;
        this.testParent = testParent;
    }
}
@Getter
public class TestDeepCopy implements Cloneable{
    private String name;
    private Integer integer;
    private int anInt;
    private TestChild testChild;

    public TestDeepCopy(String name, TestChild testChild, Integer integer, int anInt) {
        this.name = name;
        this.integer = integer;
        this.anInt = anInt;
        this.testChild = testChild;
    }
    @Override
    public TestDeepCopy clone() throws CloneNotSupportedException {
        TestChild testChild = new TestChild(this.testChild.getName());
        int aInt = this.anInt;
        Integer integer = this.integer;
        return new TestDeepCopy(this.name, testChild, integer, anInt);
    }
}
@Getter
public class TestChild {
    private String name;
    public TestChild(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
}

public class App {
    public static void main(String[] args) throws CloneNotSupportedException {
        TestChild testChild = new TestChild("testchild");
        Test test = new Test("test", testChild, 0, 0);
        Test testClone = test.clone();

        System.out.println(test != testClone); //같은 객체가 아님
        System.out.println(Objects.equals(test.getName(), testClone.getName()));//얕은 복사
        System.out.println(test.getTestParent() == testClone.getTestParent());  //얕은 복사

        System.out.println(System.identityHashCode(test.getInteger()));
        System.out.println(System.identityHashCode(testClone.getInteger()));

        System.out.println(System.identityHashCode(test.getAnInt()));
        System.out.println(System.identityHashCode(testClone.getAnInt()));

        TestChild testChild1 = new TestChild("testchild1");
        TestDeepCopy testDeepCopy = new TestDeepCopy("testDeepCopy", testChild1, 1, 1);
        TestDeepCopy testDeepCopy1 = testDeepCopy.clone();

        System.out.println(testDeepCopy != testDeepCopy1);
        System.out.println(testDeepCopy.getTestChild() == testDeepCopy1.getTestChild());

        System.out.println(System.identityHashCode(testDeepCopy.getInteger()));
        System.out.println(System.identityHashCode(testDeepCopy.getInteger()));

        System.out.println(System.identityHashCode(testDeepCopy1.getAnInt()));
        System.out.println(System.identityHashCode(testDeepCopy1.getAnInt()));
    }
}
```

Cloneable의 Default  구현체를 통해 Clone을 구현하면 얕은 복사가 이루어진다.
이번 프로토타입 패턴에서는 프로토타입 패턴에 대한 내용보다는 프로토 타입의 주의점인 객체 복사에 대한 내용에 좀 더 초점을 맞춰보는게 좋을 것 같다.