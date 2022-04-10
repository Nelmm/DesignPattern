## 예제 코드

### 요구사항
1. 사람인에 근무하고 있는 사람들의 정보를 볼 수 있는 시스템을 만들고 싶어요.
2. 근무자들은 한 조직(팀)에 속해 있으니 팀의 정보도 포함해야 해요.

이를 위해 사람인의 근무자들의 정보를 DB나 api를 통하여 데이터를 불러와 각각 Member라는 객체로 만들어 서비스를 개발할 수 있을 것이다.

여기서 우리는 Member나는 객체를 만드는 과정에 집중해서 살펴 볼 것이다.

### 패턴 적용 전 코드
```java
@AllArgsConstructor
public class Member {
    private String name;        //이름
    private LocalDate birthDay; //생일
    private String description; //기타 설명
    private String position;    //직급
    private String organizationName; //조직이름
    private String teamName;         //팀이름
    private String partName;         //파트이름
}

public class Client {
    public static void main(String[] args) {
        Member member1 = new Member("고길동", LocalDate.of(1980,12,7),"종로로 갈까요?",
                "파트장","IT연구소","사람인개발팀","D1");
        Member member2 = new Member("고둘리", LocalDate.of(1995,4,8),"호잇!",
                "팀원","IT연구소","사람인개발팀","D1");
        Member member3 = new Member("또치", LocalDate.of(1988,2,7),"난 세상에서 가장 우아하고, 잘난 귀족 타조야",
                "파트장","IT연구소","사람인개발팀","D2");
        Member member4 = new Member("도우너", LocalDate.of(1992,8,17),"안녕하세요!!",
                "팀원","IT연구소","사람인개발팀","D2");
        Member member5 = new Member("마이클", LocalDate.of(1990,3,27),"안녕하세요!!",
                "팀원","IT연구소","사람인개발팀","D2");
    }
}
```
예시에서는 팀원이 5명밖에 안되긴 하지만 5명의 예시만 봐도 팀정보와 같이 중복되며 자주 변하지 않는 데이터가 보일 것이다. 

이런 부분들을 분류하고 플라이웨이트 패턴을 적용해서 메모리를 절약해보자.

### 적용 후 코드
```java
@AllArgsConstructor
public class Member {
    private String name;
    private LocalDate birthDay;
    private String description;
    private String position;
    private Team team;      //자주 변하지 않을 팀 정보를 따로 객체로 분리
}

//팀 정보 객체
//Flyweight
@AllArgsConstructor
public class Team {
    private String organizationName;
    private String teamName;
    private String partName;
}

//FlyWeight Factory
public class TeamFactory {
    private final Map<String, Team> cache = new HashMap<>();    //캐싱하기 위한 Map
    private final Pattern teamPattern = Pattern.compile("([a-zA-Z0-9가-힣 ]*):([a-zA-Z0-9가-힣 ]*):([a-zA-Z0-9가-힣 ]*)"); //team조회를 위한 입력 표현식 (조직이름:팀이름:파트이름)

    public Team getTeam(String team) {
        Matcher teamMatcher = teamPattern.matcher(team);

        //정의한 regxr과 다르다면 error
        if(!teamMatcher.matches()){
            throw new IllegalArgumentException();
        }

        if (cache.containsKey(team)) {
            return cache.get(team);     //캐싱되어있다면 바로 반환
        } else {
            Team newTeam = new Team(teamMatcher.group(1),teamMatcher.group(2),teamMatcher.group(3));    //새로운 팀 생성
            cache.put(team, newTeam);   //캐싱
            return newTeam;
        }
    }
}

//Client
//FlyWeight를 사용
public class Client {
    public static void main(String[] args) {
        TeamFactory teamFactory = new TeamFactory();

        Member member1 = new Member("고길동", LocalDate.of(1980,12,7),"종로로 갈까요?",
                "파트장",teamFactory.getTeam("IT연구소:사람인개발팀:D1"));
        Member member2 = new Member("고둘리", LocalDate.of(1995,4,8),"호잇!",
                "팀원",teamFactory.getTeam("IT연구소:사람인개발팀:D1"));
        Member member3 = new Member("또치", LocalDate.of(1988,2,7),"난 세상에서 가장 우아하고, 잘난 귀족 타조야",
                "파트장",teamFactory.getTeam("IT연구소:사람인개발팀:D2"));
        Member member4 = new Member("도우너", LocalDate.of(1992,8,17),"안녕하세요!!",
                "팀원",teamFactory.getTeam("IT연구소:사람인개발팀:D2"));
        Member member5 = new Member("마이클", LocalDate.of(1990,3,27),"안녕하세요!!",
                "팀원",teamFactory.getTeam("IT연구소:사람인개발팀:D2"));
    }
}
```