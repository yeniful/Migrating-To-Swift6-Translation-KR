# Runtime Behavior

# 런타임 동작

Swift Cuncurrency(Swift 동시성) 이 다른 친숙한 런타임들과 어떤 차이가 있는지 배우고, 실행 측면에서 비슷한 결과를 얻기 위한 공통 패턴들을 익혀봅시다.

async/await, actors, tasks 등에 강하게 초점을 두고 있는 Swift의 동시성 모델인데요. 다른 라이브러리나 동시성 런타임이 곧바로 이러한 새로운 Swift의 동시성 모델로 변환되지 않을 수 있습니다. 이번 섹션에서는, 런타임 동작 간에 주의해야할 공통 패턴과 자이점들, Swift Concurrency 모델로 마이그레이션 하는 동안 이를 다루는 방법을 보도록 하겠습니다.

