# 프로토타입 패턴

기존 인스턴스를 복제해서 새로운 인스턴스를 만들어내는 방식. 

![Untitled](img/Untitled.png)

[https://refactoring.guru/design-patterns/prototype](https://refactoring.guru/design-patterns/prototype)

기존 인스턴스를 외부만 보고 완전히 복사하는 것은 불가능. → 내부에 private 변수 처리가 되어있으면 완전한 복사는 불가능하다 

DB/Network에서 가져온 객체들을 복사해서 ⇒ 새로운 인스턴스를 만들고 변경해서 사용하면 자원을 아낄 수 있지 않을까?에서 출발

원래 하나 만들어놓은걸 바탕으로 재사용 재사용 재사용... 하고 싶은게 이 프로토타입의 큰 맥락

![Untitled](img/Untitled%201.png)

## 자바가 프로토 타입을 제공하는 방식

clone은 object에 이미 존재... ⇒ 하지만 protected로 처리되어있어서 아쉽게도 그냥 쓸수는 없음.

```java
// Object.class 
protected native Object clone() throws CloneNotSupportedException;
```

Object 클래스에 따로  implements를 통해서 cloneable이 붙어있지는 않다.

implements 를 Cloneable을 통해 구현 가능!

⇒ shallow copy로써 작동함. 주소만 복사해서 내부 내용은 재대로 카피가 안되서  원 인스턴스가 값이 바뀌면 → 값이 바뀌어서 나올 수 도 있음. 

`super.clone()` 을 통해서 이미 정의되어있는 클래스를 그대로 완전히 복사하는 것은 어렵다.

```java
public class test {
    public static void main(String[] args) throws CloneNotSupportedException {
        A a = new A(10, new ArrayList<>());
        a.getList().add(1);
        a.getList().add(2);
        A clone = (A) a.clone();
        System.out.println(clone!=a);
        a.getList().add(909);
        System.out.println(a.getList()); //[1,2 909]
        System.out.println(clone.getList()); //[1,2 909] => 복사되어있지만, 실제로 값이 동일. 이러면 원래 copy의 의미를 잃는다 => swallow copy
    }

}

class A implements Cloneable{
    private int x;
    private List<Integer> list;

    public List<Integer> getList() {
        return list;
    }

    public void setList(List<Integer> list) {
        this.list = list;
    }

    public A(int x, List<Integer> list) {
        this.x = x;
        this.list = list;
    }

    public int getX() {
        return x;
    }

    public void setX(int x) {
        this.x = x;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

그래서 Deep Copy로 구현하기 위해서는 clone 오버라이딩 된 값을 직접 조작해서 구현해야한다.

 

```java
public class Users implements Cloneable {
....
@Override
public Object clone() throws CloneNotSupportedException{
	List temp=newArrayList();
	for(String s:this.getUserList()){
		temp.add(s);
	}
	return new Users(temp);
}
```

<aside>
☝ 괜찮은 것 하나 발견

</aside>

이러면 casting하는 과정이 필요함. 결국 Object를 통해서 리턴되니까, (Users) 객체를 묶어줘야하는 상황 생김.

```java
 @Override
  public PhoneNumber clone() {
    try {
      return (PhoneNumber) super.clone();
    } catch(ClassNotSupportedException e) {
      //아무처리를 하지 않거나, RuntimeException으로 감싸는 것이 사용하기 편하다.
    }
```

공변 반환타이핑 기능을 통해 PhoneNumber 타입으로 캐스팅하여 리턴하는 것이 가능하다.

### 프로토타입시 중요한 점.

```java
x.clone() != x // true  -> 복사와 원본은 다르다. 
x.clone().getClass() == x.getClass() // true -> 필수조건은 아님. 막 class까지 다를 수는 있되
x.clone().equals(x) //true -> 복사했기때문에 동치성 자체는 같음. 
```

- x.clone().getClass() == x.getClass()은 참이다.관례상, 이 방법으로 반환된 객체는 독립성이 있어야 한다.이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.

이 비슷한 주제로 **동등성**과 **동일성**이 있다.

- 동일성(==) 비교는 객체가 참조하고 있는 주소값을 비교
- 동등성(Equals)비교는 내용의 비교

 → String만 빼면 동일성, 동등성은 다 같은 방식으로 작동합니다.

## 장점 / 단점

### 장점

- 복잡한 객체를 만드는 과정을 숨길 수 있다.
- 기존 객체를 복제하는 과정이 새 인스턴스를 만드는 것보다는 비용적측면에서 효율적임.
- 추상적 타입 리턴 가능.

단점

- 복제한 객체를 만드는 과정 자체가 복잡할 수도... (순환 참조 이슈)

## 어디서 사용하는가?

Collection의 경우 구체적인 형태가 clone을 할 수 있게 할 수 있음. 

복사하는 방법 ⇒ 생성자 복사를 사용함.

ModelMapper 를 사용하면 DTO 관계 처리할 경우 훨씬 편하게 사용 가능하다. 

```java
ModelMapper mapper = new ModelMapper();

OrderDTO orderDTO = modelMapper.map(order, OrderDTO.class); // ORDER 객체에서 -> DTO로 쑥 빼서 사용하는 결과를 만듬.
```

[Prototype](https://refactoring.guru/design-patterns/prototype)

[Item 13. Clone 재정의는 주의해서 진행하라 | Carrey`s 기술블로그](https://jaehun2841.github.io/2019/01/13/effective-java-item13/#%EC%B0%B8%EA%B3%A0)