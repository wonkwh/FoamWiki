# What Is a Parser?: Part 3
## Introduction

The interface for mutating substrings is the same as the interface for mutating strings, but we’ll get a bit of a performance boost by working with a view into the string instead of a copy.

We still have the restriction that the entire `String` must be in memory, which means parsing a very large `String` isn’t going to be efficient, but the optimizations we’ve made so far were very low-hanging and already buy us a lot, so let’s kick that can a bit down the road.

## Constructing more parsers
We now have the “final form” of our parser: it’s a function that takes an in-out substring and produces an optional match. So let’s create a few more parsers so that we get a feel for how this goes.

First let’s build a parser that will try to parse a double off the beginning of a string:

    let double = Parser<Double> { str in
      let prefix = str.prefix(while: { $0.isNumber || $0 == "." })
      guard let match = Double(prefix) else { return nil }
      str.removeFirst(prefix.count)
      return match
    }

Let’s take it for a spin!

    double.run("42")
    
    
    double.run("42.8743289247")
    
    
    double.run("42.8743289247 Hello World")
    

This double parser isn’t perfect because we can’t consume strings with multiple decimal points:

    double.run("42.4.1.4.6")
    

With a little more work we can make it right, but we are going to just leave it here for now.

We could also make a parser that parses a constant string off the beginning of a string. This is useful for making sure that certain tokens are present in a string. This parser is a little different in the other two, in that we don’t actually care about getting back the data we parsed off the string, but only care whether or not the parsing succeeded. Therefore the parser is of type `Parser<Void>`, which may seem a little weird:

One interesting thing about this parser is that it’s a function that produces a parser. This allows us to provide some upfront configuration for how our parser behaves, in this case we provide the string that we want to match on the beginning of the input string.

    func literal(_ literal: String) -> Parser<Void> {
      return Parser<Void> { str in
        guard str.hasPrefix(literal) else { return nil }
        str.removeFirst(literal.count)
        return ()
      }
    }

    literal("cat").run("cat dog")
    
    
    literal("cat").run("dog cat")
    

We’ll encounter a bunch of other “parser generators” like `literal` as we go onward, where functions return parsers or take parsers as input to produce brand new parsers in a higher-order kind of way.

We could also cook up some seemingly pathological parsers, but they actually turn out to be pretty handy as we will later see. For example, we could cook up a parser that always succeeds and doesn’t consume anything from the input:

    func always<A>(_ a: A) -> Parser<A> {
      return Parser { _ in a }
    }

The `always` parser _always_ succeeds.

    always("cat").run("dog")
    

This may not seem like it makes a lot of sense, but the parser succeeded with “cat” and we still have “dog” to left to parse.

We could also do the opposite: a parser that never succeeds but instead immediately fails and does not consume any of the input:

    func never<A>() -> Parser<A> {
      return Parser { _ in nil }
    }

So to use it, we can give an explicit generic and run our parser.

    (never() as Parser<Int>).run("dog")
    

It’s a little awkward to use a generic function like this, but it’s necessary since Swift doesn’t support “generic variables”:

A way we can approximate this is through static computed properties, which is something we’ve used quite a lot on Point-Free:

    extension Parser {
      static var never: Parser {
        return Parser { _ in nil }
      }
    }

And now using `never` becomes a little bit nicer.

    Parser<Int>.never.run("dog")
    

## What's the point?

We’ve now built five parsers: an `int` parser that scans an `Int` off the beginning of a string, a `double` parser that scans a `Double`, the `literal` parser that scans an exact string off the beginning of another string, and then an `always` parser and a `never` parser, which always or never succeed.

Ok, now we are getting into some weird stuff. We are defining parsers that always succeed and never succeed? Before we go too much further, let’s slow down and ask “what’s the point?”. We started this episode by demoing some of the parsers that come with Swift and Foundation. They got the job done, but we claimed that they weren’t very extensible or composable. And so we went down this road building our own `Parser` type, and although we have parsed a few things, we still haven’t done anything too complicated. So, where is this going?

