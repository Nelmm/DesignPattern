# 빌더 패턴

객체의 생성과정과 표현 방법을 분리하여 동일한 생성 절차에서 다른 표현 결과를 만들 수 있게 하는 패턴.

<br><br>

## 객체 생성 방법
### 1. 점층적 생성자 패턴
```java
class Person{
    private String firstName;
    private String lastName;
    private int age;
    private int height;
    private String hobby;

    Person() {}

    public Person(String firstName) {
        this.firstName = firstName;
    }

    public Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```
파라미터의 개수에 따른 서로다른 생성자가 필요할때 그때 그때 생성자를 추가하는 방법으로 가장 일반적인 패턴이다.

이는 특정 생성모양에 따라 계속 생성자를 추가해주어야되는 귀찮음이 존재한다.

```java
public static void main(String[] args){
    //Constructor
    Person onlyFirstName = new Person("길동");
    Person onlyName = new Person("길동","홍");
    Person onlyLastName = new Person("홍"); //불가능
```
또한, 파라미터 갯수와 파라미터 타입이 동일한 생성자는 한개만 정의가 가능하기 때문에 위와같이 서로 다른 목적으로 사용하고 싶을때 사용할 수 없다.

<br>


### 2. 자바빈 패턴
```java
class Person{
    private String firstName;
    private String lastName;
    private int age;
    private int height;
    private String hobby;

    public Person() {}
    public Person(String firstName, String lastName, int age, int height, String hobby) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
        this.height = height;
        this.hobby = hobby;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public void setHobby(String hobby) {
        this.hobby = hobby;
    }
}

//use
Person person = new Person();
person.setAge(25);
person.setHeight(180);
person.setHobby("음악");
```

위와 같은 클래스가 있을때 객체를 생성하는 과정에서 setter를 이용하여 객체를 정의하는 패턴

하지만 서비스가 멀티스레드 환경에 공유하는 객체가 필요하다라고 하면 객체가 변할수있는 상태라고 한다면 큰 문제가 될 수 있기 때문에 불변성을 만족해야 한다.

하지만 setter들로 인해 해당 객체는 언제든지 변경이 될 수 있는 상태이기때문에 클래스에서 setter는 외부에 열어두지 않는 것이 좋다.

<br>

### 3. 빌더 패턴
```java
//Class
public class Person{
    String lastName;
    String firstName;

     public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}

//Absract Builder
public abstract class PersonBuilder{
    protected Person person;

    public final void createPerson() {
        person = new Person();
    }

    public abstract void buildLastName();
    public abstract void buildFirstName();

    public Person getPerson(){
        return person;
    }
}

//Builder 
public class StandardPersonBuilder{
    public void buildLastName(){
        person.setLastName("홍");
    }
    public void buildFirstName(){
        person.setFirstName("길동");
    }
}

public class CustomPersonBuilder{
    public void buildLastName(){
        person.setLastName("김");
    }
    public void buildFirstName(){
        person.setFirstName("이박");
    }
}

//Director
public class Director {
    private Builder builder;

    public Director(Builder builder){
        this.builder = builder;
    }

    public void build() {
        builder.createPerson();
        builder.buildLastName();
        builder.buildFirstName();
    }

    public Person getPerson() {
        return builder.getPerson();
    }
}
public static void main(String[] args) {
    Builder personBuilder = new PersonBuilder();
    Director director = new Director(personBuilder);
    director.build();

    Person person = director.getPerson();
}
```
객체를 만들어 내는 Builder클래스를 이용하여 사용자는 지시자(Director)에게 특정 형태의 객체를 만들도록 요청할 수 있는 패턴으로 사용자는 서로다른 Builder를 Director에게 끼워주기만 하면 다른 형태의 객체를 만들어 낼 수있는 패턴.

여기서 중요한 건 `동일한 생성 절차`에서 `다른 표현 결과`를 만들 수 있다는 것이다.

물론 해당 구현 방식도 Person이라는 클래스가 setter가 열려있기 때문에 외부에서 언제든지 값을 변경이 가능하다.

<br><br>


## Effective Java / Object의 Builder
```java
class Person{
    private String firstName;
    private String lastName;

    public static Person.PersonBuilder builder() {
        return new Person.PersonBuilder();
    }

    public static class PersonBuilder{
        private String firstName;
        private String lastName;

        PersonBuilder() {
        }

        public Person.PersonBuilder firstName(final String firstName) {
            this.firstName = firstName;
            return this;
        }

        public Person.PersonBuilder lastName(final String lastName) {
            this.lastName = lastName;
            return this;
        }

        public Person build() {
            return new Person(this.firstName, this.lastName);
        }
    }
}

//use
Person person = new Pesron.builder()
    .firstName("길동")
    .lastName("홍")
    .build();
```

