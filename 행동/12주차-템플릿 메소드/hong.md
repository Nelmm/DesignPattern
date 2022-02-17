# 템플릿 메서드 패턴
알고리즘 구조를 변경하지 않고 특정 단계의 수행방법을 재정의 하는 패턴으로 일종의 뼈대를 정의하는 패턴

다이어그램을 보면 알겠지만 특정로직의 뼈대가 담긴 슈퍼클래스가 존재해야 하기 때문에 기본적으로는 `상속`을 통해서 템플릿 메서드 패턴을 구현한다.  물론 Java8부터 생긴 interface의 default 키워드를 통해 해결할 수는 있다. (default 키워드의 원래 목적은 해당 메서드를 정의하지않아도 기본 동작을 설명해주는것이 목적이고 구현체들은 재정의가 가능해야하기에 `final`이 불가능하다. => 백기선님이 설명하신 리스코프치환원칙을 지킬수 있는 그나마의 방법이 불가능.)

또한, golang 처럼 상속을 지원하지 않고, 인터페이스에 default 키워드가 존재하지 않는다면 어떻게 템플릿 메서드 패턴을 이용할 수 있을까??

## Go 코드
```go
//
type Operator interface {
    GetResult(result,number int) int
}
type AddOperator struct {}
func (ao AddOperator)GetResult(result,number int) int {
    return result + number
}


//템플릿 메서드
type FileProcessor struct {
    path     string
    operator Operator
}

func NewFileProcessor (path string,operator Operator) *FileProcessor{
    return &FileProcessor{path: path, operator: operator}
}

func (fp *FileProcessor) process() (result int) {
    f, err := os.Open("")
	if err != nil {
		log.Fatal(fmt.Sprintf(fp.path + "에 해당하는 파일이 없습니다. error : %#v",err))
	}
	scanner := bufio.NewScanner(f)
	for scanner.Scan() {
		val,_ := strconv.Atoi(scanner.Text())
		result = fp.operator.GetResult(result,val)
	}
    return
}

func main(){
    fp := NewFileProcessor("number.txt",AddOperator{})
    result := fp.process()
    fmt.Println(result)
}
```

위처럼 템플릿 메서드가 정의된 클래스를 생성하는 타이밍에 추상적인 부분을 구현한 구현체를 주입해줌으로써 사용할 수 있다.


<br>

## 템플릿 콜백
```go
type Operator interface {
    GetResult(result,number int) int
}
type AddOperator struct {}
func (ao AddOperator)GetResult(result,number int) int {
    return result + number
}


//템플릿 메서드
type FileProcessor struct {
    path     string
}

func NewFileProcessor (path string) *FileProcessor{
    return &FileProcessor{path: path}
}

func (fp *FileProcessor) process(getResult func(x, y int) int) (result int) {
	f, err := os.Open("")
	if err != nil {
		log.Fatal(fmt.Sprintf(fp.path+"에 해당하는 파일이 없습니다. error : %#v", err))
	}
	scanner := bufio.NewScanner(f)

	for scanner.Scan() {
		val, _ := strconv.Atoi(scanner.Text())
		result = getResult(result, val)
	}
	return 
}


func main(){
    fp := NewFileProcessor("number.txt")
    result := fp.process(func(x, y int) int {
		return x + y
	})
    fmt.Println(result)
}
```
템플릿 콜백패턴은 템플릿메서드 패턴보다는 전략패턴과 비슷하다. 전략 패턴은 전략을 미리 setting하고 process를 돌렸었다면, 콜백패턴은 process하는 타이밍에 전략을 주입받아 사용한다. 

템플릿 콜백 패턴은 `콜백`이라는 이름에 맞게 매개변수로 `함수`를 받아 처리한다. java에서는 함수가 일급함수가 이니기 때문에 함수형 인터페이스와 람다, 익명객체를 이용하여 해결한다.

<br>

## 다른 패턴과의 차이점
1. 팩토리메서드 : 상속을 이용해서 특정 기능을 수행한다는 점이 매우 비슷한데 팩토리메서드패턴은 `객체 생성`이 목적이며 템플릿메서드패턴은 `행동 수행`이 목적이라는 점이 가장 큰 차이점이다.
2. 전략 패턴 : GoF가 정의한 패턴의 정의로만 보면 가장 큰 차이점은 전략 패턴은 `Composition`을 이용해서 행동을 정의하지만, 템플릿메서드 패턴은 `상속`을 이용해서 정의한다.
    
    하지만, 현대에 와서 템플릿 메서드패턴을 더 안전하고 효율적으로 구현을 하기위해 java의 default 키워드를 사용하거나 golang처럼 상속이 지원하지 않는 경우에 위의 예시 코드처럼 인터페이스를 주입하여 구현할 수 있는데 그러면서 두 패턴과의 벽이 허물어지고 있는 느낌이다. 
    <br>물론, 전략패턴은 다이어그램에도 있듯이 setStrategy가 존재해 런타임에 전략(행동)을 수정이 가능하나 템플릿메서드 패턴은 구체적인 클래스를 교체해야 가능하기에 컴파일타임에 수정이 된다는 차이점이 있다.
    
    또한, 목적부분의 차이점을 본다면 전략패턴을 행동을 추상화하여 특정 행동을 추상화하기에 인터페이스가 한개의 메서드만 정의하고 이러한 행동이 독립적이다.
    <br>템플릿메서드패턴은 행동이 정의되어있고 내부에 존재하는 여러 알고리즘을 바꿀수 있기에 여러 메서드를 추상화하고 각 메서드들의 결과가 다른 메서드의 결과에 영향을 미칠 수 있다. (매개변수의 값이 달라질 수 있고 조건문에 조건이 걸릴수도 안걸릴수도 있기 때문에)
