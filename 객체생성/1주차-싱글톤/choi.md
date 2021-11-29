싱글톤 패턴이란 한마디로 인스턴스를 한개만 제공하는 클래스

EX) 게임에 있는 설정

# 1. 기본적인 특징

- new 생성자를 통해 여러개의 객체를 생성할 수 없게 해야한다
→ 생성자를 private를 통해 막아야함

```java
public class Settings {
	private Settings() {}
}
```

- 인스턴스는  클래스 내부에서 static method를 통해 가져오도록 해야한다.
→ 클래스내부에 인스턴스를 지정하고 getInstance()를 통해 인스턴스가 비었을 때 생성, 있으면 기존 객체 반환

```java
public class Settings {
	private static Settings instance;
	private Settings() {}
	public static Settings getInstance() {
		if(instance == null) {
			instance = new Settings();
		}
		return new Settings();
	}
}
```

> 싱글톤 패턴의 궁극적인 목표는 모든 쓰레드가 똑같은 하나의 공유 객체를 가져다 쓰는 것인데 위의 코드는 그 목표를 완전히 이룰 수 있는 코드인가?
> 

# 2. synchronized

위의 코드로는 복수 개의 쓰레드가 거의 동시에 들어왔을 때 서로 다른 객체가 생성될 가능성이 존재한다. 이걸 해결하기 위해 synchronized 키워드를 사용한다.

```java
public class Settings {
	private static Settings instance;

	private Settings() {}

	public static synchronized Settings getInstance() {
		if(instance == null) {
			instance = new Settings();
		}
		return new Settings();
	}
}
```

하지만 synchronized는 성능상의 불이익이 존재

(키를 부여받은 쓰레드만 접근할 수 있게 해주는 메커니즘)

처음부터 객체를 미리 생성해놓는 이른 초기화(eager initialization)를 통해 synchronized를 사용하지 않는 방법도 있다.

```java
/*Eager Initialization)*/
public class Settings {
	private static final Settings INSTANCE = new Settings();

	private Settings() {}

	public static Settings getInstance() {
		return INSTANCE;
	}
}
```

이른 초기화의 단점은 객체를 만든다는 것 자체다. 이 객체를 생성하는것이 자원을 많이 사용하는데 쓰지 않는 객체라면 그거 자체가 낭비가 되기때문

# 3. Double Checked Locking

처음 객체를 만들때만 synchronized를 사용하게 할 수 있는 방법이 Double Checked Locking이다.

```java
public class Settings {
	private static final volatile Settings instance;
	private Settings() {}
	public static Settings getInstance() {
		if(instance == null) {
			synchronized (Settings.class) {
				if(instance == null) {
					instance = new Settings();
				}	
			}
		}
		return instance;
	}
}
```

위의 코드처럼 if문을 중첩을 사용해 첫번째 if문에 복수의 쓰레드가 두개 들어왔다 해도 다음 if문에서 synchronized를 통해 동기화를 유지할 수 있게 한다.
이 방법을 사용할 때 instance 속성에도 volatile을 선언해줘야한다.

이것의 단점은 1.5버전부터 사용가능.

# 4. static inner 클래스 사용

클래스안에 static class를 두어 생성하는 방법이다.

```java
public class Settings {
	private static final volatile Settings instance;
	private Settings() {}
	
	private static class SettingsHolder {
		private static final Settings INSTANCE = new Settings();
	}

	public static Settings getInstance() {
		return SettingsHolder.INSTANCE;
	}
}
```

이 방법을 사용하면 쓰레드가 동시에 들어왔을 때의 서로 다른 객체가 생성되는 것을 막을 수 있다.

또한 static inner class는 접근했을 때 로딩이되는 특징 때문에 객체에 접근할 때 로딩된다는 이점도 가지게 된다.

가장 무난한 방법

하지만 위의 모든 코드들은  결국 다음과 같은 코드를 통해 직접 생성자를 호출할 수 있다.

```java
public static void main(String[] args) {
	Contructor<Settings> constructor = Settings.class.getDeclaredConstructor();
	constructor.setAccessible(true);
	Settings setting1 = constructor.newInstance();
}
```

volatail, synchronized, inner class

## volatile

java 변수를 main memory에 저장하겠다고 명시