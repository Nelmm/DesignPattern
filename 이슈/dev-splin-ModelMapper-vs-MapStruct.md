# MapStruct의 사용법 및 ModelMapper와의 비교

`Controller, Service, Repository` 등 레이어 간 데이터를 주고받을 때나 비즈니스 로직에서 하나의 객체를 타입이 다른 객체로 형(Type) 변환하거나 여러 객체를 다른 객체로 합치는 일은 매우 빈번하게 발생합니다. 이런 작업을 개발자가 모두 직접 하게되면 발생하는 문제점을 몇 가지 나열해 보면 다음과 같습니다.

- 재미가 없고 반복적이고 코드 중복이 발생하기 쉽습니다.
- 실수하기 쉽습니다.
- 필드가 추가나 수정, 삭제가 일어날 경우 변환하는 로직에 대해서 수정이 필요합니다.
- 비즈니스 로직에 섞이게 되면 코드가 복잡해집니다.
- 결국 생산성을 떨어뜨립니다.



```java
UserEntity userDTOToEntity(UserDTO userDTO) {
        return new userEntity(userDTO.getId(), 
                             userDTO.getPassword(),
                             userDTO.getName());
}
```

다음과 같이 개발자가 직접 작성한 **DTO를 Entity로 변환하는 코드**인데, 여기서는 비교적 단순한 객체라 코드가 짧지만 필드가 몇 개만 더 늘어나도 가독성이 떨어지고 개발자에게 피곤한 작업이 됩니다.

`Object Mapping` 라이브러리는 이런 문제를 해결해주는 아주 좋은 도구입니다. `Object Mapping`을 위한 `ModelMapper`와 `MapStruct`에 대해서 의존성 설정, Mapping 사용법과 차이점을 알아보겠습니다.



## ModelMapper / MapStruct

`Object Mapping`을 위한 `ModelMapper`와 `MapStruct`라이브러리가 있습니다. `MapStruct`의 장점이 많기 때문에 `MapStruct`위주로 `ModelMapper`와의 차이점을 알아보겠습니다.

- `MapStruct`는 컴파일 시점에서 어노테이션을 읽어 구현체를 만들어내기 때문에 리플렉션이 발생하지 않습니다. (`ModelMapper`는 Mapping이 일어날 때 리플렉션이 발생)
- `MapStruct`의 처리속도가 10<sup>-5</sup>m/s로  압도적으로 빠릅니다. (`ModelMapper`는 0.002m/s)
- `MapStruct`는 컴파일 시 오류를 확인할 수 있습니다.
- `MapStruct`는 디버깅이 쉽습니다.
- `MapStruct`는 생성된 매핑 코드를 눈으로 직접 확인할 수 있습니다.

> 리플렉션 ? 객체를 통해 클래스의 정보를 분석해 내는 프로그래밍 기법. 구체적은 클래스 타입을 알지 못해도 컴파일된 바이트 코드를 통해 역으로 클래스의 정보를 알아내어 클래스를 사용하게 할 수 있습니다. <br>리플렉션 기법을 통해 객체의 타입을 모르는 상태에서 객체의 메서드를 호출할 수 있습니다. 즉 동적 바인딩이 되지 않던 자바에서 리플렉션이라는 프로그래밍 기법을 통해 동적 바인딩을 제공하는 것 입니다.



이렇게 `MapStruct`는 많은 장점을 가지고 있기 때문에 **MapStruct 사용을 권장**합니다.

또, 밑의 구글 트렌드로 봤을 때도 **MapStruct가 Java Mapping 라이브러리 중 가장 많이 사용**되고 있는 것을 알 수 있습니다.

<img src="https://image.toast.com/aaaadh/real/2020/techblog/1.png" alt="1.png" style="zoom:80%;" />
<img src="https://image.toast.com/aaaadh/real/2020/techblog/2.png" alt="2.png" style="zoom:80%;" />



`ModelMapper`의 간단한 사용법을 보고, `MapStruct`의 다양한 사용법을 살펴보겠습니다.

