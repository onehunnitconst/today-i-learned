# 7장: 오류 처리

## 외부 라이브러리의 null 반환 딜레마

```dart
class AuthProvider {
  final MemberRepository repository;
  late final SharedPreferences prefs;

  AuthProvider(this.repository);

  /* ... */

  bool check() {
    int? userId = prefs.getInt("userId");
    if (userId == null) {
      return false;
    }
    return true;
  }

  int getUserId() {
    int? userId = prefs.getInt("userId");
    if (userId == null) {
      throw PetCheeseAuthError("사용자를 찾을 수 없습니다.");
    }
    return userId;
  }

  /* ... */
}
```

AuthProvider는 SharedPreferences라는 외부 라이브러리를 사용한다. 해당 라이브러리의 API 중 `getInt`의 경우 값을 찾을 수 없는 경우 null을 반환한다.
* 위반 사항: null을 반환하지 말라는 클린 코드의 원칙에 어긋난다!
* 해결책: SharedPreferences를 감싸서 해결해보자.

## SharedPreferences 감싸기

```dart
class CustomSharedPreferencesError {
  final String message;

  CustomSharedPreferencesError(this.message);
}

class CustomSharedPreferences {
  late final SharedPreferences prefs;

  CustomSharedPreferences() {
    /* ... */
  }

  /* ... */

  bool checkInt(String key) {
    return prefs.getInt(key) != null;
  }

  int getInt(String key) {
    int? value = prefs.getInt(key);
    if (value == null) {
      throw CustomSharedPreferencesError("값을 찾을 수 없습니다.");
    }
    return value;
  }

  /* ... */
}

class AuthProvider {
  final MemberRepository repository;
  final CustomSharedPreferences prefs;

  AuthProvider(this.repository, this.prefs);

  /* ... */

  bool check() {
    return prefs.checkInt("userId");
  }

  int getUserId() {
    try {
      int userId = prefs.getInt("userId");
      return userId;
    } catch (e) {
      throw PetCheeseAuthError("사용자를 찾을 수 없습니다.");
    }
  }

  /* ... */
}
```

* null에 대한 처리는 감싼 `CustomSharedPreferences`가 해결하고, `AuthProvider`는 null 처리에 대해 전혀 신경 쓸 필요가 없어졌다.