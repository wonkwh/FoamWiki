# What Is a Parser?: Part 2
## Introduction

The manual, hand-rolled coordinate parsing function we made [last time](https://www.pointfree.co/episodes/ep56-what-is-a-parser-part-1) is already pretty complicated, and capturing every edge case will make it more complicated still.

But that’s not even the worst part. The worst part is that this parse function is a one-off, ad hoc solution to parsing the very specific format of latitude/longitude coordinates. There is nothing inside the body that is reusable outside of this example.

We’re definitely seeing how tricky and subtle parsing can be, and it’s easy to end up with a complex function that would need to be studied very carefully to understand what it’s doing, and even then it would be easy to overlook the potential bugs that lurk within.

Let’s back up and formally state what the problem of parsing is and analyze it from the perspective of functional programming and functions.

## What is a parser?
- parser란 문자열을 어떤 다른타입으로 변경하는 함수
- 그러므로 다음과 같이  간단한 type alias로 정의 할수 있다.
```swift
    typealias Parser<A> = (String) -> A
```

It’s going to be handy to give this a proper type so that we can extend it with methods, so let’s wrap the function in a struct:
- 함수를 가진 구조체로 확장하자
```swift
    struct Parser<A> {
      let run: (String) -> A
    }
```
However, the shape of `Parser` isn’t quite correct right now because parsing isn’t always guaranteed to succeed and produce a value. There will always be some blobs of data that are malformed and we just can’t parse it. So, let’s introduce some failability:
- parsing이 항상 성공한다고 할수 없으므로 결과값은 optional로 
```swift
    struct Parser<A> {
      let run: (String) -> A?
    }
```

- iOS application의 deep-link 라우팅을 parser로서 생각해보자
This simple type comes up quite a bit in application development. We often have unstructured data, like a bare string, and want to transform it into something structured. For example, deep-link routing in an iOS application could be thought of as a parser:

```swift
    let router = Parser<Route> { str in
      fatalError()
    }
```

`Route` represents a tree of routes that you support in your application, for example:
```swift
    enum Route {
      case home
      case profile
      case episodes
      case episode(id: Int)
    }
```
If we could construct that `router` value, then whenever a deep-link request comes through our app delegate, we can simply run the parser on it, get our first-class `Route` value of out it, `switch` on that value, and then do some basic logic to bring up the correct screen in our app:

```swift
    router.run("/")
    router.run("/episodes/42")
```    

And once you have this apparatus in place, you could route incoming deep links using a switch statement.
```swift
    switch router.run("/") {
    case .none:
    case .some(.home):
    case .some(.profile):
    case .some(.episodes):
    case let .some(.episode(id)):
    }
```
We could also think of command line tools as parsers:
```swift
    let cli = Parser<EnumPropertyGenerator> { _ in
      fatalError()
    }
```
`EnumPropertyGenerator` represents all of the ways you can use the tool.
```swift
    enum EnumPropertyGenerator {
      case help
      case version
      case invoke(urls: [URL], dryRun: Bool)
    }
```
And just like with our router, should we construct one of these values we can `switch` on it to handle the input.
```swift
    cli.run("generate-enum-properties --version")
    
    
    cli.run("generate-enum-properties --help")
    
    
    cli.run("generate-enum-properties --dry-run path/to/file")
    
    
    switch cli.run("generate-enum-properties --dry-run path/to/file") {
    case .none:
    case .some(.help):
    case .some(.version):
    case let .some(.invoke(urls, dryRun)):
    }
```
## Parsing as a multi-step process

So although we can see how useful it would be to have these parsers, the type we have defined so far is still not quite right. Creating a parser like this means we must definitely answer the question of how to turn a string into an `A` value, and so that is not conducive to being able to create small parsers that do a little bit of parsing, and piecing them together into parsers that parser a big thing.

A way to do this is to think of parsing as a multi-step process, and the `Parser` type should only describe one step in that process. This means changing the function signature to return a result alongside the rest of the string we want to parse.

We want to have this functionality for our `Parser` type. A step in parsing consists of consuming some subset of the input string, and then returning what is left of the string to parse along with the result of the parsing.
```swift
    struct Parser<A> {
      let run: (String) -> (match: A?, rest: String)
    }
```
As a very simple example, let’s build an integer parser. It will parse an integer off the beginning of a string, if possible, and return it along with the remainder of the string. Let’s start with the basic set up of the parser:
- 아주 간단한 예로 정수 파서를 만들어 보자 
- 가능하다면 문자열의 첫번째 문자를 정수로, 나머지 문자를 리턴한다.
```swift
    let int = Parser<Int> { str in
      
    }
```
Somehow in here we have to return a new string that represents the input string after we have consumed an integer off the beginning, and the result of having converted that consumed prefix into a string. Let’s start by getting a prefix of all the numeric characters of the given string:
```swift
    let int = Parser<Int> { str in
      let prefix = str.prefix(while: { $0.isNumber })
```
We can then try to convert this prefix to an integer using the `Int` initializer.
```swift
    let int = Parser<Int> { str in
      let prefix = str.prefix(while: { $0.isNumber })
      let int = Int(prefix)
```
This is a failable operation, though, and if the prefix of the string is not numeric, `int` will be `nil`, so we want to guard against that case and bail out by returning a `nil` match and the original string, since we don’t want to consume any of the string if parsing failed.
```swift
    let int = Parser<Int> { str in
      let prefix = str.prefix(while: { $0.isNumber })
      guard let int = Int(prefix) else { 
		  return (nil, str) 
	  }
```
We’re getting close, we have the integer we want to return but we don’t have the “rest” of the string that we’ve consumed. Do do so, we can use its end index to return the characters we didn’t consume:
```swift
    let int = Parser<Int> { str in
      let prefix = str.prefix(while: { $0.isNumber })
      guard let int = Int(prefix) else { 
		  return (nil, str) 
	  }
      let rest = String(str[prefix.endIndex...])
      return (int, rest)
    }
```
And this is our first parser! It’s a little complicated and we’re working with string indices, which can be tricky, but it should encapsulate all of what it means to parse an integer off the beginning of a string.

Let’s take it for a spin:
```swift
    int.run("42")
    
    
    int.run("42 Hello World")
    
    
    int.run("Hello World")
```    

And just like that we have already created a parser. It may not seem like much, but this little unit will be the basis of a lot of amazing things to come.

## Optimized parsing with inout

But before we continue developing the parser type, let’s address a few quick optimization opportunities. Right now a parser’s `run` function takes a string, and returns a whole new string that represents what is left of the input after some subset of it has been consumed in parsing. We could make this more efficient by instead mutating the input to represent that consumption, and in fact material covered in an old Point-Free episode can lead us to the exact refactor we need to do.
- 더 진행하기 전에 최적화를 진행해 보자 
In just [our second episode](https://www.pointfree.co/episodes/ep2-side-effects) on Point-Free, we showed that there’s an equivalence between functions that take a value in `A` and return a value in `A` and functions that simply take an `inout A` and return `Void`:
- episode 2의 side effect편에서 확인했듯이 A 타입을 인자로 받아 A타입을 리턴하는 함수는 `inout A` 를 인자로 받아 `Void` 타입을 리턴하는 것과 같다
You can easily build a generic transformation that moves between these two styles of functions. And more generally, if you have a type in a function signature that appears once on each side of a function arrow, you can convert that into a function that takes an `inout` input and no longer returns that type.

And this is exactly what we want to do with our `run` function. It has a `String` type on both sides of the function, and so we will convert that to an `inout`:
```swift
    struct Parser<A> {
    
      let run: (inout String) -> A?
    }
```
And now we have to fix our `int` parser. Instead of returning a new string and result, we will in-place mutate the string to consume the prefix.
```swift
    let int = Parser<Int> { str in
      let prefix = str.prefix(while: { $0.isNumber })
      guard let match = Int(prefix) else { return nil }
      str.removeFirst(prefix.count)
      return match
    }
```
Now our `run`s don’t work because they expect an `inout` argument, so for now let’s recreate the non-mutating version of `run` so that we can continue using it this way:
```swift
    extension Parser {
      func run(_ str: String) -> (match: A?, rest: String) {
        var str = str
        let match = self.run(&str)
        return (match, str)
      }
    }
```
And everything continues working just as it did before.

## Optimized parsing with substring
- 중요한 최적화가 남아있다. 
- `run ` 메소드를 실행할 때마다 새로운 문자열 전체를 생성한다. 만약 매우 큰 문자열을 입력받는다면 요건 매우 비효율적
- 
There’s one more optimization we can make, and it’s a big one. Right now we are operating on strings, and each time we consume a bit from the front of a string we are creating a whole new string to return from the `run` function. That could be very inefficient if we are trying to parse large input strings. But it doesn’t have to be that way. Since we are typically consuming from the front of the string, what if we could do something very lightweight, like change the `startIndex` of the string to point to characters inside the string. That would trick the string into thinking it’s shorter than it really is, but it will still be the same string that we started with, no copying necessary.

Turns out, this is exactly how the `Substring` type works in Swift. So if we were to work with a string’s substring instead, maybe we could move that index along and make things _really_ performant.

Substring is a wrapper around some string storage, and it’s intended to share that storage with other `Substring` instances. Then, certain mutations of substrings can be done very efficiently because internally all it does is move around indices that indicate where the string begins and ends.

Every Swift collection has an underlying `SubSequence` type, and they’re all intended to make collection usage more efficient.

Let’s try updating our parser to work on `Substring` and see what goes right and what goes wrong. We’ll start by changing its mutating `run` function to work with `inout Substring` instead of a `String`.
```swift
    struct Parser<A> {
      let run: (inout Substring) -> A?
    }
```
This breaks our non-mutating version of `run`, which is still working with plain ole `String`:
```swift
    func run(_ str: String) -> (match: A?, rest: String) {
      var str = str
      let match = self.run(&str)
```
> 🛑 Cannot invoke ‘run’ with an argument list of type ‘(inout String)’

We can now avoid the string copy by using its substring representation, which we can get by subscripting with what Swift calls an “unbounded range expression”:
```swift
    var str = str[...]
```
This looks a lil funky but it’s effectively giving us a view into the string rather than creating a brand new one.

And finally, we should update the return value to return that substring instead.
```swift
    func run(_ str: String) -> (match: A?, rest: Substring) {
      var str = str[...]
      let match = self.run(&str)
      return (match, str)
    }
```
And that’s all that needed to change! The interface for mutating substrings is the same as the interface for mutating strings, but we’ll get a bit of a performance boost by working with a view into the string instead of a copy.

We still have the restriction that the entire `String` must be in memory, which means parsing a very large `String` isn’t going to be efficient, but the optimizations we’ve made so far were very low-hanging and already buy us a lot, so let’s kick that can a bit down the road.

## Till next time

We now have the “final form” of our parser: it’s a function that takes an in-out substring and produces an optional match. So let’s create a few more parsers so that we get a feel for how this goes.

## Exercises
1. Create a parser `char: Parser<Character>` that will parser a single character off the front of the input string.
2. Create a parser `whitespace: Parser<Void>` that consumes all of the whitespace from the front of the input string. Note that this parser is of type Void because we probably don’t care about the actual whitespace we consumed, we just want it consumed.
3. Right now our int parser doesn’t work for negative numbers, for example `int.run("-123")` will fail. Fix this deficiency in int.
4. Create a parser `double: Parser<Double>` that consumes a double from the front of the input string. 
5. Define a function `literal: (String) -> Parser<Void>` that takes a string, and returns a parser which will parse that string from the beginning of the input. This exercise shows how you can build complex parsers: you can use a function to take some up-front configuration, and then use that data in the definition of the parser.
6. In this episode we mentioned that there is a correspondence between functions of the form `(A) -> A` and functions `(inout A) -> Void`. We even covered this in a previous episode, but it is instructive to write it out again. So, define two functions toInout and fromInout that will transform functions of the form (A) -> A to functions `(inout A) -> Void`, and vice-versa.
 

## Reference
- Swift Strings and Substrings
	- In this free episode of Swift talk, Chris and Florian discuss how to efficiently use Swift strings, and in particular how to use the Substring type to prevent unnecessary copies of large strings.
	- We write a simple CSV parser as an example demonstrating how to work with Swift’s String and Substring types.
	- https://talk.objc.io/episodes/S01E78-swift-strings-and-substrings
- Swift Pitch: String Consumption
	- Swift contributor Michael Ilseman lays out some potential future directions for Swift’s string consumption API. This could be seen as a “Swiftier” way of doing what the Scanner type does today, but possibly even more powerful.
	- https://forums.swift.org/t/string-consumption/21907

----
- tags : #pointfree #parser
	