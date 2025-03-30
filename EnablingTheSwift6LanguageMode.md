# 동시성 확인 (Enabling The Swift 6 Language Mode)
##### Swift 6 언어 모드를 활성화하여 코드에 데이터 경합이 없도록 합니다.

## Swift 컴파일러로 하는 방법 (Using the Swift compiler)
커맨드 라인에서 swift 또는 swiftc를 직접 실행할 때, Swift 6 언어 모드를 사용하기 위해서 -swift-version 6를 사용합니다. :
```
~ swift -swift-version 6 main.swift
```

## Swift PM으로 하는 방법 (Using SwiftPM)
### 커맨드 라인에서 호츌 (Command-line invocation)
Swift 패키지 관리자(SPM, Swift Package Manager)의 커맨트 라인에서 `-Xswiftc flag` 플래그를 통해 `-swift-version 6`를 사용할 수 있습니다.:
```
~ swift build -Xswiftc -swift-version -Xswiftc 6
~ swift test -Xswiftc -swift-version -Xswiftc 6
```

### 패키지 매니페스트 (Package manifest)
`swift-tools-version` 6.0을 사용하는 `Package.swift` 파일은 모든 타깃들에 대해 Swift 6 언어 모드를 활성화합니다. 패키지의 `swiftLanguageModes` 속성을 사용하여 패키지 전체에 대한 언어 모드를 설정하는 방법도 여전히 가능합니다. 하지만 새로운 `swiftLanguageMode` 빌드 설정을 사용하여 필요에 따라 타깃 별로 언어 모드를 변경할 수 있습니다. :
```
// swift-tools-version: 6.0

let package = Package(
    name: "MyPackage",
    products: [
        // ...
    ],
    targets: [
        // 기본 도구 언어 모드 6를 사용
        .target(
            name: "FullyMigrated",
        ),
        // 여전히 언어 모드 5가 필요
        .target(
            name: "NotQuiteReadyYet",
            swiftSettings: [
                .swiftLanguageMode(.v5)
            ]
        )
    ]
)
```
패키지가 이전 Swift 툴체인 버전을 계속 지원하고 각 타깃 별로 `swiftLanguageMode`를 사용해야 하는 경우, 버전 6 이전의 툴체인에 대한 버전별 매니페스트를 만들어야 하는 것을 유의하세요. 예를 들어, 5.9 툴체인 이상을 계속 지원하려면 `Package@swift-5.9.swift` 매니페스트를 하나 만들면 됩니다.
```
// swift-tools-version: 5.9


let package = Package(
    name: "MyPackage",
    products: [
        // ...
    ],
    targets: [
        .target(
            name: "FullyMigrated",
        ),
        .target(
            name: "NotQuiteReadyYet",
        )
    ]
)
```
Swift 툴체인 6.0 이상을 위한 Package.swift
```
// swift-tools-version: 6.0


let package = Package(
    name: "MyPackage",
    products: [
        // ...
    ],
    targets: [
        // 기본 도구 언어 모드 6를 사용
        .target(
            name: "FullyMigrated",
        ),
        // 여전히 언어 모드 5가 필요
        .target(
            name: "NotQuiteReadyYet",
            swiftSettings: [
                .swiftLanguageMode(.v5)
            ]
        )
    ]
)
```
대신 Swift 6 언어 모드를 사용할 수 있을 때만 사용하려는 경우(이전 모드를 계속 지원하면서) 하나의 `Package.swift`를 유지하면서 호환되는 방식으로 특정 버전을 지정할 수 있습니다:
```
// swift-tools-version: 5.9


let package = Package(
    name: "MyPackage",
    products: [
        // ...
    ],
    targets: [
        .target(
            name: "FullyMigrated",
        ),
    ],
    // 6.0 이전 swift-tools-version을 지원하기 위한 `swiftLanguageVersions`와 `.version("6")`
    swiftLanguageVersions: [.version("6"), .v5]
)
```

## Xcode로 하는 방법 (Using Xcode)
### 빌드 설정 (Build Settings)
Xcode 프로젝트나 여러 타깃들에 대해 "Swift Language Version" 빌드 설정을 "6"로 설정해서 언어 모드를 제어할 수 있습니다.
### XCConfig
또한 xconfig 파일에서 `SWIFT_VERSION`을 6으로 설정할 수 있습니다.
```
// Settings.xcconfig에서 설정
SWIFT_VERSION = 6;
```
