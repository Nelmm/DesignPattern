## 코드
### 요구사항
사람인서비스에 이력서를 저장하고 조회하고 삭제하는 서비스를 개발하고자 할때 `사람인 양식`과 `Pdf`를 통한 이력서 생성 서비르를 제공하고 있으며 해당 양식으로 생성한 이력서마다 조회와 삭제에 대한 비즈니스 로직이 다르다고 가정을 해보자.

간단하게 개발을 해보면 아래와 같이 개발할 수 있을 것이다.

```java
//이력서 모델
//편의를 위해 모든 데이터 String 형으로 대체
public class Resume {
    private String basicInfo;
    private String careers;
    private String educations;
    private String activities;
    private String certificates;
    private String skills;
    private String preferential;
    private String portpolio;
    private String selfIntroduction;
    private String resumeType;
    private String pdfFile;
    private String documents;
}

//사람인 폼 이력서 정보 DTO
public class SaraminFormDto {
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
public class PdfFormDto {
    private String basicInfo;
    private String careers;
    private String educations;
    private String pdfFile;
    private String documents;

    public Resume toEntity(){
        return new Resume(basicInfo,careers,educations,null,null,null,null,null,
                null, "pdf",pdfFile,documents);
    }
}

//이력서 서비스
@RequiredArgsConstructor
public class ResumeService {
    private final ResumeRepository resumeRepository;

    //사람인 폼 이력서 저장
    public void saveResumeBy(SaraminFormDto resumeData) {
        System.out.println("===사람인 이력서 작성===");
        resumeRepository.save(resumeData.toEntity());
    }

    //PDF 이력서 저장
    public void saveResumeBy(PdfFormDto resumeData) {
        System.out.println("===PDF로 이력서 작성===");
        resumeRepository.save(resumeData.toEntity());
    }

    //이력서 정보 조회
    public void getResume(int resumeId) {
       resumeRepository.getResumeById(resumeId);
    }

    //이력서 삭제
    public void deleteResume(int resumeId) {
        resumeRepository.deleteResumeById(resumeId);

    }
}

//DB이용하는 Repository가 아닌 Stub객체
//이력서 Repository
public class ResumeRepository {
    public static final int SARAMIN_RESUME_ID = 1;  //id가 1일 경우 사람인폼 이력서
    public static final int PDF_RESUME_ID = 2;      //id가 2일 경우 pdf 이력서

     public void save(Resume resume) {
        System.out.println(resume.getResumeType() + " 타입의 이력서 저장");
    }

    public void getResumeById(int id) {
            System.out.println("===이력서 조회===");
        if (id == SARAMIN_RESUME_ID) {
            System.out.println("1. 이력서 정보 반환");
        }else if (id == PDF_RESUME_ID){
            System.out.println("1. pdf 이미지로 변환");
            System.out.println("2. 이력서 정보 반환");
        }
    }

    public void deleteResumeById(int id) {
            System.out.println("===이력서 삭제===");
        if (id == SARAMIN_RESUME_ID) {
            System.out.println("1. 이력서 데이터 삭제");
        }else if (id == PDF_RESUME_ID){
            System.out.println("1. 이력서 데이터 삭제");
            System.out.println("2. pdf 파일 삭제");
        }
    }
}

//Controller와 같이 Service를 사용하는 객체
public class Client {
    public static void main(String[] args) {
        ResumeService resumeService = new ResumeService(new ResumeRepository());

        //사람인폼으로 이력서 저장
        resumeService.saveResumeBy(new SaraminFormDto());
        resumeService.getResume(SARAMIN_RESUME_ID);
        resumeService.deleteResume(SARAMIN_RESUME_ID);
        System.out.println();

        //Pdf로 이력서 저장
        resumeService.saveResumeBy(new PdfFormDto());
        resumeService.getResume(PDF_RESUME_ID);
        resumeService.deleteResume(PDF_RESUME_ID);
    }
}

결과 :
===사람인 이력서 작성===
saramin 타입의 이력서 저장
===이력서 조회===
1. 이력서 정보 반환
===이력서 삭제===
1. 이력서 데이터 삭제

===PDF로 이력서 작성===
pdf 타입의 이력서 저장
===이력서 조회===
1. pdf 이미지로 변환
2. 이력서 정보 반환
===이력서 삭제===
1. 이력서 데이터 삭제
2. pdf 파일 삭제
```

주어진 요구사항을 필요한 부분만 나타내어 개발해보면 위와 같이 개발 할 수 있을 것이다. 여기서 만약에 `URL`을 통한 이력서 서비스를 추가로 제공해야 한다면 `Service`에 초점을 맞춰서 보면 아래와 같이 코드를 추가해야 할 것이다.

```java
public class UrlFormDto {
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

    public void saveResumeBy(SaraminFormDto resumeData) {
        System.out.println("===사람인 이력서 저장===");
        resumeRepository.save(resumeData.toEntity());
    }

    public void saveResumeBy(PdfFormDto resumeData) {
        System.out.println("===PDF로 이력서 저장===");
        resumeRepository.save(resumeData.toEntity());
    }


    //저장 메서드 추가
    public void saveResumeBy(UrlFormDto resumeData) {
        System.out.println("===URL로 이력서 저장===");
        resumeRepository.save(resumeData.toEntity());
    }

    public void getResume(int resumeId) {
        Resume resume = resumeRepository.getResumeById(resumeId);
        if (resume.getResumeType().equals("pdf")) {
            System.out.println("pdf 파일 이미지로 변환");
        }else if(resume.getResumeType().equals("url")) {        //이력서 조회 로직 추가
            System.out.println("url에서 html 다운");
        }
    }

    public void deleteResume(int resumeId) {
        Resume resume = resumeRepository.getResumeById(resumeId);

        resumeRepository.deleteResumeById(resumeId);
        if (resume.getResumeType().equals("pdf")) {     //만일 다른 로직이 필요하다면 로직 추가
            System.out.println("pdf 파일 삭제");
        }
    }
}
```