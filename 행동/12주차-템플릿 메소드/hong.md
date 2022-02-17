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

func (fp *FileProcessor) process() int {
    f, err := os.Open("")
	if err != nil {
		log.Fatal(fmt.Sprintf(fp.path + "에 해당하는 파일이 없습니다. error : %#v",err)
	}
	scanner := bufio.NewScanner(f)
	var result int

	for scanner.Scan() {
		val,_ := strconv.Atoi(scanner.Text())
		result = fp.operator.GetResult(result,val)
	}
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

func (fp *FileProcessor) process(operator Operator) int {
    f, err := os.Open("")
	if err != nil {
		log.Fatal(fmt.Sprintf(fp.path + "에 해당하는 파일이 없습니다. error : %#v",err)
	}
	scanner := bufio.NewScanner(f)
	var result int

	for scanner.Scan() {
		val,_ := strconv.Atoi(scanner.Text())
		result = operator.GetResult(result,val)
	}
}

func main(){
    fp := NewFileProcessor("number.txt")
    result := fp.process(AddOperator{})
    fmt.Println(result)
}
```
콜백패턴은 `전략 패턴`과 비슷한 형태인데 전략 패턴은 전략을 미리 setting하고 process를 돌렸었다면, 콜백패턴은 process하는 타이밍에 전략을 주입받아 사용
