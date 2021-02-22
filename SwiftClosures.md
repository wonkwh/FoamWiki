
According to Apple:

> _클로저 (Closures)_ 는 코드에서 사용도 하고 전달할 수도 있는 독립된 ‘기능 블럭 (blocks of functionality)’ 입니다.

A closure is just a block of code that can be assigned to a variable. 
This variable can be used just like any other variable. 
E.g. you can pass that to a function and the function executes the closure.

Let’s write our first closure to demonstrate how simple it really is:

```swift
let greet = {  
    print("Hello!")  
}
greet() // prints "Hello!"
```

Let’s break it down:

*   We create a constant `greet` in which we _assign a closure_. The _closure_ is all the code after the equals sign.
*   We call `greet` in a similar fashion as if it was a function.

By the way, the type of closure above is `() -> ()`. This means that the closure takes in no parameters and returns nothing (as it just prints `"Hello!"`).

Now, let’s slightly modify the implementation of `greet`. We are going to add a name parameter so that we can greet a person with their name:

```swift
let greet: (String) -> () = { name **in**  
    print("Hello, \\(name)!")  
}
greet("Nick") // prints "Hello, Nick!"
```
Let’s inspect this slightly more complex closure:

*   The closure accepts a parameter. Thus, its type needs to be specified. In this case, it takes a name string and returns nothing, thus the type `(String) -> ()`.
*   There is a `name` parameter within the closure. It is the name that we pass to the closure when we greet someone. Notice that it can be named however you like.

Let’s create a closure that compares two integers by checking if the second one is greater than the first one:

```swift
let compare: (Int, Int) -> Bool = {  
    (n1, n2) **in**   
    return n1 < n2  
}
compare(1,4) // returns **true**
```
Next, let’s sort an array of integers with the help of this closure by passing it into a sorting function. First, let’s create an array of integers to sort:

```swift
**let** nums = \[10, 2, 3, 54, 3, 2, 1030\]
```
In Swift, an array has a sorting method called `sort(by:)`. The `by:` parameter passed to this function is a closure. This closure acts as a sorting criterion. In this case, it compares two integers and sorts the array based on the result of the comparisons.


Let’s pass our `compare` closure into the `sort(by:)` function to sort the list from smallest to largest:
```swift
let sortedNums = nums.sorted(by: compare)print(sortedNums) 
// prints sorted list \[2, 2, 3, 3, 10, 54, 1030\]
```
Now we know what closures are and how they work + how they can be passed to a function to e.g. sort a list of integers. Next, we are going to take a look at completion handlers that are closures in action.

Probably the most common use-case for closures in Swift is completion handlers.

Say you perform a lengthy process, such as an HTTP request. You want to do something _immediately_ after the request completes. But you do _not_ want to waste resources in checking multiple times if the process is still going on or not.

This is when completion handlers come in. A completion handler is a closure that _calls back_ as soon as the lengthy process completes.

As an example, when making a networking request to a web page, the response _may_ contain data, an URL response, and an error.

Let’s create a closure that handles this kind of response and prints the HTML data of the requested page:
```swift
**let** handler: (Data?, URLResponse?, Error?) -> () = { (data, response, error) **in**  
    **guard** **let** data = data **else** { **return** }  
    print(String(data: data, encoding: .utf8)!)  
}
```
The closure above accepts three optional parameters, data, a response, and an error, and returns nothing. (The parameters are optional as each may be `nil` due to possible issues with the request.)

Let’s now create an URL object to StackOverflow’s main page:

```swift
let url = URL(string: "http://www.stackoverflow.com")!
```

Next, we create the network task to request StackOverflow’s contents. The `dataTask` method takes in a target URL _and a completion handler,_ that is, a _closure_ that is executed when the network request completes.

This _completion handler_ needs to be a _closure_ that is of type `(Data?, URLResponse?, Error?) -> ()` as described above. This is exactly what we just created, so let’s just pass it into the `dataTask` method:

```swift
**let** task = URLSession.shared.dataTask(with: url, completionHandler: handler)task.resume()
```
As a result, StackOverflow page’s HTML content is printed in the console.

And that’s it. Now you know how completion handlers use closures to call back after a lengthy process completes.

One more thing about writing closures:
--------------------------------------

You do not actually need to declare closures separately as I’ve done in the above examples.

To illustrate this, let’s rewrite the network request example so that we give the closure directly into the `dataTask` function without declaring it separately:

```swift
let task = URLSession.shared.dataTask(with: url) { (data, response, error) **in**  
    guard let data = data else { **return** }  
    print(String(data: data, encoding: .utf8)!)  
}
```
This is a perfectly valid, and actually even more common way to pass a closure into a function in Swift.

A closure is a block of code assigned to a variable. This variable can be used just like other variables to e.g. pass it to a function.

The common use-case for closures in Swift is completion handlers. A completion handler is a block of code executed after a lengthy action gets completed. This is handy as you do not need to worry about checking if the action is completed or not.

References
----------
- [Swift—Closures Made Very Simple | CodeX (medium.com)](https://medium.com/codex/swift-closures-made-simple-cb81dd15b543)
- [Swift 5.3: Closures (클로저; 잠금 블럭) (xho95.github.io)](http://xho95.github.io/swift/language/grammar/closure/2020/03/03/Closures.html)