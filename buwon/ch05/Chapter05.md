## Bean Validation 이란 무엇일까?

어노테이션을 통하여 검증을 쉽게 해주는 것이 Bean Validation이라고 볼 수 있다. 즉, 검증 어노테이션과 여러 인터페이스의 모음이며 Bean Validation 구현한 기술 중에 대부분이 사용하는 구현체는 hibernate Validator이다.

사용하기 위해서는 의존성을 추가해줘야 한다.

```
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

### Bean Validation 사용

아래와 같이 uid에는 공백이 들어갈 수 없고, contactType에는 null이 들어갈 수 없고 content는 1600자를 넘기면 안된다는 규정을 세웠습니다.

```java
@Builder
public class Contact {
    @Length(max = 64)
    @NotBlank
    private String uid;

    @NotNull
    private ContactType contactType;

    @Length(max = 1600)
    private String content;
}
```

한번 Test 코드를 통해 검증해보도록 하겠습니다.

```java
class ContactTest {
    @Test
    public void test_validate() {
        Validator validator = Validation.buildDefaultValidatorFactory().getValidator();

        Contact content = Contact.builder()
                .uid("")  // 공백으로 설정
                .contactType(ContactType.PHONE)
                .content("하하호호")
                .build();

        Set<ConstraintViolation<Contact>> validate = validator.validate(content);

        // uid가 null이므로 @NotNull에 위배되는 오류가 예상됨
        ConstraintViolation<Contact> violation = validate.iterator().next();
        Assertions.assertEquals("공백일 수 없습니다", violation.getMessage());
        Assertions.assertEquals("uid", violation.getPropertyPath().toString());
    }
}
```

![](https://velog.velcdn.com/images/bw1611/post/23fa7c7d-d4e8-445f-9910-1f312e85a6b5/image.png)

사진과 같이 테스트가 통과한 것으로 보아 uid가 null인 부분에서 공백이 에러가 된것으로 보입니다. 그렇다면 Bean Validation에는 어떤 종류가 있을지 찾아보겠습니다.

### Bean Validation의 종류

자주 사용되는 어노테이션 위주로 정리해보겠습니다.

- @Max(value = )

요소는 value보다 작거나 같아야 한다. 
지원되는 타입으로는 byte, short, int, long, BigDeciaml, BigInteger 등이 있습니다. 또한, null 요소는 유효한 것으로 간주됩니다.

- @Min(value = )

요소는 value보다 크거나 같아야 한다.
지원되는 타입으로는 byte, short, int, long, BigDeciaml, BigInteger 등이 있습니다. 또한, null 요소는 유효한 것으로 간주됩니다.

- @NotBalnk

요소는 null이 아니어야 하며, 하나 이상의 공백이 아닌 문자를 포함해야 한다.

- @NotEmpty

요소는 null이 아니어야 한다. 또한, 모든 타입을 허용합니다.

- @Null

요소는 null 이어야 합니다. 또한, 모든 타입을 허용합니다.

- @Pattern(regexp = , flags = )

CharSequence는 지정된 정규식과 일치해야 한다. 정규식 규칙은 Java 정규식을 규칙을 따르며 null 요소는 유효한 것으로 간주한다.

- @Size(min = , max = )

요소의 크기는 지정된 min보다는 이상 max보다는 이하의 값을 가져야 한다.
min의 default 값은 0이고 max는 MAX_INTEGER 값을 갖고 있습니다.

- @Range(min =, max = )

요소는 지정한 범위 내에 있어야 합니다.
숫자 값 또는 숫자 값의 문자열 표현에 적용할 수 있습니다.
min의 default 값은 0L이고 max는 MAX_LONG 값을 갖고 있습니다.

- @UniqueElements

요소는 객체가 고유한지 확인합니다. 즉, 중복되는 값의 요소가 들어갈 수 없음을 나타냅니다.


외에도 많은 어노테이션이 존재하지만 저희가 만나볼듯한 것들 위주로 정리하였습니다.

### Spring에서 Bean Validation 사용하기

Bean Validation을 사용하고 싶다면 Controller, Service, Bean에서는 `@Validated`와 `@Valid`를 추가해주어야 합니다.

```java
@Validated
@Service
@Slf4j
public class ContactService {
    public void createContact(@Valid Contact contact) { // '@Valid'가 설정된 메서드가 호출될 때 유효성 검사를 진행한다.
        // Do Something
    }
}
```

Service 단에서 @Validated 어노테이션을 사용한다면 AOP 기반으로 메서도의 요청을 가로채서 유효성 검증을 진행해줄 수 있습니다. (입력 파라미터의 유효성 검정은 컨트롤러에서 최대한 처리하고 넘겨주는 것이 좋습니다.)
또한 `@Validated`는 Spring 프레임워크에서 제공하는 어노테이션 및 기능입니다.

```java
public void createContact(@Valid Contact contact)
```

또한 사용할 메서드 앞에 @Valid 어노테이션을 넣어주면서 유효성 검증을 진행할 수 있습니다.

> @Validated 동작 원리
@Validated를 클래스 레벨에 선언하면 해당 클래스에 유효성 검증을 위한 AOP의 어드바이스 또는 인터셉터가 등록이 된다. 그리고 해당 클래스의 메서드들이 호출될 때 AOP의 포인트 컷으로써 요청을 가로채서 유효성 검증을 진행한다. 이러한 이유로 스프링 빈이라면 유효성 검증을 진행할 수 있게 된다.

- Bean Validation을 통하여 컨테이너에서 유효성 검증하기!

Bean Validation 2.0부터 컨테이너 요소에 대한 유효성 검사기능이 추가되었습니다.

```java
    @Min(10)
    private List<@Length(max = 64) String> address;
