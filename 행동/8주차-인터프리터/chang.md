# 인터프리터 패턴

자주 등장하는 문제를 간단한 언어로 정의하고 재사용하는 패턴

반복되는 문제 패턴을 언어 또는 문법으로 정의하고 확장할 수 있다.

![](https://sourcemaking.com/files/v2/content/patterns/Interpreter_example1.png?id=6e18c37efd5aa7086a60)

[Interpreter Design Pattern](https://sourcemaking.com/design_patterns/interpreter)

종료되는 표현식, 종료되지 않는 표현식(다른 표현식을 참조하는 표현식) 을 바탕으로 섞어서 현 문장을 해석하는 방식으로 진행되어진다. 

UML을 보자면 다음과 같은데,

![Interpreter Pattern | 밥줄과 취미 사이 ːː 못 먹어도 고!](https://dejavuhyo.github.io/assets/img/2021-01-28-interpreter-pattern/img001.png)

AST트리 형태로 해석하는 것에 도움을 주는 방식이다. 

![Study] Abstract Syntax Tree (AST)](https://t1.daumcdn.net/cfile/tistory/2468CD39516B9E2B30)



컴포짓 패턴과 상당히 유사하게 생김. 즉, 이 표현식 방식이 결국 트리구조를 가지게 되는 형태이다.

컴포짓도 컴포넌트안에 컴포넌트가 아래로 계속 존재하게 되고 트리구조처럼 이뤄지는 면에서는 또 인터프리터 패턴이 또 유사한 점이 있다. 

![](https://gmlwjd9405.github.io/images/design-pattern-composite/composite-pattern.png)

컴포짓은 컴포넌트에 재귀형식으로 구현되어있다면, 인터프리터 패턴은 `non-terminalExpression`이 재귀형식으로 내려가고 Leaf 역할을 `terminalExpression`이 역할을 대체하는 것 같다. 



## 장점 / 단점

### 장점

- 자주 사용하는 패턴을 우리만의 문법으로 정의해서 재사용할 수 있음
- 코드 변경 없이도 확장 가능.

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