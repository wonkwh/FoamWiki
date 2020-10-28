# Swift Naming Guide

## Table of Contents
- [Swift Naming Guide](#swift-naming-guide)
  - [Table of Contents](#table-of-contents)
  - [FileNameing](#filenameing)
  - [Type(Class, Protocol, Enum) and Method Naming](#typeclass-protocol-enum-and-method-naming)
  - [산문 (Prose)](#%ec%82%b0%eb%ac%b8-prose)
  - [Class Prefixes](#class-prefixes)
  - [Delegates](#delegates)
    - [Use Type Inferred Context](#use-type-inferred-context)
  - [Generics](#generics)
    - [Language](#language)


## 파일 이름
- 실제 파일 패스와 프로젝트 폴더,파일 패스가 일치해야 한다. 
- 폴더명과 파일명 다 `snake_case`가 아닌 대문자로 시작하는 `CamelCase` 로 한다. 
  - 예) `Scenes/Hub/Login/LoginViewController` 

## Type(Class, Protocol, Enum) 과 Method 이름
- Type(class, Protocol, Enum)명은 대문자로 시작하는 `CamelCase`로 한다 
- 그외 다른것은 소문자로 시작하는 `camelCase`로 한다. 

- 생략하지 말고 정확한 단어를 사용한다. 
- type보다는 역활과 책임을 기반으로 이름을 정한다.
  - 특히 헝가리안 노테이션( IStyle ) 은 사용하지 않는다.
- sometimes compensating for weak type information
- striving for fluent usage
- factory methods 는 `make` 로 시작
- 부작용(side effects)에 대한 메소드 이름 지정하기
  - 동사 메소드는 비 수정(non-mutating) 버젼에 대해 `-ed`, `-ing`규칙을 따른다.
  - 명사 메소드는 비 수정(non-mutating) 버젼에 대해 formX 규칙을 따른다.
  - boolean 타입은 assertions처럼 읽어야 한다
  - 무엇을 해야 하는지(what something is) 설명하는 프로토콜은 명사로 읽어야 한다
  - 가능성(capability)을 설명하는 프로토콜은 `-able` 또는 `-ible`으로 끝나야 한다

- using terms that don't surprise experts or confuse beginners
- abbreviations 축약어는 사용하지 않는다. 
  - 예외적으로 사용하게 되는 약어(id, ..)는 `swiftlint.yml`파일에 추가
- 이미 많이 사용하고 있는 선례가 있는경우 그것을 사용하자
- casing acronyms and initialisms uniformly up or down
- giving the same base name to methods that share the same meaning
- avoiding overloads on return type
- choosing good parameter names that serve as documentation
- preferring to name the first parameter instead of including its name in the method name, except as mentioned under Delegates
- closure 와 tuple parameters 에 레이블 지정
- default parameters를 적극적으로 사용

## 산문 (Prose)

산문(prose)에서 메소드를 언급할때, 모호하지 않는 것이 중요하다. 메소드 이름을 참조하려면, 가능하면 가장 간단한 형식을 사용한다

1. 매개변수 없이 메소드 이름을 작성한다.  
**Example:
** Next, you need to call `addTarget`.
3. 인자 레이블을 가지는 메소드 이름을 작성한다  **Example:** Next, you need to call `addTarget(_:action:)`.
4. 인자 레이블과 타입을 가지는 메소드 전체 이름을 작성한다 **Example:** Next, you need to call `addTarget(_: Any?, action: Selector?)`.

`UIGestureRecognizer`, 을 사용하는 위의 예제에서, 1은 모호하지 않고 선호된다.

**Pro Tip:** XCode의 점프 바(jump bar)를 사용하여 인자 레이블이 있는 메소드들을 조회 할수 있다. (**Ctrl-6** key)

**Shift-Control-Option-Command-C** (all 4 modifier keys) and Xcode will kindly put the signature on your clipboard.

![Methods in Xcode jump bar](screens/xcode-jump-bar.png)


## Class Prefixes

- Swift types are automatically namespaced by the module that contains them and you should not add a class prefix such as RW. 
- If two names from different modules collide you can disambiguate by prefixing the type name with the module name. 
- However, only specify the module name when there is possibility for confusion which should be rare.

```swift
import SomeModule

let myClass = MyModule.UsefulClass()
```

## Delegates

- custom delegate methods를 생성 시에 첫번쨰 파라미터의 익명으로 할때 delegate 원본이름과 같아야 한다
  - UIKit에 많은 예제가 있으니 참조하자 
  - 아래 예를 보면 첫번째 파라미터는 `_ namePickerView` 로 naming

**Preferred**:
```swift
func namePickerView(_ namePickerView: NamePickerView, didSelectName name: String)
func namePickerViewShouldReload(_ namePickerView: NamePickerView) -> Bool
```

**Not Preferred**:
```swift
func didSelectName(namePicker: NamePickerViewController, name: String)
func namePickerShouldReload() -> Bool
```

### Use Type Inferred Context

- 컴파일러에 의해 추론된 짧고, 명확한 코드를 사용한다.
- 스타일가이드에 [Type Inference](https://github.com/balmbees/ios-best-practice/wiki/SwiftStyleGuide#type-inference)을 참조
- swiftlint rule에 custom rule로 추가하였으니 완벽하게 동작하지는 않는다. 
  - swiftlint.yml file에 `type_inferred_context rule`
**Preferred**:
```swift
let selector = #selector(viewDidLoad)
view.backgroundColor = .red
let toView = context.view(forKey: .to)
let view = UIView(frame: .zero)
```

**Not Preferred**:
```swift
let selector = #selector(ViewController.viewDidLoad)
view.backgroundColor = UIColor.red
let toView = context.view(forKey: UITransitionContextViewKey.to)
let view = UIView(frame: CGRect.zero)
```

## Generics 
- 제네릭 타입 매개변수는 대문자 카멜 표기법 이름으로 서술해야 한다. 
- 타입 이름에 의미있는 관계나 규칙이 없을때, 전통적으로 **T, U, V**처럼 하나의 대문자를 사용한다.


**Preferred**:
```swift
struct Stack<Element> { ... }
func write<Target: OutputStream>(to target: inout Target)
func swap<T>(_ a: inout T, _ b: inout T)
```

**Not Preferred**:
```swift
struct Stack<T> { ... }
func write<target: OutputStream>(to target: inout target)
func swap<Thing>(_ a: inout Thing, _ b: inout Thing)
```

### Language

 - Apple's API 와 맞는 US English 를 사용

**Preferred**:
```swift
let color = "red"
```

**Not Preferred**:
```swift
let colour = "red"
```

----
- [Swift Api Design Guidelines 한글](http://xho95.github.io/swift/language/grammar/revision/history/2020/10/08/API-Design-Guidelines.html)

