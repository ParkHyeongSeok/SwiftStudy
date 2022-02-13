# Operation
하나의 작업을 나타내는 객체. 

보통 Operation을 상속한 Block Operation을 사용하거나 Operation을 서브 클래싱하고 직접 커스텀한다. Operation은 Single-shot Object다. 실행이 완료된 인스턴스는 다시 실행할 수 없다. 동일한 작업을 반복하고자 한다면 매번 새로 생성해야 한다.

Operation은 API를 통해 직접 실행할 수 있다. 하지만 다른 코드와 동시에 실행되는 것을 보장해주지 않는다. 이것은 Operation이 동기 방식으로 실행되기 때문. 비동기로 할 수 있지만 추천하지는 않는다. Operation을 직접 실행하는 것보다 Operation Queue에 추가하는 것을 추천한다.

### 장점
* Interoperation Dependency
 Operation 사이에 의존성을 추가해서 실행 순서를 제어할 수 있다.

* Cancellation
 실행취소 기능을 구현할 수 있다.

* Completion Handler
 컴플리션 핸들러를 추가하는데 필요한 API를 제공한다.

* Key & Value Observing을 통해 실행 상태를 감시하고 우선순위 설정에 필요한 API도 제공한다.

### Operation의 4가지 상태
1. Ready는 작업을 실행할 수 있는 상태
2. 작업이 시작되면 Executing 상태로 전환
3. 작업이 정상 종료 -> Finished
4. 상태작업을 취소하면  -> Cancelled 상태

Single shot이기 때문에 finish나 cancel에서 다시 ready로 돌아갈 수 없다.
Operation 각 상태를 observing할 때는 Operation 클래스가 제공하는 속성을 사용하거나 KVO를 사용한다.

### Operation Queue
Operation으로 실행할 작업을 구현하고 Queue에 추가하면 Operation Queue가 담당한다. 작업 우선순위는 우선순위나 의존성에 따라서 자동으로 결정된다. 정상적으로 작업이 완료되면, Queue에서 자동으로 제거된다. 아직 실행되지 않은 작업은 언제든지 구현할 수 있지만, 실행된 작업은 cancellation 기능을 구현했을 때만 취소할 수 있다. Custom Operation을 구현할 때는 항상 취소 기능을 고려해서 구현해야 한다.

### Operation 우선순위 : QueuePriority, QualityIfService
두 가지 속성으로 결정한다. QueuePriority, QualityIfService.
1. QueuePriority 속성은 동일한 Queue에 추가되어 있는 Operation 사이의 상대적인 우선순위. 기본.
2. QualityIfService 속성은 리소스 사용 우선순위를 결정한다. qos라고 하기도 한다. 우선순위가 높을 수록 CPU, Network, Disk를 더 오래 사용, 더 빠르게 실행된다. 기본은 Background, 보통 기본을 사용.

### OperationQueue 생성 방법
1. 메인속성을 생성하면 메인큐에서 작업을 실행하는 OperationQueue가 리턴, UI 업데이트
```swift
let queue = OperationQueue.main
```

2. 백그라운드에서 작업을 실행하는 OperationQueue가 생성된다.
```swift
let queue = OperationQueue()
```