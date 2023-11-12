## 단일 책임 원칙 (SRP)을 위반한 사례

주말 스터디를 통해서 스프링 토이 프로젝트를 수행 중, 스터디원 한 분에게 기능 추가를 부탁드린 적이 있습니다. 기존의 투두 리스트 앱에 메모 기능을 추가하고자 하니, 메모 관련 API를 추가해달라는 요청이었습니다.

프로젝트에는 투두 API 핸들러를 작성하는 `TodoController`와 서비스 로직인 `TodoService`가 있습니다. `TodoService` 는 데이터베이스와 투두 엔티티 객체를 주고받으며 간단한 CRUD를 수행하는 클래스입니다.

```java
// TodoService.java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class TodoService {
    private final TodoJpaRepository todoRepository;

    @Transactional
    public Long add(TodoEntity todo) {
        todoRepository.save(todo);
        return todo.getId();
    }

    public List<TodoEntity> searchAll() {
        return this.todoRepository.findAll();
    }

    public TodoEntity searchById(Long id) {
        return todoRepository.findById(id)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
    }

    @Transactional
    public void updateTodo(TodoEntity todo, String title, Boolean completed) {
        todo.update(title, completed);
        todoRepository.save(todo);
    }

    @Transactional
    public void deleteTodo(TodoEntity todo) {
        todoRepository.delete(todo);
    }
}
```

### 초기 PR: 두 개의 책임을 가진 TodoService

스터디원에게 기능 추가를 완료하면 PR을 올려 저에게 알려달라고 하였고, 스터디원 분이 리뷰를 요청하여 코드를 살펴보았습니다.

PR에서는 `TodoController`와 `TodoService`를 변경한 이력이 있었습니다. 투두 기능과는 전혀 관계가 없는 기능 추가를 요청했기에 두 클래스가 변경될 이유가 없었는데, 해당 클래스에 메모 기능을 추가하고, 메소드명에 `memo`를 붙여서 기능을 구분되게 한 형태였습니다.

```java
// 변경된 TodoService.java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class TodoService {
    private final TodoJpaRepository todoRepository;
    private final TodoMemoJpaRepository todoMemoRepository;

    @Transactional
    public Long add(TodoEntity todo) {
        todoRepository.save(todo);
        return todo.getId();
    }

    @Transactional
    public Long addMemo(TodoMemo todo){
        todoMemoRepository.save(todo);
        return todo.getId();
    }

    public List<TodoEntity> searchAll() {
        return todoRepository.findAll();
    }

    public List<TodoMemo> searchAllMemo(){ return todoMemoRepository.findAll(); }

    public TodoEntity searchById(Long id) {
        return todoRepository.findById(id)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
    }

    public TodoMemo searchMemoById(Long id){
        return todoMemoRepository.findById(id)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
    }

    @Transactional
    public void updateTodo(TodoEntity todo, String title, Boolean completed) {
        todo.update(title, completed);
    }

    @Transactional
    public void updateMemo(TodoMemo todo, String title,String memo, Boolean completed) { todo.update(title,memo,completed);}

    @Transactional
    public void deleteTodo(TodoEntity todo) {

        todoRepository.delete(todo);
    }

    @Transactional
    public void deleteMemo(TodoMemo todo){
        todoMemoRepository.delete(todo);
    }
}
```

### 초기 PR에 대한 수정 요청

투두 기능을 수정할 필요가 있을 경우에 투두와 관련된 클래스를 손을 대어야 하는데, 다른 기능을 덧붙여 여러 책임을 가진 클래스가 되었습니다. 따라서 클래스를 분리할 필요가 있어보였고, 아래와 같이 요청을 하였습니다.

![Alt text](image/10-1.png)

또한 지코바를 하면서 10장을 읽은 지 얼마 안 되었기 때문에, 구두로 왜 `TodoService`에 기능을 추가하면 안되고 다른 클래스로 분리를 해야하는가에 대해 단일 책임 원칙(SRP)을 근거로 설명할 수 있었습니다.

### 최종 PR: 분리된 책임

```java
// 분리된 클래스 TodoMemoService.java

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class TodoMemoService {

    private final TodoMemoJpaRepository todoMemoRepository;

    @Transactional
    public Long addMemo(TodoMemo todo){
        todoMemoRepository.save(todo);
        return todo.getId();
    }

    public List<TodoMemo> searchAllMemo(){ return todoMemoRepository.findAll(); }

    public TodoMemo searchMemoById(Long id){
        return todoMemoRepository.findById(id)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
    }

    @Transactional
    public void updateMemo(TodoMemo todo, String title,String memo, Boolean completed) {
        todo.update(title,memo,completed);
    }

    @Transactional
    public void deleteMemo(TodoMemo todo){
        todoMemoRepository.delete(todo);
    }
}
```

수정된 PR에서는 개선된 사항을 확인할 수 있었습니다. 투두 기능과 메모 기능을 분리하였고, 각 클래스가 하나의 리소스만을 담당하는것을 코드로 확인할 수 있었습니다.

물론 현재의 서비스 클래스는 기능이 추가가 된다면 서비스 클래스에 새로운 함수를 추가하거나 기존 함수를 수정해야 하는 위험이 있습니다. 그러나 클래스를 쪼갬으로써 클래스의 책임을 명확하게 정리할 수 있었던 사례로서 적절하다고 생각하여 가져와봤습니다!