```

- Bean Validation Groups

동일한 모델 객체를 등록할 때와 수정할 때 각각 다르게 검증하기 위하여 도입되었다. 즉, 하나의 클래스에서 2가지의 다른 제약조건이 설정되어야 할 경우 Group으로 묶어서 진행하는 것이 좋다.

그룹으로 묶기위하여 interface 마커를 만들어 준다.
```java
public interface SaveCheck {}
public interface UpdateCheck {}
```

마커를 통하여 2가지의 제약조건으로 분리되어야 하는 곳에 적용합니다.

```java
@Data
public class Item {
 @NotNull(groups = UpdateCheck.class) //수정시에만 적용
 private Long id;
 
 @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
 private String itemName;
 
 @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
 @Range(min = 1000, max = 1000000, groups = {SaveCheck.class, 
UpdateCheck.class})
 private Integer price;
 
 @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
 @Max(value = 9999, groups = SaveCheck.class) //등록시에만 적용
 private Integer quantity;
```

Controller 부분에서 내가 적용하고 싶은 제약조건을 `@Validated(SaveCheck.class)`와 같이 넣어줍니다. 여기서 @Valid는 group을 지정할 수 없으므로 꼭 @Validated로 진행해야 합니다.

```java
@PostMapping("/add")
public String addItemV2(@Validated(SaveCheck.class) @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
}
```

하지만 group에는 단점이 존재하는데 가독성입니다. 또한 코드의 전반적인 복잡도가 올라간것을 확인할 수 있습니다. 그래서 이 방법으로 밖에 진행할 수 있다면 그럴 경우 사용하는 것을 추천한다.

가장 추천하는 방법은 각 Dto를 만들어서 내가 사용할만한 필드에 제약조건을 걸어주는 것이 좋습니다.
만약 무언가를 추가할 때 제약 조건을 걸고 싶다면 AddItemDto클래스를 만들어 제약조건을 걸어주는 것이 가장 바람직한 방법입니다. (만약 update를 해야한다면 UpdateItemDto 클래스를 생성한다.)

```java
public class AddItemDto {

    @NotBlank
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    @NotNull
    @Max(value = 9999)
    private Integer quantity;
}
```

```java
public class UpdateItemDto {

    @NotNull
    private Long id;

    @NotBlank
    private String itemName;
    
    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    private Integer quantity;
}
```

상황에 맞는 Dto클래스를 만들어 제약조건을 사용하고 그에 맞는 Controller, Service에서 호출하여 사용하는 것이 가장 좋습니다.

- 참고 자료

[망나니 개발자 - @Valid와 @Validated를 이용한 유효성 검증의 동작 원리 및 사용법 예시](https://mangkyu.tistory.com/174)
[NHN-Cloud Validation 어디까지 해봤니?](https://meetup.nhncloud.com/posts/223)
