# GCD
Grand Central Dispatch, iOS Concurrency Programming의 근간을 이루는 기술.

GCD는 Thread를 자동으로 생성하고 효율적으로 관리한다. 특히 Thread Pool을 통해서 Thread를 재사용하기 때문에 시스템 리소스를 상대적으로 적게 사용하면서 빠른 성능을 유지한다. 그리고 Dispatch framwork를 통해 직관적이고 단순한 API를 제공하며 모든 Apple 플랫폼에서 동일한 API를 사용하는 장점이 있다.

## Dispatch Queue
GCD의 핵심 객체. Dispatch Queue는 FIFO 방식으로 task를 관리하면서 시스템 상태와 실행 방식에 따라 순서를 제어한다. 이런 모델을 Work Queue Programming Model이라고 하는데, Dispatch Queue에게는 thread에 관한 모든 것을 맡겨두고 task를 구현하는데 집중할 수 있다는 장점이 있다.

### Dispatch Queue에 task를 추가하는 방법
1. 블럭 형태 : {}
2. DispatchWorkItem으로 캡슐화해서 추가 : DispatchWorkItem()

### Dispatch Queue가 task를 실행하는 방법
1. Concurrent Queue
 추가된 작업을 동시에 실행한다. 동시 실행 작업 수는 시스템 상태에 따라 자동으로 결정. 이 큐는 Concurrent 옵션을 통해서 직접 생성하거나 system이 제공하는 global Dispatch Queue를 사용할 수 있다.

2. Serial Queue
 큐에 추가된 순서대로 하나씩 실행. 동시에 실행되지 않기 때문에 큐 기반 동기화에 자주 사용된다. 새로운 Dispatch Queue를 생성하면 Serial로 생성된다. Main Queue는 Main Thread에서 동작하는 특별한 Serial Dispatch Queue다. Main Queue는 앱 시작 시점에 자동으로 생성되고 Dispatch Queue에 선언되어 있는 속성으로 언제든 접근할 수 있다.

### Dispatch Queue의 우선순위, qos
Dispatch Queue의 우선순위는 qos를 통해 설정한다. qos는 리소스 사용 우선순위를 4가지로 설정한다. 우선순위가 높을 수록 CPU Network Disk를 더 오랫동안 사용, 더 빠르게 실행된다.

1. QualityOfService.userInteractive
2. QualityOfService.userInitiated
3. QualityOfService.utility
4. QualityOfService.background

### DispathQueue 구현하기

1. 시스템에서 제공하는 API를 사용해서 Background와 main queue를 사용할 수 있다.
 ```swift
 DispathQueue.global().async {
     // Operation과 달리 Dispatch Queue는 autoreleasPool을 자동으로 관리하기 때문에 추가할 필요 없다.
     var total = 0
     // ...
     DispatchQueue.main.async {
         self.valueLabel.text = "\(total)"
     }
 }
 ```
2. 직접 구현해 사용할 수 있다.
 ```swift
 // 별도의 attributes를 지정하지 않으면 serial queue를 생성한다.
 let serialWorkQueue = DispatchQueue(label: "SerialWorkQueue")
 let concurrentWorkQueue = DispatchQueue(label: "ConcurrentWorkQueue", attributes: .concurrent)
 ```

### DispatchQueue에 작업 추가하기
작업을 추가하는 방법은 2가지, sync, async가 있다. 작업을 전달할 때는 클로저나 DispatchWorkItem으로 전달한다.

1. sync
 동기방식. 작업을 추가한 다음 바로 리턴하지 않고 완료될 때까지 기다린다. 완료되면 이어지는 코드를 실행. Lock과 유사한 기능을 구현할 때 사용. **sync를 메인에서 사용하면 Crash가 난다. 주의!!

2. async
 비동기방식, 작업을 추가하고 바로 리턴. 이어지는 코드가 바로 실행. 어떤 큐에서 호출하더라도 연관된 쓰레드를 블로킹하지 않는다. 대부분 aysnc

Notice: 이 메서드는 DispatchQueue에 '작업을 전달하는 메소드'지 실행하는 메소드가 아니다. 작업을 실제로 실행하는 것은 DispatchQueue다. 두 메서드가 DispatchQueue의 동작 방식에 영향을 주는게 아니다. 그래서...

* Serial Queue에 작업을 추가할 때는 어떤 메서드를 사용하는지와 관계 없이 항상 추가된 순서대로 실행된다.
* Concurrent Queue에 sync 메서드를 사용하면 작업이 추가된 순서대로 하나씩 실행되는 것처럼 보이지만 ConcurrentQueue가 작업을 동시에 실행할 수 있다는게 달라지지는 않는다. 다른 곳에서 작업을 추가하면 현재 실행 중인 sync작업과 동시에 실행된다.

### 실행 시간
Dispatch Framwork에서 사용하는 '시간'은 'DispatchTime' 구조체로 생성한다. 
이 구조체의 now()는 지금.

```swift
var start = DispatchTime.now()
// ...
var end = DispatchTime.now()
print(Double(end.uptimeNanoseconds = start.uptimeNanoseconds)) // 주석코드 실행 시간
```

### 반복 하기
반복 순서가 중요하지 않다면 for문의 시간을 더 줄일 수 있다.
DispatchQueue.concurrentPerform은 
첫 번째 파라미터로 전달하는 인덱스 범위에서 시스템 자원이 허락하는 만큼 병렬로 실행한다. 
두번째는 실행할 코드를 넣는데, 여기서는 0~20사이의 값이 병렬로 들어온다.
```swift
DispatchQueue.concurrentPerform(iterations: 20) { value in
    // code
}
```

## DispatchWorkItem
task를 캡슐화하는 클래스. 캡슐화된 task를 직접 실행할 수 있지만 보통 DispatchQueue나 DispatchSource에 추가하는 형식으로 구현한다. 

1. 생성 : 기본값이 있는 Block으로 간단히 생성 가능
2. 작업 추가 : 동일하게 sync, async 메서드를 사용
3. 취소 기능 : DispatchQueue는 취소 기능을 제공하지 않기 때문에 DispatchworkItem에 대한 참조를 사용해 cancel 메서드를 사용. 취소여부는 isCancel. 물론, 취소 기능 API를 제공하지만 Operation에 비해 효율적이지 않다. 취소기능이 필요하다면 Operation과 OperationQueue를 이용해서 구현하자. 