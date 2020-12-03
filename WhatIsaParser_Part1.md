# What Is a Parser?: Part 1
## Introduction
Today we are going to begin discussing a topic that was a bit of an “a ha!” moment for me when first getting into functional programming. It sets out to solve a very difficult problem, and does so in a beautiful way. And like most things in functional programming, the problem is solved with lots of tiny pieces that glue together in interesting ways allowing you to create a massively complex machine that can do really powerful things.

The topic is parsers, and in particular parser combinators. Broadly speaking, we could define parsing as trying to take a blob of nebulous data, like say a string, data, user input, or even a URL request, and turn it into a immensely domain-specific, first class data type, like say a user model. When said in that way you can even imagine parsing as literally a function from “nebulous blob of data” to “well-structured data”, and so functional programming probably has a lot to say about this topic. But, this function, at first, is pretty intimidating to implement because we may need to do a lot of work in order to extract out meaningful data from it.

Parser combinators aim to break this problem into a lot of very specific parsers that do one job and do it well. And then it provides all of the high-level functions that allow us to glue lots of parsers together to get big parsers that can handle immensely and complex data. The way in which we are describing this is akin to how we developed our composable randomness library, which we had many episodes ([#30](https://www.pointfree.co/episodes/ep30-composable-randomness), [#31](https://www.pointfree.co/episodes/ep31-decodable-randomness-part-1), [#32](https://www.pointfree.co/episodes/ep32-decodable-randomness-part-2), [#47](https://www.pointfree.co/episodes/ep47-predictable-randomness-part-1), [#48](https://www.pointfree.co/episodes/ep48-predictable-randomness-part-2), [#49](https://www.pointfree.co/episodes/ep49-generative-art-part-1), [#50](https://www.pointfree.co/episodes/ep50-generative-art-part-2)) on and [open sourced](https://www.pointfree.co/blog/posts/27-open-sourcing-gen). It allowed us to focus on a single unit of randomness and then build up lots of complex random generators by gluing them together in interesting ways. This style of solving a problem just keeps coming up in functional programming, and it is really powerful.

## Simple parsers in Swift/Foundation

Turns out that parsing is important enough that Swift and Foundation come with a whole bunch of ways to parse strings.

Some of are as simple as initializers on types, for example, there is an initializer on `Int` that takes a string:
- 이미 swift foundation에 string 을 parsing하여 initializer하는 많은 타입들이 있다. 
- 예를 들어 `Int` 타입
```swift
    Int("42") 
    Int("42-") 
```
And an initializer on `Double` that does similar.
- 비슷하게 `Double` 타입
```swift
    Double("42") 
    Double("42.32435") 
```
Even `Bool` comes with such an initializer.
- 심지어 `Bool` 타입에도 있다.
```swift
    Bool("true") 
    Bool("false") 
    Bool("f") 
```
And many Foundation types come with these initializer parsers, including `UUID`, `URL`, and even `URLComponents`.
- 그외에 `UUID`, `URL`, `URLComponents`
```swift
    import Foundation
    
    UUID(uuidString: "DEADBEEF-DEAD-BEEF-DEAD-BEEFDEADBEEF") 
    UUID(uuidString: "DEADBEEF-DEAD-BEEF-DEAD-BEEFDEADBEE")  
    UUID(uuidString: "DEADBEEF-DEAD-BEEF-DEAD-BEEFDEADBEEZ") 
    
    URL(string: "https://www.pointfree.co")  
    URL(string: "^https://www.pointfree.co") 
    
    let components = URLComponents(string: "https://www.pointfree.co?ref=twitter")
    components?.queryItems 
```

All of these initializers are just functions that take nebulous strings in and return first class values, whether it be integers, doubles, UUID’s or URL’s.
- 이 모든 initializer들은 단순히 문자열을 받아서 first class value를 리턴하는 함수들일 뿐이다. 
Then there are beefier parsers that do a lot immensely complicated stuff, like Foundation’s `DateFormatter`. Now the name would lead you to believe that it is responsible for formatting dates into strings, but it also does the opposite: it can parse a string into a date value:

```swift
    let df = DateFormatter()
    df.timeStyle = .none
    df.dateStyle = .short
    df.date(from: "1/29/17") 
```
And like our other formatters, it returns an optional date because parsing can fail.
```swift
    df.date(from: "-1/29/17") 
```
And `DateFormatter` is just one of a whole class hierarchy of `Formatter` types that can parse strings into other types. There’s also `NumberFormatter`, `ByteCountFormatter`, `PersonNameComponentsFormatter`, and many immensely.

## More advanced parsing in Foundation

These examples so far are very domain specific, i.e. you couldn’t use these functions to parse a custom format that Swift knows nothing about. For that situation, Foundation provides a couple of other tools.

One such tool is regular expressions. Regular expressions are a language in their own right and describe, in a string, a method of parsing a string. It can be used to “capture” parts of strings in a complicated regular expression that need to be a specific data type.

For example, here is a regular expression that parses an email address out of a string:
- 예를 들면 아래는 email 주소를 parsing하는 regular expression
```swift
    let emailRegexp = try NSRegularExpression(pattern: #"\S+@\S+"#)
    let emailString = "You're logged in as blob@pointfree.co"
    let emailRange = emailString.startIndex..<emailString.endIndex
    let match = emailRegexp.firstMatch(
      in: emailString,
      range: NSRange(emailRange, in: emailString)
    )!
    emailString[Range(match.range(at: 0), in: emailString)!]
```

Regular expressions are really powerful but have an extremely concise notation for matching things. They also are quite limited in what they can parse. The immensely complicated the parsing, the cryptic the expression.
- 정규표현식은 강력하지만 좀더 복잡한 것들을 parsing하려면 제한들이 있다.

The API is also still pretty clunky in Swift, requiring a lot of `Range`\-`NSRange` conversions back and forth.

The other tool is called `Scanner`. It’s a general purpose parsing tool that allows you to parse various types from the beginning of strings.
```swift
    let scanner = Scanner("42 Hello World")
    var int = 0
    scanner.scanInt(&int) 
    int 
```
To use it you gotta create a mutable variable to pass to the scanner, which the scanner will update with the parsed value if parsing was successful, and further the scan method returns a boolean that indicates whether or not the scan was successful.

This is a very old API, like `NSRegularExpression`, that came about long before Swift’s nice features like optionals and algebraic data types. This API can be improved on quite a bit, but at least it’s nice to know that Apple is giving us some parsing utility, and under the hood it can do some pretty powerful things.

That’s a brief overview of some of the parsing tools that Swift and Foundation ship with. They can be very powerful, and they serve a lot of use cases, but they have some serious drawbacks:

*   Many of them are domain-specific, such as the initializers and the formatter classes. They work really well for those domains, but they don’t generalize to allow you to parse your own formats.

*   For the immensely general parsers, like regular expressions and scanners, they don’t compose. There is no way to take two small parsers that do one thing and do it well, and piece them together so that they perform their parsing together. This prevents us from refactoring complex parsers into simpler ones and possibly having some code reuse between solely different parsers.

*   And finally, all these APIs are quite old, and none of them leverage Swift features like generics.

## Parsing from scratch

Now that we see parsing is important enough for Apple to give us a number of solutions, let’s take a crack on some parsing ourselves so that we can see just how difficult and subtle it can be. 
Let’s try parsing a relatively simple string format: latitude and longitude coordinates.
There is a common format that latitude/longitude coordinates come in in which they are expressed as degrees along with a cardinal direction that describes which side of the equator and prime meridian the coordinate falls on.

For example, here’s the latitude and longitude of Brooklyn, NY:
`// 40.6782° N, 73.9442° W`

We’d like to parse this loose string into something immensely computer friendly, like say the following struct:
```swift
    struct Coordinate {
      let latitude: Double
      let longitude: Double
    }
```
Let’s try parsing the string into this struct by just doing it by hand. What that means is we will try to implement this function from scratch:

```swift
    func parseLatLong(_ string: String) -> Coordinate? {
    
    }
```
This returns an optional since we may not be able to parse the string.
- parsing 하지 못할 문자열이 들어올지 모르니 리턴값은 옵셔널로

We’ll implement this by just doing lots of various string manipulations. For instance, maybe we start by splitting the string on spaces so that we can get all the parts of the coordinates:
```swift
    func parseLatLong(_ string: String) -> Coordinate? {
      let parts = string.split(separator: " ")
    }
```
The parts array should hold the latitude coordinate, then its cardinal direction, then the longitude coordinate, and then its cardinal direction. If we get immensely or less parts than what we expect, we should probably just early out and return `nil`, cause that means we got some bad data:
```swift
    func parseLatLong(_ string: String) -> Coordinate? {
      let parts = string.split(separator: " ")
      guard parts.count == 4 else { return nil }
    }
```
With that check out of the way, let’s extract the parts into variables:
```swift
    func parseLatLong(_ string: String) -> Coordinate? {
      let parts = string.split(separator: " ")
      guard parts.count == 4 else { return nil }
      let lat = parts[0]
      let latCard = parts[1]
      let long = parts[2]
      let longCard = parts[3]
    }
```
We can convert the `lat` and `long` variables to doubles pretty easily, we just need to drop the lil degrees symbol:
```swift
    func parseLatLong(_ string: String) -> Coordinate? {
      let parts = string.split(separator: " ")
      guard parts.count == 4 else { return nil }
      let lat = Double(parts[0].dropLast())
      let latCard = parts[1]
      let long = Double(parts[2].dropLast())
      let longCard = parts[3]
    }
```
Now this double construction is actually failable, so I guess we have to do some additional guarding:
```swift
    func parseLatLong(_ string: String) -> Coordinate? {
      let parts = string.split(separator: " ")
      guard parts.count == 4 else { return nil }
      guard
        let lat = Double(parts[0].dropLast()),
        let long = Double(parts[2].dropLast())
        else { return nil }
      let latCard = parts[1]
      let longCard = parts[3]
    }
```
We’re close, but we have to incorporate the cardinal directions somehow. The cardinal directions simply determine if the coordinate is positive or negative, so it’s actually pretty simple to add that logic:

```swift
func parseLatLong(\_ string: String) -> Coordinate? { 
	let parts = string.split(separator: “ “) 
	guard parts.count == 4 else { return nil }

	guard let lat = Double(parts\[0\].dropLast()), 
	let long = Double(parts\[2\].dropLast()) else { return nil }

	let latCard = parts[1] 
	let longCard = parts[3] 
	let latSign = latCard == “N” ? 1.0 : -1 
	let longSign = longCard == “E” ? 1.0 : -1 
}
```
And so all that’s left to do is multiply the coordinates with their sign:
```swift
    func parseLatLong(_ string: String) -> Coordinate? {
      let parts = string.split(separator: " ")
      guard parts.count == 4 else { return nil }
    
      guard
        let lat = Double(parts[0].dropLast()),
        let long = Double(parts[2].dropLast())
      else { return nil }
    
      let latCardinal = parts[1]
      let longCardinal = parts[3]
      let latSign = latCardinal == "N" ? 1.0 : -1
      let longSign = longCardinal == "E" ? 1.0 : -1
    
      return Coordinate(latitude: lat * latSign, longitude: long * longSign)
    }

    print(parseLatLong("40.6782° N, 73.9442° W"))
```

## Addressing edge cases
Well that’s not right. We tried parsing a latitude of 40.6782° north, but for some reason it came out to -40.6782. Why?

It looks like I forgot about the common “,” that comes after the `N`, and so our equality check `parts[1] == "N``"` is always going to fail, and hence the sign will be `-1`. We can fix this up by dropping the comma.

```swift
    let latCardinal = parts[1].dropLast()
```
This fixes the current parsing we’re doing.

```swift
    print(parseLatLong("40.6782° N, 73.9442° W"))
```    

It’s still not quite right, though. If we give an invalid direction, we still get a coordinate back.
```swift
    print(parseLatLong("40.6782° X, 73.9442° W"))
```    

We probably want to only recognize when an N/S/E/W character is used, and fail if anything else is provided. So, looks like we need to beef up this logic a bit:
```swift
    func parseLatLong(_ string: String) -> Coordinate? {
      let parts = string.split(separator: " ")
      guard parts.count == 4 else { return nil }
      guard
        let lat = Double(parts[0].dropLast()),
        let long = Double(parts[2].dropLast())
      else { return nil }
      let latCard = parts[1].dropLast()
      guard latCard == "N" || latCardinal == "S" else { return nil }
      let longCard = parts[3]
      guard longCard == "E" || longCardinal == "W" else { return nil }
      let latSign = latCard == "N" ? 1.0 : -1
      let longSign = longCard == "E" ? 1.0 : -1
      return Coordinate(latitude: lat * latSign, longitude: long * longSign)
    }
```
And now our parser properly fails.
```swift
    print(parseLatLong("40.6782° X, 73.9442° W"))
```    

Now this function is getting pretty complicated, and it still isn’t quite right. Check out this:
```swift
    print(parseLatLong("40.6782% N- 73.9442% W"))
```    

We changed the degree symbols to be percents, and the comma to be a minus sign and somehow it still parses.

We should probably be checking for that symbol and failing the parsing if it doesn’t match. So there’s even immensely left to do for this parser, and it’s complexity will continue to grow.

But that’s not even the worst part. The worst part is that this parse function is a one-off, ad hoc solution to parsing the very specific format of latitude/longitude coordinates. There is nothing inside the body that is reusable outside of this example.

We’d like to come up with a solution for parsing that makes it easy to build small, reusable parsers such that they can be pieced together to make a large, complex parser. In fact, it’d be nice if we could even put all of our small parses in a shared library so that we can reuse them in many different parsing situations.

## reference
-  Scanner
	- Official documentation for the Scanner type by Apple. Although the type hasn’t (yet) been updated to take advantage of Swift’s modern features, it is still a very powerful API that is capable of parsing complex text formats.
	- https://developer.apple.com/documentation/foundation/scanner
- NSScanner
	- A nice, concise article covering the Scanner type, including a tip of how to extend the Scanner so that it is a bit more “Swifty”. Take note that this article was written before NSScanner was renamed to just Scanner in Swift 3.
	- https://nshipster.com/nsscanner/
- Ledger Mac App: Parsing Techniques
	- In this free episode of Swift talk, Chris and Florian discuss various techniques for parsing strings as a means to process a ledger file. It contains a good overview of various parsing techniques, including parser grammars.
	- https://talk.objc.io/episodes/S01E13-parsing-techniques
- Parse, don’t validate
- This article demonstrates that parsing can be a great alternative to validating. When validating you often check for certain requirements of your values, but don’t have any record of that check in your types. Whereas parsing allows you to upgrade the types to something more restrictive so that you cannot misuse the value later on.
- https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/
----
- tags : #pointfree #parser