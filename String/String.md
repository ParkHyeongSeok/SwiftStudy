# String

* String과 Character 기본

```swift
// 문자열
let s = "String"
let c = "C"

// 문자로 처리하고 싶다면?
let d: Character = "C"
let d: Character = "CC" // Error! 하나만 가능하기 때문에 두 개("CC")는 에러

// 빈 문자
let e: String = ""
let e = String()
let e: Character = "" // Error! 큰 따옴표 사이에 아무것도 없으면 항상 문자열로 추론
let e: String = " " // 빈 문자가 아니다. 문자열은 길이(count)를 기준으로 빈 문자열을 판단
```

* Swift의 String Type, String vs NSString
swift에서는 두 개의 문자열 자료형을 사용
1. String: 구조체, 값 형식 (Swift 문자열)
2. NSString: 클래스, 참조 형식 (Foundation 문자열)

이 두 자료형은 Toll-Free Birdged(브릿징)이 가능한 타입으로 타입캐스팅으로 호환되는 자료형이다. 이 브릿징은 일종의 공짜 변환인데, 이는 타입 자체가 자동으로 변환되는 것이 아니라 타입과 관련을 맺는 API들이 자동으로 호환 대상 타입을 위한 타입으로 변환된다는 뜻이다.

하지만 유니코드를 처리하는 방식이 다르기 때문에 조심해서 사용해야 한다. Swift의 String 타입은 '인코딩 독립적인 유니코드 문자'들로 구성되고 이러한 문자들에 접근할 수 있는 다양한 방법을 지원한다. 반면 NSString 타입은 유니코드 호환 텍스트를 인코딩해 일련의 UTF-16 코드 유닛들로 표현된다. 이런 차이 때문에 String은 정수 범위가 아닌 자체적인 Index를 사용해 Character에 접근한다.

```swift
// 문자열 리터럴로 초기화할 시 String 타입의 영향을 받음.
var string = "str" 

// swift 문자열을 NSString에 저장하는 것은 가능
var nsString: NSString = "str"
var swiftString: String = nsString // 하지만 반대는 불가능
let swiftString: String = nsString as String // 타입캐스팅으로 저장 가능
```

* Swift의 문자 인코딩 방식, Unicode
위에서 언급했던 것처럼, swift는 문자열을 저장할 때 Unicode의 독립적인 문자로 저장하고 이에 관련한 기능을 지원한다.
```swift
let str = "Swift String"

// 하지만 아래와 같이 간단히 인코딩 할 수 있다.
str.utf8
str.utf16
```
하지만 대부분의 경우 해당 인코딩으로 처리할 필요가 없다.

Unicode 문자열은 Swift에서 간단하게 사용 가능하다.

```swift
// 직접 입력해서 사용할 수 있고,
var thumbUp = "⭐️"

// 유니코드 문자에 할당되어 있는 코드 값(유니코드 스칼라값)으로 입력하는 것도 가능하다.
thumbUp = "\u{2B50}"
```
