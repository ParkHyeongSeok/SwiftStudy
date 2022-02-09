# String 이론

### String과 Character 기본

* 문자열은 문자들의 집합
* Swift 문자열은 String Type
* Swift의 문자는 Character Type
* 즉, String은 Character의 값의 집합이다.
* 유니코드에 호환이 되지만 유니코드와 독립적으로 문자를 관리한다. 

```swift
// 문자열
let s = "String"
let c = "C"

// 문자로 처리하고 싶다면?
let d: Character = "C"
let d: Character = "CC" // Error! 하나만 가능

// 빈 문자
let e: String = ""
let e = String()
let e: Character = "" // Error! 큰 따옴표 사이에 아무것도 없으면 항상 문자열로 추론
let e: String = " " // 빈 문자가 아니다. 문자열은 길이(count)를 기준으로 빈 문자열을 판단
```

### Swift와 Unicode

* Swift의 String, Character Type은 유니코드 스칼라 값(unique 21-bit number)에서 만들어진다. 따라서 Swift의 String과 Character는 기본적으로 유니코드를 완벽하게 준수한다. 

* 하지만 유니코드와 독립적으로 문자를 관리한다. Swift Character Type의 모든 인스턴스는 하나의 extended grapheme cluster(가장 작은 의미 단위)를 표현한다. Character는 하나 또는 하나 이상의 유니코드 스칼라로 이루어진 sequence이고, 결합했을 시 human-readable한 문자를 만든다. 이렇게 Swift의 String 타입은 '인코딩 독립적인 유니코드 문자'들로 구성되고 이러한 문자들에 접근할 수 있는 다양한 방법(String.Index)을 지원한다.

```swift
let precomposed: Character = "\u{D55C}"                  // 한
let decomposed: Character = "\u{1112}\u{1161}\u{11AB}"   // ᄒ, ᅡ, ᆫ
// precomposed is 한, decomposed is 한
```

* Swift에 String의 count이 NSString의 length 속성과는 항상 같지 않다. NSString은 UTF-16 기반이기 때문에 extended grapheme cluster를 지원하지 못하기 때문이다. 이 기능의 지원 문제 때문에 count나 == 과 같은 연산이 언제나 눈에 보이는대로 작동하지 않을 수 있다.

* 세 가지 문자열 인코딩에 접근할 수 있는 프로퍼티를 제공한다. (UTF-8, UTF-16, Unicode Scalar) 유니코드 스칼라 값은 UTF-32인데, 실제로 21 bits만 필요해서, 21bits라고 한다. 반면 NSString은 UTF-16 기반이다.

```swift
let str = "Swift String"

// 아래와 같이 간단히 인코딩 할 수 있다.
// 하지만 대부분의 경우 해당 인코딩으로 처리할 필요가 없다.
str.utf8
str.utf16
```

* Unicode 문자열은 Swift에서 간단하게 사용 가능하다.

```swift
// 직접 입력해서 사용할 수 있고,
var thumbUp = "⭐️"

// 유니코드 문자에 할당되어 있는 코드 값(유니코드 스칼라값)으로 입력하는 것도 가능하다.
thumbUp = "\u{2B50}"
```

### Swift의 String Type, String vs NSString
* swift에서는 두 개의 문자열 자료형을 사용
1. String: 구조체, 값 형식 (Swift 문자열)
2. NSString: 클래스, 참조 형식 (Foundation 문자열)

이 두 자료형은 Toll-Free Birdged(브릿징)이 가능한 타입으로 타입캐스팅으로 호환되는 자료형이다. 이 브릿징은 일종의 공짜 변환인데, 이는 타입 자체가 자동으로 변환되는 것이 아니라 타입과 관련을 맺는 API들이 자동으로 호환 대상 타입을 위한 타입으로 변환된다는 뜻이다. 하지만 유니코드를 처리하는 방식이 다르기 때문에 조심해서 사용해야 한다. 위에서 언급했던 것처럼 String은 extended grapheme cluster를 사용하는 반면 NSString 타입은 유니코드 호환 텍스트를 인코딩해 일련의 UTF-16 코드 유닛들로 표현된다. 

```swift
// 문자열 리터럴로 초기화할 시 String 타입의 영향을 받음.
var string = "str" 

// swift 문자열을 NSString에 저장하는 것은 가능
var nsString: NSString = "str"
var swiftString: String = nsString // 하지만 반대는 불가능
let swiftString: String = nsString as String // 타입캐스팅으로 저장 가능
```

### SubString Type

* 하나의 문자열에서 특정 범위에 있는 문자열을 SubString이라고 한다. 막상 사용하면 String.SubSequence로 표시되는데, typealias로 이름만 붙인거니 두 개는 같은 타입이다. (String.Element가 Character인 것과 같음)

* SubString을 왜 사용할까?
 문자열을 처리할 때 메모리를 절약하기 위해서다. Swift3까지는 SubString을 사용하지 않고 그냥 String의 일부를 가지고 새로운 메모리에 등록했다. 하지만 Swift4부터 Copy-on-Write Optimization(최적화)를 지원하며 SubString이라는 것이 생겨났다.  SubString은 새로운 메모리 공간을 사용하지 않고 원본 메모리를 공유해 그 중에 일부 문자만 가리킨다. 이전과 달리 메모리 공간을 절약하며 빠르게 처리하게 된다. 다만 쓰기 작업이 일어날 때 Swift에서는 새로운 메모리를 생성하고 원본 메모리와는 독립적인 공간에서 작업이 수행된다.(즉 insert같은 원본 변경 메서드를 사용해도 원본이 변경되지 않는다) 이렇게 불필요한 복사와 메모리 생성을 최대한 줄여서 코드의 실행 성능을 최적화한 것이다.
 
* Copy-on-Write Optimization(최적화)를 사용하지 않고 새로운 메모리에 바로 생성하고 싶다면?

```swift
// 생성자로 바로 생성하면 된다.
let newStr = String(str.prefix(1))
```
