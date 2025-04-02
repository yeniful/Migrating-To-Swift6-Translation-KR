# Incremental Adoption
Swift 동시성 기능을 프로젝트에 점진적으로 도입하는 방법을 알아보세요.

Swift 6 언어 모드로 프로젝트를 마이그레이션하는 것은 일반적으로 단계적으로 이루어집니다. 사실, 많은 프로젝트들이 Swift 6가 출시되기도 전에 프로세스를 시작했습니다. 동시성 기능을 점진적으로 계속 도입하여 그 과정에서 발생하는 문제를 해결할 수 있습니다. 이를 통해 전체 프로젝트를 중단하지 않고 점진적인 진행을 이룰 수 있습니다.

Swift에는 다양한 언어 기능과 표준 라이브러리 API가 포함되어 있어 점진적 채택을 더 쉽게 할 수 있습니다.

## 콜백 기반 함수 래핑 (Wrapping Callback-Based Functions)

완료 시 단일 함수를 인자로 받아 호출하는 API는 Swift에서 매우 일반적인 패턴입니다. 비동기 컨텍스트에서 직접 사용할 수 있는 이러한 함수의 버전을 만들 수 있습니다.

```swift
func updateStyle(backgroundColor: ColorComponents, completionHandler: @escaping () -> Void) {
    // ... 
}
```

이것은 콜백을 사용하여 클라이언트에게 작업이 완료되었음을 알리는 함수의 예입니다. 호출자가 문서를 참조하지 않고는 콜백이 언제 또는 어떤 스레드에서 호출될지 결정할 방법이 없습니다.

_컨티뉴에이션(continuations)_ 을 사용하여 이 함수를 비동기 버전으로 래핑할 수 있습니다.

```swift
func updateStyle(backgroundColor: ColorComponents) async {
    await withCheckedContinuation { continuation in
        updateStyle(backgroundColor: backgroundColor) {
            // ... 여기서 일을 수행 ...
            
            continuation.resume()
        }
    }
}
```

> 참고
정확히 한 번 컨티뉴에이션을 재개(resume)하려면 주의를 기울일 필요가 있다. 호출을 놓치면 호출 작업이 무기한 중단됩니다. 반면에, 체크된 컨티뉴에이션을 두 번 이상 재개하면 예상되는 충돌이 발생하여 정의되지 않은 동작으로부터 사용자를 보호합니다.

비동기 버전을 사용하면 더 이상 모호함이 없습니다. 함수가 완료된 후, 실행은 항상 시작했던 것과 동일한 컨텍스트에서 재개됩니다.

```swift
await updateStyle(backgroundColor: color)
// 스타일이 업데이트되었습니다
```

