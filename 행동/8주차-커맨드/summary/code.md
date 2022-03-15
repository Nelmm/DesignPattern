## 코드
### 요구사항
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

이력서의 종류가 추가될 수록 서비스가 담고있는 비즈니스 로직은 무거워지고 요구사항이 변경될때마다 서비스의 객체가 빈번히 수정이 일어나게 된다. 수정히 빈번히 발생하게 되면 다른 곳에는 영향이 있는지없는지 체크하기 힘들며, 테스트코드를 제아무리 꼼꼼히 작성했다하더라도 많은 객체의 의존성이 묶여있어 단위테스트가 어려워지게 된다. 

이를 커맨드 패턴을 이용해서 저장,조회,삭제에 대한 행동을 더 세부객체에게 위임해보자.

### 커맨드 패턴 적용 후
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
