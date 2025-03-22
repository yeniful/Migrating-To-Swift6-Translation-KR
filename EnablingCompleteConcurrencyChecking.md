# Enabling Complete Concurrency Checking
##### 프로젝트에서 경고를 알려주는 진단을 활성화하여 데이터 경쟁 방지 이슈를 단계적으로 조금씩 해결합니다.

Swift 6 언어 모드의 데이터 경쟁 방지는 점차적인 마이그레이션을 위해 설계되었습니다. 프로젝트 모듈에서 데이터 경쟁 방지 이슈를 모듈별로 해결할 수 있으며, Swift 5 언어 모드에서 컴파일러의 액터 격리(`Actor` Isolation) 및 `Sendable` 경고를 알려주는 검사를 활성화하여 Swift 6 언어 모드로 바꾸기 전 데이터 경쟁을 방지하는 과정에 대해 판단할 수 있습니다.

완전한 데이터 경쟁 방지 검사는 `-strict-concurrency` 컴파일러 플래그를 사용하여 Swift 5 언어 모드에서 경고 검사를 활성화할 수 있습니다.

## Swift 컴파일러로 하는 방법 (Using the Swift compiler)
커맨드 라인에서 swift 또는 swiftc를 직접 실행할 때, 전체 동시성 검사를 사용하기 위해서 -strict-concurrency=complete를 사용합니다. :
```
~ swift -strict-concurrency=complete main.swift
```

## Swift PM으로 하는 방법 (Using SwiftPM)
### SwiftPM 커맨드 라인에서 실행할 때 (In a SwiftPM command-line invocation)
Swift 패키지 관리자(SPM, Swift Package Manager)의 커맨트 라인에서 `-Xswiftc` 플래그를 통해 `-strict-concurrency=complete`를 사용할 수 있습니다.:
```
~ swift build -Xswiftc -strict-concurrency=complete
~ swift test -Xswiftc -strict-concurrency=complete
```
이와같은 방식은, 이어서 설명할 패키지 매니페스트(Package Manifest)에 플래그를 영구적으로 추가하기 전에 동시성 경고의 수를 파악하는 데 유용할 수 있습니다.
