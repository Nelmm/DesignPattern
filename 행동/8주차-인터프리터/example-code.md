## 2. 인터프리터 적용 예제

### 2-1. 요구사항

자주 등장하는 문제 ➡️ 간단한 언어 로 정의하고 재사용하는 패턴으로

반복되는 문제 패턴을 언어 또는 문법으로 정의하고 확장할 수 있기 때문에 후위표현식의 연산을 문법으로 정의하는 코드를 작성하고자 하고 싶다.

아래 코드를 살펴보자.

```java
public class PostfixNotation {

    private final String expression;

    public PostfixNotation(String expression) {
        this.expression = expression;
    }

    public static void main(String[] args) {
        // 후위표현식 연산
        PostfixNotation postfixNotation = new PostfixNotation("123+-");
        postfixNotation.calculate();
    }

    private void calculate() {
        Stack<Integer> numbers = new Stack<>();

        for (char c : this.expression.toCharArray()) {
            switch (c) {
                // Java 14부터 switch-case break 없이 사용할 수 있다!
                case '+' -> numbers.push(numbers.pop() + numbers.pop());
                case '-' -> {
                    int right = numbers.pop();
                    int left = numbers.pop();
                    numbers.push(left - right);
                }
                default -> numbers.push(Integer.parseInt(c + ""));
            }
        }

        System.out.println(numbers.pop());
    }
}
```

`123+-`연산을 `xyz+-`문법으로 정의하여 재사용하는 경우, `xyz`는 TerminalExpression이고 `+-`는 NonTerminalExpression이다.

- `TerminalExpression` : 표현에서 사용되는 기호들(x,y,z)의 해석값을 구하는 클래스(즉 x일때 무엇, y일때 무엇, z일때 무엇으로 해석하나)
- `NonTerminalExpression` : 기호만으로는 결과가 나오지 않고 특정 `Expression`들을 참조해서 결과를 얻어야하는 클래스(+, -만으로는 결과가 나오지 않고 z, x, y과 같은 `TerminalExpression`, 또는 또 다른 `NontermialExpresion` 참조해서 결과값을 얻을 때까지 재귀적으로 참조한다. )

이제 클라이언트 코드부터, 인터프리터 패턴으로 개선한 코드를 살펴보자.

```java
// 클라이언트
public class App {
    public static void main(String[] args) {
        // PostfixParser - 문자열을 parse 해준다.
        PostfixExpression expression = PostfixParser.parse("xyz+-a+");
        int result = expression.interpret(Map.of('x', 1, 'y', 2, 'z', 3, 'a', 4));
    }
}

// 받은 문자열을 Expression들을 이용해 파싱하는 리턴 값은 Expression 객체들로 이루어진 Expression을 반환
public class PostfixParser {
    // 문자열을 char 배열로 받아서 getExpression 메소드 호출 후 stack push
    public static PostfixExpression parse(String expression) {
        // Stack 타입 - PostfixExpression
        Stack<PostfixExpression> stack = new Stack<>();
        for (char c : expression.toCharArray()) {
            stack.push(getExpression(c, stack));
        }
        return stack.pop();
    }
  
    private static PostfixExpression getExpression(char c, Stack<PostfixExpression> stack) {
        switch (c) {
            case '+':
                return new PlusExpression(stack.pop(), stack.pop());
            case '-':
                PostfixExpression right = stack.pop();
                PostfixExpression left = stack.pop();
                return new MinusExpression(left, right);
            default:
                return new VariableExpression(c);
        }
    }
}

// AbstractExpression
public interface PostfixExpression {
    // 모든 표현에서 공통적으로 구현할 메소드
    int interpret(Map<Character, Integer> context);
}

// TerminalExpression
// 특정 기호들에 대한 해석값을 정의
public class VariableExpression implements PostfixExpression {

    private Character character;

    public VariableExpression(Character character) {
        this.character = character;
    }

    @Override
    public int interpret(Map<Character, Integer> context) {
        // 값을 그대로 반환
        return context.get(this.character);
    }
}

// NonTerminalExpression1 (Plus)
public class PlusExpression implements PostfixExpression {
    private PostfixExpression left;
    private PostfixExpression right;
    public PlusExpression(PostfixExpression left, PostfixExpression right) {
        this.left = left;
        this.right = right;
    }
    @Override
    public int interpret(Map<Character, Integer> context) {
        // 받은 기호만으로 결과가 나오지 않아 다른 Expression들을 참조하여 결과 도출
        return left.interpret(context) + right.interpret(context);
    }
}

// NonTerminalExpression2 (Minus)
public class MinusExpression implements PostfixExpression {
    private PostfixExpression left;
    private PostfixExpression right;
    public MinusExpression(PostfixExpression left, PostfixExpression right) {
        this.left = left;
        this.right = right;
    }
    @Override
    public int interpret(Map<Character, Integer> context) {
        // Plus와 마찬가지
        return left.interpret(context) - right.interpret(context);
    }
}
```

디버깅을 해보면 아래와 같고, 트리 형태이다.

<img width="366" alt="image" src="https://user-images.githubusercontent.com/42997924/162049490-b7f7b1a7-6fdf-4d8a-8351-093c06c4ed7f.png">

<img width="709" alt="image" src="https://user-images.githubusercontent.com/42997924/162050558-98e7b7b8-0f2b-44ab-bd83-4dbf05ae7c0e.png">
