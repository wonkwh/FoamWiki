# Parsing and Performance Strings
## Introduction

Over the past many weeks we have built up a pretty impressive parser library. Our parser is a very general type that allows you to parse any kind of nebulous input into any kind of well-structured output. It supports lots of interesting forms of composition that allow you to break large problems down into smaller ones.

On top of all of that we were able to build up an impressive library of parsers and higher-order parsers that work on strings. They allowed us to scan values off the fronts of strings in an efficient manner, such as characters, numbers, prefixes and more. And these parsers dealt with a lower-level API than just plain `String`, called `Substring`. A `Substring` is like a view into a string. It’s not the actual string itself, but rather just a representation of a portion of a base string that is stored somewhere else. This means we can consume little bits off the front of a substring while only changing our view of the underlying string, which is a very lightweight thing to do. If we were dealing with raw `String`s then we would need to make a whole copy of the string, which can be a very expensive operation.

So it seems that maybe we are ready to close the book on parsers and open source it! But not so fast. There is a very important topic to consider, especially when it comes to parsers, which is performance. Sometimes we need to parse megabytes or even gigabytes of data, and we need to be as efficient as possible when it comes to scanning the input. And although we have taken a huge step by using `Substring`s instead of `String`s, it turns out there is a lot more we can do.

We want to start off by giving everyone a quick deep dive into Swift strings and their performance characteristics. It’s a tricky subject, and there are a few subtle edge cases to think about, but once that’s done we will find that there is an even lower level representation of strings for which parsing is even more efficient than `Substring`. It’s really quite amazing to see, but it also kind of opens up a whole can of worms that requires more work to wrangle in.

### String vs. substring

Let’s get our feet wet by really understanding what performance benefits `Substring` is giving us over `String`. We’ve previously stated that it helps us prevent copies, but let’s dig into the details.

Let’s start by creating a really big string:
```swift
    let bigString = String(repeating: "A", count: 1_000_000)
```
This string should take up something like a megabyte of memory when executed. Right now it’s immutable, but if we wanted to create a mutable copy we could do the following:
```swift
    var copy = bigString
```
Naively we may think that this immediately creates a copy and so now we have 2 megabytes of data being taken. However Swift is a little smarter when it comes to making copies of data by adopting a strategy it calls “copy-on-write”. It will try to wait as long as possible until it makes a copy, and in this case since we haven’t made any mutations to `copy` there really is no need to create a copy.

However, once I make a mutation to `copy` then Swift has no choice but to make a real copy, because then we have two different variables that need to hold two different values:

    copy.removeFirst()

And of course if I make another copy and another mutation then we will get a 3rd copy of this megabyte chunk of data:

    var anotherCopy = copy
    anotherCopy.removeFirst()

So, this isn’t great. It’s nice that Swift delays making copies sometimes, but eventually we will do an operation that will force Swift to make a copy.

This is where `Substring` comes in. It allows us to even further delay the moment that Swift must make a copy of a string. It’s a lightweight view into a subset of the string, specified by a start and end index. Turns out there are a lot of operations that we do with strings that boil down to just moving those start and end indices around. Not all, but a lot.

So this is why many of `String`’s APIs return `Substring`s instead of honest `String`s. Because it allows the API to return something meaningful without performing a copy of the string. For example, the `dropFirst` method on string is similar to `removeFirst` except instead of mutating the string it returns a new value that represents the first character being chopped off:

    bigString.dropFirst()

This value is not a string:

    bigString.dropFirst() as String

> 🛑 Cannot convert value of type ‘String.SubSequence’ (aka ‘Substring’) to type ‘String’ in coercion

It’s a substring.

    bigString.dropFirst() as Substring

Because this substring is just a view into the string Swift did not have to make a copy. It’s just a lightweight representation of a subset of the big string, in particular its start index is the one after the start index of the big string:

    let truncatedBigString = bigString.dropFirst()
    
    bigString.index(after: bigString.startIndex) == truncatedBigString.startIndex
    