Well, the real point of this episode was for us to all get comfortable with the problem space of parsers, and to properly define what a parser is. As far as we are concerned, it’s just a function that takes a string and returns an optional value of some type, and it will possibly consume some subset of the input string.

And although the type we have defined so far doesn’t seem to be able to do too much, we promise that there is an entire world of composability lurking in that type, we should haven’t explored it yet. But even before we get to all of that, we think this type is already showing some promise.

## Revisiting coordinate parsing

Let’s go back to the latitude/longitude coordinate parsing function and update it to use our new `Parser` type.

We previously defined this parser as a plain ole function, where we did a bunch of manual work using `split`, checking array `count`s, character equality checks, and so on.

    func parseLatLong(_ string: String) -> Coordinate? {
      let parts = string.split(separator: " ")
      guard parts.count == 4 else { return nil }
      guard
        let lat = Double(parts[0].dropLast()),
        let long = Double(parts[2].dropLast())
      else { return nil }
      let latCardinal = parts[1].dropLast()
      guard latCardinal == "N" || latCardinal == "S" else { return nil }
      let longCardinal = parts[3]
      guard longCardinal == "E" || longCardinal == "W" else { return nil }
      let latSign = latCardinal == "N" ? 1.0 : -1
      let longSign = longCardinal == "E" ? 1.0 : -1
      return Coordinate(latitude: lat * latSign, longitude: long * longSign)
    }

Let’s redo this function using the parsers we constructed.

The first thing we need to do is get our string input into the `Substring` format because that is what are parsers understand:

    func parseLatLong(_ coordString: String) -> Coordinate? {
      var str = coordString[...]
    }

Then we could first parse off a double from the front of the input:

    func parseLatLong(_ coordString: String) -> Coordinate? {
      var str = coordString[...]
      guard let lat = double.run(&str)
        else { return nil }
    }

