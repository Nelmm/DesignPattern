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
        System.out.println("복사 실행");
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

public class Editor {
    public static void main(String[] args) {
        Button button = new SaveButton();button.click();
        button  = new CopyButton();button.click();
        button = new PasteButton();button.click();
        button = new CutButton();button.click();
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
        System.out.println("복사 실행");
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
Editor의 기능을 추가하고자 할때마다 생성해야 하는 클래스가 기능을 수행하는 주체가 많을 수록 늘어나게 되고 그 클래스에서 수행하는 행동은 코드 중복이 발생하는 것을 볼 수 있다.

이 코드에 되돌리기(작업 취소)나 재실행 기능을 추가하려고 한다면 어떻게 코드를 추가 할 수 있을까?
```java
public interface Button {
    void click();
    void undo();
}
public class CopyButton implements Button {
    @Override
    public void click() {
        System.out.println("복사 실행");
    }

    @Override
    public void undo() {
        System.out.println("복사 실행 취소");
    }
}
public class History {
    private final Deque<HistoryInfo> history = new ArrayDeque<>();

    public void push(HistoryInfo b) {
        history.addLast(b);
    }

    private HistoryInfo pop() {
        return history.pollLast();
    }

    public HistoryInfo peek() {
        return history.peekLast();
    }

    public void redo() throws InvocationTargetException, IllegalAccessException {
        HistoryInfo historyInfo = peek();
        assert historyInfo != null;
        historyInfo.getMethod().invoke(historyInfo.getObject());
        push(historyInfo);
    }

    public void undo() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        HistoryInfo historyInfo = pop();
        assert historyInfo != null;
        historyInfo.getObject().getClass().getMethod("undo").invoke(historyInfo.getObject());
    }

    public boolean isEmpty() {
        return history.isEmpty();
    }

    public static class HistoryInfo {
        private Method method;
        private Object object;

        public HistoryInfo(Method method, Object object) {
            this.method = method;
            this.object = object;
        }

        public Method getMethod() {
            return method;
        }

        public Object getObject() {
            return object;
        }
    }
}

public class Editor {
    static History history = new History();

    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        Button button = new SaveButton();
        button.click();
        history.push(new History.HistoryInfo(button.getClass().getDeclaredMethod("click"),button));
        button  = new CopyButton();button.click();
        history.push(new History.HistoryInfo(button.getClass().getDeclaredMethod("click"),button));
        button = new PasteButton();button.click();
        history.push(new History.HistoryInfo(button.getClass().getDeclaredMethod("click"),button));
        button = new CutButton();button.click();
        history.push(new History.HistoryInfo(button.getClass().getDeclaredMethod("click"),button));

        //마지막 명령 재실행
        history.redo();

        //명령 취소
        history.undo();
    }

    private static void execute(Component component) {
        component.getCommand().execute();
        history.push(component);
    }
}
```
구현 방법은 다양할 텐데 나는 위와 같이 Button/Keyboard에 undo메서드를 추가하고 각각 다 구현해주었다. 그런다음 실행기록을 저장하는 History 객체를 만들어 관리하도록 코드를 추가했다. 보면 알겠지만 invoke메서드를 이용하기 때문에 Method이름으로 저장,호출하기 때문에 매우 불안정하다. 또한 버튼이나 키보드의 undo 이벤트도 click을 통해 수행되는 것인데 메서드역할의 모호성이 존재한다.

이 코드에서 명령(요청/행동)을 분리하여 따로 요청을 객체로 만들어 관리해보자.

## Command 도입
```java
public interface Command {
    public void execute();
}
public class CopyCommand implements Command{
    @Override
    public void execute() {
        System.out.println("복사 실행");
    }

    @Override
    public void undo() {
        System.out.println("복사 실행 취소");
    }
}

public class CutCommand implements Command {
    @Override
    public void execute() {
        System.out.println("잘라내기 실행");
    }

    @Override
    public void undo() {
        System.out.println("잘라내기 실행 취소");
    }
}

public class PasteCommand implements Command {
    @Override
    public void execute() {
        System.out.println("붙여넣기 실행");
    }

    @Override
    public void undo() {
        System.out.println("붙여넣기 실행 취소");
    }
}

public class SaveCommand implements Command {
    @Override
    public void execute() {
        System.out.println("저장 실행");
    }

    @Override
    public void undo() {
        System.out.println("저장 실행 취소");
    }
}

public interface Component {
    public Command getCommand();
}

public class Button implements Component{
    private final Command command;

    public Button(Command command) {
        this.command = command;
    }

    public Command getCommand() {
        return this.command;
    }
}

public class Keyboard implements Component{
    private final Command command;

    public Keyboard(Command command) {
        this.command = command;
    }

    public Command getCommand() {
        return this.command;
    }
}

public class History {
    private final Deque<Component> history = new ArrayDeque<>();

    public void push(Component b) {
        history.addLast(b);
    }

    private Component pop() {
        return history.pollLast();
    }

    public Component peek() {
        return history.peekLast();
    }

    public void redo() {
        Component component = peek();
        assert component != null;
        component.getCommand().execute();
        push(component);
    }

    public void undo() {
        Component component = pop();
        assert component != null;
        component.getCommand().undo();
    }

    public boolean isEmpty() {
        return history.isEmpty();
    }
}

public class Editor {
    static History history = new History();

    public static void main(String[] args) {
       execute(new Button(new CopyCommand()));
       execute(new Button(new CutCommand()));
       execute(new Button(new PasteCommand()));
       execute(new Keyboard(new SaveCommand()));
       execute(new Keyboard(new PasteCommand()));

        //명령 취소
        history.undo();

        //마지막 명령 재실행
        history.redo();
    }

    private static void execute(Component component) {
        component.getCommand().execute();
        history.push(component);
    }
}

복사 실행
잘라내기 실행
붙여넣기 실행
저장 실행
붙여넣기 실행
붙여넣기 실행 취소
저장 실행
```

## Java 에서 사용 예
java.lang.Runnable / javax.swing.Action과 같이 어떠한 명령(액션)으로 결과를 매개변수화 하기 위한 콜백의 대안으로 사용된다.