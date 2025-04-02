# Library Evolution

소스 및 ABI 호환성을 유지하면서 동시성을 위해 라이브러리 API에 주석을 달 수 있습니다.

`메인 액터(@MainActor)` 및 `전송 가능한(@Sendable)` 과 같은 동시성 어노테이션은 소스 및 ABI 호환성에 영향을 줄 수 있습니다. 라이브러리 작성자는 기존 API에 주석을 달 때 이러한 영향을 인지하고 있어야 합니다.

## 사전 동시성 어노테이션 (Preconcurrency annotations)

`사전 동시성(@preconcurrency)` 속성은 라이브러리 API에서 직접 사용하여 클라이언트에 대한 소스 또는 ABI 호환성을 깨지 않고 컴파일 시 확인되는 새로운 동시성 요구 사항을 단계적으로 설정할 수 있습니다:

```swift
@preconcurrency @MainActor
struct S { ... }


@preconcurrency
public func performConcurrently(
  completion: @escaping @Sendable () -> Void
) { ... }
```

클라이언트는 새로운 에러를 다운그레이드하기 위해 `@preconcurrency` `임포트(import)`를 사용할 필요가 없습니다. 클라이언트가 최소한의 동시성 검사(minimal concurrency checking)로 빌드하는 경우 `@preconcurrency` API의 오류는 억제됩니다. 클라이언트가 완전한 동시성 검사 또는 Swift 6 언어 모드를 사용하여 빌드하는 경우 오류는 경고로 다운그레이드됩니다.

ABI 호환성을 위해 `@preconcurrency`는 동시성 어노테이션 없이 심볼 이름을 변형하고 고유하게 만든다. API가 일부 동시성 어노테이션과 함께 도입되었고 나중에 추가 동시성 어노테이션을 포함하도록 업데이트되는 경우 `@preconcurrency` 를 적용하는 것만으로는 맹글링을 유지하는 데 충분하지 않습니다. 동시성 어노테이션을 보다 정밀하게 제어해야 하는 경우 `@_silgen_name`을 사용할 수 있습니다.

C, C++ 및 Objective-C에서 가져온 모든 API는 자동으로 `@preconcurrency` 로 간주된다는 점에 유의하세요. 소스 또는 ABI 호환성을 깨뜨리지 않고 `__attribute__((__swift_attr__("<attribute name>")))` 를 사용하여 이러한 API에 항상 동시성 속성을 적용할 수 있습니다.

## 전송 가능한 (Sendable)

### 특정 유형에 대한 준수 (Conformances on concrete types)

`Sendable`을 특정 타입에 준수하도록 추가하는 것은, 조건부 준수를 포함하여, 일반적으로 **소스 호환성이 유지되는 변경사항**입니다.

**소스와 ABI 적합성:**

```swift
-public struct S
+public struct S: Sendable
```

다른 프로토콜 준수(conformance)와 마찬가지로, `Sendable`을 준수하도록 만들면 특정 타입이 더 구체적인 요구사항을 만족하는 경우 오버로드 해석(overload resolution)에 영향을 미칠 수 있습니다.  
그러나 `Sendable` 준수를 기준으로 오버로드되는 API가 타입 추론을 변경하여 소스 호환성을 깨뜨리거나 프로그램 동작에 영향을 미칠 가능성은 낮습니다.

특정 타입의 제네릭 타입 파라미터가 아닌, 해당 타입 자체에 `Sendable` 준수를 추가하는 것은 항상 ABI 호환성이 유지되는 변경입니다.

### 제너릭 요구사항 (Generic requirements)

제너릭 유형 또는 함수에 `Sendable` 준수 요구사항을 추가하는 것은 소스 호환성이 유지되지 않는 변경입니다. 
그 이유는, 클라이언트 코드에서 전달하는 제네릭 인자에 새로운 제한이 생기기 때문입니다.

**소스와 ABI가 호환되지 않습니다:**

```swift
-public func generic<T>
+public func generic<T> where T: Sendable
```

**해결 방법:** 유형 또는 함수 선언에 `@preconcurrency`를 적용하여 요구 사항 불일치를 오류가 아닌 경고로 다운그레이드하고 ABI 호환성을 유지합니다:

```swift
@preconcurrency
public func generic<T> where T: Sendable { ... }
```

### 함수 타입 (Function types)

제너릭 요구사항과 마찬가지로 함수 유형에 `@Sendable`을 추가하는 것은 소스 및 ABI와 호환되지 않는 변경 사항입니다:

**소스와 ABI가 호환되지 않습니다::**

```swift
-public func performConcurrently(completion: @escaping () -> Void)
+public func performConcurrently(completion: @escaping @Sendable () -> Void)
```

**해결방법**: 둘러싸는 함수 선언에 `@preconcurrency를` 적용하여 요구 사항 불일치를 오류가 아닌 경고로 다운그레이드하고 ABI 호환성을 유지합니다:

```swift
@preconcurrency
public func performConcurrently(completion: @escaping @Sendable () -> Void)
```
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