withCheckedContinuation 함수는 비동기가 아닌 코드 혹은 비동기 코드를 인터페이스할 수 있도록 존재하는 [표준 라이브러리 API 제품군]([suite of standard library APIs](https://developer.apple.com/documentation/swift/concurrency#continuations)) 중 하나입니다.

> 참고
> 프로젝트에 비동기 코드를 도입하면 데이터 격리 검사 위반 사항을 표면화할 수 있습니다. 이러한 문제를 이해하고 해결하려면 [격리 경계를 넘나들기]([Crossing Isolation Boundaries](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/incrementaladoption/commonproblems#Crossing-Isolation-Boundaries)) 를 참조하세요.


## 동적 격리 (Dynamic Isolation)

주석(annotaions) 및 기타 언어 구성을 사용하여 프로그램의 격리를 정적으로 표현하는 것은 강력하고 간결합니다. 하지만 모든 종속성을 동시에 업데이트하지 않고 정적 격리를 도입하는 것은 어려울 수 있습니다.

동적 격리는 데이터 격리를 설명하기 위한 대체 수단으로 사용할 수 있는 런타임 메커니즘을 제공합니다.  _동일_ 모듈 내에 있는 구성 요소라도 아직 업데이트되지 않은 다른 구성 요소와 Swift 6 구성 요소를 인터페이스하는 데 필수적인 도구가 될 수 있습니다.

### 내부 전용 격리 (Internal-Only Isolation)

프로젝트 내의 참조 유형이 `메인액터 (MainActor)` 정적 격리로 가장 잘 설명될 수 있다고 판단했다고 가정해 보겠습니다.

```swift
@MainActor
class WindowStyler {
    private var backgroundColor: ColorComponents

    func applyStyle() {
        // ...
    }
}
```

`메인액터 (MainActor)` 격리는 논리적으로 옳을 수 있습니다. 그러나 이 유형이 마이그레이션되지 않은 다른 위치에서 사용되는 경우 여기에 정적 격리를 추가하려면 많은 추가 변경이 필요할 수 있습니다. 대안으로 동적 격리를 사용하여 범위를 제어할 수 있습니다.

```swift
@MainActor
class WindowStyler {
    private var backgroundColor: ColorComponents

    func applyStyle() {
        MainActor.assumeIsolated {
            // 다른 `MainActor` 상태 사용 및 상호작용
        }
    }
}
```

여기서 `메인액터 (MainActor)` 격리는 클래스에 내재화되었습니다. 이렇게 하면 모든 변경 사항이 해당 유형에 국한되므로 해당 유형의 클라이언트에 영향을 주지 않고 변경할 수 있습니다.

그러나 이 기법의 가장 큰 단점은 유형의 실제 격리 요구 사항이 보이지 않는다는 것입니다. 클라이언트가 이 공개 API를 기반으로 변경 여부나 방법을 결정할 수 있는 방법이 없습니다. 이 접근 방식은 다른 옵션을 모두 사용한 경우에만 임시 해결책으로 사용해야 합니다.

### 사용 전용 격리 (Usage-Only Isolation)

한 유형 내에서만 격리하는 것이 비현실적인 경우, 대신 해당 API 사용만 포함하도록 격리를 확장할 수 있습니다.
이렇게 하려면 먼저 유형에 정적 격리를 적용한 다음 모든 사용 위치에서 동적 격리를 사용합니다:

```swift
@MainActor
class WindowStyler {
    // ...
}


class UIStyler {
    @MainActor
    private let windowStyler: WindowStyler
    
    func applyStyle() {
        MainActor.assumeIsolated {
            windowStyler.applyStyle()
        }
    }
}
```

정적 격리 및 동적 격리를 결합하면 변경 범위를 점진적으로 유지하는 데 강력한 도구가 될 수 있습니다.

### 명시적 메인 액터 컨텍스트 (Explicit MainActor Context)

`assumeIsolated` 메서드는 동기식이며, 가정이 잘못된 경우 실행을 방지하여 런타임에서 유형 시스템으로 격리 정보를 복구하기 위해 존재합니다. `메인 액터(MainActor)` 유형에는 비동기 컨텍스트에서 수동으로 격리를 전환하는 데 사용할 수 있는 메서드도 있습니다.

```swift
// MainActor여야 하지만 아직 업데이트되지 않은 타입을 예로 들어 보겠습니다.
class PersonalTransportation {
}


await MainActor.run {
    // 여기서 메인 액터로 격리
    let transport = PersonalTransportation()
    
    // ...
}
```


정적 격리를 사용하면 컴파일러가 필요에 따라 격리를 전환하는 프로세스를 확인하고 자동화할 수 있다는 점을 기억하세요. 정적 격리와 함께 사용하더라도 MainActor.run이 정말 필요한 시기를 판단하기는 어려울 수 있습니다. MainActor.run은 마이그레이션 중에 유용할 수 있지만, 시스템의 격리 요구 사항을 정적으로 표현하는 대신 사용해서는 안 됩니다. 궁극적인 목표는 여전히 @MainActor를 PersonalTransportation에 적용하는 것이어야 합니다.

## 누락된 주석 (Misssing Annotations)

동적 격리는 런타임에 격리를 표현할 수 있는 도구를 제공합니다. 그러나 마이그레이션되지 않은 모듈에서 누락된 다른 동시성 속성을 설명해야 할 수도 있습니다.

### 표시되지 않은 전송 가능한 클로저 (Unmarked Sendable Closures)

클로저의 전송 가능성(sendability)은 컴파일러가 본문에 대한 격리를 추론하는 방식에 영향을 줍니다. 실제로 격리 경계를 넘나들지만 `Sendable` 어노테이션이 누락된 콜백 클로저는 동시성 시스템의 중요한 불변성을 위반합니다.

```swift
// Swift 6 이전 모듈 내의 정의
extension JPKJetPack {
    // @Sendable 어노테이션이 없음에 유의하세요.
    static func jetPackConfiguration(_ callback: @escaping () -> Void) {
        // 잠재적으로 격리 도메인을 교차할 수 있음
    }
}


@MainActor
class PersonalTransportation {
    func configure() {
        JPKJetPack.jetPackConfiguration {
            // 메인 액터 격리는 여기서 추론됩니다.
            self.applyConfiguration()
        }
    }


    func applyConfiguration() {
    }
}
```

`jetPackConfiguration` 이 다른 격리 도메인에서 클로저를 호출할 수 있는 경우, `@Sendable`로 표시해야 합니다. 마이그레이션되지 않은 모듈이 아직 이 작업을 수행하지 않은 경우 잘못된 액터 추론이 발생합니다. 이 코드는 문제 없이 컴파일되지만 런타임에 크래시됩니다.

> 참고
컴파일러가 컴파일러에 표시되는 정보 부족을 감지하거나 진단할 수 없습니다.

이 문제를 해결하려면 `@Sendable` 을 사용하여 클로저에 수동으로 주석을 달 수 있습니다. 이렇게 하면 컴파일러가 `메인 액터(MainActor)` 격리를 유추하지 못합니다. 컴파일러는 이제 액터 격리가 변경될 수 있다는 것을 알기 때문에 콜사이트(callsite)에 태스크가 필요하고 태스크에 대기(await) 상태가 필요합니다.

```swift
@MainActor
class PersonalTransportation {
    func configure() {
        JPKJetPack.jetPackConfiguration { @Sendable in
            // Sendable 클로저는 액터 격리를 유추하지 않습니다,
            // 이 컨텍스트를 격리되지 않게 만듭니다.
            Task {
                await self.applyConfiguration()
            }
        }
    }


    func applyConfiguration() {
    }
}
```

또는 컴파일러 플래그를 사용하여 모듈에 대한 런타임 격리 어설션을 비활성화할 수도 있습니다(-disable-dynamic-actor-isolation). 이렇게 하면 동적 액터 격리의 모든 런타임 적용이 억제됩니다.

> 경고
이 플래그는 주의해서 사용해야 합니다. 이러한 런타임 검사를 비활성화하면 데이터 격리 위반이 허용될 수 있습니다.

## 디스패치 시리얼 큐와 액터 통합하기 (Integrating DispatchSerial Queue with Actors)

기본적으로 액터가 작업을 예약하고 실행하는 데 사용하는 메커니즘은 시스템 정의입니다. 하지만 이를 재정의하여 사용자 정의 구현을 제공할 수 있습니다. `DispatchSerialQueue` 유형에는 이 기능에 대한 기본 지원이 포함되어 있습니다.

```swift
actor LandingSite {
    private let queue = DispatchSerialQueue(label: “something”)

    nonisolated var unownedExecutor: UnownedSerialExecutor {
        queue.asUnownedSerialExecutor()
    }


    func acceptTransport(_ transport: PersonalTransportation) {
        // 이 함수는 대기열에서 실행됩니다
    }
}
```

`DispatchQueue`에 의존하는 코드와의 호환성을 유지하면서 액터 모델로 유형을 마이그레이션하려는 경우 유용할 수 있습니다.

## 이전 버전과의 호환성 (Backwards Compatibility)

타입 시스템의 일부인 정적 격리는 공용 API에 영향을 미친다는 점을 명심해야 합니다. 하지만 기존 클라이언트를 손상시키지 않고도 Swift 6용 API를 개선하는 방식으로 자체 모듈을 마이그레이션할 수 있습니다.

`WindowStyler` 가 공용 API라고 가정해 봅시다. `메인 액터(MainActor)`와 격리되어야 한다고 결정했지만 클라이언트에 대한 하위 호환성을 보장하고 싶다고 가정해 보겠습니다.

```swift
@preconcurrency @MainActor
public class WindowStyler {
    // ...
}
```

이런 식으로 `@preconcurrency` 를 사용하면 클라이언트 모듈에서도 전체 검사를 활성화하는 조건부 격리가 표시됩니다. 이렇게 하면 아직 Swift 6을 도입하지 않은 클라이언트와의 소스 호환성을 유지할 수 있습니다.

## 종속성 (Dependencies)

종속성으로 임포트해야 하는 모듈을 제어할 수 없는 경우가 종종 있습니다. 이러한 모듈이 아직 Swift 6를 채택하지 않은 경우 해결하기 어렵거나 불가능한 오류가 발생할 수 있습니다.
마이그레이션되지 않은 코드를 사용하면 여러 가지 종류의 문제가 발생할 수 있습니다. 이러한 상황 중 상당수는 `@preconcurrency` 어노테이션이 도움이 될 수 있습니다:

- [전송할 수 없는 유형](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/commonproblems#Crossing-Isolation-Boundaries)
- [프로토콜 적합성 격리](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/commonproblems#Crossing-Isolation-Boundaries)의 불일치

## C/Objective-C

주석을 사용하여 C 및 Objective-C API에 대한 Swift 동시성 지원을 노출할 수 있습니다. 이것은 Clang의 [동시성-특화 주석](https://clang.llvm.org/docs/AttributeReference.html#customizing-swift-import):에 의해 가능해졌다.


```
__attribute__((swift_attr(“@Sendable”)))
__attribute__((swift_attr(“@_nonSendable”)))
__attribute__((swift_attr("nonisolated")))
__attribute__((swift_attr("@UIActor")))
__attribute__((swift_attr("sending")))


__attribute__((swift_async(none)))
__attribute__((swift_async(not_swift_private, COMPLETION_BLOCK_INDEX))
__attribute__((swift_async(swift_private, COMPLETION_BLOCK_INDEX)))
__attribute__((__swift_async_name__(NAME)))
__attribute__((swift_async_error(none)))
__attribute__((__swift_attr__("@_unavailableFromAsync(message: \"" msg "\")")))
```

파운데이션을 가져올 수 있는 프로젝트로 작업할 때 `NSObjCRuntime.h` 에서 다음 주석 매크로를 사용할 수 있습니다.


```
NS_SWIFT_SENDABLE
NS_SWIFT_NONSENDABLE
NS_SWIFT_NONISOLATED
NS_SWIFT_UI_ACTOR
NS_SWIFT_SENDING


NS_SWIFT_DISABLE_ASYNC
NS_SWIFT_ASYNC(COMPLETION_BLOCK_INDEX)
NS_REFINED_FOR_SWIFT_ASYNC(COMPLETION_BLOCK_INDEX)
NS_SWIFT_ASYNC_NAME
NS_SWIFT_ASYNC_NOTHROW
NS_SWIFT_UNAVAILABLE_FROM_ASYNC(msg)
```

## Objective-C 라이브러리에서 누락된 격리 어노테이션 처리하기

`SDK` 및 기타 `Objective-C` 라이브러리가 Swift 동시성을 채택하는 과정에서 문서로만 설명되어 있던 컨트랙트를 코딩하는 작업을 거치는 경우가 많습니다. 예를 들어, Swift 동시성 이전에는 API가 “이것은 항상 메인 스레드에서 호출됩니다” 와 같은 주석으로 스레딩 동작을 문서화해야 하는 경우가 많았습니다.
Swift 동시성을 사용하면 이러한 코드 주석을 컴파일러 및 런타임 강제 격리 검사로 전환할 수 있으며, 이러한 API를 채택할 때 Swift가 이를 확인합니다.
예를 들어, 가상의 `NSJetPack` 프로토콜은 일반적으로 메인 스레드에서 모든 델리게이트 메서드를 호출하므로 이제 `메인 액터(MainActor)` 격리 상태가 되었습니다.
라이브러리 작성자는 Swift에서 `@MainActor`를 사용하여 유형에 주석을 다는 것과 동일한 `NS_SWIFT_UI_ACTOR` 속성을 사용하여 `메인 액터 (MainActor)` 가 격리된 것으로 표시할 수 있습니다:

```swift
NS_SWIFT_UI_ACTOR
@protocol NSJetPack // 가상의 프로토콜
  // ...
@end
```

덕분에 이 프로토콜의 모든 멤버 메서드는 `@MainActor` 격리를 상속하며 대부분의 메서드에서 이것이 정확합니다.
하지만 이 예제에서는 이전에 다음과 같이 문서화된 메서드를 고려해 보겠습니다:

```swift
NS_SWIFT_UI_ACTOR // 최근 SDK 감사에서 MainActor를 사용하여 주석을 단 SDK 작성자
@protocol NSJetPack // 가상의 프로토콜
/* 이 제트팩이 정말 높은 고도에서 비행을 지원하면 YES를 반환합니다!
 
 JetPackKit은 이 메서드를 항상 메인 스레드에서 호출하는 것이 아니라 다양한 시점에 호출합니다. 예를 들어, ...
*/
@property(readonly) BOOL supportsHighAltitude;


@end
```

이 메서드의 격리는 둘러싸는 타입의 주석으로 인해 실수로 `@MainActor`로 유추되었습니다. 메인 액터에서 호출될 수도 있고 그렇지 않을 수도 있는 다른 스레딩 전략을 구체적으로 문서화했지만 메서드에 이러한 의미를 어노테이션하는 것이 실수로 누락되었습니다.

이는 가상의 `JetPackKit` 라이브러리에서 발생한 어노테이션 문제입니다. 구체적으로, 메서드에 `분리되지 않은(nonisolated)` 어노테이션이 누락되어 Swift에 정확하고 예상되는 실행 시맨틱을 알려줄 수 있는 어노테이션이 누락되었습니다.

이 라이브러리를 채택한 Swift 코드는 다음과 같이 보일 수 있습니다:

```swift
@MainActor
final class MyJetPack: NSJetPack {
  override class var supportsHighAltitude: Bool { // Swift 6 모드에서 런타임 크래시 발생
    true
  }
}
```

위의 코드는 런타임 검사와 함께 충돌하며, 이는 Objective-C의 Swift 동시성 환경이 아닌 영역에서 Swift로 넘어올 때, 실제로 메인 액터에서 실행되고 있는지를 보장하기 위한 것입니다.

이러한 문제를 자동으로 감지하고, 해당 기대가 충족되지 않을 경우 런타임에 크래시를 발생시키는 것은 Swift 6의 새로운 기능입니다. 이러한 문제를 진단하지 않은 채로 두면, 실제로 감지하기 어려운 데이터 경쟁 상태가 발생할 수 있으며, Swift 6가 보장하는 데이터 경쟁 방지 원칙을 약화시킬 수 있습니다.

이러한 실패에는 다음과 같은 백트레이스(backtrace)가 포함될 것입니다:

```
* thread #5, queue = 'com.apple.root.default-qos', stop reason = EXC_BREAKPOINT (code=1, subcode=0x1004f8a5c)
  * frame #0: 0x00000001004..... libdispatch.dylib`_dispatch_assert_queue_fail + 120
    frame #1: 0x00000001004..... libdispatch.dylib`dispatch_assert_queue + 196
    frame #2: 0x0000000275b..... libswift_Concurrency.dylib`swift_task_isCurrentExecutorImpl(swift::SerialExecutorRef) + 280
    frame #3: 0x0000000275b..... libswift_Concurrency.dylib`Swift._checkExpectedExecutor(_filenameStart: Builtin.RawPointer, _filenameLength: Builtin.Word, _filenameIsASCII: Builtin.Int1, _line: Builtin.Word, _executor: Builtin.Executor) -> () + 60
    frame #4: 0x00000001089..... MyApp.debug.dylib`@objc static JetPack.supportsHighAltitude.getter at <compiler-generated>:0
    ...
    frame #10: 0x00000001005..... libdispatch.dylib`_dispatch_root_queue_drain + 404
    frame #11: 0x00000001005..... libdispatch.dylib`_dispatch_worker_thread2 + 188
    frame #12: 0x00000001005..... libsystem_pthread.dylib`_pthread_wqthread + 228
```


> 참고
이러한 문제가 발생했을 때, 그리고 문서와 API 주석을 조사하여 무언가가 잘못 주석이 달린 것을 발견했을 때, 문제의 근본 원인을 해결하는 가장 좋은 방법은 문제를 라이브러리 관리자에게 다시 보고하는 것입니다.

보시다시피, 런타임은 호출에 메인 액터 (MainActor)에서 실행되는 실행자 검사를 주입했고, 디스패치 대기열 어설션이 실패했습니다. 이것은 교활하고 디버깅하기 어려운 데이터 경주를 방지합니다.

이 문제에 대한 올바른 장기적 해결책은 라이브러리가 메소드의 주석을 `격리되지 않은(nonisolated)`로 표시하여 수정하는 것입니다.

```swift
// API를 제공하는 라이브러리의 솔루션:
@property(readonly) BOOL supportsHighAltitude NS_SWIFT_NONISOLATED;
```

라이브러리에서 주석 문제를 해결할 때까지 다음과 같이 올바르게 `격리되지 않은(nonisolated)` 메서드를 사용하여 메서드를 확인할 수 있습니다.

```swift
// Swift 6 모드에서 실행하기를 원하는 클라이언트 코드를 채택하는 해결책:
@MainActor
final class MyJetPack: NSJetPack {
    // Correct
    override nonisolated class var supportsHighAltitude: Bool {
        true
    }
}
```

이런 식으로 Swift는 그 방법이 메인 액터 격리를 필요로 한다는 올바르지 않은 가정을 확인하지 않는 것을 알게 된다.
