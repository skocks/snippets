# My Snippets

## Lombok
```gradle
    compileOnly 'org.projectlombok:lombok:1.18.20'
    annotationProcessor 'org.projectlombok:lombok:1.18.20'

    testCompileOnly 'org.projectlombok:lombok:1.18.20'
    testAnnotationProcessor 'org.projectlombok:lombok:1.18.20'
```

lombok.config
```
lombok.addLombokGeneratedAnnotation = true
```

```Java
@Slf4j

@Synchronized

@Cleanup InputStream is = this.getClass().getResourceAsStream("res.txt");
```

## Mockito

```gradle
    testImplementation 'org.mockito:mockito-core:3.12.4'
    testImplementation 'org.mockito:mockito-junit-jupiter:3.12.4'
```

```Java
@ExtendWith(MockitoExtension.class)
@RunWith(JUnitPlatform.class)
public class UserServiceUnitTest {

    @Mock UserRepository userRepository;

    UserService userService;

    @BeforeEach
    void init(@Mock SettingRepository settingRepository) {
      userService = new DefaultUserService(userRepository, settingRepository);
        
      Mockito.lenient().when(settingRepository.getUserMinAge()).thenReturn(10);
          
      when(settingRepository.getUserNameMinLength()).thenReturn(4);
          
      Mockito.lenient().when(userRepository.isUsernameAlreadyExists(any(String.class)))
              .thenReturn(false);
  }

  @Test
  void givenValidUser_whenSaveUser_thenSucceed(@Mock MailClient mailClient) {
      // Given
      user = new User("Jerry", 12);
      when(userRepository.insert(any(User.class))).then(new Answer<User>() {
          int sequence = 1;
              
          @Override
          public User answer(InvocationOnMock invocation) throws Throwable {
              User user = (User) invocation.getArgument(0);
              user.setId(sequence++);
              return user;
          }
      });

      userService = new DefaultUserService(userRepository, settingRepository, mailClient);

      // When
      User insertedUser = userService.register(user);
          
      // Then
      verify(userRepository).insert(user);
      Assertions.assertNotNull(user.getId());
      verify(mailClient).sendUserRegistrationMail(insertedUser);
  }
}
```
## Jackson

```gradle
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.12.5'
    implementation 'com.fasterxml.jackson.datatype:jackson-datatype-jsr310:2.12.5'
```

```Java
  objectMapper.findAndRegisterModules()
    .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    .configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
```
[jackson-modules-java8](https://github.com/FasterXML/jackson-modules-java8)

---
```Java
String result = new ObjectMapper().writeValueAsString(bean);

@JsonPropertyOrder({ "name", "id" })
public class MyBean {
    public int id;
    public String name;
}

public enum TypeEnumWithValue {
    TYPE1(1, "Type A"), TYPE2(2, "Type 2");

    private Integer id;
    private String name;

    // standard constructors

    @JsonValue
    public String getName() {
        return name;
    }
}
```
---
```Java
public class Zoo {
    public Animal animal;

    @JsonTypeInfo(
      use = JsonTypeInfo.Id.NAME, 
      include = As.PROPERTY, 
      property = "type")
    @JsonSubTypes({
        @JsonSubTypes.Type(value = Dog.class, name = "dog"),
        @JsonSubTypes.Type(value = Cat.class, name = "cat")
    })
    public static class Animal {
        public String name;
    }

    @JsonTypeName("dog")
    public static class Dog extends Animal {
        public double barkVolume;
    }

    @JsonTypeName("cat")
    public static class Cat extends Animal {
        boolean likesCream;
        public int lives;
    }
}
```

```json
{
    "animal": {
        "type": "dog",
        "name": "lacy",
        "barkVolume": 0
    }
}
```

```Java
@Test
public void whenDeserializingPolymorphic_thenCorrect() throws IOException {
    String json = "{\"animal\":{\"name\":\"lacy\",\"type\":\"cat\"}}";

    Zoo zoo = new ObjectMapper()
      .readerFor(Zoo.class)
      .readValue(json);

    assertEquals("lacy", zoo.animal.name);
    assertEquals(Zoo.Cat.class, zoo.animal.getClass());
}
```

## MapStruct

```gradle
    compileOnly 'org.mapstruct:mapstruct-processor:1.4.2.Final'
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'
    implementation 'org.mapstruct:mapstruct:1.4.2.Final'
```

### HashIds

```Java
public class IdMapper {

  private final Hashids hashIds;

  public IdMapper() {
    this.hashIds = new Hashids("Salt42", 4);
  }

  public String toHash(Long id) {
    return hashIds.encode(id);
  }

  public Long toId(String hash) {
    return hashIds.decode(hash)[0];
  }
}
```

```Java
@Mapper(componentModel = "spring", uses = {IdMapper.class}, injectionStrategy = CONSTRUCTOR)
public interface UserMapper {

  User toUser(UserEntity userEntity);

  List<User> toUserList(List<UserEntity> users);
}
```

## Hibernate

```gradle
    compileOnly 'org.hibernate:hibernate-jpamodelgen:5.5.7-Final'
    annotationProcessor 'org.hibernate:hibernate-jpamodelgen:5.5.7-Final'
```

```Java
package de.example;

import lombok.EqualsAndHashCode;
import lombok.Getter;
import org.hibernate.annotations.CreationTimestamp;
import org.hibernate.annotations.UpdateTimestamp;

import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@Getter
@MappedSuperclass
@EqualsAndHashCode
public abstract class Persisted {

  public abstract Long getId();

  @CreationTimestamp
  private LocalDateTime dateCreated;

  @UpdateTimestamp
  private LocalDateTime lastUpdated;

}
```
```Java
package de.skocks.ecommerce.user;

import de.skocks.ecommerce.Persisted;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.EqualsAndHashCode;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.experimental.Accessors;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.SequenceGenerator;
import javax.persistence.Table;
import javax.persistence.UniqueConstraint;
import javax.validation.constraints.Pattern;
import javax.validation.constraints.Size;

@Entity
@Builder
@Getter
@Setter
@Accessors(chain = true)
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode(callSuper = true)
public class UserEntity extends Persisted {

  @Id
  @SequenceGenerator(name = "user-sequence-generator", sequenceName = "user_sequence")
  @GeneratedValue(generator = "user-sequence-generator")  @GeneratedValue
  private Long id;

  @Column(name = "email")
  @Size(min = 3, max = 80)
  private String email;

  @Column(name = "first_name")
  @Pattern(regexp = "^[a-zA-Z\\s]+$")
  @Size(min = 3, max = 80)
  private String firstName;

}

```

## H2

```
spring.jpa.hibernate.ddl-auto=update
#spring.jpa.hibernate.ddl-auto=validate

spring.datasource.url=jdbc:h2:mem:testdb
#spring.datasource.url=jdbc:h2:file:/data/demo
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect

spring.h2.console.enabled=true
```

```
testImplementation group: 'com.h2database', name: 'h2', version: '1.4.200'
```