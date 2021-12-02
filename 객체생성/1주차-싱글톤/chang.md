# 싱글톤 패턴

인스턴스를 오직 한개만 제공하는 클래스 

시스템 런타임, 환경 세팅에 대한 정보등 여러개면 문제가 생기는 케이스에 써야됨. → 오직 한개만 만들어야함.

새로 인스턴스가 만들어지면 같지 않은 경우가 생길수 있고 그러니까 무조건 **new 를 통해서 생성자를 만들어선 안됨**.  

private 생성자를 통해서 하면 → **new로는 절대 객체를 만들 수 없다.** 

그러면 이 객체를 생성하기 위해서는.... → static를 통해서 생성자 메소드를 구현하면... 생성자가 나오게 된다.

```java
public class Settings {
	private static Settings instance;

	private Settings() {}

	private static Setiings getInstance(){
		if(instance == null){
			instance = new Settings();
		}
		return instance;
	}
}
```

하지만 이 방식 스레드 세이프하지 않음.. why? 

1 thread와 2thread가 if문을 거의 동시에 타면 인스턴스가 두개 생기면서 다를 수 있다. 

## Synchronized 사용하기

```java
public class Settings {
	private static Settings instance;

	private Settings() {}

	private static synchronized Setiings getInstance(){
		if(instance == null){
			instance = new Settings();
		}
		return instance;
	}
}
```

→ 싱크로나이즈드 걸면 동기화를 위한 블럭이 걸려서, 성능 이슈가 있을 수 있다. 

## Eager initialize

```java
public class Settings {
	private static final Settings INSTANCE = new Settings();

	private Settings() {}

	private static Setiings getInstance(){
		return INSTANCE;
	}
}
```

이른 초기화 전략 ⇒ 초기화 빨리하면 쓰레드 세이프한데, 단점: 생성되는데 시간이 많이 걸리면... 초기화에 성능 이슈가 존재할 수 있다.

## Double checked Locking

```java
public class Settings {
	private static volatile Settings instance;

	private Settings() {}

	private static Setiings getInstance(){
		if(instance == null){
			synchronized (settings.class){
				if(instance == null){
					instance = new Settings();
				}
			}
		}
		return instance;
	}
}
```

멀티스레드를 많이 쓰는 경우에는 유리하다. 

## static inner 클래스 사용하기

```java
public class Settings {
	private static Settings instance;

	private Settings() {}

	private static class SetiingsHolder {
		private static final Settings instance = new Settings();		
	}

	pulblic static Settings getInstance(){
		return SettingsHolder.instance;
	}
}
```

# 박살내는 방법

1. 리플렉션 쓰기.

```java
Settings settings = Settings.getInstance();
Constructor<Settings> declaredConstructor = Settings.class.getDeclaredConstructor();
declaredConstructor.setAccessible(true);
Settings settings1 = declaredConstructor.newInstance();
System.out.println(settings == settings1);
```

class 자체를 내부에 구현되어있는 instance 선언하기 방식을 통해서 ⇒ 억지로 생성하는 방법이 존재

1. 직렬화 & 역직렬화 사용하기

```java
Settings settings = Settings.getInstance();
Settings settings1 = null;
try (ObjectOutput out = new ObjectOutputStream(new FileOutputStream("settings.obj"))) {
 out.writeObject(settings);
}
try (ObjectInput in = new ObjectInputStream(new FileInputStream("settings.obj"))) {
 settings1 = (Settings) in.readObject();
}
System.out.println(settings == settings1);
```

직렬화를 통해서 일단 파일 만든다음에... 역직렬화해서 객체를 만들면 → 직렬화하는 방식을 세팅해놓지 않아서 그냥 지멋대로 객체를 만들어서 다르게 나옴. 

이런 박살내는 방법을 피해서 구현하려면...?

## Enum

```java
public enum Settings{
	INSTANCE;

}
```

리플렉션으로 박살낼수가 없음.  이미 직렬화가 이미 구현되어있음.

단점: 일단 미리 만들어짐. enum은 enum끼리만 상속 가능 그냥은 상속안됨. 

[https://gist.github.com/schwarzeszeux/2347102](https://gist.github.com/schwarzeszeux/2347102)