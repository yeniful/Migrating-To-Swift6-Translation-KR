# Source Compatibility

# 소스 호환성

##### 잠재적으로 발생할 수 있는 소스 호환성에 대한 개요를 알아봅니다.



Swift 6에 대해 소스 호환성에 영향을 주는 많은 보완 제안(evolution proposals)이 있습니다. 이는 모두 Swift 5 언어 모드에서는 선택적으로 활성화(opt-in) 해야하는 기능들입니다.

> **Note**
>
> 이전 릴리즈된 모드에 대한 마이그레이션 가이드를 보고 싶다면, [Migrating to Swift 5](https://www.swift.org/migration-guide-swift5/) 를 참고하세요.



### Future Enum Cases 다루기

[SE-0192](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0192-non-exhaustive-enums.md): `NonfrozenEnumExhastivity`

enum 에서 @unknown default case가 없는 경우 발생가능한 경고가 오류로 변경되었습니다.



### 간결한 magic file names

[SE-0274](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0274-magic-file.md): `ConciseMagicFile`

특별한 표현인 #file 이 인간이 읽을 수 있는 파일 이름과 모듈 이름이 포함된 문자열로 변경되었습니다.



### 후행클로져를 위한 Forward-scan 매칭

[SE-0286](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0286-forward-scan-trailing-closures.md): `ForwardTrailingClosures`

다수의, default closure parameter 들을 포함하는 코드에 발생할 수 있습니다.



### 동시성 체크를 위한 점진적 마이그레이션

[SE-0337](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0337-support-incremental-migration-to-concurrency-checking.md): `StrictConcurrency`

데이터 경쟁(data races) 문제가 발생 가능한 코드에 대한 에러를 소개합니다.

> **Note**
> 이 기능은 암시적으로 [IsolatedDefaultValues](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/sourcecompatibility#Isolated-default-value-expressions), [GlobalConcurrency](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/sourcecompatibility#Strict-concurrency-for-global-variables), and [RegionBasedIsolation](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/sourcecompatibility#Region-based-Isolation)을 가능하게 합니다.



### 암시적으로 개방된 실존 타입들 (Existentials)

[SE-0352](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0352-implicit-open-existentials.md): `ImplicitOpenExistentials`

실존 타입(existentials), 제네릭 타입(generic types)들을 포함하는 기능을 위한 오버로드 해결에 영향을 줄 수 있습니다.



### 정규식 리터럴 (Regex Literals)

[SE-0354](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0354-regex-literals.md): `BareSlashRegexLiterals`

이전에 `/`를 사용하던 코드 파싱 방식에 영향을 줄 수 있습니다.



### @UIApplicationMain and @NSApplicationMain 미사용

[SE-0384](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0384-importing-forward-declared-objc-interfaces-and-protocols.md): `ImportObjcForwardDeclarations`

기존 소스들과 충돌할 수 있는, 이전에는 보이지 않았던 유형들을 노출합니다.



### 이전에 선언된 Objective-C interface, protocol들 import 하기 

[SE-0384](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0384-importing-forward-declared-objc-interfaces-and-protocols.md): `ImportObjcForwardDeclarations`

기존 소스들과 충돌할 수 있는 이전에 보이지 않았던 타입들을 노출합니다.



### Property Wrapper에 의한 Actor 고립 추론 제거

[SE-0401](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0401-remove-property-wrapper-isolation.md): `DisableOutwardActorInference`

타입과 해당 멤버들의 추론된 격리를 변경할 수 있습니다.



### 고립된 기본 값 표현

[SE-0411](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0411-isolated-default-values.md): `IsolatedDefaultValues`

데이터 경쟁 문제를 발생하는 코드 에러를 소개합니다.



### 전역 변수를 위한 Strict concurrency

[SE-0412](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0412-strict-concurrency-for-global-variables.md): `GlobalConcurrency`

데이터 경쟁 문제를 발생하는 코드 에러를 소개합니다.



### 지역 기반 격리

[SE-0414](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0414-region-based-isolation.md): `RegionBasedIsolation`

`Actor.assumeIsolated` 메서드의 제약을 증가시킵니다.



### 메서드 및 키 경로(key path) 리터럴에 대한 `Sendable` 유추

[SE-0418](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0418-inferring-sendable-for-methods.md): `InferSendableFromCaptures`

전송 가능성(sendability)만 다른 기능 들에 대한 오버로드 해결에 영향을 줄 수 있습니다.



### strict concurrency 미적용 문맥에 동적 actor 격리 적용하기

[SE0423](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0423-dynamic-actor-isolation.md): `DynamicActorIsolation`

런타임 격리가 기대와 맞지 않을 경우, 기존 코드에 영향을 미칠 수 있는 새로운 assertion들을 소개합니다.

> 💬 번역자의 추가 설명
> assertion : 코드에서 특정 조건이 반드시 충족되어야할 때 사용하는 디버깅 도구입니다.