그럼 먼저 테스트에 사용할 Entity 객체와 DTO 객체를 만들겠습니다. `Entity, VO, DTO`를 사용할 때의 주의점을 따라`Lombok`을 사용하여 객체를 생성하였습니다. 

[Entity, VO, DTO의 차이 포스트 링크](https://dev-splin.github.io/spring/Spring-Entity-DTO-VO/)

[Lombok의 사용법 및 주의점 포스트 링크](https://dev-splin.github.io/spring/Spring-Lombok/)



#### UserEntity

**Entity는 Setter 사용을 지양**하여야 합니다.

```java
@ToString
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class UserEntity {
    private Long id;
    private String password;
    private String name;
    private String nickName;
    private Date createDate;
    
    @Builder
    private UserEntity(Long id, String password, String name,
    		String nickName, Date createDate) {
    	this.id = id;
    	this.password = password;
    	this.name = name;
    	this.nickName = nickName;
    	this.createDate = createDate;
    }
}
```



#### UserDTO

```java
@Getter
@Setter
@ToString
public class UserDTO {
    private Long id;
    private String password;
    private String name;
   
}
```



### ModelMapper



#### 의존성 설정

```xml
<!-- https://mvnrepository.com/artifact/org.modelmapper/modelmapper -->
<dependency>
    <groupId>org.modelmapper</groupId>
    <artifactId>modelmapper</artifactId>
    <version>2.3.0</version>
</dependency>
```



#### Bean Configuration

```java
@Configuration
public class ApplicationConfig {

    @Bean
    public ModelMapper modelMapper(){
        ModelMapper modelMapper = new ModelMapper();
        /* modelMapper.getConfiguration().setMatchingStrategy(MatchingStrategies.STRICT); */
        return modelMapper;
    }
}
```

주석이 된 부분은 매칭 전략을 설정하는 부분이며 `Standard`, `Loose`, `Strict` 세 가지가 있고 **Default는 Standard**로 되어있습니다.

추가적인 내용은 [http://modelmapper.org/user-manual/configuration/#matching-strategies](http://modelmapper.org/user-manual/configuration/#matching-strategies) 레퍼런스를 참조하면 되겠습니다.



#### Test

간단하게 테스트로 결과만 확인해보겠습니다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {ApplicationConfig.class})
public class ModelMapperTest {

	@Autowired
	ModelMapper modelMapper;
	
	@Test
	public void ModelMapperTest() throws Exception { 
		
		// Entity에는 Setter가 없기 때문에 ModelMapper는 밑의 방식으로 값을 넣어줄 수 있습니다.
		modelMapper.createTypeMap(UserDTO.class, UserEntity.class).setProvider(
			// Setter가 없는 클래스를 넣어줍니다.
			new Provider<UserEntity>() {
				public UserEntity get(ProvisionRequest<UserEntity> request) {
					// 변환할 클래스를 가져와서 Setter가 없는 클래스를 만듭니다.
					UserDTO dto = UserDTO.class.cast(request.getSource());
					return UserEntity.builder()
                        	.id(1L)
							.password(dto.getPassword())
							.name(dto.getName())
							.nickName(dto.getNickName())
							.createDate(new Date())
							.build();
				}
		});
		
		UserEntity userEntity = UserEntity.builder().id(1L)
				.password("testPassword")
				.name("testName")
				.nickName("testNickName")
				.createDate(new Date())
				.build();;
		
		// Entity -> DTO
		UserDTO resultDTO = modelMapper.map(userEntity, UserDTO.class);
		// DTO -> Entity, Setter가 없을 시 변환할 수 없음
		UserEntity resultEntity = modelMapper.map(resultDTO, UserEntity.class);
		
		System.out.println(resultDTO);
		System.out.println(resultEntity);
	}
}
```

1. `Entity -> DTO`, `modelMapper.map(Entity 객체, DTO클래스명.class)`
2. `DTO -> Entity`, `modelMapper.map(DTO 객체, Entity클래스명.class)`

사용방법은 위와 같이 변경할 객체와 변경될 객체의 클래스명을 전달하면 변환이 이루어 지는 것을 알 수 있습니다.

[https://github.com/modelmapper/modelmapper/blob/master/core/src/test/java/org/modelmapper/functional/Immutable.java](https://github.com/modelmapper/modelmapper/blob/master/core/src/test/java/org/modelmapper/functional/Immutable.java)를 참고하여 Setter가 없는 경우에 값을 넣어주었습니다.



#### 주의 사항

- **만들어지는 대상은 Getter, 만드는 대상은 Setter가 필요합니다.**
  - Entity가 DTO로 변환된다고 한다면 Entity에는 각 필드값을 읽을 수 있는 `Getter`가 DTO는 필드값을 넣을 수 있는 `Setter`들이 존재해야합니다.
  - createTypeMap를 통해 Setter가 없을 경우도 설정해줄 수 있지만, 그다지 좋아보이지는 않습니다.
- **필드 작명, Standard(Default Staragy) 기준**, 필드 이름이 같을 경우 자동으로 매핑이 이루어지지만 필드이름이 다를 경우 매핑이 이루어지지 않습니다



### MapStruct



#### 의존성 설정

```xml
<properties>
    <org.mapstruct.version>1.4.2.Final</org.mapstruct.version>
</properties>


<dependencies>
    <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${org.mapstruct.version}</version>
    </dependency>
</dependencies>

...

    <plugins>
        <!-- maven 버전 설정을 위한 플러그인 입니다.-->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.5.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <!-- mapstruct-processor을 설정해줍니다. -->
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>${org.mapstruct.version}</version>
                    </path>
                    <!-- other annotation processors -->
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    <plugins>
```

mapstruct 설정을 할 때, 몇 가지 주의할 점이 있습니다.



##### 주의 사항

###### plugins와 pluginManagement의 차이

의존성을 설정할 때, `plugins`와 `pluginManagement`의 차이에 대해 주의하면서 작성합니다.

[pluginManagement와 plugins의 차이와 주의할 점 포스팅](https://dev-splin.github.io/spring/Spring-pluginManagement,plugins/)



###### Lombok과 MapStruct의 순서

- **Lombok 다음 MapStruct dependency**가 오는 경우
  - 먼저 실행된 Lombok이  Getter, Setter, Builder등을 만들고 그 다음에 실행된 MapStruct가 만들어진 Getter, Setter, Builder등을 이용해 코드를 생성합니다. 
- **MapStruct 다음 Lombok dependency**가 오는 경우
  - 먼저 실행된 MapStruct가 Getter, Setter, Builder등을 이용해 코드를 생성하여야 하는데, 만들어진 Getter, Setter, Builder등이 없기 때문에 생성에 실패합니다.

위와 같이 Lombok과 MapStruct를 같이 사용할 때, `<dependencies> </dependencies>`에서 **Lombok과 MapStruct의 순서가 중요합니다.**



```java
<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok-mapstruct-binding -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok-mapstruct-binding</artifactId>
    <version>0.2.0</version>
</dependency>
```

`Lombok 1.18.16 버전 이상`에서는 위와 같은 의존성을 추가할 경우 순서와 상관없이 사용할 수 있습니다.



#### Mapper 만들기

```java
@Mapper
public interface UserMapper {
	UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);
	
	UserEntity userDTOToEntity(UserDTO userDTO);
	
	UserDTO userEntityToDTO(UserEntity userEntity);
}
```

- 기본적으로는 인터페이스에 `@Mapper`를 붙이면 됩니다. **@Mapper를 사용하게 되면 MapStruct가 자동으로 UserMapper를 상속받아서 UserMapperImpl를 구현**해줍니다. (아래와 같이 구현된 부분은 `target -> generated-sources->...->Mapper가 있는 폴더`에서 볼 수 있습니다.)
- `MapStruct`에서는 `UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);` 같은 방식으로 Mapper Bean을 만드는 것을 권장한다고 합니다.
  - `@Mapper(componentModel = "spring")`과 `@Autowired`를 이용하여 만들 수도 있습니다.

```java
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2021-06-03T16:00:52+0900",
    comments = "version: 1.4.2.Final, compiler: Eclipse JDT (IDE) 1.3.1200.v20200916-0645, environment: Java 15.0.2 (Oracle Corporation)"
)
public class UserMapperImpl implements UserMapper {

