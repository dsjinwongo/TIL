# Spring
## DTA, DAO, Domain
_(linger0310.log 참고)_

### Domain(=Entity)
- 실제 DB의 테이블과 매칭시키는 클래스
- Entity와 DTO를 분리하는 이유
  - Entity는 DB Layer를 위한, DTO는 View Layer를 위한 것이다.
  - 따라서 Entity는 변경 시 여러 클래스에 영향을 주기 때문에 설계 시에 신중히 만들어야한다.
  - 반면에 DTO는 요청과 응답에 대한 presentation logic(비즈니스 로직과는 달리 화면상의 디자인 구성을 위한 로직을 말한다.)을 가지므로 자주 변경된다. 따라서 이 둘을 분리할 필요가 있다.
  - 다시 말해서 DTO는 Entity에서 presentation logic을 추가하거나 DTO 상에서 필요한 필드멤버만 추가한 구조인 것이다.
  
### DTO(Data Transfer Object)
- View와 통신하기 위한 클래스이다.
- DB와 받은 데이터들을 어떤 방식, 타입으로 주고받을 것인지 정의하는 역할이다.
- ex) DTO
```
@Getter
public class UserDto {
    public String userid;
    public String password;

    public SignInReq(String userid, String password) {
        this.userid = userid;
        this.password = password;
    }
}
```

### DAO(Data Access Object)
- 실제로 DB에 접근하여 Data를 CRUD하는 객체
- Service와 DB를 연결해주는 역할
- 인터페이스와 구현체를 만들어서 구현체에 CRUD 기능을 구현하고 DI 해준다.
- ex) 인터페이스
```
public interface MemberRepository {
    Member save(Member member);
    Optional<Member> findById(Long id);
    Optional<Member> findByName(String name);
    List<Member> findAll();
}
```
- ex) 구현체
```
public class JpaMemberRepository implements MemberRepository{
    ...
    
    @Override
    public Member save(Member member) {
        ...
    }

    @Override
    public Optional<Member> findById(Long id) {
        ...
    }

    @Override
    public Optional<Member> findByName(String name) {
        ...
    }

    @Override
    public List<Member> findAll() {
        ...
    }
    ...
}
```

## Jackson vs Gson
- Jackson과 Gson 모두 Java에 대한 JSON 데이터 바인딩 지원을 제공하는 라이브러리입니다.
- Jackson은 Spring 프레임 워크에 내장되어있지만
- Gson은 dependency를 따로 추가해 줘야 한다.
```java
@GetMapping("/")
    public String list(@RequestParam(value = "subject") String subject) throws JsonProcessingException {
        
        List<Board> boards = boardService.getBoards(subject);
        return mapper.writeValueAsString(boards);
    }
```

## SpringConfig 작성
```java
@Configuration
public class SpringConfig {
    private final EntityManager em;

    public SpringConfig(EntityManager em) {
        this.em = em;
    }

    @Bean
    public BoardRepository boardRepository() {
        return new JpaBoardRepository(em);
    }

    @Bean
    public LikeRepository likeRepository() {
        return new JpaLikeRepository(em);
    }
}
```

## @RequestParam
GET방식으로 넘어온 URI의 queryString을 받기에 적절하다.
```java
@GetMapping("/")
    public String list(@RequestParam(value = "subject") String subject, Model model) throws JsonProcessingException {

        List<Board> boards = boardService.getBoards(subject);
        model.addAttribute("boards", mapper.writeValueAsString(boards));

        return "board/list";
    }
```

## @Valid 를 이용해 @RequestBody 객체 검증
@Valid를 이용하면, service 단이 아닌 객체 안에서, 들어오는 값에 대해 검증을 할 수 있다.

## CORS
[참고](https://shinsunyoung.tistory.com/86)
### SOP(Same-origin policy)
SOP란 같은 Origin에만 요청을 보낼 수 있게 제한하는 보안 정책을 말한다.
Origin은 아래와 같은 구성을 이루어져 있다.
- URI Schema(ex. http, https)
- Hostname(ex. localhost, naver.com)
- Post(ex. 80, 8081)
이중에 하나라도 구성이 다르면 SOP 정책에 걸려 요청을 보낼 수 없다.

_http://www.example.com/dir/page.html에 요청을 보낼 때 예시_
![](https://velog.velcdn.com/images/dsjinwongo/post/ff599b81-b074-45bb-aa3c-7f54e360d726/image.png)

### CORS(Cross-Origin Resource Sharing)
CORS란 서로 다른 Origin끼리 요청을 주고 받을 수 있게 정해둔 표준이다. Spring에서는 ```@CrossOrigin```이라는 어노테이션을 이용하여 간단하게 CORS를 사용할 수 있다.

1. @CrossOrigin 추가
```java
@CrossOrigin(origins = "http://localhost:8080") // 옆의 origin에서 접근 가능
@RestController
public class GreetingController {

  @GetMapping("/hello")
  public String hello() {
    return "안녕하세요?";
  }
}
```
2. WebConfig에 CORS 설정
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

  @Override
  public void addCorsMappings(CorsRegistry registry) {
    registry.addMapping("/**")
        .allowedOrigins("http://localhost:8080");
  }
}
```

# Intellij, Java
- 테스트 클래스 생성 단축
> Ctrl + Shift + T
- 생성자 자동 생성
> Alt+insert
- Stream
> Stream은 컬렉션에 저장되어 있는 엘리먼트를 하나씩 순회하면서 처리할 수 있는 코드패턴이다.(for문 대신에 사용)