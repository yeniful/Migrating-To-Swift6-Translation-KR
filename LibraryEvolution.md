# Library Evolution

### Main Actor 주석

### 프로토콜과 타입

프로토콜이나 타입 선언에 @MainActor를 추가하는 것은 소스 및 ABI와 호환이 깨질 수 있는 변경입니다.

>💬 번역자의 의견
>
>ABI (Application Binary Interface)는 컴파일된 바이너리 코드가 실행 시간에 상호 작용하는 방식을 정의하는 인터페이스입니다.



**소스, ABI와 호환되지 않는 변경**

```diff
-public protocol P
+@MainActor public protocol P

-public class C
+@MainActor public class C
```

프로토콜, 타입 선언에 `@MainActor`를 추가하는 것은 다른 동시성 주석보다 넓은 영향을 갖습니다. @MainActor 주석은 프로토콜 준수, 서브클래스, extension 메서드를 포함하는 클라이언트 코드 내내 추론될 수 있기 때문입니다.



프로토콜이나 타입 선언에 `@preconcurrency`를 적용하는 것은 동시성 체크 단계 기반에서 발생할 수 있는 actor 고립 오류들을 경고로 다운그레이드 해줍니다. 하지만, `@preconcurrency`는 클라이언트 코드 내 다른 선언들에서 `@preconcurrency @MainActor`가 추론되는 상황에서 ABI 호환성을 유지하기에 충분하지 않습니다. 예를들면, 클라이언트 라이브러리의 다음 API를 고려하세요.

```swift
extension P {
  public func onChange(action: @escaping @Sendable () -> Void)
}
```

만약 P에 `@preconcurrency @MainActor`로 주석이 달린 경우, 이런 주석들은 extension 메서드 상에서 추론됩니다. 만약 extension 메서드가 ABI 호환성 제약을 가진 라이브러리의 일부라면, `@preconcurrency`는 모든 동시성 관련 주석들을 바꾸어 제거해버립니다. 이는 클라이언트 라이브러리 상에서 적절한 고립을 명시적으로 적용해서 해결될 수 있습니다.

```swift
extension P {
  nonisolated public func onChange(action: @escaping @Sendable () -> Void)
}
```

선언의 ABI를 정확하게 제어하기 위한 언어 지원이 [개발 중](https://forums.swift.org/t/pitch-controlling-the-abi-of-a-declaration/75123)에 있습니다.



### 메서드 선언 및 타입들

메서드 선언이나 메서드 타입에 `@MainActor`를 추가하는 것은 소스 및 ABI와의 호환이 깨질 수 있는 변경입니다.

```diff
-public func runOnMain()
+@MainActor public func runOnMain()

-public func preformConcurrently(completion: @escaping () -> Void)
+public func performConcurrently(completion: @escaping @MainActor () -> Void)
```



**해결하려면:** 아래와 같이 메서드 선언에 `@preconcurrency`를 적용해서 요구사항 실패를 경고로 다운그레이드 하고 ABI를 유지할 수 있습니다.

```swift
@preconcurrency @MainActor
public func runOnMain() { ... }

@preconcurrency
public func performConcurrently(completion: @escaping @MainActor () -> Void) { ... }
```



### `sending` 매개변수 및 결과들

결과에 `sending`을 추가하는 것은 클라이언트 코드에서 제한을 해제해주는데, 항상 소스와 ABI 호환 가능한 변경입니다.



**소스 및 ABI와 호환 가능함:**

```diff
-public func getValue() -> NotSendable
+public func getValue() -> sending NotSendable
```

하지만, 매개변수에 `sending`을 추가하는 것은 호출자 시점에서 더욱 제한적입니다.

```diff
-public func takeValue(_: NotSendable)
+public func takeValue(_: sending NotSendable)
```

이 경우에는 소스 호환성을 깨지 않으면서 매개변수에 새로운 `sending` 주석을 추가하는 방법이 없습니다.



### `@Sendable`을 `sending`으로 교체하기

클로져 매개변수에 대해 존재하는 `@Sendable` 주석을 `sending`으로 교체하는 것은 소스 호환 가능하지만, ABI에 호환되지는 않는 변경입니다.



**소스 호환 가능, ABI 호환 불가능한 변경:**

```diff
-public func takeValue(_: @Sendable @escaping () -> Void)
+public func takeValue(_: sending @escaping () -> Void)
```



**해결 하려면:** 매개변수에 `sending`을 추가하면, 이름 맹글링을 변경하므로, 모든 채택은 `@_silgen_name`을 사용해서 맹글링을 유지해야합니다. 매개변수 위치에서 `sending`을 채택하면, 매개변수의 소유권 규칙을 유지해야합니다. 만약 매개변수가 암시적인 소유권 수정자를 갖고 있다면, 추가적인 주석은 필요하지 않습니다. 생성자를 제외한 모든 메서드들은 `_shared sending`을 사용해서 소유권 규칙을 보존합니다:

```swift
public func takeValue(_: __shared sending NotSendable)
```

생성자의 경우, `sending`이 기본적인 소유권 규칙을 유지해줍니다. 따라서 생성자 매개변수들에 `sending`을 채택할때, 소유권 수정자를 명시할 필요가 없습니다.

```swift
public class C {
  public init(ns: sending NotSendable)
}
```
