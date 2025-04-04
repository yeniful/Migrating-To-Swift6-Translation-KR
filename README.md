# Swift6 마이그레이션 한국어 번역
원문: https://www.swift.org/migration/documentation/migrationguide/

## 개요 (Overview)
Swift 5.5에 도입된 Swift의 동시성 시스템(Swift concurrency system)은 비동기 및 병렬 코드를 더 쉽게 작성하고 이해할 수 있도록 합니다. 
Swift 6 언어 모드를 통해, 컴파일러는 동시 프로그램들에 데이터 경쟁이 없도록 합니다.
Swift 6가 활성화되면, 이전에 선택 사항(optional)이었던 컴파일러 안전 점검이 필수(required)로 요구됩니다.

스위프트 6 언어 모드를 채택하는 것은 완전히 여러분의 통제 아래에 있습니다. 또한 각 타깃 별로 채택할 수 있습니다.
Swift에 노출된 다른 언어의 코드뿐만 아니라 이전 모드로 빌드하는 대상은 모두 Swift 6 언어 모드로 마이그레이션된 모듈과 상호 운용할 수 있습니다.

동시성 기능이 등장했을 때, 여러분은 동시성 기능을 점진적으로 채택했을 수도 있습니다. 
또는 동시성 기능을 사용하기 시작하기 위해 Swift 6 릴리즈를 기다리고 있었을 수도 있습니다. 
여러분의 프로젝트가 어느 과정에 있든지, 이 문서는 마이그레이션을 돕기 위한 개념과 실용적인 도움을 제공합니다.

이 곳에서 다음에 해당하는 글과 코드 예제를 찾을 수 있습니다:
- 스위프트의 데이터 경쟁 안전 모델(Data-race safety model)에 사용된 개념을 설명
- 마이그레이션 시작을 위한 개요
- Swift 5 프로젝트에 대한 완전한 동시성 검사를 활성화하는 방법
- 스위프트 6 언어 모드를 활성화하는 방법 시연
- 흔하게 나타나는 문제를 해결하기 위한 방법
- 점진적 도입을 위한 방법


> 중요  
Swift 6 언어 모드는 옵트인(opt-in)입니다. 기존 프로젝트에서 설정을 변경하지 않으면 해당 모드로 전환되지 않습니다.  
'컴파일러 버전'과 '언어 모드'를 구분합니다. 'Swift 6 컴파일러'는 다음의 네 가지 '언어 모드'를 지원합니다: "6", "5", "4.2", "4".

## 기여하기 (Contributing)
이 문서는 계속 업데이트 중입니다. 여러분도 소스와 전체 코드 예제를 보고, 레파지토리에 기여할 수 있습니다. 다음에 해당하는 여러분들의 기여를 기다리고 있습니다.
- 특정 코드 패턴 또는 가이드의 추가 섹션을 다루는 이슈 제출
- 기존 콘텐츠를 개선하거나 새로운 콘텐츠를 추가하기 위한 풀 리퀘스트 열기
- 작성 및 코드 예제의 명확성과 정확성에 대한 다른 사람들의 풀 리퀘스트 검토
자세한 정보는 기여 문서를 참조하세요.

## 주제 (Topics)
1. [**데이터 경쟁으로부터 안전하게 (Data Race Safety)**](https://github.com/yeniful/Migrating-To-Swift6-Translation-KR/blob/main/DataRaceSafety.md)  
데이터 경쟁 없는(data-race-free) 동시 코드를 위한 Swift 기본 개념에 대해서 알아보기
3. [**마이그레이션 전략 (Migration Strategy)**](https://github.com/yeniful/Migrating-To-Swift6-Translation-KR/blob/main/MigrationStrategy.md)  
프로젝트를 Swift 6 언어 모드로 마이그레이션하기 시작하기
4. [**완전한 동시성 검사 활성화 (Enabling Complete Concurrency Checking)**](https://github.com/yeniful/Migrating-To-Swift6-Translation-KR/blob/main/EnablingCompleteConcurrencyChecking.md)  
프로젝트에서 경고로 진단을 하도록 활성화해서 데이터 경쟁 안전 문제를 점차적으로 해결하기
5. [**동시성 확인 (Enabling The Swift 6 Language Mode)**](https://github.com/yeniful/Migrating-To-Swift6-Translation-KR/blob/main/EnablingTheSwift6LanguageMode.md)  
Swift 6 언어 모드를 활성화하여 코드에 데이터 경합이 없도록 보장하기
6. [**흔한 컴파일러 에러 (Common Compiler Errors)**](https://github.com/yeniful/Migrating-To-Swift6-Translation-KR/blob/main/CommonCompilerErrors.md)  
Swift Concurrency로 작업하면서 발생할 수 있는 흔한 문제를 파악하고, 이해하고, 해결하기
7. [**점진적인 채택 (Incremental Adoption)**](https://github.com/yeniful/Migrating-To-Swift6-Translation-KR/blob/main/IncrementalAdoption.md)  
프로젝트에 Swift 동시성 기능을 점진적으로 도입하는 방법 배우기
8. [**소스 호환성 (Source Compatibility)**](https://github.com/yeniful/Migrating-To-Swift6-Translation-KR/blob/main/SourceCompatibility.md)  
잠재적인 소스 호환성 문제에 대한 개요 보기
9. [**라이브러리 진화 (Library Evolution)**](https://github.com/yeniful/Migrating-To-Swift6-Translation-KR/blob/main/LibraryEvolution.md)  
소스 및 ABI 호환성을 유지하면서 동시성을 위해 라이브러리 API에 대해 설명

## 심도있는 Swift 동시성 (Swift Concurrency in Depth)
8. [**런타임 특성 (Runtime Behavior)**](https://github.com/yeniful/Migrating-To-Swift6-Translation-KR/blob/main/RuntimeBehavior.md)  
동시성 런타임 의미 체계가 다른 런타임들과 어떻게 다른지 알아보고, 실행 의미 체계 측면에서 비슷한 최종 결과를 얻기 위해 일반적인 패턴에 익숙해지기


## 기여 (Contribution)
<table>
  <tr height="150px">
  <td align="center">
    <a href="https://github.com/applebuddy"><img height="150px" width="150px" src="https://avatars.githubusercontent.com/u/4410021?v=4"/></a>
    <br/>
    <a href="https://github.com/applebuddy">Applebuddy</a>
  </td>
  <td align="center">
    <a href="https://github.com/kangddong"><img height="150px" width="150px" src="https://avatars.githubusercontent.com/u/50406861?v=4"/></a>
    <br/>
    <a href="https://github.com/kangddong">kangddong</a>
  </td>
  <td align="center">
    <a href="https://github.com/yeniful"><img height="150px" width="150px" src="https://avatars.githubusercontent.com/u/77262576?v=4"/></a>
    <br />
    <a href="https://github.com/yeniful">yeniful</a>
  </td>
  </tr>
</table>
