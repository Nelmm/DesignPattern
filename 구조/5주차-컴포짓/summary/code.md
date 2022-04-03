## 2. 컴포짓 패턴(Composite Pattern)이란?

객체들의 관계를 트리 구조로 구성하여 `부분-전체 계층`을 표현하는 패턴으로, 사용자가 `단일 객체`와 `복합 객체` 모두 동일한 객체로 인식할 수 있는 구조로 이루어 진다.



## 코드

### 요구사항
IT전략팀의 이창섭 팀원님이 개발하신 `mohani (mohani.saraminhr.co.kr/board)` 프로젝트를 컴포짓 패턴을 이용하여 예시를 들어보도록 하겠습니다.

해당 서비스를 이용해보면 가장 먼저보이는 기능은 다음과 같습니다.

1. 팀단위 통합적인 이슈 시작 ~ 종료 날짜를 보여주는 기능
2. 각 멤버 별 통합적인 이슈 시작 ~ 종료 날짜 보여주는 기능
3. 각각의 이슈의 세부사항을 보여주는 기능

이슈정보라는 단일 객체들의 집합을 가지고 있는 멤버와 멤버들을 가지고있는 팀객체에서 `가장 빠른 이슈 시작 / 가장 늦은 이슈 종료 날짜`를 조회하는 기능은 자기가 가지고 있는 객체의 정보들 중에 min/max를 구하는 것으로 처리 방법이 같습니다.

때문에, 이슈 날짜를 조회하는 행동을 동일한 인터페이스로 분리하여 취급 할 수 있게 됩니다. 이런 경우 `컴포짓 패턴`을 적용할 수 있습니다.

해당 예제에서는 `가장 빠른 이슈 시작 조회 / 가장 늦은 이슈 종료 날짜 조회` 라는 행위의 집합을 IssueUser라고 명명하고 코드를 한 번 보겠습니다.

```java
//web에 노출할 이슈에 대한 정보를 담은 VO클래스
public class IssueInfo {
    private final String name;
    private final LocalDate startDate;
    private final LocalDate endDate;
    private final String state;
    private final LocalDateTime updatedDt;
    private final String manager;
    private final String link;
    private final String reporter;
    private final String creator;

    public IssueInfo(String name,LocalDate startDate, LocalDate endDate, String state, LocalDateTime updatedDt, String manager, String link, String reporter, String creator) {
        this.name = name;
        this.startDate = startDate;
        this.endDate = endDate;
        this.state = state;
        this.updatedDt = updatedDt;
        this.manager = manager;
        this.link = link;
        this.reporter = reporter;
        this.creator = creator;
    }

    public String getName() {
        return name;
    }

    public LocalDate getStartDate() {
        return startDate;
    }

    public LocalDate getEndDate() {
        return endDate;
    }
}


//Component
//이슈에 대한 정보를 보여주기위한 서비스인터페이스
public interface IssueUser {
    LocalDate getEarliestStartDateInIssues();   //속한 이슈들 중 가장 빠른 이슈시작날짜
    LocalDate getLatestEndDateInIssues();       //속한 이슈들 중 가장 느린 이슈 종료날짜
}

//Composite
//사람인 개발팀, 팀원들의 집합정보
public class SaraminDevelopmentTeam implements IssueUser{
    private final String name = "사람인개발팀";
    private final List<IssueUser> memberList;

    public SaraminDevelopmentTeam(List<IssueUser> memberList) {
        this.memberList = memberList;
    }

    //사람인 개발팀의 가장 빠른 이슈 시작날짜
    @Override
    public LocalDate getEarliestStartDateInIssues() {
        if(this.memberList.isEmpty()){
            return LocalDate.EPOCH;         //팀원이 없다면 1970.01.01 UTC 00:00:00
        }
        return memberList.stream().min(Comparator.comparing(IssueUser::getEarliestStartDateInIssues)).orElseThrow().getEarliestStartDateInIssues();    //팀의 속한 멤버들의 가장 빠른 이슈시작날짜 중 가장 빠른 이슈 시작 날짜를 가진 멤버 find하여 날짜 get
    }

    //사람인 개발팀의 가장 늦은 이슈 종료날짜
    @Override
    public LocalDate getLatestEndDateInIssues() {
        if(this.memberList.isEmpty()){
            return LocalDate.EPOCH;     //팀원이 없다면 1970.01.01 UTC 00:00:00
        }
        return memberList.stream().min(Comparator.comparing(IssueUser::getLatestEndDateInIssues)).orElseThrow().getLatestEndDateInIssues();     //팀의 속한 멤버들의 이슈 종료날짜 중 가장 늦은 이슈 종료 날짜를 가진 멤버 find 하여 날짜 get
    }

    public List<IssueUser> getIssueUserList() {
        return issueUserList;
    }
}

//Leaf
//팀원 한명의 정보, 멤버가 Leaf객체로 Issue정보들의 집합을 가지고 있다.
public class Member implements IssueUser {
    private final String name;
    private final List<IssueInfo> issueInfoList;    //이슈정보들 집합

    public Member(String name, List<IssueInfo> issueInfoList) {
        this.name = name;
        this.issueInfoList = issueInfoList;
    }

    @Override
    public LocalDate getEarliestStartDateInIssues() {
        if(this.issueInfoList.isEmpty()){
            return LocalDate.EPOCH;
        }
        return issueInfoList.stream().min(Comparator.comparing(IssueInfo::getStartDate)).orElseThrow().getStartDate();  //가진 이슈들 중 가장 빠른 시간 get
    }

    @Override
    public LocalDate getLatestEndDateInIssues() {
        if(this.issueInfoList.isEmpty()){
            return LocalDate.EPOCH;
        }
        return issueInfoList.stream().min(Comparator.comparing(IssueInfo::getEndDate)).orElseThrow().getEndDate();      //가진 이슈들 중 가장 늦은 시간 get
    }
}
```

