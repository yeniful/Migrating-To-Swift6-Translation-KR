# 마이그레이션 방법 (Migration Strategy)

모듈에서 전체 동시성 검사(Complete Concurrency Checking)를 활성화하면 컴파일러에 의해 많은 데이터 경쟁 방지(Data-race Safety) 문제가 발생할 수 있습니다. 수백 혹은 수천 개의 경고가 발생하는 것은 흔한 일입니다. 이렇게 많은 수의 문제가 발생했을 때, 특히 여러분이 Swift의 데이터 격리(Data Isolation) 모델에 대해 이제 막 알아가기 시작했다면 이 상황이 넘을 수 없는 벽처럼 느껴질 수 있습니다.

**두려워하지 마세요. (Don't Panic)**

여러분은 몇 가지 수정만으로 상당한 진전을 이루고 있는 것을 자주 깨닫게 될 것입니다. 그리고 Swift Concurrency(동시성) 시스템이 어떻게 동작하는지에 대해서도 여러분의 이해가 깊어질 것입니다.

> 중요  
이 가이드는 정답이 아닙니다. 다른 접근법을 생각하는 것에 대해서도 자신감을 가지세요.

## 방법 (Strategy)
이 문서는 좋은 출발점이 될 수 있는 일반적인 방법에 대해서 다룹니다. 모든 프로젝트에 효과가 있는 단 하나의 정답, 즉 은총알은 없습니다. 접근 방식에는 세 가지 핵심 단계가 있습니다:
- 모듈 선택하기
- Swift 5에서 엄격한 검사(Stricter Checking) 활성화하기
- 경고 해결하기
이 과정은 반복될 것입니다. 한 모듈에서 단 하나의 변화가 있더라도 전체 프로젝트 상태에 큰 영향을 미칠 수 있습니다.

## 외부 모듈부터 차근차근 (Begin from the Outside)
프로젝트 가장 바깥쪽의 루트 모듈부터 시작하는 것이 쉬울 수 있습니다. 정의상 이것은 다른 모듈의 dependency(의존성)가 아닙니다. 이 곳에서의 변경은 로컬에만 영향을 미치기 때문에 작업을 계속 유지할 수 있습니다.

그러나 변경 사항을 모듈에 포함할 필요는 없습니다. 안전하지 않은 전역 상태([Unsafe Global State](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/commonproblems#Unsafe-Global-and-Static-Variables))이거나 사소한 [`Sendable`](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/commonproblems#Implicitly-Sendable-Types) 유형의 dependency들은 프로젝트 전체에 많은 경고를 만들 수 있습니다. 보통 이런 dependency들이 가장 먼저 집중해야 할 대상이 될 수 있습니다.

## Swift 5 언어 모드 사용하기 (Use the Swift 5 Language Mode)
직접 확인하지 않고 프로젝트 언어 모드를 Swift 5에서 Swift 6로 바꾸는 것은 상당히 어려울 수 있습니다. 대신, Swift 5 모드를 유지하면서 더 많은 Swift 6 검사 메커니즘을 단계적으로 활성화하는 것이 가능합니다. 이 경우 문제가 경고로만 표시되기 때문에 빌드 및 기능 테스트가 가능한 상태를 유지합니다.

곧 도입될 단일 동시성(Single Concurrency) 기능을 사용하면 시작할 수 있습니다. 이를 통해 한 번에 하나의 특정 유형 문제에 집중할 수 있습니다.

|제안 (Proposal)|설명 (Description)|기능 플래그 (Feature Flag)|
|:-----------:|------------------------------------------------------------|----------------------------|
|[SE-0401](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0401-remove-property-wrapper-isolation.md)|Remove Actor Isolation Inference caused by Property Wrappers|DisableOutwardActorInference|
|[SE-0412](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0412-strict-concurrency-for-global-variables.md)|Strict concurrency for global variables|GlobalConcurrency|
|[SE-0418](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0418-inferring-sendable-for-methods.md)|Inferring `Sendable` for methods and key path literals|InferSendableFromCaptures|

이러한 기능은 순서 상관없이 독립적으로 활성화할 수 있습니다.  
향후 기능 플래그에서 발견한 문제를 해결한 후 다음 단계는 모듈에 대한 [전체 검사 활성화](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/completechecking)를 하는 것입니다. 이를 통해 컴파일러의 나머지 모든 데이터 격리 검사를 켭니다.

## 경고 해결하기 (Address Warnings)
경고를 확인할 때 따라야 하는 한 가지 원칙이 있습니다: 지금 무엇이 사실인지(**what is true now**) 분명하게 하는 것입니다. 문제를 해결하기 위해 코드를 리팩토링하려는 충동을 참아야 합니다.

전체 동시성 검사를 통해 경고를 없애기 위해 필요한 변경량을 최소화하는 것이 좋습니다. 안전하지 않은 옵트아웃(opt-out)을 적용한 경우 이를 후속 리팩토링 기회로 삼아 더 안전한 격리 메커니즘을 도입하세요.
> 참고  
[흔한 컴파일러 에러](https://github.com/yeniful/Migrating-To-Swift6-Translation-KR/blob/main/CommonCompilerErrors.md) 문서를 참조하면 흔하게 마주할 수 있는 문제를 해결하는 방법에 대해 알 수 있습니다.

## 반복 (Iteration)
처음에는 데이터 격리 오류들을 비활성화하는 방법이나 임시 방편을 사용할 가능성이 높습니다. 상위 레벨 모듈의 정지점에 도달했다는 생각이 들면, 임시 방편으로 해결해야하는 여러 종속성 중 하나를 대상으로 지정하세요.

앞으로 나아가기 위해 모든 경고를 없앨 필요는 없습니다. 아주 사소한 변화가 중대한 영향을 미칠 수 있다는 것을 기억하세요. 모듈의 의존성 중 하나가 업데이트되면 언제든지 모듈로 돌아갈 수 있습니다.

