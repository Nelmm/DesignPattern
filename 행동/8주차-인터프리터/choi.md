# 인터프리터 패턴

> 문법 규칙을 클래스화 한 구조로, **일련의 규칙**으로 정의된 **문법적 언어**를 해석하는 패턴
>

![Untitled](https://user-images.githubusercontent.com/32676275/149166785-f682ffa3-44a1-447f-8ccc-d558d7d36750.png)

어떠한 해석을 정의하여 특정 값이 왔을 때 정의된 해석에 따라 결과를 도출하는 패턴이라 볼 수 있다.

- `AbstractExpression` : 모든 표현에서 공통적으로 사용할 `interpret()`를 정의한 추상클래스
- `TerminalExpression` : 표현에서 사용되는 기호들(x,y,z)의 해석값을 구하는 클래스(즉 x일때 무엇, y일때 무엇, z일때 무엇으로 해석하나)
- `NonterminalExpression` : 기호만으로는 결과가 나오지 않고 특정 `Expression`들을 참조해서 결과를 얻어야하는 클래스(+, -만으로는 결과가 나오지 않고 z, x, y과 같은 `TerminalExpression`, 또는 또 다른 `NontermialExpresion` 참조해서 결과값을 얻을 때까지 재귀적으로 참조한다. )
- `Context` : 표현을 분석할 때 사용될 포괄적인 정보
- `Client` : 언어로 정의한 특정 문장을 나타내는 클래스. `interpret()`를 호출하는 클래스

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

또 보면 Composite 패턴과 많이 비슷한 것을 볼 수 있다. composite 패턴도 결국은 부분과 전체 객체를 똑같은 선상에 표현하기에 NonterminalExpression이 다른 Expression들을 모으는 거랑 비슷하다 볼 수 있다.

> 장점
>
- 자주 등장하는 패턴을 기호와 문법으로 정의할 수 있다
- 기존 코드 변경하지 않고 새로운 `Expression` 만드는 것이 가능 (OCP)

> 단점
>
- 만드는 것 자체가 어려움
- 표현이 너무 적거나 많아도 이 패턴을 쓰기에는 적절하지 않은 것 같음