```java
//사용 예제 코드
public class Client {
    public static void main(String[] args) {
        //임의의 멤버와 이슈 생성
        //이 부분의 코드는 db, api 등 에서 조회후 해당 객체로 mapping 시켜 생성함으로써 대체 될 수 있다.
        Member member1 = new Member("고길동", List.of(
                new IssueInfo("ITIN-1554",
                        LocalDate.of(2022,1,30), LocalDate.of(2022,2,22),
                        "진행중", LocalDateTime.of(LocalDate.of(2022,3,31), LocalTime.now()),
                            "관리자1", "https://example.com", "보고자1", "고길동"),
                new IssueInfo("기업정보뷰 추가",
                        LocalDate.of(2021,12,31), LocalDate.of(2022,2,22),
                        "진행중", LocalDateTime.of(LocalDate.of(2022,4,12), LocalTime.now()),
                        "관리자2", "https://example.com", "보고자2", "고길동"))
        );
        Member member2 = new Member("김둘리",List.of(
                new IssueInfo("노쇼 이벤트",
                        LocalDate.of(2021,10,31), LocalDate.of(2022,2,22),
                        "완료", LocalDateTime.of(LocalDate.of(2021,11,1), LocalTime.now()),
                        "관리자1", "https://example.com", "보고자3", "김둘리"),
                new IssueInfo("채용관리형",
                        LocalDate.of(2022,2,28), LocalDate.of(2022,2,22),
                        "진행중", LocalDateTime.of(LocalDate.of(2022,4,28), LocalTime.now()),
                        "관리자2", "https://example.com", "보고자4", "김둘리"))
        );

        //사람인 서비스 개발팀에 위에서 생성한 고길동, 고둘리 멤버 추가
        SaraminDevelopmentTeam saraminDevelopmentTeam = new SaraminDevelopmentTeam(List.of(member1,member2));

        System.out.println(saraminDevelopmentTeam.getEarliestStartDateInIssues());  //사람인 개발팀의 가장 빠른 이슈 시작 날짜 조회
        System.out.println(saraminDevelopmentTeam.getLatestEndDateInIssues());      //사람인 개발팀의 가장 늦은 이슈 종료 날짜 조회
        for(IssueUser member: saraminDevelopmentTeam.getIssueList()){
            System.out.println(member.getEarliestStartDateInIssues());      //사람인 개발팀에 속한 특정 멤버의 가장 빠른 이슈 시작 날짜 조회
            System.out.println(member.getLatestEndDateInIssues());          //사람인 개발팀에 속한 특정 멤버의 가장 늦은 이슈 종료 날짜 조회
        }
    }
}
```