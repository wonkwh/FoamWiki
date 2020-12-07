# Swift Style Guide.
### Updated for Swift 5.1
 - Raywenderlich 의 스타일가이드의 대부분을 사용한다. 
 - 필요에 따라 변경사항들이 있으며 이를 반드시 문서로 기록한다. 
 - swiftlint, swiftformat으로 최대한 warning, error, 혹은 auto formatting 되도록 처리한다.
   - 해당항목에 lint, formatting rule을 명시한다. 

## Correctness
- 반드시 경고(warnings)없이 컴파일되도록 한다. 
- 이 규칙은 문자열 원문(literals) 대신 `#selector`타입을 사용하는 것과 같이 많은 스타일을 결정하는 것을 제공한다.

## Code Organization

- extensions을 사용하여 각각의 로직을 분리한다. 
  - 특히 각각의 protocol 구현은 extension을 사용하여 분리한다. 
- 각 extension앞에는 `// MARK: -` 을 붙인다.


- UIViewController대신 `ViewController<ViewBinder>` 를 상속하여 사용하면 강제하게 된다.  (todo)
  - 파일템플릿 추가 (todo)
- UITableListController 대신 ListController<T,U> 를 상속하여 사용
  - 파일템플릿 추가 (todo)

### Protocol Conformance

In particular, when adding protocol conformance to a model, prefer adding a separate extension for the protocol methods. This keeps the related methods grouped together with the protocol and can simplify instructions to add a protocol to a class with its associated methods.

**Preferred**:
```swift
class MyViewController: UIViewController {
  // class stuff here
}

// MARK: - UITableViewDataSource
extension MyViewController: UITableViewDataSource {
  // table view data source methods
}

// MARK: - UIScrollViewDelegate
extension MyViewController: UIScrollViewDelegate {
  // scroll view delegate methods
}
```

**Not Preferred**:
```swift
class MyViewController: UIViewController, UITableViewDataSource, UIScrollViewDelegate {
  // all methods
}
```

Since the compiler does not allow you to re-declare protocol conformance in a derived class, it is not always required to replicate the extension groups of the base class. This is especially true if the derived class is a terminal class and a small number of methods are being overridden. When to preserve the extension groups is left to the discretion of the author.

For UIKit view controllers, consider grouping lifecycle, custom accessors, and IBAction in separate class extensions.

### Unused Code
- 사용하지 않는 xcode template code 나 코멘트들은 제거한다.

**Preferred**:
```swift
override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
  return Database.contacts.count
}
```

**Not Preferred**:
```swift
override func didReceiveMemoryWarning() {
  super.didReceiveMemoryWarning()
  // Dispose of any resources that can be recreated.
}

override func numberOfSections(in tableView: UITableView) -> Int {
  // #warning Incomplete implementation, return the number of sections
  return 1
}

override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
  // #warning Incomplete implementation, return the number of rows
  return Database.contacts.count
}

```
### Minimal Imports

Import only the modules a source file requires. For example, don't import `UIKit` when importing `Foundation` will suffice. Likewise, don't import `Foundation` if you must import `UIKit`.

**Preferred**:
```swift
import UIKit
var view: UIView
var deviceModels: [String]
```

**Preferred**:
```swift
import Foundation
var deviceModels: [String]
```

**Not Preferred**:
```swift
import UIKit
import Foundation
var view: UIView
var deviceModels: [String]
```

**Not Preferred**:
```swift
import UIKit
var deviceModels: [String]
```

## Spacing

* tab 문자 대신 4space indent를 사용한다 
* Xcode setting에서 다음과 같이 설정하면 된다. 

