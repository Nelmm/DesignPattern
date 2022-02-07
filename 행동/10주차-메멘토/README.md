# 메멘토(Memento) 패턴

메멘토 패턴은 **캡슐화를 유지하면서 객체 내부 상태를 외부에 저장하는 패턴**입니다.

## 1. 다이어그램

![memento](https://user-images.githubusercontent.com/79291114/151184142-6a60b443-3ba4-4eea-855a-981aaa82d9fd.PNG)

- `Originator` : 객체의 정보를 가지고 있는 오리지널 객체
  - 제 3자가 객체 내부 정보를 알지 못하게 하기 위하여 **Memento를 만들 수 있는 메서드가 필요**함
- `Memento` : 오리지널 객체의 정보를 담는 불변 객체
- `CareTaker` : 오리지널이 반환한 **Memento를 가지고 있다가 필요할 때 Memento의 값을 Originator에게 전달**함
  
  

### 2. 예제

```java
//캐릭터 객체
//Originator
@Getter
@Setter
@AllArgsConstructor
public class Character {
    private String name;
    private Long level;
    private String job;

    public void levelUp() {
        this.level += 1;
    }

    public CharacterMemento createMementor() {
        return new CharacterMemento(level, name, job);
    }

    public static Character restore(CharacterMemento characterMemento) {
                //Memento 객체를 통해 새로운 Character 생성
        return new Character(characterMemento.getName(), characterMemento.getLevel(), characterMemento.getJob());
    }
}

//캐릭터 정보 저장할 Memento
@Getter
@AllArgsConstructor
public class CharacterMemento {
    private final Long level;
    private final String name;
    private final String job;
}

//Memento 관리할 CareTaker
public class CharacterCareTaker {
    private final Map<String, CharacterMemento> characterInfo = new HashMap<>();
    //캐릭터 정보 저장
    public void saveCharacter(CharacterMemento characterMemento) {
        characterInfo.put(characterMemento.getName(), characterMemento);
    }
    //캐릭터 목록 출력
    public void printCharacterList() {
        for (String characterName : this.characterInfo.keySet()) {
            System.out.println("캐릭터 이름 : " + characterInfo.get(characterName).getName());
        }
    }
    //특정 캐릭터 반환
    public CharacterMemento getCharacter(String name) {
        if (!characterInfo.containsKey(name)) {
            return null;
        }
        return characterInfo.get(name);
    }
    //캐릭터 삭제
    public void clearInfo() {
        characterInfo.clear();
    }
}

//클라이언트
public class GameClient {
    public static void main(String[] args) {
        CharacterCareTaker characterCareTaker = new CharacterCareTaker();
        //캐릭터 생성
        Character character = new Character("name1", 1L, "전사");
        //캐릭터 저장
        characterCareTaker.saveCharacter(character.createMementor());

        character.levelUp();
        System.out.println("캐릭터가 레벨업을 했다 현재 레벨 : " + character.getLevel());

        System.out.println("-------갑자기 게임이 강제 종료됐다-----");
        System.out.println("유저는 캐릭터 목록을 확인한다");
        characterCareTaker.printCharacterList();

        System.out.println("아까 만든 캐릭터를 클릭한다");
        Character restartCharacter = Character.restore(characterCareTaker.getCharacter("name1"));
        System.out.println("현재 캐릭터 레벨 : " + restartCharacter.getLevel());

        System.out.println("젠장....빡쳐서 캐릭터 삭제");
        characterCareTaker.clearInfo();

        System.out.println("캐릭터 목록");
        characterCareTaker.printCharacterList();
    }
}
```

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