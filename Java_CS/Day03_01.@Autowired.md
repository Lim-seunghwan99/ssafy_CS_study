```java
@Service
public class BoardService{

  //private BoardRepository boardRepository;
  //private ReplyRepository replyRepository;

  //// -> 이렇게 되면 기본 생성자가 아님, bRepo, rRepo를 넣어줘야 생성 가능
  //public BoardService(BoardRepository bRepo, ReplyRepository rRepo){
  //  this.boardRepository = bRepo;
  //  this.replyRepository = rRepo;
  //}

  // 위 코드는 밑에 코드와 같음
  public BoardService(BoardRepository bRepo, ReplyRepository rRepo){
  }

  @Autowired
  private BoardRepository boardRepository;

  @Autowired
  private ReplyRepository replyRepository;
  

  @Transactional
  public void 글쓰기(Board board, User user){
  }
}
```

# Autowired
- repository라는 패키지에
  - UserRepository
  - BoardRepository
  - ReplyRepository 라고 있는데
- 스프링이 시작될 때 component들을 스캔해
- 스프링 IoC 컨테이너에 빈(오브젝트, 객체)들을 등록한다
- 등록될 때 UserRepository, BoardRepository, ReplyRepository가 new를 통해 등록되는데<br>
  생성자를 통해 등록된단 말이지
- 그다음에 Service 패키지를 스캔해
  -  BoardService 스캔
- BoardService를 등록할라고 보니까 타입 2개를 넣어줘야 하네?<br>
  근데 가능해!! 이미 난 BoardRepository랑 ReplyRepository를 등록해놨거든<br>
  이게 DI, 의존성 주입

```java
@Service
@RequiredArgsConstructor
// 초기화되지 않은 애들을 생성자 호출할 때 초기화해줘~할 때 쓰는 어노테이션
public class BoardService{

  private final BoardRepository boardRepository;
  // 원래 final 붙이려면 private final BoardRepository boardRepository = null;
  // 이렇게 초기화해줘야 함
  private final ReplyRepository replyRepository;

  @Transactional
  public void 글쓰기(Board board, User user){
  }
}
```