So now any operations we do on `truncatedBigString` will secretly be routed to the underlying big string under the hood, but we don’t have to know about that.

This is really powerful. We can create tons of these lightweight values, all without ever creating a copy:
```swift
    let copy1 = truncatedBigString.dropFirst(100)
    let copy2 = copy1.dropFirst(1_000)
    let copy3 = copy2.dropFirst(10_000)
```
All of this executes without making a single copy. It’s just moving the `startIndex` to point at different parts of the underlying base string.

So, this all sounds great in theory, but does it actually matter in practice? Do these memory copies really slow things down? Since the memory is only used for a moment and then discarded it doesn’t really bloat the net memory usage of your application. And perhaps when we build our applications for release the compiler does some magic under the hood to minimize these copies.

## How to benchmark

Well, that’s a lot of conjecture without any real proof. Let’s prove it to ourselves that these copies can actually be problematic. We’re going to write a benchmark that exposes what happens when we make a copy of a string and mutate it versus making a copy of a substring and mutating it. Benchmarks are notoriously tricky to get right. You need to make sure you run the code in release mode so that all Swift optimizations are turned on, you need to make sure to get a bunch of samples so that any strange blips in the data can be averaged out, and you need to be sure that you are truly measuring the thing you think you are. It is all too easy to let something extra slip into a benchmark that shouldn’t be there, which can cause those numbers to be inaccurate and misleading.

So, we are going to use a project from Google called “swift-benchmark” that provides a nice interface for creating benchmarks.

It’s got lots of nice options, such as specifying how many times you want to iterate when gathering benchmark data, as well as how many initial iterations should be discarded as a part of a “warm up” period.

So, let’s get started. We’re going to create a new Swift package that is an executable:
```bash
    $ mkdir string-performance
    $ cd string-performance
    $ swift package init --type executable
    Creating executable package: string-performance
    Creating Package.swift
    Creating README.md
    Creating .gitignore
    Creating Sources/
    Creating Sources/string-performance/main.swift
    Creating Tests/
    Creating Tests/LinuxMain.swift
    Creating Tests/string-performanceTests/
    Creating Tests/string-performanceTests/string_performanceTests.swift
    Creating Tests/string-performanceTests/XCTestManifests.swift
    $ open Package.swift
```
Then we’ll add the “swift-benchmark” dependency to our `Package.swift`:
```swift
    .package(
      name: "Benchmark",
      url: "https://github.com/google/swift-benchmark",
      .branch("main")
    ),
```

## Correction

