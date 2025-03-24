# 전체 동시성 검사 활성화 Enabling Complete Concurrency Checking
##### 프로젝트에서 경고를 알려주는 진단을 활성화하여 데이터 경쟁 방지 이슈를 단계적으로 조금씩 해결합니다.

Swift 6 언어 모드의 데이터 경쟁 방지는 점차적인 마이그레이션을 위해 설계되었습니다. 프로젝트 모듈에서 데이터 경쟁 방지 이슈를 모듈별로 해결할 수 있으며, Swift 5 언어 모드에서 컴파일러의 액터 격리(`Actor` Isolation) 및 `Sendable` 경고를 알려주는 검사를 활성화하여 Swift 6 언어 모드로 바꾸기 전 데이터 경쟁을 방지하는 과정에 대해 판단할 수 있습니다.

완전한 데이터 경쟁 방지 검사는 `-strict-concurrency` 컴파일러 플래그를 사용하여 Swift 5 언어 모드에서 경고 검사를 활성화할 수 있습니다.

## Swift 컴파일러로 하는 방법 (Using the Swift compiler)
커맨드 라인에서 swift 또는 swiftc를 직접 실행할 때, 전체 동시성 검사를 사용하기 위해서 -strict-concurrency=complete를 사용합니다. :
```
~ swift -strict-concurrency=complete main.swift
```

## Swift PM으로 하는 방법 (Using SwiftPM)
### SwiftPM 커맨드 라인에서 하는 법 (In a SwiftPM command-line invocation)
Swift 패키지 관리자(SPM, Swift Package Manager)의 커맨트 라인에서 `-Xswiftc` 플래그를 통해 `-strict-concurrency=complete`를 사용할 수 있습니다.:
```
~ swift build -Xswiftc -strict-concurrency=complete
~ swift test -Xswiftc -strict-concurrency=complete
```
이와같은 방식은, 이어서 설명할 패키지 매니페스트(Package Manifest)에 플래그를 영구적으로 추가하기 전에 동시성 경고의 수를 파악하는 데 유용할 수 있습니다.

### SwiftPM 패키지 매니페스트에서 하는 법 (In a SwiftPM package manifest)
Swift 5.9 또는 Swift 5.10 도구로 Swift 패키지 타깃에 대한 전체 동시성 검사를 활성화하려면, 해당 타깃의 Swift 설정에서 [SwiftSetting.enableExperimentalFeature](https://developer.apple.com/documentation/packagedescription/swiftsetting/enableexperimentalfeature(_:_:))를 사용합니다.
```
.target(
  name: "MyTarget",
  swiftSettings: [
    .enableExperimentalFeature("StrictConcurrency")
  ]
)
```
Swift 6.0 도구나 그 이후의 최신 버전 도구를 사용한다면, Swift 6 이전 언어 모드 타깃에 대해 Swift 설정에서 [SwiftSetting.enableUpcomingFeature](https://developer.apple.com/documentation/packagedescription/swiftsetting/enableupcomingfeature(_:_:))를 사용합니다.
```
.target(
  name: "MyTarget",
  swiftSettings: [
    .enableUpcomingFeature("StrictConcurrency")
  ]
)
```
Swift 6 언어 모드를 설정한 타깃들은 전체 검사가 무조건 활성화되기 때문에 설정을 따로 변경하지 않아도 됩니다.

## Xcode로 하는 방법 (Using Xcode)
Xcode 프로젝트에서 전체 동시성 검사를 활성화하려면 Xcode 빌드 설정에서 "엄격한 동시성 검사(Strict Concurrency Checking)" 설정을 "전체(Complete)"로 설정합니다. 혹은 xcconfig 파일에서 `SWIFT_STRICT_CONCURRENCY`를 complete로 설정하는 방법도 가능합니다.
```
// In a Settings.xcconfig
SWIFT_STRICT_CONCURRENCY = complete;
```
