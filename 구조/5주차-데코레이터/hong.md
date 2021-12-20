# 데코레이터
![decorator](/common/image/decorator.PNG)
사진 출처 : https://refactoring.guru/design-patterns/decorator

기존 코드를 변경하지 않고 추가 기능을 추가하는 패턴으로 `런타임`에도 기능을 추가하는 것이 가능하다. 기능 확장이 필요할 때 상속대신 사용할 수 있는 패턴.

특정 DataSource의 인터페이스가 있고 그 구현체로 FileDataSource가 있다고해보자 그런데 파일데이터를 압축해달라는 요구사항이 생기고 차례로 암호화, 암호화와 압축화를 동시에 해달라는 요구사항이 생겼을 때 아래와 같이 상속을 이용하여 해결할 수 있을 것이다.
```java
public interface DataSource {
    void writeData(String data);
    String readData();
}

public class FileDataSource implements DataSource{
    private String fileName;

    public FileDataSource(String fileName) {
        this.fileName = fileName;
    }

    @Override
    public void writeData(String data) {
        System.out.println(fileName + "에 " + data + "쓰기");
    }

    @Override
    public String readData() {
        return "data";
    }
}

public class CompressionFileDataSource extends FileDataSource{
    public CompressionFileDataSource(String fileName) {
        super(fileName);
    }

    @Override
    public void writeData(String data) {
        super.writeData("압축 " + data);
    }

    @Override
    public String readData() {
        String data = super.readData();
        return "압축해제 " + data;
    }
}
public class EncryptionFileDataSource extends FileDataSource{
    public EncryptionFileDataSource(String fileName) {
        super(fileName);
    }

    @Override
    public void writeData(String data) {
        super.writeData("암호화 " + data);
    }

    @Override
    public String readData() {
        String data = super.readData();
        return "복호화 " + data;
    }
}
public class EncryptionCompressionFileDataSource extends FileDataSource{
    public EncryptionCompressionFileDataSource(String fileName) {
        super(fileName);
    }

    @Override
    public void writeData(String data) {
        super.writeData("암호화 압축 " + data);
    }

    @Override
    public String readData() {
        String data = super.readData();
        return "복호화 압축해제 " + data;
    }
}
public class App {
    public static void main(String[] args) {
        String fileName = "README.md";
        String data = "데코레이터";

        DataSource dataSource = new FileDataSource(fileName);
        dataSource.writeData(data);                 //README.md에 데코레이터쓰기
        System.out.println(dataSource.readData());  //data

        dataSource = new CompressionFileDataSource(fileName);
        dataSource.writeData(data);                 //README.md에 압축 데코레이터쓰기
        System.out.println(dataSource.readData());  //압축해제 data

        dataSource = new EncryptionFileDataSource(fileName);
        dataSource.writeData(data);                 //README.md에 암호화 데코레이터쓰기
        System.out.println(dataSource.readData());  //복호화 data

        dataSource = new EncryptionCompressionFileDataSource(fileName);
        dataSource.writeData(data);                 //README.md에 암호화 압축 데코레이터쓰기
        System.out.println(dataSource.readData());  //복호화 압축해제 data
    }
}
```
그런데 위와 같은 상속을 통해 기능을 확장하는 방식은 구현체가 많이 존재하는데 추가하려는 기능이 많아질 수록  문제가 발생한다. 만일 DataSource의 구현체가 DbDataSource / CacheDataSource ... 와 같이 다양하게 존재하고 해당 구현체마다 기능들을 추가하려고 한다면 생성할 클래스의 갯수가 NxM 갯수만큼 증가하기 때문이다.

이를 해결하기 위한 방법으로 데코레이터 패턴을 이용할 수 있다.

```java
public class Decorator implements DataSource{
    private DataSource dataSource;

    public Decorator(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public void writeData(String data) {
        dataSource.writeData(data);
    }

    @Override
    public String readData() {
        return dataSource.readData();
    }
}
public class CompressionDecorator extends Decorator{
    public CompressionDecorator(DataSource dataSource) {
        super(dataSource);
    }

    @Override
    public void writeData(String data) {
        super.writeData("압축 " + data);
    }

    @Override
    public String readData() {
        String data = super.readData();
        return "압축해제 " + data;
    }
}
public class EncryptionDecorator extends Decorator{
    public EncryptionDecorator(DataSource dataSource) {
        super(dataSource);
    }

    @Override
    public void writeData(String data) {
        super.writeData("암호화 " + data);
    }

    @Override
    public String readData() {
        String data = super.readData();
        return "복호화 " + data;
    }
}
public class App {
    boolean compressionFlag = true;
    boolean encryptionFlag = true;

    public static void main(String[] args) {
        String fileName = "README.md";
        String data = "\'데코레이터\'";

        DataSource dataSource = new FileDataSource(fileName);

        if(compressionFlag){
            dataSource = new CompressionDecorator(dataSource);
        }

        if(encryptionFlag){
            dataSource = new EncryptionDecorator(dataSource);
        }

        dataSource.writeData(data);                 //README.md에 암호화 압축 '데코레이터' 쓰기
    }
}
```
데코레이터를 이용하는 객체가 많지 않은 현재 예제같은 경우에는 생성한 클래스의 개수가 비슷해 보일 수 있지만 객체가 많아진다면 첫번째 예제처럼 NxM이 아닌 N+M의 갯수만큼만 생성해주면 된다.


## 장점
1. 새로운 클래스를 만들지 않고 기존 기능을 조합 가능하다.
2. 런타임에 동적으로 기능을 확장할 수 있다.
3. 단일 책임원칙


## 다른 패턴들과 비교

- 어댑터 : 기능을 확장한다는 점에서 비슷하다고 할 수 있으나 어댑터는 기존 인터페이스와는 다른 인터페이스를 반환하고, 데코레이터는 인터페이스를 변경하지 않고 기능이 향상된 인터페이스를 제공한다.
- 컴포짓 : 합성을 이용한 패턴이라는 점에서는 비슷하나 컴포짓은 관련된 기능을 가지고 있는 객체들을 하나의 추상층으로 묶어 결과를 집계(요약) 한다면, 데코레이터는 해당 객체에 책임을 추가한다.
  
  ![Composite](/객체생성/3주차-빌더/image/abstractFactory-architecture.png)