    @Override
    public UserEntity userDTOToEntity(UserDTO userDTO) {
        if ( userDTO == null ) {
            return null;
        }

        UserEntityBuilder userEntity = UserEntity.builder();

        userEntity.id( userDTO.getId() );
        userEntity.name( userDTO.getName() );
        userEntity.password( userDTO.getPassword() );

        return userEntity.build();
    }

    @Override
    public UserDTO userEntityToDTO(UserEntity userEntity) {
        if ( userEntity == null ) {
            return null;
        }

        UserDTO userDTO = new UserDTO();

        userDTO.setId( userEntity.getId() );
        userDTO.setName( userEntity.getName() );
        userDTO.setPassword( userEntity.getPassword() );

        return userDTO;
    }
}
```

메서드의 파라미터에 오는 객체는 Getter, 메서드가 리턴하는 객체는 Setter가 필요하지만, **Target 객체(메서드가 리턴하는 객체)에 @Builder 어노테이션이 달려있다면 Builder 메소드를 우선 사용**합니다. (`userDTOToEntity`메서드를 보면 Builder를 이용한 것을 볼 수 있습니다.)

또, `@Mapping` 어노테이션과 속성을 이용해 다양한 경우를 설정해 줄 수 있습니다. (이에 대한 내용은 아래에서 다루겠습니다.)



#### Test

```java
	@Test
	public void MapStructTest() throws Exception {
		
		UserEntity userEntity = UserEntity.builder()
            	.id(1L)
				.password("testPassword")
				.name("testName")
				.nickName("testNickName")
				.createDate(new Date())
				.build();
		
		// Entity -> DTO
		UserDTO resultDTO = UserMapper.INSTANCE.userEntityToDTO(userEntity);
		// DTO -> Entity, Setter가 없어도 @Builder가 붙어있다면 변환 가능
		UserEntity resultEntity = UserMapper.INSTANCE.userDTOToEntity(resultDTO);
		
		System.out.println(resultDTO);
		System.out.println(resultEntity);
	}