Since recording this episode, [swift-benchmark](https://github.com/google/swift-benchmark) has not only had a release, but has changed its main branch to `main`. We’ve updated our episode code and samples accordingly.

And we’ll make the executable depend on the `Benchmark` module:

    .target(
      name: "performance-intro",
      dependencies: [.product(name: "Benchmark", package: "Benchmark")]
    ),

With this we can go over to `main.swift`, which is the entry point for of our executable, and import `Benchmark`:

    import Benchmark

The simplest way to get a benchmark going is to use the global function `benchmark`, which allows you to give the benchmark a name and provide a closure which is where we will perform the work that we want to measure:

    benchmark("Copying: String") {
    }
    
    benchmark("Copying: Substring") {
    }

Then, to kick off these benchmarks we just have to call the global `main` function in the `Benchmark` module:

    Benchmark.main()

If we run this we will see that the executable immediately completes with an error:

> Error: Please build with optimizations enabled (`-c release` if using SwiftPM, `-c opt` if using bazel, or `-O` if using swiftc directly). If you would really like to run the benchmark without optimizations, pass the `--allow-debug-build` flag.

This is a good error to have. This is the benchmark library forcing us to build for release before we are allowed to benchmark. This is important because benchmarking code built for development can be highly misleading. Not only does that code run much slower than code built for release, but also two sets of code could have wildly different performance characteristics when built for `DEBUG` but be nearly identical when built for `RELEASE`. 

So, we need to open up our scheme settings and set the build configuration for our executable to be release.

Now when we run we get some actual benchmark results:

    running Copying: String... done! (78.31 ms)
    running Copying: Substring... done! (80.37 ms)
    
    name               time      std        iterations
    ---------------------------------------------
    Copying: String    29.000 ns ± 286.46 %    1000000
    Copying: Substring 30.000 ns ± 271.05 %    1000000
    Program ended with exit code: 0

This prints out an entry for each benchmark we added, and tells us on average how much time it took to execute the closure, along with its standard deviation, as well as how many times it executed the closure.

Nothing too surprising here. Our closures aren’t doing anything yet and so both benchmarks basically take the same amount of time.

It’s worth noting that even these empty benchmarks take some amount of time, about 30 nanoseconds, and a nanosecond is one billionth of a second. As short as it is, so we should keep that cost in mind when evaluating the results of the benchmarks we run.

Alright, let’s put some real work in those closures.

What we want to do is have a large string, and in one closure we mutate copies of it, and it another we mutate copies of its substring. So let’s start with a big-ish string:

    let string = String(repeating: "A", count: 1_000)

Then in our string benchmark we will make a copy, mutate it:

    benchmark("Copying: String") {
      var copy = string
      copy.removeFirst()
    }

And in our substring benchmark we will make a copy of the substring representation of the string, and then mutate it:

    benchmark("Copying: Substring") {
      var copy = string[...]
      copy.removeFirst()
    }

Running this we already see there’s a pretty substantial difference in execution times:

    running Copying: String... done! (458.44 ms)
    running Copying: Substring... done! (174.44 ms)
    
    name               time       std        iterations
    ----------------------------------------------
    Copying: String    334.000 ns ± 208.79 %    1000000
    Copying: Substring 104.000 ns ± 429.39 %    1000000
    Program ended with exit code: 0

The `Substring` version of this code seems to be about 3 times faster than the `String` version, although if we factor in the 30ish nanoseconds of an empty benchmark it’s actually even 4 times faster. Looks like all of those little string copies really do add up.

Let’s see what happens if we amp this up a bit. Right now our string is only 1,000 characters. Let’s bump that back up to a million characters:

    let string = String(repeating: character, count: 1_000_000)

    running Copying: String... done! (1785.61 ms)
    running Copying: Substring... done! (145.08 ms)
    
    name              time         std        iterations
    ------------------------------------------------
    Copying.String    29945.000 ns ±  28.86 %      45404
    Copying.Substring    95.000 ns ± 196.38 %    1000000
    Program ended with exit code: 0

Wow! The amount of time for string copying increased dramatically, nearly 100 fold. But the time for substrings remained the same. We did not incur any additional cost from making copies by making the input string much larger.

And this dramatic change happened with just a single copy. In a real world parsing problem we would probably make dozens or more copies as we apply more and more parsers. So let’s add a few more copies to our benchmarks:

    benchmark("Copying: String") {
      var copy = string
      copy.removeFirst()
      var copy1 = copy
      copy1.removeFirst()
      var copy2 = copy1
      copy2.removeFirst()
      var copy3 = copy2
      copy3.removeFirst()
    }
    
    benchmark("Copying: Substring") {
      var copy = string[...]
      copy.removeFirst()
      var copy1 = copy
      copy1.removeFirst()
      var copy2 = copy1
      copy2.removeFirst()
      var copy3 = copy2
      copy3.removeFirst()
    }

    running Copying: String... done! (1540.30 ms)
    running Copying: Substring... done! (453.30 ms)
    
    name              time          std        iterations
    -----------------------------------------------------
    Copying.String    101229.000 ns ±  89.94 %      11513
    Copying.Substring    333.000 ns ± 200.08 %    1000000
    Program ended with exit code: 0

All the numbers have basically gotten three times slower due to the three new copies we performed.

## Copying substrings

So it really does seem that our decision to lean on `Substring` for our parser was a good one. It is preventing us from needlessly making copies of large strings all over the place.

However, this doesn’t mean that `Substring` doesn’t prevent us from making copies 100% of the time. `Substring` will be forced to make a copy if you do something with it that cannot be expressed simply as moving the start and end indices around. For example, if we were to append a string to the end of a `Substring`, then it would be forced to make a whole new copy of the base string so that it could append the new string to it.

We can even see this in the benchmarks by appending an exclamation mark to one of our copied substrings:

    var copy = string[...]
    copy.removeFirst()
    var copy1 = copy
    copy1.removeFirst()
    var copy2 = copy1
    copy2.removeFirst()
    var copy3 = copy2 + "!"
    copy3.removeFirst()

Now when we run this our substring benchmark has gotten much slower, but it’s still a little faster than the string benchmark:

    running Copying: String... done! (1286.08 ms)
    running Copying: Substring... done! (1360.65 ms)
    
    name              time          std        iterations
    -----------------------------------------------------
    Copying.String    123570.500 ns ±  85.96 %       7618
    Copying.Substring  90107.500 ns ±  39.69 %      12310
    Program ended with exit code: 0

We can even witness these string copies right in Xcode by using the memory debugger. What we can do is put a breakpoint in each of these closures, and we should see memory jump whenever we step over a particular line.

Unfortunately the debugger doesn’t work super well with release builds. This is because optimizations have been performed that may no longer correspond to how our code looks, and so stepping over lines can act a little funny. So, order to use use the debugger we need to switch back to `DEBUG` mode in our scheme settings.

And remember that swift-benchmark doesn’t allow us to run benchmarks unless we are building for release, which is good. But we can also circumvent that check by adding the `--allow-debug-build` launch argument to our scheme settings.

Now let’s put a few break points in and run.

We start by getting to this line and our memory is at about 7 MB:

    var copy = string

When we step over this line we would have created this `copy` variable, but we also see that the memory doesn’t change. It’s still 7 MB.

    copy.removeFirst()

And when we step over this line we instantly see the memory jump up to 8 MB. This is us seeing the copy being made in real time. Because we wanted to remove the first character from this string it had no choice but to make a copy of the base string and then remove the character.

If we step through the rest of the code we will see our memory increase to 9 MB, then 10 MB, and finally to 11 MB:

    var copy1 = copy
    copy1.removeFirst()
    var copy2 = copy1
    copy2.removeFirst()
    var copy3 = copy2
    copy3.removeFirst()

So this is pretty clear proof that copies are being made all over the place, and that seems like the real culprit to our performance woes in this benchmark.

Now let’s try the same for the substring benchmark. Let’s first comment out the string benchmark so that it’s memory usage doesn’t infect this benchmark. When we run we stop at this line:

    var copy = string[...]

At this point memory usage is basically what it was previously, right now at 7.3mb. When we step over this line it stays the same. However, once we step over the next line:

    copy.removeFirst()

We something much different. Our memory has not increased at all. It’s holding steady at 7 MB. In fact, we can step over the next two `removeFirst`s and we see the same:

    var copy1 = copy
    copy1.removeFirst()
    var copy2 = copy1
    copy2.removeFirst()

Memory usage is still at 7 MB because `Substring` didn’t need to actually make any copies. It just needed to move the `startIndex` forward one unit.

However, once we step over line that creates a new string by appending an exclamation mark we finally see the memory jump up to 8 MB:

    var copy3 = copy2 + "!"

And then finally when we step over the `removeFirst` on that string we see that memory does not jump up:

    copy3.removeFirst()

So that’s pretty impressive. Memory usage grew by 4 megabytes in the string benchmark, but only grew by 1 megabyte in the substring benchmark, and that only happened because we did the silly thing of appending an exclamation mark. Had we not done that then memory wouldn’t have grown at all.

Before moving out, and before we forget, let’s quickly switch the build configuration of our scheme back to release and remove this string appending.

Another quick thing we can do is organize our benchmarks into a suite, and move the code into a separate file. This will be handy as we write more benchmarks and want to better organize them for visual display:

```swift    
    import Benchmark
    
    let copyingSuite = BenchmarkSuite(name: "Copying") { suite in
      let string = String(repeating: "A", count: 1_000_000)
    
      suite.benchmark("String") {
        var copy = string
        copy.removeFirst()
        var copy1 = copy
        copy1.removeFirst()
        var copy2 = copy1
        copy2.removeFirst()
        var copy3 = copy2
        copy3.removeFirst()
      }
    
      suite.benchmark("Substring") {
        var copy = string[...]
        copy.removeFirst()
        var copy1 = copy
        copy1.removeFirst()
        var copy2 = copy1
        copy2.removeFirst()
        var copy3 = copy2
        copy3.removeFirst()
      }
    }
    
    
    import Benchmark
    
    Benchmark.main([
      copyingSuite,
    ])
```

## Substring vs. unicode scalars

So, we’ve definitely convinced ourselves that there is a huge win to be had by working on `Substring` rather than `String`. It’s simply an abstraction layer that allows us to prevent needless copies of strings by sharing an underlying storage. The abstraction of `Substring` over `String` really isn’t all that substantial though. You can use `Substring` in all the same ways you would use a regular `String`.

There are other forms of string abstractions out there that are even lower level than `Substring`, and they offer other possible performance benefits. You might not notice it right away, but it turns out that `String` and `Substring` are doing a ton of work to hide a bunch of complexity from you.

For example, it turns out there are many ways to represent what visually appears to be the same string. If you wanted an “e” character with an accent you could just type it directly by pressing option+e followed by “e” again:

    "é"

This is a single “e with acute”, which we can also see by looking it up in macOS’s character panel.

However, there’s another way of representing this character. We can represented by a plain “e” character followed by a “combining acute accent”:

    "é"

This character visually looks the same as the other one, but there are some subtle differences. To explore this let’s assign these strings to some variables:

    let acuteE1 = "é"
    let acuteE2 = "é"

The first interesting thing we can notice is that even though we constructed these strings in two very different ways, they are still equal as strings:

    acuteE1 == acuteE2 

Even stranger, they are both collections of just one element:

    acuteE1.count 
    acuteE2.count 

We think this is a little peculiar because you’ll remember that we constructed `acuteE2` by typing in two characters: first the plain “e” and then the combining accent. So it seems that somehow `String` has squashed those two things into just one visual thing, which is nice.

In order to see differences between these two values we need to drop down to a lower level. Underlying Swift strings is an alternative representation known as “unicode scalars.” This is where you get access to more granular, individual bits that make up a string:

    acuteE1.unicodeScalars 
    acuteE2.unicodeScalars 

Well, according to the playground sidebar these two things still look the same. However, the playground is doing a bit of extra work to render this for us. In reality, `unicodeScalars` is a collection of the individual scalars that make up the string, and that includes that little combining accent we added. To detect it let’s count the number of scalars:

    acuteE1.unicodeScalars.count 
    acuteE2.unicodeScalars.count 

There we go! We can now clearly see that the second accented e is definitely a little different than the first, even though they look the same, and even though the `String` API does some work to squash the difference between them.

If we convert these collections to arrays we can even get a closer look at its raw data:

    Array(acuteE1.unicodeScalars) 
    Array(acuteE2.unicodeScalars) 

So this further shows the difference between these two strings. We can even ask if the unicode scalars are equal to each other, but we can’t do it with the double equals sign because these collections aren’t equatable. However, we can use the `elementsEqual` method, which allows us to compare any two sequences:

    accentedE1.unicodeScalars
      .elementsEqual(accentedE2.unicodeScalars)
    

Remember that the strings are equal:

    accentedE1 == accentedE2 

It’s just their unicode scalar representations that are not equal. This is because the `String` API is doing a lot of work to normalize the representation of the string to make it behave how you would expect. We can clearly see that there is only one single character in this string, and so it would be very strange if we asked for its `count` and for some reason got back 2!

Things get even more complicated when you start considering emojis. There are a ton of emojis out there, and many of them are not just a simple, single unicode scalar. Some are built up out of many scalars so that you can customize how the emoji looks.

A really good example is flags. Every country flag can be represented by two unicode characters smashed together. For example, there are these weird unicode characters that aren’t really meant to be used independently:

    "🇺"
    "🇸"

If you concatenate them together they magically become the flag emoji:

    "🇺" + "🇸" 

You can even express them from their escaped numeric values:

    "\u{1F1FA}\u{1F1F8}" 

Swift defaults to giving you the friendliest API up front, and then gives you the ability to drop to a lower level if you need more raw access to the underlying representation. But what do these representations mean as far as performance is concerned?

Let’s create another benchmark that does the copy and `removeFirst` dance, but using the unicode scalars instead of string or substring:
```swift
    suite.benchmark("UnicodeScalars") {
      var copy = string[...].unicodeScalars
      copy.removeFirst()
      var copy1 = copy
      copy1.removeFirst()
      var copy2 = copy1
      copy2.removeFirst()
      var copy3 = copy2
      copy3.removeFirst()
    }
```
    running Copying: String... done! (1537.80 ms)
    running Copying: Substring... done! (466.35 ms)
    running Copying: UnicodeScalars... done! (289.70 ms)
    
    name                   time         std        iterations
    ---------------------------------------------------------
    Copying.String         99846.000 ns ±  29.56 %      12686
    Copying.Substring        343.000 ns ± 232.34 %    1000000
    Copying.UnicodeScalars   200.000 ns ± 260.24 %    1000000
    Program ended with exit code: 0

It appears that working with unicode scalars is a little bit faster than working with substrings. This is because the logic for performing `removeFirst` on unicode scalars is a little simpler. The `Substring` API has to do extra work in order to figure out what is the single, contiguous character that needs to be dropped. For example, if the first character is a flag it needs to be able to detect that there are actually two scalars next to each other that form a single grapheme cluster, and therefore they should both be dropped:

    "🇺🇸".dropFirst() 

Whereas the unicode scalar view can simply drop the first scalar in the duet:

    String(("🇺🇸").unicodeScalars.dropFirst()) 

We can make these benchmarks even more dramatic if we make our large string a little more complicated. Rather than it being just 1 million A’s, let’s put in a million flag emojis:

    let string = String(repeating: "🇺🇸", count: 1_000_000)

    running Copying: String... done! (1818.20 ms)
    running Copying: Substring... done! (1887.07 ms)
    running Copying: UnicodeScalars... done! (288.51 ms)
    
    name                   time           std        iterations
    -----------------------------------------------------------
    Copying.String         2416261.500 ns ±  70.46 %        510
    Copying.Substring         4020.000 ns ±  69.51 %     304277
    Copying.UnicodeScalars     207.000 ns ± 257.70 %    1000000
    Program ended with exit code: 0

Now the substring benchmark has gotten about 10x slower from what it was before, but the unicode scalar benchmark hasn’t changed at all. That’s because the string API is now doing even more work to figure out how to remove the first character.

If we put in an even more complicated emoji the benchmarks will get even more dramatic:

    let string = String(repeating: "👨‍👨‍👧‍👧", count: 1_000_000)

    running Copying: String... done! (1331.52 ms)
    running Copying: Substring... done! (1308.10 ms)
    running Copying: UnicodeScalars... done! (310.02 ms)
    
    name                   time           std        iterations
    -----------------------------------------------------------
    Copying.String         8144004.000 ns ±   4.83 %        170
    Copying.Substring         9728.000 ns ±  39.64 %     126753
    Copying.UnicodeScalars     205.000 ns ± 243.12 %    1000000
    Program ended with exit code: 0

Now the substring benchmark is a whopping 30 times slower from when we just had a bunch of As. So clearly the `Substring` abstraction comes with some baggage. It provides a friendly interface to strings and unicode, but that comes at a price.

## Substring vs. UTF-8

It’s possible to drop down to an even lower abstraction than unicode scalars. Each unicode scalar is made up of multiple UTF8 code units, which you can get access to through the `UTF8View` on substrings:

    Array(acuteE1.utf8) 
    Array(acuteE2.utf8) 

This output is contrast to what we saw with unicode scalars:

    Array(acuteE1.unicodeScalars) 
    Array(acuteE2.unicodeScalars) 

Unicode scalars can be up to 21 bits in length, whereas UTF8 code units can only be 8 bits, and so it may take a lot more code units to express a single unicode scalar. For example, the flag emoji was only 2 unicode scalars, but it requires 8 code units:

    Array("🇺🇸".unicodeScalars)
    
    Array("🇺🇸".utf8)
    

And the family emoji is 25 is a whopping code units:

    "👩‍👩‍👧‍👧".utf8.count 

So this abstraction has gotten us even further away from “what you see is what you get” strings. Seemingly simple characters can have quite a large number of code units backing it.

And so while it seemed that the `UnicodeScalarView` was quite low level compared to `Substring`, it turns out it’s still doing quite a bit of work to shield us from the messy details of code units. For example, it does work to roll up a bunch of code units into a single scalar so that we can do things like check if a scalar equals something by simply doing:

    "🇺🇸".unicodeScalars.first == "🇺" 

This is made even nicer due to the fact that `UnicodeScalarView` is a collection whose element is `Unicode.Scalar`, and that conforms to `ExpressibleByUnicodeScalarLiteral`, which is what allows us to use this literal “🇺”.

If we wanted to do the same with `UTF8View` we would need to check that it starts with a sequence of explicit code units that represents the “🇺” character:

    "🇺🇸".utf8.starts(with: [240, 159, 135, 186])

Quite a bit more complicated! So the `UTF8View` has gotten us to an even lower abstraction level, and so then the question is are there any performance benefits that come with it? If `UnicodeScalarView` is doing work in order to abstract away code units from us so that we can instead focus on scalars, is there a chance it’s slower? Let’s add a new benchmark just for UTF8:
```swift
    suite.benchmark("UTF8") {
      var copy = string[...].utf8
      copy.removeFirst()
      var copy1 = copy
      copy1.removeFirst()
      var copy2 = copy1
      copy2.removeFirst()
      var copy3 = copy2
      copy3.removeFirst()
    }
```
    running Copying: String... done! (1323.69 ms)
    running Copying: Substring... done! (1302.53 ms)
    running Copying: UnicodeScalars... done! (296.98 ms)
    running Copying: UTF8... done! (91.69 ms)
    
    name                   time            std        iterations
    -----------------------------------------------------------
    Copying.String         10048176.000 ns ±  46.70 %        100
    Copying.Substring          9921.000 ns ±  48.31 %     104808
    Copying.UnicodeScalars      206.000 ns ± 291.37 %    1000000
    Copying.UTF8                 42.000 ns ± 656.14 %    1000000
    Program ended with exit code: 0

Wow! UTF8 is so fast that it’s just a hair over the speed of an empty benchmark. If we factor in those 30ish nanoseconds that means it’s 10 times faster than unicode scalars, which is 50 times faster than substring, which is 800 times faster than string.

## Next time: parser performance

So there really are quite substantial performance gains to be had by dropping to lower and lower abstractions levels. The biggest gain is just by using substring over string, but even using unicode scalars and UTF8 are big enough to consider using it if possible.

Now how do we apply what we’ve learned to parsers?

Well, so far all of our string parsers have been defined on `Substring`, and we’ve used lots of string APIs such as `removeFirst`, `prefix` and range subscripting. As we have just seen in very clear terms, these operations can be a little slow on `Substring` because of the extra work that must be done to properly handle traversing over grapheme clusters and normalized characters. The time differences may not seem huge, measured in just a few microseconds, but if you are parsing a multi-megabyte file that can really add up.

So, let’s see what kind of performance gains can be had by switching some of our parsers to work with UTF8 instead of `Substring`…next time!

----
- tags : #pointfree #parser