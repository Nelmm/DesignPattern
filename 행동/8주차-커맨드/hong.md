# 커맨드 패턴

`어떠한 요청`이 존재할때 이 요청에 대한 정보들이 포함된 `객체로 캡슐화`하는 패턴으로 이 객체를 메서드 인수로 전달하거나,요청을 재사용하거나, 요청 실행을 큐에 넣어 지연시키거나 실행을 취소할 수 있게 해준다.

## 코드

### 기존

```java
public interface Button {
    void click();
}
public class CopyButton implements Button{
    @Override
    public void click() {
        System.out.println("붙여넣기 실행");
    }
}
public class CutButton implements Button{
    @Override
    public void click() {
        System.out.println("잘라내기 실행");
    }
}
public class PasteButton implements Button{
    @Override
    public void click() {
        System.out.println("붙여넣기 실행");
    }
}
public class SaveButton implements Button {
    @Override
    public void click() {
        System.out.println("저장 실행");
    }
}
public class History {
    private final Deque<Button> history = new ArrayDeque<>();

    public void push(Button b) {
        history.addLast(b);
    }

    public Button pop() {
        return history.pop();
    }

    public boolean isEmpty() {
        return history.isEmpty();
    }
}

public class Editor {
    History history = new History();

    public static void main(String[] args) {
        Button button = new SaveButton();
        button.click();
        history.push(button);
        button  = new CopyButton();button.click();
        history.push(button);
        button = new PasteButton();button.click();
        history.push(button);
        button = new CutButton();button.click();
        history.push(button);
    }
}

```

여기에 keyboard로도 해당 명령을 실행하고싶다면 다음과 같이 추가하게 될 것이다.

```java
public interface Keyboard {
    void click();
}
public class CopyKeyboard implements Keyboard{
    @Override
    public void click() {
        System.out.println("붙여넣기 실행");
    }
}
public class CutKeyboard implements Keyboard{
    @Override
    public void click() {
        System.out.println("잘라내기 실행");
    }
}
public class PasteKeyboard implements Keyboard{
    @Override
    public void click() {
        System.out.println("붙여넣기 실행");
    }
}
public class SaveKeyboard implements Keyboard {
    @Override
    public void click() {
        System.out.println("저장 실행");
    }
}
```