```

`INSTANCE`를 이용해 변환 메서드를 가져와 객체를 Mapping할 수 있습니다.



#### 주의 사항

- 메서드의 파라미터에 오는 객체는 Getter, 메서드가 리턴하는 객체는 Setter가 필요하지만, **Target 객체(메서드가 리턴하는 객체)에 @Builder 어노테이션이 달려있다면 Builder 메소드를 우선 사용**합니다.





## @Mapping을 이용한 다양한 Mapping

`MapStruct`는 `@Mapping` 어노테이션의 속성을 이용해 다양한 경우의 Mapping을 설정해줄 수 있습니다.



### 서로 다른 속성 매핑

위의 `UserDTO`에서 닉네임을 `nick`이라는 이름으로 추가하고<br>**UserEntity의 nickName을 UserDTO의 nick에 Mapping해야 하는 경우** `source, target 속성`을 이용해 설정해줄 수 있습니다.

```java
@Getter
@Setter
@ToString
public class UserDTO {
    private Long id;
    private String password;
    private String name;
    private String nick;
}
```

아래와 같이 Mapping 메서드에 `@Mapping(source = "파라미터 객체의 변수이름", target = "리턴 객체의 변수이름")`와 같이 설정해주면 됩니다.

```java
@Mapper
public interface UserMapper {
	UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);
	
    @Mapping(source = "nick", target = "nickName")
	UserEntity userDTOToEntity(UserDTO userDTO);
	
	@Mapping(source = "nickName", target = "nick")
	UserDTO userEntityToDTO(UserEntity userEntity);
}
```



### 객체 합치기

UserDTO에 message 변수를 추가하고 파라미터를 통해 따로 받아줄 수 있습니다.

```java
@Getter
@Setter
@ToString
public class UserDTO {
    private Long id;
    private String password;
    private String name;
    private String nick;
    private String message;
}
```

아래와 같이 파라미터에 message를 추가하여 사용할 수 있습니다.

이 때 기존의 `@Mapping(source = "nickName", target = "nick")`에서 **source(파라미터 객체)의 nickName이 어떤 파라미터 객체의 것인지 알 수 없기 때문**에 `@Mapping(source = "userEntity.nickName", target = "nick")` 같이 객체명을 명시해줍니다.

`expression = "java(message + \".msg\")"`처럼 `expression`속성을 사용해 message뒤에 자동으로 .msg를 붙일 수 있습니다. `java()`안에는 객체가 들어갈수도 있고, 메서드를 호출하는 등 다양하게 활용할 수 있습니다.

```java
@Mapper
public interface UserMapper {
	UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);
	
    @Mapping(source = "nick", target = "nickName")
	UserEntity userDTOToEntity(UserDTO userDTO);
	
	@Mapping(source = "userEntity.nickName", target = "nick")
    @Mapping(target = "message", expression = "java(message + \".msg\")")
	UserDTO userEntityToDTO(UserEntity userEntity, String message, Address address);
}
```



### 객체 합치기 2

Address라는 객체를 만들고, UserDTO에 Address객체를 가지는 속성을 추가해줍니다.

```java
@Getter
@Setter
@ToString
public class Address {
	private String address;
	private int addressNum;
}
```

```java
@Getter
@Setter
@ToString
public class UserDTO {
    private Long id;
    private String password;
    private String name;
    private String nick;
    private String message;
    private Address add;
}
```

아래와 같이 여러개의 `@Mapping`을 이용해 `source, target`을 지정해줄 수 있습니다.

**UserDTO의 Address속성의 이름은 add**이고 **파라미터로 들어온 Address 객체의 이름은 address**이기 때문에 `source, target`으로 지정해주어야 합니다.

```java
@Mapper
public interface UserMapper {
	UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);
	
    @Mapping(source = "nick", target = "nickName")
	UserEntity userDTOToEntity(UserDTO userDTO);
	
	@Mapping(source = "userEntity.nickName", target = "nick")
	@Mapping(source = "address", target = "add")
	UserDTO userEntityToDTO(UserEntity userEntity, String message, Address address);
}
```



### 속성 무시하기

**UserEntity에는 createDate가 있는데 DTO에는 없기 때문**에 DTO -> Entity로 변환할 때 **createDate에는 null 값**이 들어가게 됩니다.

이 때, `@Mapper(unmappedTargetPolicy = ReportingPolicy.ERROR)`을 이용해 정책을 설정해서 target 객체에 매핑 시 **매핑되지 않은 속성이 있다면 컴파일 에러**를 발생시키게 할 수 있습니다. 

하지만 `@Mapping`의 `ignore`속성을 이용해 매핑되지 않는 속성을 무시하게 할 수 있습니다.

```java
@Mapper(unmappedTargetPolicy = ReportingPolicy.ERROR)
public interface UserMapper {
	UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);
	
	@Mapping(target = "createDate", ignore = true)
	@Mapping(source = "nick", target = "nickName")
	UserEntity userDTOToEntity(UserDTO userDTO);
	
	@Mapping(source = "userEntity.nickName", target = "nick")
	@Mapping(source = "address", target = "add")
	UserDTO userEntityToDTO(UserEntity userEntity, String message, Address address);
}
```



#### 정책

`매핑 정책(Policy)`과 `전략(Strategy)`를 설정할 수 있습니다. 아래는 몇 가지 유용한 매핑 정책과 전략에 대한 설명입니다.

|         정책         |              값              |                             설명                             |
| :------------------: | :--------------------------: | :----------------------------------------------------------: |
| unmappedSourcePolicy | IGNORE(default), WARN, ERROR | Source의 필드가 Target에 매핑되지 않을 때 정책입니다. 예, ERROR로 설정하면 매핑 시 Source.aField가 사용되지 않는다면 컴파일 오류가 발생시킵니다. |
| unmappedTargetPolicy | IGNORE, WARN(default), ERROR | Target의 필드가 매핑되지 않을 때 정책입니다. 예, ERROR로 설정하면 매핑 시 Target.aField에 값이 매핑되지 않는다면 컴파일 오류가 발생시킵니다. |
| typeConversionPolicy | IGNORE(default), WARN, ERROR | 타입 변환 시 유실이 발생할 수 있을 때 정책입니다. 예, ERROR로 설정하면 long에서 int로 값을 넘길 때 값에 유실이 발생할 수 있습니다. 이런 경우에 컴파일 오류를 발생시킵니다. |

|               전략               |                      값                      |                 설명                  |
| :------------------------------: | :------------------------------------------: | :-----------------------------------: |
|     nullValueMappingStrategy     |     RETURN_NULL(default), RETURN_DEFAULT     |    Source가 null일 때 정책입니다.     |
| nullValuePropertyMappingStrategy | SET_TO_NULL(default), SET_TO_DEFAULT, IGNORE | Source의 필드가 null일 때 정책입니다. |



### Json String으로 변환하는 Mapper 직접 구현하기

가끔 **MapStruct에서 매핑 코드를 구현하지 못하거나 직접 구현해야될 때**가 있습니다. `MapStruct`는 `default` 메서드를 이용해 Mapping 메서드를 직접 구현할 수 있게 해줍니다.

다음은 객체를 String으로 Mapping하는 메서드를 직접 구현한 것입니다.

```java
@Mapper
public interface JsonMapper {
    ObjectMapper OBJECT_MAPPER = new ObjectMapper();

