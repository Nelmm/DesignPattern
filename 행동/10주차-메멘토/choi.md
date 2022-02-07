# 메멘토 패턴

> `캡슐화`를 유지하면서 객체 내부 상태를 외부에 저장하는 방법
>

![Untitled](https://user-images.githubusercontent.com/32676275/151112064-eae9dec5-f841-4ee5-b20a-b87a3af58775.png)

 객체의 데이터를 `CareTaker` 에 저장하고 필요할 때 데이터를 이용해 원본 객체를 만드는 패턴

---

### 예제

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

---

### 장/단점

> 장점
>
1.  저장된 상태를 핵심 객체와는 다른 별도의 객체에 보관하기 때문에 안전하다.
2.  핵심 객체의 데이터를 계속해서 캡슐화된 상태로 유지할 수 있다.
3.  복구 기능을 구현하기 쉽다.

> 단점
>
1. 상태를 저장하고 복구하는 데 시간이 오래 걸리 수 있다는 단점이 있다.