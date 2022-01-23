# 인터프리터 패턴

자주 등장하는 문제를 간단한 언어로 정의하고 재사용하는 패턴

반복되는 문제 패턴을 언어 또는 문법으로 정의하고 확장할 수 있다.

![](https://sourcemaking.com/files/v2/content/patterns/Interpreter_example1.png?id=6e18c37efd5aa7086a60)

[Interpreter Design Pattern](https://sourcemaking.com/design_patterns/interpreter)

종료되는 표현식, 종료되지 않는 표현식(다른 표현식을 참조하는 표현식) 을 바탕으로 섞어서 현 문장을 해석하는 방식으로 진행되어진다. 

UML을 보자면 다음과 같은데,

![Interpreter Pattern | 밥줄과 취미 사이 ːː 못 먹어도 고!](https://dejavuhyo.github.io/assets/img/2021-01-28-interpreter-pattern/img001.png)

어떠한 해석을 정의하여 특정 값이 왔을 때 정의된 해석에 따라 결과를 도출하는 패턴이라 볼 수 있다.

- `AbstractExpression` : 모든 표현에서 공통적으로 사용할 `interpret()`를 정의한 추상클래스
- `TerminalExpression` : 표현에서 사용되는 기호들(x,y,z)의 해석값을 구하는 클래스(즉 x일때 무엇, y일때 무엇, z일때 무엇으로 해석하나)
- `NonterminalExpression` : 기호만으로는 결과가 나오지 않고 특정 `Expression`들을 참조해서 결과를 얻어야하는 클래스(+, -만으로는 결과가 나오지 않고 z, x, y과 같은 `TerminalExpression`, 또는 또 다른 `NontermialExpresion` 참조해서 결과값을 얻을 때까지 재귀적으로 참조한다. )
- `Context` : 표현을 분석할 때 사용될 포괄적인 정보
- `Client` : 언어로 정의한 특정 문장을 나타내는 클래스. `interpret()`를 호출하는 클래스





## 예시 코드

```java
//AbstractExpression
//모든 표현에서 공통적으로 사용하는 메소드 선언
public interface PostfixExpression {
    int interpret(Map<Character, Integer> context);
}

//TerminalExpression
//특정 기호들에 대한 해석값을 정의
public class VariableExpression implements PostfixExpression {

    private Character character;

    public VariableExpression(Character character) {
        this.character = character;
    }

    @Override
    public int interpret(Map<Character, Integer> context) {
        return context.get(this.character);
    }
}

//NonterminalExpression
//받은 기호만으로 결과가 나오지 않아 다른 Expression들을 참조하여 결과 도출
public class PlusExpression implements PostfixExpression {
    private PostfixExpression left;
    private PostfixExpression right;
    public PlusExpression(PostfixExpression left, PostfixExpression right) {
        this.left = left;
        this.right = right;
    }
    @Override
    public int interpret(Map<Character, Integer> context) {
        return left.interpret(context) + right.interpret(context);
    }
}

//NonterminalExpression
//받은 기호만으로 결과가 나오지 않아 다른 Expression들을 참조하여 결과 도출
public class MinusExpression implements PostfixExpression {
    private PostfixExpression left;
    private PostfixExpression right;
    public MinusExpression(PostfixExpression left, PostfixExpression right) {
        this.left = left;
        this.right = right;
    }
    @Override
    public int interpret(Map<Character, Integer> context) {
        return left.interpret(context) - right.interpret(context);
    }
}

//NonterminalExpression
//받은 기호만으로 결과가 나오지 않아 다른 Expression들을 참조하여 결과 도출
public class MultiplyExpression implements PostfixExpression{
    private PostfixExpression left, right;
    public MultiplyExpression(PostfixExpression left, PostfixExpression right) {
        this.left = left;
        this.right = right;
    }
    @Override
    public int interpret(Map<Character, Integer> context) {
        return left.interpret(context) * right.interpret(context);
    }
}

//Client
//받은 문자열을 Expression들을 이용해 파싱하는 리턴 값은 Expression 객체들로 이루어진 Expression을 반환
public class PostfixParser {
    public static PostfixExpression parse(String expression) {
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

public class App {
    public static void main(String[] args) {
        PostfixExpression expression = PostfixParser.parse("xyz+-a+");
				//여기서 Map은 Context를 의미
        int result = expression.interpret(Map.of('x', 1, 'y', 2, 'z', 3, 'a', 4));
        System.out.println(result);
    }
}
```

결국 이 패턴은 정말 특정 언어(SQL)를 해석할 때에만 크게 효율적인 패턴이라는 것을 알 수 있다.

AST트리 형태로 해석하는 것에 도움을 주는 방식이다. 

![Study] Abstract Syntax Tree (AST)](https://t1.daumcdn.net/cfile/tistory/2468CD39516B9E2B30)



컴포짓 패턴과 상당히 유사하게 생김. 즉, 이 표현식 방식이 결국 트리구조를 가지게 되는 형태이다.

컴포짓도 컴포넌트안에 컴포넌트가 아래로 계속 존재하게 되고 트리구조처럼 이뤄지는 면에서는 또 인터프리터 패턴이 또 유사한 점이 있다. 

![](https://gmlwjd9405.github.io/images/design-pattern-composite/composite-pattern.png)

컴포짓은 컴포넌트에 재귀형식으로 구현되어있다면, 인터프리터 패턴은 `non-terminalExpression`이 재귀형식으로 내려가고 Leaf 역할을 `terminalExpression`이 역할을 대체하는 것 같다. 





## 장점 / 단점

### 장점

- 자주 사용하는 패턴을 우리만의 문법으로 정의해서 재사용할 수 있음
- 기존 코드 변경하지 않고 새로운 `Expression` 만드는 것이 가능 (OCP)

### 단점

- 너무 복잡함
- 문법이 복잡하면, 구현이 어려움.
- 구성 가성비가 떨어질수도 있음. 굳이... 이게 정말 필요한지에 대해서 잘 판단해서 구현하는게 좋을 수 있다.





## 어디에서 쓰는지?

### 자바

- Compiler
- Regex ⇒ Pattern.matches()

### 스프링

Spring Expression Language ⇒ `SpEL`

```java
public class SpelExpressionParser extends TemplateAwareExpressionParser {

}
```

파서가 따로있고, 그 컴파일러도 존재

```java
public final class SpelCompiler implements Opcodes {

}
```

여기서 대부분의 표현식이 컴파일링 된다.