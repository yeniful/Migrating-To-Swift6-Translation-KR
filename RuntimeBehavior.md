# Runtime Behavior

# 런타임 동작

Swift Cuncurrency(Swift 동시성) 이 다른 친숙한 런타임들과 어떤 차이가 있는지 배우고, 실행 측면에서 비슷한 결과를 얻기 위한 공통 패턴들을 익혀봅시다.

Swift Concurrency 모델은 async/await, actors, tasks 등에 강하게 초점을 두고 있습니다. 다른 라이브러리나 동시성 런타임이 곧바로 이러한 새로운 Swift의 동시성 모델로 변환되지 않을 수 있습니다. 이번 섹션에서는, 런타임 동작 간에 주의해야할 공통 패턴과 차이점들, Swift Concurrency 모델로 마이그레이션 하는 동안 이를 다루는 방법을 보도록 하겠습니다.



### Task Group을 사용해서 동시성 제한하기

처리해야할 작업이 많은 경우, "모든" 작업들을 아래와 같이 Task Group 큐에 추가할 수 있습니다.

```swift
// 비효율적일 가능성이 있습니다. -- 수천 개의 작업이 한번에 생성되어 메모리 낭비가 될 수 있음!

let lotsOfWork: [Work] = ...
await withTaskGroup(of: Something.self) { group in
  for work in lotsOfWork {
    // 만약 작업 항목이 수천 개라면, 수많은 작업들이 생성될 겁니다.
    group.addTask {
      await work.work()
    }
  }

  for await result in group {
    process(result) // 필요에 따라서 작업들에 대한 결과를 처리
  }
}
```

만약 위와 같은 방식으로 수백 개 혹은 수천 개의 작업을 처리하려고 했다면, 한번에 Task Group 큐에 추가하는 것은 비효율적일 수 있습니다. addTask 메서드 내에서 작업을 추가하는 것은 해당 작업이 중지 혹은 실행 되기 위한 작업 메모리를 할당하게 됩니다. 각각의 작업 메모리는 크지 않을지라도, 바로 실행되지 않는 수천 개의 작업들은 상당한 메모리를 차지할 수 있습니다.

이런 상황을 겪었을때 동시에 추가할 작업 수를 제한하는 방식으로 아래와 같이 수동 조절(throttling)이 가능합니다.

```swift
let lotsOfWork: [Work] = ... 
// 최대 작업 갯수를 10개 이하로 제한
let maxConcurrentWorkTasks = min(lotsOfWork.count, 10)
assert(maxConcurrentWorkTasks > 0)


await withTaskGroup(of: Something.self) { group in
    var submittedWork = 0
    for _ in 0..<maxConcurrentWorkTasks {
        group.addTask { // 'addTaskUnlessCancelled'를 사용할 수도 있음
            await lotsOfWork[submittedWork].work() 
        }
        submittedWork += 1
    }
    
    for await result in group {
        process(result) // 필요에 따라 작업 결과를 처리
    
      	// 추가로 처리할 작업이 있다면, 새로운 작업 추가
        if submittedWork < lotsOfWork.count, 
           let remainingWorkItem = lotsOfWork[submittedWork] {
            group.addTask { // 'addTaskUnlessCancelled'를 사용할 수도 있음
                await remainingWorkItem.work() 
            }  
            submittedWork += 1
        }
    }
}
```

> 💬 번역자의 추가 설명  
위와 같이 코드를 작성하면, 한번에 최대 10개의 작업만 실행되며, 새로운 작업 결과가 도착할때마다 새로운 작업이 추가로 실행됩니다. 이렇게 최대 실행가능 작업 수를 제한하는 방식으로 작업 실행 간 과도한 메모리 사용을 방지하면서, 효율적인 동시성 처리가 가능합니다.