두 책에서의 말하고 있는 빌더 패턴은 생성자의 인자가 많을 경우 고려하라고 말하고 있으며 다음과 같은 문제를 해결할 수 있다.

1. 불필요한 생성자를 제거
1. 데이터의 순서에 상관없이 객체 생성
1. 사용자입장에서 이해하기 쉬워야한다.

setter가 존재하지 않기 때문에 외부에서 값을 건드려 일관성이 깨지는 일이 존재하지 않으며 프로퍼티값을 검증하는 로직을 클래스에 두지 않고 객체를 생성해낼 수 있다.

<br><br>


## 단점

1. 약간의 성능이슈
   Builder를 생성하는 것도 하나의 객체를 생성하는 것이고 해당 객체가 다른 객체를 생성하는 것이기 때문에 약간의 메모리를 소비하게 된다.

1. 코드의 유지보수
   이는 대부분의 디자인패턴들의 공통적인 단점으로 관리하는 클래스들이 많아질 수 있으며 빌더를 inner 클래스에 정의해도 클래스가 길어져 가독성이 떨어질 수 있다. 또한 php와 같은 언어에서는 inner 클래스를 지원해주지 않기 때문에 사용이 불가능하다.

<br><br>


## @Builder

Lombok 라이브러리의 @Builder 어노테이션을 사용하면 별도의 클래스 생성없이 컴파일러가 내부클래스로 Builder를 생성해준다.

```java
class Person{
    private String firstName;
    private String lastName;
    private int age;
    private int height;
    private String hobby;

    public static Person.PersonBuilder builder() {
        return new Person.PersonBuilder();
    }

    public static class PersonBuilder{
        private String firstName;
        private String lastName;
        private int age;
        private int height;
        private String hobby;

        PersonBuilder() {
        }

        public Person.PersonBuilder firstName(final String firstName) {
            this.firstName = firstName;
            return this;
        }

        public Person.PersonBuilder lastName(final String lastName) {
            this.lastName = lastName;
            return this;
        }

        public Person.PersonBuilder age(final int age) {
            this.age = age;
            return this;
        }

        public Person.PersonBuilder height(final int height) {
            this.height = height;
            return this;
        }

        public Person.PersonBuilder hobby(final String hobby) {
            this.hobby = hobby;
            return this;
        }

        public Person build() {
            return new Person(this.firstName, this.lastName, this.age, this.height, this.hobby);
        }

        public String toString() {
            return "Person.PersonBuilder(firstName=" + this.firstName + ", lastName=" + this.lastName + ", age=" + this.age + ", height=" + this.height + ", hobby=" + this.hobby + ")";
        }
    }
}
```

다만 build()를 정의하면서 AllargumentsConstructor를 필요로 하기 때문에 원본클래스의 해당 생성자는 필수로 있어야 컴파일에러가 발생하지 않는다.

<br><br>


## 팩토리패턴과 비교해보자

- 팩토리 메서드 패턴 : 객체를 생성하는 패턴에 있어 추상화를 적용할 수 있는 가작 간단한 패턴으로 단일 메서드를 사용하여 간단한 객체를 생성

    빌더패턴은 Director가 Builder를 제어하지만, 팩토리 메서드 패턴은 상위 클래스가 하위 클래스를 제어한다.

- 추상 팩토리 패턴 : 복잡한 객체 생성을 담당한다는 점에서는 빌더 패턴과 유사하나 여러 팩토리 메서드를 사용하여 객체 생성하며 각 메서드들은 하위 객체를 바로 리턴한다. 


- 빌더 패턴 : 단계별로 복잡한 객체를 생성이 가능하며 각 중간단계에서의 상태를 유지하고 최종적으로 하위 객체들을 합친 최종 객체를 반환한다.

    한마디로 빌더패턴은 사용자가 최종결과물을 받고 추상 팩토리 패턴은 최종 결과물에 필요한 재료들을 받게 된다.


<br><br>

## 사용 예시
![abstractFactory-architecture](/객체생성/3주차-빌더/image/abstractFactory-architecture.png)
```php
$partsFactory = new BioDataEngineerFactory($this->getRequest, $this->identity, $this->cookie);
$header = $partsFactory->createHeader();
$head = $partsFactory->createHead();
$body = $partsFactory->createBody();
$parameter = $partsFactory->createParameter();

$page = new Page($header,$head,$body,$parameter);
$page->assign();    
```


![builder](/객체생성/3주차-빌더/image/builder.png)

```php
$partsBuilder = new BioDataEngineerBuilder($this->getRequest, $this->identity, $this->cookie);
$director = new Director(partsBuilder)
$director->createPage();

$page = $director->getPage();
$page->assign();    
```