And then we could parse off the degree symbol and whitespace from the input:

    func parseLatLong(_ coordString: String) -> Coordinate? {
      var str = coordString[...]
      guard let lat = double.run(&str)
        else { return nil }
    
      guard literal("° ").run(&str) != nil
        else { return nil }

Before going further, let’s combine our `guard` statements to clean things up a bit.

    func parseLatLong(_ coordString: String) -> Coordinate? {
      var str = coordString[...]
      guard
        let lat = double.run(&str),
        literal("° ").run(&str) != nil
        else { return nil }

Now we need to parse an “N” or “S” character from the string, and convert that to a +1 or -1. Previously we did that in multiple steps, but now we can build up a specialized parsers that does just that:

    let northSouth = Parser<Double> { str in
      guard
        let cardinal = str.first,
        cardinal == "N" || cardinal == "S"
        else { return nil }
      str.removeFirst(1)
      return cardinal == "N" ? 1 : -1
    }

And we can use this self-contained, reusable unit of parsing rather simply:

    func parseLatLong(_ coordString: String) -> Coordinate? {
      var str = coordString[...]
      guard
        let lat = double.run(&str),
        literal("° ").run(&str) != nil,
        let latSign = northSouth.run(&str)
      else { return nil }
    }

Then we need to parse off the comma and whitespace:

    func parseLatLong(_ coordString: String) -> Coordinate? {
      var str = coordString[...]
      guard
        let lat = double.run(&str),
        literal("° ").run(&str) != nil,
        let latSign = northSouth.run(&str),
        literal(", ").run(&str) != nil
      else { return nil }
    }

And then we do the process all over again by parsing off a double, then the degree symbol, and then the cardinal direction.

    func parseLatLong(_ coordString: String) -> Coordinate? {
      var str = coordString[...]
      guard
        let lat = double.run(&str),
        literal("° ").run(&str) != nil,
        let latSign = northSouth.run(&str),
        literal(", ").run(&str) != nil,
        let long = double.run(&str),
        literal("° ").run(&str) != nil,
        let longSign = eastWest.run(&str)
        else { return nil }

We’re going to need to define `eastWest` though:

    let eastWest = Parser<Double> { str in
      guard
        let cardinal = str.first,
        cardinal == "E" || cardinal == "W"
        else { return nil }
      str.removeFirst(1)
      return cardinal == "E" ? 1 : -1
    }

And bringing it all together we have:

    func parseLatLong(_ coordString: String) -> Coordinate? {
      var str = coordString[...]
      guard
        let lat = double.run(&str),
        literal("° ").run(&str) != nil,
        let latSign = northSouth.run(&str),
        literal(", ").run(&str) != nil,
        let long = double.run(&str),
        literal("° ").run(&str) != nil,
        let longSign = eastWest.run(&str)
        else { return nil }
    
      return Coordinate(latitude: lat * latSign, longitude: long * longSign)
    }

And now our previous bit of parsing, which was erroneously succeeding, now fails!

    print(parseLatLong("40.6782% N- 73.9442% W"))
    

If we switch things to use the correct symbols, it passes.

    print(parseLatLong("40.6782° N, 73.9442° W"))
    

I think this is already looking quite a bit better than the hand rolled parsing we were doing before. For one thing, all of the incremental consumption of the input is happening in a linear, line-by-line fashion and telling a very direct story. First we parse off a double, then the degree sign, then the cardinal direction, then a comma, then another double, then another degree sign, and then another cardinal direction. Once all the data is parsed from the string we bring it all together to create the actual `Coordinate` value.

Something else that is better about this style is that we got our first glimpse at code reuse. The `northSouth` and `eastWest` parsers we built can be used anywhere, not just for parsing this specific coordinate format. And we can peel off as many little helper parsers as we want. For example, right now we are repeating the `literal("° ")` parser twice, so maybe we should extract it out:

The `Scanner` style of parsing has many of these benefits, but because its API hasn’t been updated for Swift it can be quite cumbersome to use. We’ll save you from the details of coding it up from scratch, but this is what it would look like:

    func parseLatLongWithScanner(_ string: String) -> Coordinate? {
      let scanner = Scanner(string: string)
    
      var lat: Double = 0
      guard scanner.scanDouble(&lat) else { return nil }
    
      guard scanner.scanString("° ", into: nil) else { return nil }
    
      var northSouth: NSString? = ""
      guard scanner.scanCharacters(from: ["N", "S"], into: &northSouth) else { return nil }
      let latSign = northSouth == "N" ? 1.0 : -1
    
      guard scanner.scanString(", ", into: nil) else { return nil }
    
      var long: Double = 0
      guard scanner.scanDouble(&long) else { return nil }
    
      guard scanner.scanString("° ", into: nil) else { return nil }
    
      var eastWest: NSString? = ""
      guard scanner.scanCharacters(from: ["E", "W"], into: &eastWest) else { return nil }
      let longSign = eastWest == "E" ? 1.0 : -1
    
      return Coordinate(latitude: lat * latSign, longitude: long * longSign)
    }

Each time we parse something we need two lines: one to set up a mutable variable, and another to do the scanning, which also requires a `guard` to check if it scanned successfully.

So at the very least our `Parser` type provides a more ergonomic interface to parsing that is more conducive to sharing parsers and embracing code reuse. That alone might be reason enough to use this type, but it’s only the beginning. Now that we’ve laid the foundation for the structure of what a parser is we can begin to define a whole bunch of useful, reusable parsers and build up far more complex ones. And we’ll be able to do so with all of those universal operators that we have covered many times on Point-Free, like `map`, `zip` and `flatMap`, which it turns out the Parser type has! And it’s those operators that unleash a whole world of composability, which allow us to piece together lots of small parsers to build up a huge, complex parser. This is what we will discuss next time!

## Exercises
1. Right now all of our parsers (int, double, literal, etc.) are defined at the top-level of the file, hence they are defined in the module namespace. While that is completely fine to do in Swift, it can sometimes improve the ergonomics of using these values by storing them as static properties on the Parser type itself. We have done this a bunch in previous episodes, such as with our Gen type and Snapshotting type.Move all of the parsers we have defined so far to be static properties on the Parser type. You will want to suitably constrain the A generic in the extension in order to further restrict how these parsers are stored, i.e. you shouldn’t be allowed to access the integer parser via `Parser<String>.int.`
2. 	We have previously devoted an entire episode (here) to the concept of map, then 3 entire episodes (part 1, part 2, part 3) to zip, and then 5 (!) entire episodes (part 1, part 2, part 3, part 4, part 5) to flatMap. In those episodes we showed that those operations are very general, and go far beyond what Swift gives us in the standard library for arrays and optionals. Define map, zip and flatMap on the Parser type. Start by defining what their signatures should be, and then figure out how to implement them in the simplest way possible. What gotcha to be on the look out for is that you do not want to consume any of the input string if the parser fails.
3. 	Create a parser end: `Parser<Void>` that simply succeeds if the input string is empty, and fails otherwise. This parser is useful to indicate that you do not intend to parse anymore.
4. 	Implement a function that takes a `predicate (Character) -> Bool` as an argument, and returns a parser `Parser<Substring>` that consumes from the front of the input string until the predicate is no longer satisfied. It would have the signature `func pred: ((Character) -> Bool) -> Parser<Substring>.`
5. 	Implement a function that transforms any parser into one that does not consume its input at all. It would have the signature `func nonConsuming: (Parser<A>) -> Parser<A>.`
6. 	Implement a function that transforms a parser into one that runs the parser many times and accumulates the values into an array. It would have the signature `func many: (Parser<A>) -> Parser<[A]>`
7. 	Implement a function that takes an array of parsers, and returns a new parser that takes the result of the first parser that succeeds. It would have the signature `func choice: (Parser<A>...) -> Parser<A>`
8. 	Implement a function that takes two parsers, and returns a new parser that returns the result of the first if it succeeds, otherwise it returns the result of the second. It would have the signature `func either: (Parser<A>, Parser<B>) -> Parser<Either<A, B>>` where Either is defined:

	```swift
	enum Either<A, B> {
	  case left(A)
	  case right(B)
	}
	```
	
9. 	Implement a function that takes two parsers and returns a new parser that runs both of the parsers on the input string, but only returns the successful result of the first and discards the second. It would have the signature `func keep(_: Parser<A>, discard: Parser<B>) -> Parser<A>.` Make sure to not consume any of the input string if either of the parsers fail.
10. Implement a function that takes two parsers and returns a new parser that runs both of the parsers on the input string, but only returns the successful result of the second and discards the first. It would have the signature `func discard(_: Parser<A>, keep: Parser<B>) -> Parser<B>.` Make sure to not consume any of the input string if either of the parsers fail.
11. Implement a function that takes two parsers and returns a new parser that returns of the first if it succeeds, otherwise it returns the result of the second. It would have the signature `func choose: (Parser<A>, Parser<A>) -> Parser<A>.` Consume as little of the input string when implementing this function.
12. Generalize the previous exercise by implementing a function of the form `func choose: ([Parser<A>]) -> Parser<A>.`
13. Right now our parser can only fail in a single way, by returning nil. However, it can often be useful to have parsers that return a description of what went wrong when parsing. Generalize the Parser type so that instead of returning an A? value it returns a Result<A, String> value, which will allow parsers to describe their failures. Update all of our parsers and the ones in the above exercises to work with this new type.
14. Right now our parser only works on strings, but there are many other inputs we may want to parse. For example, if we are making a router we would want to parse URLRequest values. Generalize the Parser type so that it is generic not only over the type of value it produces, but also the type of values it parses. Update all of our parsers and the ones in the above exercises to work with this new type (you may need to constrain generics to work on specific types instead of all possible input types).


## Reference
- Difficulties With Efficient Large File Parsing
	- This question on the Swift forums brings up an interesting discussion on how to best handle large files (hundreds of megabytes and millions of lines) in Swift. The thread contains lots of interesting tips on how to improve performance, and contains some hope of future standard library changes that may help too.
	- https://forums.swift.org/t/difficulties-with-efficient-large-file-parsing/23660
----
- tags : #pointfree #parser