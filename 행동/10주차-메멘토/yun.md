# 메멘토(Memento) 패턴

메멘토 패턴은 **캡슐화를 유지하면서 객체 내부 상태를 외부에 저장하는 패턴**입니다.



## 1. 다이어그램

![memento](https://user-images.githubusercontent.com/79291114/151184142-6a60b443-3ba4-4eea-855a-981aaa82d9fd.PNG)

- `Originator` : 객체의 정보를 가지고 있는 오리지널 객체
  - 제 3자가 객체 내부 정보를 알지 못하게 하기 위하여 **Memento를 만들 수 있는 메서드가 필요**함
- `Memento` : 오리지널 객체의 정보를 담는 불변 객체
- `CareTaker` : 오리지널이 반환한 **Memento를 가지고 있다가 필요할 때 Memento의 값을 Originator에게 전달**함





## 2. 예시 코드

너무 간단한 패턴이라 코드만 봐도 쉽게 이해할 수 있을 거라 생각합니다.



### 2-1. 메멘토 패턴 적용 전

Game 객체의 정보를 복원하기 위하여 저장을 해야할 때 **외부에 객체의 정보를 노출함으로써 캡슐화를 지킬 수 없게 됩니다.**

```java
// 게임 객체
public class Game implements Serializable {

    private int redTeamScore;

    private int blueTeamScore;

    public int getRedTeamScore() {
        return redTeamScore;
    }

    public void setRedTeamScore(int redTeamScore) {
        this.redTeamScore = redTeamScore;
    }

    public int getBlueTeamScore() {
        return blueTeamScore;
    }

    public void setBlueTeamScore(int blueTeamScore) {
        this.blueTeamScore = blueTeamScore;
    }
}

// 클라이언트, CareTaker로 볼 수 있음
public class Client {

    public static void main(String[] args) {
        Game game = new Game();
        game.setRedTeamScore(10);
        game.setBlueTeamScore(20);
        
        // 게임을 복원하기 위해서 게임의 정보를 노출
        int blueTeamScore = game.getBlueTeamScore();
        int redTeamScore = game.getRedTeamScore();

        // 외부에 저장한 게임의 정보를 그대로 가져와서 새로운 게임을 만듬 -> 캡슐화 깨짐
        Game restoredGame = new Game();
        restoredGame.setBlueTeamScore(blueTeamScore);
        restoredGame.setRedTeamScore(redTeamScore);
    }
}
```



### 2-2. 메멘토 패턴 적용 후

Game 객체의 정보를 복원하기 위하여 저장을 해야할 때 **GameSave 불변 객체를 이용함으로써 외부에 객체의 정보를 감추어, 캡슐화**를 지킬 수 있습니다.

```java
// 게임 객체
public class Game {

    private int redTeamScore;

    private int blueTeamScore;

    public int getRedTeamScore() {
        return redTeamScore;
    }

    public void setRedTeamScore(int redTeamScore) {
        this.redTeamScore = redTeamScore;
    }

    public int getBlueTeamScore() {
        return blueTeamScore;
    }

    public void setBlueTeamScore(int blueTeamScore) {
        this.blueTeamScore = blueTeamScore;
    }

    public GameSave save() {
        return new GameSave(this.blueTeamScore, this.redTeamScore);
    }

    public void restore(GameSave gameSave) {
        this.blueTeamScore = gameSave.getBlueTeamScore();
        this.redTeamScore = gameSave.getRedTeamScore();
    }

}

// 게임 정보를 저장하는 불변 객체
public final class GameSave {

    private final int blueTeamScore;

    private final int redTeamScore;

    public GameSave(int blueTeamScore, int redTeamScore) {
        this.blueTeamScore = blueTeamScore;
        this.redTeamScore = redTeamScore;
    }

    public int getBlueTeamScore() {
        return blueTeamScore;
    }

    public int getRedTeamScore() {
        return redTeamScore;
    }
}

// 클라이언트, CareTaker로 볼 수 있음
public class Client {

    public static void main(String[] args) {
        Game game = new Game();
        game.setBlueTeamScore(10);
        game.setRedTeamScore(20);

        // 위와 다르게 클라이언트에서 게임의 정보를 가지고 있는 것이 아니라 불변 객체인 GameSave를 가짐
        GameSave save = game.save();

        game.setBlueTeamScore(12);
        game.setRedTeamScore(22);

        // GameSave를 받아서 정보 복원
        game.restore(save);

        System.out.println(game.getBlueTeamScore());
        System.out.println(game.getRedTeamScore());
    }
}
```



### 2-3. 객체 간의 관계도

![memento2](https://user-images.githubusercontent.com/79291114/151471132-19ee4fd1-a684-469f-98c9-b8cc0c8f8421.PNG)





## 3. 장점과 단점

메멘토 패턴의 장점과 단점은 아래와 같습니다.

### 3-1. 장점

- 캡슐화를 지키면서 상태 객체 상태 스냅샷을 만들 수 있습니다.
  - `스냅샷` : 객체의 특정 시점의 데이터를 저장해 놓는 것
- 객체 상태 저장하고 또는 복원하는 역할을 CareTaker에게 위임할 수 있습니다.

- 객체 상태가 바뀌어도 클라이언트 코드는 변경되지 않습니다.



### 3-2. 단점

- 많은 정보를 저장하는 Mementor를 자주 생성하는 경우 메모리 사용량에 많은 영향을 줄 수 있습니다.





## 4. 마치며...

메멘토 패턴은 객체의 특정 시점 상태를 저장할 수 있는데, 이 때, 객체와 거의 동일한 메멘토 객체를 만들기 때문에 객체의 크기가 커질수록 메모리 사용량에 영향을 줄 확률이 높아지므로 상황에 따라 적절히 사용할 수 있어야 합니다.



---

참고 : [코딩으로 학습하는 GoF의 디자인 패턴](https://www.inflearn.com/course/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4/dashboard)
