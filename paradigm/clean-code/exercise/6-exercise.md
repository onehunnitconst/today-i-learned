## 자료 추상화

### Cita의 `AnswerChoiceController`

`AnswerChoiceController`는 사용자가 질문에 

* 추상 클래스
```dart
abstract class AnswerChoiceController<T> with ChangeNotifier {
  T get answer;
  bool get isAnswerEmpty;
}
```

답을 구하는 answer 메서드와, 사용자가 답을 선택했는 지 여부를 판별하는 isAnswerEmpty 메소드로 구성되었다.

* 구현 중 하나
```dart
class MultipleImageChoiceSingleSelectController
    extends AnswerChoiceController<int> {
  late List<ImageChoiceCubit> _cubits;
  late int? _activatedIndex;

  @override
  int get answer => _activatedIndex!;

  @override
  bool get isAnswerEmpty => _activatedIndex == null;

  MultipleImageChoiceSingleSelectController();

  void initialize(List<ImageChoiceCubit> cubits) {
    _cubits = cubits;
    _cubits.asMap().entries.forEach(_attachListener);
  }

  void _attachListener(MapEntry<int, ImageChoiceCubit> entry) {
    // ...
  }

  // ...
}
```

구현체에서는 여러 개의 ImageChoiceCubit을 관리하고 이 중 현재 상태가 true인 Cubit의 인덱스를 저장하는데, `answer` 메서드는 이 선택된 인덱스를 값으로 보여주고, `isAnswerEmpty`는 아무것도 선택되지 않았는 지에 대한 검사를 한다.

개발자의 입장에서 굳이 내부의 `_activatedIndex` 값을 알 필요 없고, 추상 클래스로 정의된 `answer`와 `isAnswerEmpty`

* 실제 구현체를 사용하는 파트
```dart
final buttonActivateCubit = ButtonActivateCubit();

controller.addListener(() {
  buttonActivateCubit.setState(!controller.isAnswerEmpty);
});
```

구현체가 어떤 것이냐에 상관 없이, isAnswerEmpty를 호출하여 사용자가 값을 입력했는 지 판별할 수 있다. 

## 디미터 법칙

```dart
Widget build(BuildContext context) {
  final viewModel = Provider.of<HomeViewModel>(context);

  // ...

  if (viewModel.testStatusCubit.state.isEmpty) {
    viewModel.initialize(organs);
  }
}
```

* 다음 코드는 디미터 법칙을 어겼을까?
  * `viewModel`은 클래스의 메서드 $build$가 불러낸 객체
  * viewModel의 getter를 통해 `TestStatusCubit`의 인스턴스를 얻어옴
  * testStatusCubit 이후 state 호출, 이 역시 getter를 통해 얻어옴
  * isEmpty 역시 객체의 api 콜

### 디미터 법칙을 지키도록 수정
```dart
Widget build(BuildContext context) {
  final viewModel = Provider.of<HomeViewModel>(context);

  final testStatusCubit = viewModel.testStatusCubit;

  // ...
  // testStatusCubit 내부에 메서드로 처리

  if (testStatusCubit.isStateEmpty()) {
    viewModel.initialize(organs);
  }
}
```