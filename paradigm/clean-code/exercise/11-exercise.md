## 스프링 부트 컨트롤러가 POJO라면

```java
public class TodoController {
    private final TodoService todoService;

    public ResponseEntity<TodoResponseDto> createTodo(TodoRequestDto todoRequestDto) {
        TodoEntity todo = todoRequestDto.toEntity();
        todoService.add(todo);
        TodoResponseDto todoResponseDto = TodoResponseDto.of(todo);
        return new ResponseEntity<>(todoResponseDto, HttpStatus.CREATED);
    }

    public ResponseEntity<List<TodoResponseDto>> searchAll() {
        List<TodoEntity> result = todoService.searchAll();
        List<TodoResponseDto> todoList = result.stream().map(e -> TodoResponseDto.of(e)).toList();
        return new ResponseEntity<>(todoList, HttpStatus.OK);
    }

    public ResponseEntity<TodoResponseDto> searchById(@PathVariable Long id) {
        TodoEntity result = todoService.searchById(id);
        TodoResponseDto todoResponseDto = TodoResponseDto.of(result);
        return new ResponseEntity<>(todoResponseDto, HttpStatus.OK);
    }

    public ResponseEntity<TodoResponseDto> updateTodo(@PathVariable Long id, @Valid @RequestBody TodoRequestDto todoRequestDto) {
        TodoEntity result = todoService.searchById(id);
        todoService.updateTodo(result, todoRequestDto.getTitle(),todoRequestDto.getCompleted());
        TodoResponseDto todoResponseDto = TodoResponseDto.of(result);
        return new ResponseEntity<>(todoResponseDto, HttpStatus.OK);

    }

    public ResponseEntity<TodoResponseDto> deleteTodo(@PathVariable Long id) {
        TodoEntity result = todoService.searchById(id);
        todoService.deleteTodo(result);
        return ResponseEntity.noContent().build();
    }
}

```

## 어노테이션 기반으로 HTTP 통신에 이용되는 컨트롤러 객체
```java
@Slf4j
@RestController
@RequestMapping("/todos")
@RequiredArgsConstructor
public class TodoController {
    private final TodoService todoService;


    @PostMapping("")
    public ResponseEntity<TodoResponseDto> createTodo(@Valid @RequestBody TodoRequestDto todoRequestDto) {
        TodoEntity todo = todoRequestDto.toEntity();
        todoService.add(todo);
        TodoResponseDto todoResponseDto = TodoResponseDto.of(todo);
        return new ResponseEntity<>(todoResponseDto, HttpStatus.CREATED);
    }

    @GetMapping("")
    public ResponseEntity<List<TodoResponseDto>> searchAll() {
        List<TodoEntity> result = todoService.searchAll();
        List<TodoResponseDto> todoList = result.stream().map(e -> TodoResponseDto.of(e)).toList();
        return new ResponseEntity<>(todoList, HttpStatus.OK);
    }


    @GetMapping("{id}")
    public ResponseEntity<TodoResponseDto> searchById(@PathVariable Long id) {
        TodoEntity result = todoService.searchById(id);
        TodoResponseDto todoResponseDto = TodoResponseDto.of(result);
        return new ResponseEntity<>(todoResponseDto, HttpStatus.OK);
    }

    @PatchMapping("{id}")
    public ResponseEntity<TodoResponseDto> updateTodo(@PathVariable Long id, @Valid @RequestBody TodoRequestDto todoRequestDto) {
        TodoEntity result = todoService.searchById(id);
        todoService.updateTodo(result, todoRequestDto.getTitle(),todoRequestDto.getCompleted());
        TodoResponseDto todoResponseDto = TodoResponseDto.of(result);
        return new ResponseEntity<>(todoResponseDto, HttpStatus.OK);

    }

    @DeleteMapping("{id}")
    public ResponseEntity<TodoResponseDto> deleteTodo(@PathVariable Long id) {
        TodoEntity result = todoService.searchById(id);
        todoService.deleteTodo(result);
        return ResponseEntity.noContent().build();
    }
}
```