# Swift6 마이그레이션 한국어 번역
원문: https://www.swift.org/migration/documentation/migrationguide/

## 개요 (Overview)
Swift 5.5에 도입된 Swift의 동시성 시스템(Swift concurrency system)은 비동기 및 병렬 코드를 더 쉽게 작성하고 이해할 수 있도록 합니다. 
Swift 6 언어 모드를 통해 컴파일러는 동시 프로그램이 데이터 경쟁이 없음을 보장할 수 있습니다. 
Swift 6가 활성화되면, 이전에 선택 사항(optional)이었던 컴파일러 안전 점검이 필수(required)로 요구됩니다.

스위프트 6 언어 모드를 채택하는 것은 대상별로 전적으로 당신의 통제 하에 있습니다. 
Swift에 노출된 다른 언어의 코드뿐만 아니라 이전 모드로 빌드하는 대상은 모두 Swift 6 언어 모드로 마이그레이션된 모듈과 상호 운용할 수 있습니다.

도입된 동시성 기능을 점진적으로 채택했을 가능성이 있습니다. 
또는, 당신은 그것들을 사용하기 시작하기 위해 Swift 6 릴리스를 기다리고 있었을 수도 있습니다. 
이 과정에서 당신의 프로젝트가 어디에 있든 상관없이, 이 가이드는 마이그레이션을 용이하게 하기 위한 개념과 실용적인 도움을 제공합니다.

여기에서 다음과 같은 기사와 코드 예제를 찾을 수 있습니다:
- 스위프트의 데이터 레이스 안전 모델에 사용된 개념을 설명
- 마이그레이션 시작을 위한 개요
- Swift 5 프로젝트에 대한 완전한 동시성 검사를 활성화하는 방법
- 스위프트 6 언어 모드를 활성화하는 방법을 시연
- 일반적인 문제를 해결하기 위한 전략
- 점진적인 채택을 위한 기술


> 중요
Swift 6 언어 모드는 옵트인(opt-in)입니다. 기존 프로젝트에서 설정을 변경하지 않으면 해당 모드로 전환되지 않습니다.
컴파일러 버전과 언어 모드 사이에는 차이가 있습다. Swift 6 컴파일러는 네 가지 언어 모드(버전)를 지원합니다: "6", "5", "4.2", "4".

## 기여하기 (Contributing)
이 가이드는 업데이트 중입니다. 소스와 전체 코드 예제를 보고, 레파지토리에 기여하는 방법을 알 수 있습니다. 우리는 다음에 해당되는 여러분들의 기여를 기다리고 있습니다.
- 특정 코드 패턴 또는 가이드의 추가 섹션을 다루는 이슈 제출
- 기존 콘텐츠를 개선하거나 새로운 콘텐츠를 추가하기 위한 풀 리퀘스트 열기
- 작성 및 코드 예제의 명확성과 정확성에 대한 다른 사람들의 풀 요청 검토
자세한 정보는 기여 문서를 참조하세요.

## 주제 (Topics)
1. 데이터 레이스 안전 (Data Race Safety): Learn about the fundamental concepts Swift uses to enable data-race-free concurrent code.
2. 마이그레이션 전략 (Migration Strategy): Get started migrating your project to the Swift 6 language mode.
3. 마이그레이션 안전 (Migration Safety): Incrementally address data-race safety issues by enabling diagnostics as warnings in your project.
4. 동시성 확인 (Common Compiler Errors): Guarantee your code is free of data races by enabling the Swift 6 language mode.
5. 흔한 컴파일러 에러 (Common Compiler Errors): Identify, understand, and address common problems you can encounter while working with Swift concurrency.
6. 점진적인 채택 (Incremental Adoption): Learn how you can introduce Swift concurrency features into your project incrementally.
7. 소스 호환성 (Source Compatibility): See an overview of potential source compatibility issues.

## 심도있는 Swift 동시성 (Swift Concurrency in Depth)
8. 런타임 특성 (Runtime Behavior): Learn how Swift concurrency runtime semantics differ from other runtimes you may be familiar with, and familiarize yourself with common patterns to achieve similar end results in terms of execution semantics.