    default String toString(Object obj) {
        try {
            return OBJECT_MAPPER.writeValueAsString(obj);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

**ObjectMapper는 Jackson에서 Java Object와  JSON사이의 변환을 쉽게 해주는 클래스**입니다.

- `writeValueAsString(obj)`  : obj객체를 Json String으로 변환시켜 줍니다.
- `readValue(arg, type)` : arg을 type에 해당하는 클래스로 변환할 수 있습니다. Class객체, TypeReference가 올 수 있습니다.
  - **ex )** `OBJECT_MAPPER.readValue(arg, ArrayList.class);` (arg에 해당하는 Json String을 ArrayList로 변환합니다.)



### Mapper 속성 설정 공통화 하기

Mapper가 많아져서 Mapper마다의 설정 중 중복이 발생할 때 속성 설정을 공통화할 수 있습니다. `MapStructMapperConfig`를 만들어 `@MapperConfig`어노테이션을 이용해 공통화할 속성을 설정해줍니다.

특정 타입이나 객체 간 Mapping을 스스로할 수 없거나 다른 Mapper를 이용해야 할 때 `uses`속성을 사용할 수 있습니다. `uses = JsonMapper.class`로 설정해주면 객체에서 String으로 변환이 필요할 때 JsonMapper를 사용합니다.

```java
@MapperConfig(unmappedTargetPolicy = ReportingPolicy.ERROR, uses = JsonMapper.class)
public interface MapStructMapperConfig {

}
```

아래와 같이 Mapper마다 사용할 수 있지만, 주석처럼 JsonMapper에 적용하게 되면 `uses = JsonMapper.class`때문에 순환참조가 일어날 수 있으니 조심해야 합니다.

```java
@Mapper(config = MapStructMapperConfig.class)
public interface UserMapper {
	...
}

// @Mapper(config = MapStructMapperConfig.class)
@Mapper(unmappedTargetPolicy = ReportingPolicy.ERROR)
public interface JsonMapper {
    ...
}
```



---

참고 : [https://huisam.tistory.com/entry/mapStruct](https://huisam.tistory.com/entry/mapStruct)

[https://mangchhe.github.io/spring/2021/01/25/ModelMapperAndMapStruct/](https://mangchhe.github.io/spring/2021/01/25/ModelMapperAndMapStruct/)

[https://meetup.toast.com/posts/213](https://meetup.toast.com/posts/213)

[https://wise-develop.tistory.com/18](https://wise-develop.tistory.com/18)

[https://asfirstalways.tistory.com/221](https://asfirstalways.tistory.com/221)

[https://engaspect.tistory.com/27](https://engaspect.tistory.com/27)

