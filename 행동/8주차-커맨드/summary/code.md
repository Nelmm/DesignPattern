## 1. 커맨드 패턴(Command Pattern)이란?

> 요청을 캡슐화 하여 호출자(invoker)와 수신자(receiver)를 분리하는 패턴

요청을 호출하는 쪽과 수신하는 쪽을 커맨드 객체를 사용해서 **디커플링**하여 분리시키는게 핵심이다.

요청을 처리하는 방법이 바뀌더라도, 호출자의 코드는 변경되지 않는다.

코드를 통해 실제로 필요한 상황을 알아보자.

![command](https://user-images.githubusercontent.com/79291114/162621083-24761d9f-91af-4675-946b-df74a4062ad5.PNG)

- `Invoker` (Button) : Command 에서 제공하는 메소드를 실행함
  - 주로 `execute()` 메소드를 호출하고, 경우에 따라 undo() 와 같은 메소드도 사용
- `Command` : 구체적인 커맨드를 추상화 시킨 인터페이스
  - Interface 또는 abstract 로 선언
- `ConcreteCommand` : 구현체, 상속받은 클래스
  - 실제로 어떤 Receiver를 사용할지, 요청에 대한 작업을 수행하는 메소드와 파라미터가 필요한지 구현
- `Receiver (Light & Police)` : 실제 요청에 대한 작업을 수행

위와 같이 커맨드 패턴은 Command 를 재사용할 수 있는 장점이 있고, Command를 로깅한다거나, undo 등 추가적인 작업을 관리하기 쉬워진다.





## 2. 코드로 알아보는 커맨드 패턴

### 2-1. 요구사항
사람인서비스에 이력서를 저장하고 조회하고 삭제하는 서비스를 개발하고자 할때 `사람인 양식`과 `Pdf`를 통한 이력서 생성 서비스를 제공하고 있으며 해당 양식으로 생성한 이력서마다 저장하는 과정에서 내부 파라미터를 셋팅하는 로직이 다르다고 가정을 해보자.

```java
//이력서 모델
//편의를 위해 모든 데이터 String 형으로 대체
public class Resume {
    private String basicInfo;       //기본 정보
    private String careers;         //경력 사항
    private String educations;      //교육 사항
    private String activities;      //대외 활동
    private String certificates;    //자격증
    private String skills;          //보유 기술
    private String preferential;    //우대 사항
    private String portpolio;       //포트폴리오
    private String selfIntroduction;//자기소개서
    private String resumeType;      //이력서 타입(saramin,pdf)
    private String pdfFile;         //pdf file
    private String documents;       //추가 문서,자료
}

//사람인 폼 이력서 정보 DTO
public class SaraminFormResume {
    private String basicInfo;
    private String careers;
    private String educations;
    private String activities;
    private String certificates;
    private String skills;
    private String preferential;
    private String portpolio;
    private String selfIntroduction;

    public Resume toEntity(){
        return new Resume(basicInfo,careers,educations,activities,certificates,skills,preferential,portpolio,selfIntroduction,
                "saramin",null,null);
    }
}

//PDF 이력서 정보 DTO
public class FileFormResume {
    private String basicInfo;
    private String careers;
    private String educations;
    private String pdfFile;
    private String documents;

    public Resume toEntity(){
        return new Resume(basicInfo,careers,educations,null,null,null,null,null,
                null, "pdf",pdfFile,null,documents);
    }
}

//이력서 서비스
@RequiredArgsConstructor
public class ResumeService {
    private final ResumeRepository resumeRepository;

    //이력서 저장
    public void saveResume(Resume resume) {
        //공통 로직
        System.out.println("====이력서 저장====");
        System.out.print("기본정보, 경력사항 등 기본 값 validate와 셋팅");

        //이력서 타입에 따른 개별 비즈니스 로직 수행
        if (resume.getResumeType().equals("saramin")) {
            System.out.println("사람인 이력서 제외 item 조회");
            System.out.println("불러오기 가능한 이력서 리스트 조회");
            System.out.println("인적성 응시 시험 리스트 조회");
        }else if(resume.getResumeType().equals("file")) {
            System.out.println("파일 정보 파라미터 할당");
            System.out.println("파일 이력서 제외 item 조회");
            System.out.println("파일 썸네일 추출");
            System.out.println("파일 서버에 업로드");
        }
        resumeRepository.save(resume);
    }
}



//DB이용하는 Repository가 아닌 Stub객체
//이력서 Repository
public class ResumeRepository {
    public static final int SARAMIN_RESUME_ID = 1;  //사람인 폼 이력서 id
    public static final int FILE_RESUME_ID = 2;     //File 폼 이력서 id

    //이력서 저장
    public void save(Resume resume) {
        System.out.println(resume.getResumeType() + " 타입의 이력서 저장");
    }
}


//Controller와 같이 Service를 사용하는 객체
public class Client {
    public static void main(String[] args) {
        ResumeService resumeService = new ResumeService(new ResumeRepository());

        //사람인폼으로 이력서 저장
        resumeService.saveResume(new SaraminFormResume().toEntity());
        System.out.println();

        //Pdf로 이력서 저장
        resumeService.saveResume(new FileFormResume().toEntity());
    
    }
}

결과 :
====이력서 저장====
기본정보, 경력사항 등 기본 값 validate와 셋팅사람인 이력서 제외 item 조회
불러오기 가능한 이력서 리스트 조회
인적성 응시 시험 리스트 조회
saramin 타입의 이력서 저장

====이력서 저장====
기본정보, 경력사항 등 기본 값 validate와 셋팅파일 정보 파라미터 할당
파일 이력서 제외 item 조회
파일 썸네일 추출
파일 서버에 업로드
file 타입의 이력서 저장
```

주어진 요구사항을 필요한 부분만 나타내어 개발해보면 위와 같이 개발 할 수 있을 것이다. 여기서 만약에 `URL`을 통한 이력서 서비스를 추가로 제공해야 한다면 `Service`에 초점을 맞춰서 보면 아래와 같이 코드를 추가해야 할 것이다.

```java
//새로 추가된 URL폼 이력서 DTO
public class UrlFormResume {
    private String basicInfo;
    private String careers;
    private String educations;
    private String url;
    private String documents;

    public Resume toEntity(){
        return new Resume(basicInfo,careers,educations,null,null,null,null,null,
                null, "url",null,url,documents);
    }
}

@RequiredArgsConstructor
public class ResumeService {
    private final ResumeRepository resumeRepository;

    public void saveResume(Resume resume) {
        System.out.println("====이력서 저장====");
        System.out.print("기본정보, 경력사항 등 기본 값 validate와 셋팅");
        if (resume.getResumeType().equals("saramin")) {
            System.out.println("사람인 이력서 제외 item 조회");
            System.out.println("불러오기 가능한 이력서 리스트 조회");
            System.out.println("인적성 응시 시험 리스트 조회");
        }else if(resume.getResumeType().equals("file")) {
            System.out.println("파일 정보 파라미터 할당");
            System.out.println("파일 이력서 제외 item 조회");
            System.out.println("파일 썸네일 추출");
            System.out.println("파일 서버에 업로드");
        }else if(resume.getResumeType().equals("url")) {        //새롭게 추가되는 분기(비즈니스 로직)
            System.out.println("URL 이력서 제외 item 조회");
            System.out.println("URL 통해 html 다운");
            System.out.println("URL 썸네일 추출");
            System.out.println("파일 서버에 업로드");
        }
        resumeRepository.save(resume);
    }
}

public class Client {
    public static void main(String[] args) {
        //URL로 이력서 저장
        resumeService = new ResumeService(new UrlResumeCommand(),new ResumeRepository());
        resumeService.saveResume(new UrlFormResume().toEntity());
    }
}
결과 :
====이력서 저장====
기본정보, 경력사항 등 기본 값 validate와 셋팅URL 이력서 제외 item 조회
URL 통해 html 다운
URL 썸네일 추출
파일 서버에 업로드
url 타입의 이력서 저장
```

이력서의 종류가 추가될 수록 서비스가 담고있는 비즈니스 로직은 무거워지고 요구사항이 변경될때마다 서비스의 객체가 빈번히 수정이 일어나게 된다. 

또한, 수정이 빈번히 발생하게 되면 다른 곳에는 영향이 있는지없는지 체크하기 힘들다.

 테스트코드를 제아무리 꼼꼼히 작성했다하더라도 많은 객체의 의존성이 묶여있어 단위테스트가 어려워지게 된다. 

이를 커맨드 패턴을 이용해서 저장,조회,삭제에 대한 행동을 더 세부객체에게 위임해보자.



### 2-2. 커맨드 패턴 적용 후

```java
//Command 인터페이스
public interface ResumeCommand {
    void saveResume();     //저장 명령 
}

//Concrete Command
//사람인 폼 이력서 Command
@RequiredArgsConstructor
public class SaraminResumeCommand implements ResumeCommand {
    private final Resume resume;    //Receiver
    private final ResumeRepository resumeRepository; //Receiver

    //저장 세부 비즈니스로직을 수행
    @Override
    public void saveResume() {
        System.out.print("기본정보, 경력사항 등 기본 값 validate와 셋팅");
        System.out.println(resume.getResumeType() + " 이력서 제외 item 조회");
        System.out.println("불러오기 가능한 이력서 리스트 조회");
        System.out.println("인적성 응시 시험 리스트 조회");
        resumeRepository.save(resume);
    }
}

//Concrete Command
//파일 폼 이력서 Command
@RequiredArgsConstructor
public class FileResumeCommand implements ResumeCommand {
    private final Resume resume;    //Receiver
    private final ResumeRepository resumeRepository; //Receiver

    //저장 세부 비즈니스로직을 수행
    @Override
    public void saveResume(Resume resume) {
        System.out.print("기본정보, 경력사항 등 기본 값 validate와 셋팅");
        System.out.println(resume.getResumeType() + " 이력서 제외 item 조회");
        System.out.println("파일 정보 파라미터 할당");
        System.out.println("파일 썸네일 추출");
        System.out.println("파일 서버에 업로드");
        resumeRepository.save(resume);
    }
}

//Concrete Command
//URL 폼 이력서 Command
@RequiredArgsConstructor
public class UrlResumeCommand implements ResumeCommand {
    private final Resume resume;    //Receiver
    private final ResumeRepository resumeRepository; //Receiver

    //저장 세부 비즈니스로직을 수행
    @Override
    public void saveResume(Resume resume) {
        System.out.print("기본정보, 경력사항 등 기본 값 validate와 셋팅");
        System.out.println(resume.getResumeType() + " 이력서 제외 item 조회");
        System.out.println("URL 통해 html 다운");
        System.out.println("URL 썸네일 추출");
        System.out.println("파일 서버에 업로드");
        resumeRepository.save(resume);
    }
}

//이력서 서비스
//Invoker
@RequiredArgsConstructor
public class ResumeService {
    private final ResumeCommand resumeCommand;  

    public void saveResume(Resume resume) {
        System.out.println("====이력서 저장====");
        resumeCommand.saveResume();     //명령을 통해 비즈니스 로직 수행을 세부객체에게 위임
    }
}

public class Client {
    public static void main(String[] args) {
        //사람인폼으로 이력서 저장
        Resume resume = new SaraminFormResume().toEntity();
        ResumeService resumeService = new ResumeService(new SaraminResumeCommand(resume,new ResumeRepository()));
        resumeService.saveResume();

        System.out.println();

        //Pdf로 이력서 저장
        resume = new FileFormResume().toEntity();
        resumeService = new ResumeService(new FileResumeCommand(resume,new ResumeRepository()));
        resumeService.saveResume();

        System.out.println();

        //URL로 이력서 저장
        resume = new UrlFormResume().toEntity();
        resumeService = new ResumeService(new UrlResumeCommand(resume,new ResumeRepository()));
        resumeService.saveResume();
    }
}
결과 :
====이력서 저장====
기본정보, 경력사항 등 기본 값 validate와 셋팅saramin 이력서 제외 item 조회
불러오기 가능한 이력서 리스트 조회
인적성 응시 시험 리스트 조회
saramin 타입의 이력서 저장

====이력서 저장====
기본정보, 경력사항 등 기본 값 validate와 셋팅file 이력서 제외 item 조회
파일 정보 파라미터 할당
파일 썸네일 추출
파일 서버에 업로드
file 타입의 이력서 저장

====이력서 저장====
기본정보, 경력사항 등 기본 값 validate와 셋팅url 이력서 제외 item 조회
URL 통해 html 다운
URL 썸네일 추출
파일 서버에 업로드
url 타입의 이력서 저장
```





## 3. 장점과 단점

### 3-1. 장점

- 기존 코드를 변경하지 않고 새로운 Command를 만들 수 있음 `(OCP 원칙)`
- 수신자(Receiver)의 코드가 변경되어도, 호출자(Invoker)의 코드는 변경되지 않음 `(단일책임원칙)`
- Command 객체를 로깅, DB에 저장, 네트워크로 전송하는 등 다양한 방법으로 활용 가능



### 3-2. 단점

- 대부분의 디자인 패턴가 마찬가지로 코드가 복잡하고 클래스가 많아짐





## 4. Command 패턴과 State 패턴과의 비교

공부를 하면서 `전략 패턴, 커맨드 패턴, 상태 패턴`이 비슷하다고 생각했는데, 이 3개 패턴의 다이어그램을 보면서 차이점을 간단히 정리했다.



### 4-1. 전략 패턴

[![strategy](https://user-images.githubusercontent.com/79291114/153312563-a830529c-8f3e-47f8-b047-9c84225f3401.PNG)](https://user-images.githubusercontent.com/79291114/153312563-a830529c-8f3e-47f8-b047-9c84225f3401.PNG)

여러 알고리즘을 캡슐화하고 상호 교환 가능하게 만드는 패턴

- 여러 알고리즘을 `상호 교환` 한다는 것은 이미 알고리즘이 실행되는 것은 정해져있고 해당 알고리즘만 교체해준다는 의미입니다.
- 다이어그램을 보면, **Client가 ConcreteStrategy를 교체**하는 것을 볼 수 있습니다.
- 즉, **행동이 정해져 있는 상태에서 어떠한 방법으로 수행**할지만 달라집니다.



### 4-2. 커맨드 패턴

[![command](https://user-images.githubusercontent.com/79291114/153312559-5b068040-1e19-493c-a680-499744f56f08.PNG)](https://user-images.githubusercontent.com/79291114/153312559-5b068040-1e19-493c-a680-499744f56f08.PNG)

요청을 캡슐화하여 호출자와 수신자를 분리하는 패턴

- 호출자와 수신자를 분리한다는 것은 `명령(호출자)`과 `행동(수신자)`을 분리한다는 의미입니다.
- 다이어그램을 보면, **Invoker에 따라서 다양한 Receiver가 호출**되는 것을 알 수 있습니다.
- 즉, **명령에 따라서 다른 행동을 수행**한다는 것입니다.



### 4-3. 상태 패턴

[![state](https://user-images.githubusercontent.com/79291114/153312562-2f70656c-3cd2-4537-8acd-5bfeea7dcfe6.PNG)](https://user-images.githubusercontent.com/79291114/153312562-2f70656c-3cd2-4537-8acd-5bfeea7dcfe6.PNG)

객체 내부 상태 변경에 따라 객체의 행동이 달라지는 패턴

- 말 그대로 **객체 내부 상태에 따라서 다른 행동을 수행**한다는 것입니다.
- 다이어그램을 보면, **Client가 changeState로 상태를 변경해줌에 따라 ConcreteState가 바뀌면서 다른 행동**을 하게됩니다.





## 5. 마치며

일단 전략 패턴, 커맨드 패턴, 상태 패턴과 같이 비슷한 패턴은 생김새로 구분하는 것이 아니라, 목적으로 구분해야 한다.

커맨드 패턴은 명령에 따라 커맨드 구현체를 교체하여 다른 행동을 할 수 있기 때문에 유용하게 사용할 수 있다. 