![Xcode indent settings](https://github.com/balmbees/ios-best-practice/blob/master/screens/xcode_text_setting.png)

* 메소드 와 (`if`/`else`/`switch`/`while` etc.) 등에 사용하는 brace는 항상 같은 line에 사용한다.
* Tip: Xcode의 자동 indent 기능 단축키는  **Control-I** (or **Editor ▸ Structure ▸ Re-Indent** in the menu). 
  - swiftlint rule : `statement_position`


**Preferred**:
```swift
if user.isHappy {
  // Do something
} else {
  // Do something else
}
```

**Not Preferred**:
```swift
if user.isHappy
{
  // Do something
}
else {
  // Do something else
}
```

* There should be exactly one blank line between methods to aid in visual clarity and organization. Whitespace within methods should separate functionality, but having too many sections in a method often means you should refactor into several methods.

* There should be no blank lines after an opening brace or before a closing brace.

* Colons always have no space on the left and one space on the right. Exceptions are the ternary operator `? :`, empty dictionary `[:]` and `#selector` syntax `addTarget(_:action:)`.

**Preferred**:
```swift
class TestDatabase: Database {
  var data: [String: CGFloat] = ["A": 1.2, "B": 3.2]
}
```

**Not Preferred**:
```swift
class TestDatabase : Database {
  var data :[String:CGFloat] = ["A" : 1.2, "B":3.2]
}
```

* Long lines should be wrapped at around 70 characters. A hard limit is intentionally not specified.

* Avoid trailing whitespaces at the ends of lines.

* Add a single newline character at the end of each file.

## Comments

When they are needed, use comments to explain **why** a particular piece of code does something. Comments must be kept up-to-date or deleted.

Avoid block comments inline with code, as the code should be as self-documenting as possible. _Exception: This does not apply to those comments used to generate documentation._

Avoid the use of C-style comments (`/* ... */`). Prefer the use of double- or triple-slash.

## Classes and Structures

### 어떤 것을 사용해야 하나?

  - struct 는  [value semantics](https://developer.apple.com/library/mac/documentation/Swift/Conceptual/Swift_Programming_Language/ClassesAndStructures.html#//apple_ref/doc/uid/TP40014097-CH13-XID_144).
  - 구조체는 식별성(identity)을 가지지 않은 것에 사용한다. [a, b, c]가 포함하는 배열은 [a, b, c]을 포함하는 다른 배열과 정말 똑같고 교환이 가능하다. 그것들은 똑같은 것을 나타내기 때문에, 첫번째 배열이나 두번째 배열을 사용하는 것은 중요하지 않다. 그것은 배열이 구조체인 이유이다.
  - Classes have [reference semantics](https://developer.apple.com/library/mac/documentation/Swift/Conceptual/Swift_Programming_Language/ClassesAndStructures.html#//apple_ref/doc/uid/TP40014097-CH13-XID_145).
  - 식별성(identity)을 가지거나 특정 생명주기(lifecycle)가 있는 것은 클래스를 사용한다.
  - 두 사람에 대한 객체들은 둘 다 다르기 때문에 사람을 하나의 클래스로 모델링한다. 두 사람의 이름과 생년월일이 같기 때문에, 그들이 같은 사람이라는 것을 의미하지는 않는다. 하지만, 1950년 3월 3일의 날짜는 1950년 3월 3일에 대한 다른 날짜 객체와 같기 때문에, 사람들의 생년월일은 구조체가 될것이다. 날짜는 스스로 식별성(identity)이 없다.
  - 가끔씩, 물건은 구조체가 되어야 하지만 `AnyObject`나 이미 클래스로 모델링된 것들(`NSDate`, `NSSet`)을 준수해야 한다. 
  - 이 가이드라인을 준수하도록 최대한 노력해야한다.

### Example definition

 잘 꾸며진 클래스 정의의 예:

```swift
class Circle: Shape {
  var x: Int, y: Int
  var radius: Double
  var diameter: Double {
    get {
      return radius * 2
    }
    set {
      radius = newValue / 2
    }
  }

  init(x: Int, y: Int, radius: Double) {
    self.x = x
    self.y = y
    self.radius = radius
  }

  convenience init(x: Int, y: Int, diameter: Double) {
    self.init(x: x, y: y, radius: diameter / 2)
  }

  override func area() -> Double {
    return Double.pi * radius * radius
  }
}

extension Circle: CustomStringConvertible {
  var description: String {
    return "center = \(centerString) area = \(area())"
  }
  private var centerString: String {
    return "(\(x),\(y))"
  }
}
```

위의 예제는 다음과 같은 스타일 가이드라인을 따른다.
 - 프로퍼티, 변수, 상수, 인자 선언과 콜론 다음에(이전은 아님) 공백있는 구문 타입을 지정한다.      - 예를들어, `x: Int`와 `Circle: Shape`
 - 공통의 목적와 / 맥락(context)을 공유하는 경우에, 한줄에 여러개의 변수와 구조체를 정의한다.
 - getter과 setter 정의와 프로퍼티 옵져버는 들여쓰기 한다.
 - 기본값으로 이미 되어 있는 경우, `internal`과 같은 수정자를 추가하지 않는다. 
 - 비슷하게, 메소드를 오버라이딩(overriding) 할때 접근자를 반복하지 않는다.
 - 확장에서 추가 기능을 구성한다 (예를들어, 프린팅)
 - `private` 접근 제어를 사용하여 확장에서 centerString 같은 구현 세부 정보는 숨긴다.

### Use of Self
 - 간결함을 위해, Swift는 객체의 프로퍼티에 접근하거나 메소드 호출할 필요가 없는 경우에 self 사용을 피한다.
 - 컴파일러에 의해 요구될때에 self를 사용한다
   - `@escaping` 클로저 
 - `init()` 메소드에서 프로퍼티 초기화시에는 `self`를 사용한다.
 - 위 사항은 `swiftformat` 으로 빌드시 자동으로 self키워드를 remove하거나 insert한다. 
   - [redundantself](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#redundantself) 항목 참조 
   - 옵션으로 `--self init-only` 를 사용

### Computed Properties

간결함을 위해, 계산 프로퍼티가 읽기 전용인 경우, get 클로저는 생략한다. get 클로저는 set 클로저가 제공될때에만 필요하다.

**Preferred**:
```swift
var diameter: Double {
  return radius * 2
}
```

**Not Preferred**:
```swift
var diameter: Double {
  get {
    return radius * 2
  }
}
```

### Final

- 클래스나 멤버를 `final`로 표시하는 것은 핵심 주제를 혼란시킬수 있고 필요하지도 않다. 
- 그렇지만, `final`의 사용은 가끔씩 의도를 명확하게 해준다. 
- 아래 예제에서, `Box`는 특별한 목적을 가지고 파생된 클래스의 사용자 정의는 의도되지 않는다. final로 표시하면 분명해진다.

```swift
// Turn any generic type into a reference type using this Box class.
final class Box<T> {
  let value: T
  init(_ value: T) {
    self.value = value
  }
}
```

## Function Declarations

- open brace 를 func 선언과 같이 한라인에 선언하는 `K&R` 방식을 사용한다 
- open brace를 내리는 방식 `Allman` 방식일 경우 `swiftlint:opening_brace` 에서 에러처리 
```swift
func reticulateSplines(spline: [Double]) -> Bool {
  // reticulate code goes here
}
```

- parameter가 많아 한줄에 쓰기 어려울때는 파라미터당 각각 한라인씩 
- swiftformat rule 로 다음 두 라인을 추가하면 자동으로 포맷 맞춰준다.
  - `--wraparguments before-first`
  - `--wrapcollections before-first`

```swift
func reticulateSplines(
  spline: [Double], 
  adjustmentFactor: Double,
  translateConstant: Int, comment: String
) -> Bool {
  // reticulate code goes here
}
```

- 빈 function argument에는  `(Void)` 라고 사용하지 말고 걍 `()` 사용.
- closure functon return 값이 void 일시 `()`대신 `Void`  사용.
  - `swiftlint:void_return` 워닝 처리

**Preferred**:

```swift
func updateConstraints() -> Void {
  // magic happens here
}

typealias CompletionHandler = (result) -> Void
var mentionTapHandler: ((String) -> (Void))?
```

**Not Preferred**:

```swift
func updateConstraints() -> () {
  // magic happens here
}

typealias CompletionHandler = (result) -> ()
var mentionTapHandler: ((String) -> ())?
```

## Function Calls

Mirror the style of function declarations at call sites. Calls that fit on a single line should be written as such:

```swift
let success = reticulateSplines(splines)
```

If the call site must be wrapped, put each parameter on a new line, indented one additional level:

```swift
let success = reticulateSplines(
  spline: splines,
  adjustmentFactor: 1.3,
  translateConstant: 2,
  comment: "normalize the display")
```

## Closure Expressions

- 인자 목록의 끝에 단일 클로져 표현 매개변수가 있는 경우에만 후행 클로저 문법을 사용한다. 
- 클로져 매개변수를 설명하는 이름을 준다.

**Preferred**:
```swift
UIView.animate(withDuration: 1.0) {
  self.myView.alpha = 0
}

UIView.animate(withDuration: 1.0, animations: {
  self.myView.alpha = 0
}, completion: { finished in
  self.myView.removeFromSuperview()
})
```

**Not Preferred**:
```swift
UIView.animate(withDuration: 1.0, animations: {
  self.myView.alpha = 0
})

UIView.animate(withDuration: 1.0, animations: {
  self.myView.alpha = 0
}) { f in
  self.myView.removeFromSuperview()
}
```

문맥(context)이 명확한 단일 표현 클로져에 대해, 암시적인 반환을 사용한다.:

```swift
attendeeList.sort { a, b in
  a > b
}
```
- 후행 클로저를 사용하여 연결된(chained) 메서드는 문맥(context)에 따라 명확하고 읽기 쉬워야 한다. 
- 익명으로 이름지어진 인자를 사용할때에는 간격, 줄바꿈을 결정하는 것은 작성자 자유이다

Examples:

```swift
let value = numbers.map { $0 * 2 }.filter { $0 % 3 == 0 }.index(of: 90)

let value = numbers
  .map {$0 * 2}
  .filter {$0 > 50}
  .map {$0 + 10}
```

## Types
- 가능하면 언제나 Swift의 기본 타입을 사용한다. 
- Swift는 Objective-C에 연결(bridging)을 제공하며, 필요에 따라 전체 메소드 세트를 계속 사용 할 수 있다.

**Preferred**:
```swift
let width = 120.0                                    // Double
let widthString = "\(width)"                         // String
```

**Less Preferred**:
```swift
let width = 120.0                                    // Double
let widthString = (width as NSNumber).stringValue    // String
```

**Not Preferred**:
```swift
let width: NSNumber = 120.0                          // NSNumber
let widthString: NSString = width.stringValue        // NSString
```

In drawing code, use `CGFloat` if it makes the code more succinct by avoiding too many conversions.

### Constants

- 가능하면 `let` keyword 로 
- 대부분 xcode 에서 compiler가 알려준다. 충실히 따라서 let으로 바꿔주라 하면 바꿔주자. 
- 타입 프로퍼티를 사용하여 타입의 인스턴스가 아닌 타입에 상수를 정의 할수 있다. 
  - 타입 프로퍼티를 상수로 선언하려면 단순히 `static let`을 사용한다. 
  - 타입 프로퍼티 선언된 이 방법은 인스턴스 프로퍼티와 구별하기 쉽기 때문에, 일반적으로 전역상수보다 선호한다.

Example:

**Preferred**:
```swift
enum Math {
  static let e = 2.718281828459045235360287
  static let root2 = 1.41421356237309504880168872
}

let hypotenuse = side * Math.root2

```
**Note:** 위의 예제는 case가 없는 열거형 을 사용하였다 위와 같이 사용하면 실수를 방지하고 순수하게 namespace로서 작용한다. 

**Not Preferred**:
```swift
let e = 2.718281828459045235360287  // pollutes global namespace
let root2 = 1.41421356237309504880168872

let hypotenuse = side * root2 // what is root2?
```

### Static Methods and Variable Type Properties

Static methods and type properties work similarly to global functions and global variables and should be used sparingly. They are useful when functionality is scoped to a particular type or when Objective-C interoperability is required.

### Optionals

Declare variables and function return types as optional with `?` where a `nil` value is acceptable.

Use implicitly unwrapped types declared with `!` only for instance variables that you know will be initialized later before use, such as subviews that will be set up in `viewDidLoad()`. Prefer optional binding to implicitly unwrapped optionals in most other cases.

When accessing an optional value, use optional chaining if the value is only accessed once or if there are many optionals in the chain:

```swift
textContainer?.textLabel?.setNeedsDisplay()
```

Use optional binding when it's more convenient to unwrap once and perform multiple operations:

```swift
if let textContainer = textContainer {
  // do many things with textContainer
}
```

When naming optional variables and properties, avoid naming them like `optionalString` or `maybeView` since their optional-ness is already in the type declaration.

For optional binding, shadow the original name whenever possible rather than using names like `unwrappedView` or `actualLabel`.

**Preferred**:
```swift
var subview: UIView?
var volume: Double?

// later on...
if let subview = subview, let volume = volume {
  // do something with unwrapped subview and volume
}

// another example
UIView.animate(withDuration: 2.0) { [weak self] in
  guard let self = self else { return }
  self.alpha = 1.0
}
```

**Not Preferred**:
```swift
var optionalSubview: UIView?
var volume: Double?

if let unwrappedSubview = optionalSubview {
  if let realVolume = volume {
    // do something with unwrappedSubview and realVolume
  }
}

// another example
UIView.animate(withDuration: 2.0) { [weak self] in
  guard let strongSelf = self else { return }
  strongSelf.alpha = 1.0
}
```

### Lazy Initialization

- Consider using lazy initialization for finer grained control over object lifetime. 
- This is especially true for `UIViewController` that loads views lazily. 
- You can either use a closure that is immediately called `{ }()` or call a private factory method. Example:

```swift
lazy var locationManager = makeLocationManager()

private func makeLocationManager() -> CLLocationManager {
  let manager = CLLocationManager()
  manager.desiredAccuracy = kCLLocationAccuracyBest
  manager.delegate = self
  manager.requestAlwaysAuthorization()
  return manager
}
```

**Notes:**
  - `[unowned self]` is not required here. A retain cycle is not created.
  - Location manager has a side-effect for popping up UI to ask the user for permission so fine grain control makes sense here.


### Type Inference

 - Prefer compact code and let the compiler infer the type for constants or variables of single instances. 
 - Type inference is also appropriate for small, non-empty arrays and dictionaries. 
 - `CGFloat` 나  `Int16` 같은 타입은 명시적으로 타입을 선언해주어야 할 경우가 있다 이경우에는 사용한다.

**Preferred**:
```swift
let message = "Click the button"
let currentBounds = computeViewBounds()
var names = ["Mic", "Sam", "Christine"]
let maximumWidth: CGFloat = 106.5
```

**Not Preferred**:
```swift
let message: String = "Click the button"
let currentBounds: CGRect = computeViewBounds()
var names = [String]()
```

#### Type Annotation for Empty Arrays and Dictionaries

  - 빈 arrays 나 dictionaries 를 선언시에는 inference type 선언을 사용하지 말고 type을 명시해준다.

**Preferred**:
```swift
var names: [String] = []
var lookup: [String: Int] = [:]
```

**Not Preferred**:
```swift
var names = [String]()
var lookup = [String: Int]()
```

**NOTE**: Following this guideline means picking descriptive names is even more important than before.


### Syntactic Sugar

- full generics syntax를 사용하지 말고 Syntactic Sugar를 사용하여 타입선언

**Preferred**:
```swift
var deviceModels: [String]
var employees: [Int: String]
var faxNumber: Int?
```

**Not Preferred**:
```swift
var deviceModels: Array<String>
var employees: Dictionary<Int, String>
var faxNumber: Optional<Int>
```

## Functions vs Methods

Free functions, which aren't attached to a class or type, should be used sparingly. When possible, prefer to use a method instead of a free function. This aids in readability and discoverability.

Free functions are most appropriate when they aren't associated with any particular type or instance.

**Preferred**
```swift
let sorted = items.mergeSorted()  // easily discoverable
rocket.launch()  // acts on the model
```

**Not Preferred**
```swift
let sorted = mergeSort(items)  // hard to discover
launch(&rocket)
```

**Free Function Exceptions**
```swift
let tuples = zip(a, b)  // feels natural as a free function (symmetry)
let value = max(x, y, z)  // another free function that feels natural
```

## Memory Management

Code (even non-production, tutorial demo code) should not create reference cycles. Analyze your object graph and prevent strong cycles with `weak` and `unowned` references. Alternatively, use value types (`struct`, `enum`) to prevent cycles altogether.

### Extending object lifetime

Extend object lifetime using the `[weak self]` and `guard let self = self else { return }` idiom. `[weak self]` is preferred to `[unowned self]` where it is not immediately obvious that `self` outlives the closure. Explicitly extending lifetime is preferred to optional chaining.

**Preferred**
```swift
resource.request().onComplete { [weak self] response in
  guard let self = self else {
    return
  }
  let model = self.updateModel(response)
  self.updateUI(model)
}
```

**Not Preferred**
```swift
// might crash if self is released before response returns
resource.request().onComplete { [unowned self] response in
  let model = self.updateModel(response)
  self.updateUI(model)
}
```

**Not Preferred**
```swift
// deallocate could happen between updating the model and updating UI
resource.request().onComplete { [weak self] response in
  let model = self?.updateModel(response)
  self?.updateUI(model)
}
```

## Access Control
 - `fileprivate` 보다 `private` 을 사용.
 - Only explicitly use `open`, `public`, and `internal` when you require a full access control specification.

Use access control as the leading property specifier. The only things that should come before access control are the `static` specifier or attributes such as `@IBAction`, `@IBOutlet` and `@discardableResult`.

**Preferred**:
```swift
private let message = "Great Scott!"

class TimeMachine {  
  private dynamic lazy var fluxCapacitor = FluxCapacitor()
}
```

**Not Preferred**:
```swift
fileprivate let message = "Great Scott!"

class TimeMachine {  
  lazy dynamic private var fluxCapacitor = FluxCapacitor()
}
```

## Control Flow

-  `for-in` style 를 사용
-  `while-condition-increment` style. 의 루프는 되도록 지양.

**Preferred**:
```swift
for _ in 0..<3 {
  print("Hello three times")
}

for (index, person) in attendeeList.enumerated() {
  print("\(person) is at position #\(index)")
}

for index in stride(from: 0, to: items.count, by: 2) {
  print(index)
}

for index in (0...3).reversed() {
  print(index)
}
```

**Not Preferred**:
```swift
var i = 0
while i < 3 {
  print("Hello three times")
  i += 1
}


var i = 0
while i < attendeeList.count {
  let person = attendeeList[i]
  print("\(person) is at position #\(i)")
  i += 1
}
```

### Ternary Operator

- The Ternary operator, `?:` ,는 하나의 조건일떄만 사용.
- 여러 조건일 떄는 `if`를 사용하거나 Extract Refactoring을 사용하여 조건을 instance variables 로 분리시킨다.
- In general, the best use of the ternary operator is during assignment of a variable and deciding which value to use.

**Preferred**:

```swift
let value = 5
result = value != 0 ? x : y

let isHorizontal = true
result = isHorizontal ? x : y
```

**Not Preferred**:

```swift
result = a > b ? x = c > d ? c : d : y
```

## Golden Path

When coding with conditionals, the left-hand margin of the code should be the "golden" or "happy" path. That is, don't nest `if` statements. Multiple return statements are OK. The `guard` statement is built for this.

**Preferred**:
```swift
func computeFFT(context: Context?, inputData: InputData?) throws -> Frequencies {

  guard let context = context else {
    throw FFTError.noContext
  }
  guard let inputData = inputData else {
    throw FFTError.noInputData
  }

  // use context and input to compute the frequencies
  return frequencies
}
```

**Not Preferred**:
```swift
func computeFFT(context: Context?, inputData: InputData?) throws -> Frequencies {

  if let context = context {
    if let inputData = inputData {
      // use context and input to compute the frequencies

      return frequencies
    } else {
      throw FFTError.noInputData
    }
  } else {
    throw FFTError.noContext
  }
}
```

When multiple optionals are unwrapped either with `guard` or `if let`, minimize nesting by using the compound version when possible. In the compound version, place the `guard` on its own line, then indent each condition on its own line. The `else` clause is indented to match the conditions and the code is indented one additional level, as shown below. Example:

**Preferred**:
```swift
guard let number1 = number1,
    let number2 = number2,
    let number3 = number3 else {
    fatalError("impossible")
}
// do something with numbers
```

**Not Preferred**:
```swift
if let number1 = number1 {
  if let number2 = number2 {
    if let number3 = number3 {
      // do something with numbers
    } else {
      fatalError("impossible")
    }
  } else {
    fatalError("impossible")
  }
} else {
  fatalError("impossible")
}
```

### Failing Guards

Guard statements are required to exit in some way. Generally, this should be simple one line statement such as `return`, `throw`, `break`, `continue`, and `fatalError()`. Large code blocks should be avoided. If cleanup code is required for multiple exit points, consider using a `defer` block to avoid cleanup code duplication.

## Semicolons
- semicolon은 사용하지 않는다.

**Preferred**:
```swift
let swift = "not a scripting language"
```

**Not Preferred**:
```swift
let swift = "not a scripting language";
```

**NOTE**: Swift is very different from JavaScript, where omitting semicolons is [generally considered unsafe](http://stackoverflow.com/questions/444080/do-you-recommend-using-semicolons-after-every-statement-in-javascript)

## Parentheses

Parentheses around conditionals are not required and should be omitted.

**Preferred**:
```swift
if name == "Hello" {
  print("World")
}
```

**Not Preferred**:
```swift
if (name == "Hello") {
  print("World")
}
```

In larger expressions, optional parentheses can sometimes make code read more clearly.

**Preferred**:
```swift
let playerMark = (player == current ? "X" : "O")
```

## Multi-line String Literals

When building a long string literal, you're encouraged to use the multi-line string literal syntax. Open the literal on the same line as the assignment but do not include text on that line. Indent the text block one additional level.

**Preferred**:

```swift
let message = """
  You cannot charge the flux \
  capacitor with a 9V battery.
  You must use a super-charger \
  which costs 10 credits. You currently \
  have \(credits) credits available.
  """
```

**Not Preferred**:

```swift
let message = """You cannot charge the flux \
  capacitor with a 9V battery.
  You must use a super-charger \
  which costs 10 credits. You currently \
  have \(credits) credits available.
  """
```

**Not Preferred**:

```swift
let message = "You cannot charge the flux " +
  "capacitor with a 9V battery.\n" +
  "You must use a super-charger " +
  "which costs 10 credits. You currently " +
  "have \(credits) credits available."
```

## No Emoji

Do not use emoji in your projects. For those readers who actually type in their code, it's an unnecessary source of friction. While it may be cute, it doesn't add to the learning and it interrupts the coding flow for these readers.

## Organization and Bundle Identifier
 
Where an Xcode project is involved, the organization should be set to `Vingle` and the Bundle Identifier set to `com.Vingle.ProductName` where `ProductName` is the name of the product project.

![Xcode Project settings](https://github.com/balmbees/ios-best-practice/blob/master/screens/project_settings.png)

## Copyright Statement
다음 copyright statement가 각각의 파일 top comment에 포한되어야 한다. 

```swift
/// ...
/// Copyright © 2019 Vingle. All rights reserved.
/// ... 
```

## References

* [The Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)
* [Swift Api Design Guidelines 한글](http://xho95.github.io/swift/language/grammar/revision/history/2020/10/08/API-Design-Guidelines.html)
* [The Swift Programming Language](https://developer.apple.com/library/prerelease/ios/documentation/swift/conceptual/swift_programming_language/index.html)
* [Using Swift with Cocoa and Objective-C](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/BuildingCocoaApps/index.html)
* [Swift Standard Library Reference](https://developer.apple.com/library/prerelease/ios/documentation/General/Reference/SwiftStandardLibraryReference/index.html)

----
- tags: #codingConvention, #bestPractice, #styleguide 