# Derived Behavior: The Problem

### [Introduction](https://www.pointfree.co/episodes/ep146-derived-behavior-the-problem#t5)

There’s a large complex problem that we all grapple with when making applications, but it’s not often addressed head on and given a systematic study. Our applications are built as large blobs of state, which represents the actual data of the application, and behavior to evolve that state over time. Now this big blob of data and behavior is extremely difficult to understand all at once. It’s of course all there, even if some parts are hidden away in little implicit chunks of state and behavior, but ideally we could cook up tools and techniques that allow us to understand only a small part of the application at once.

Doing this comes with huge benefits. Things like improved compile times, allowing yourself to build and run subsets of your application in isolation, strengthening the boundaries between components to make them more versatile, and more. We’ve definitely harped on these concepts over and over on Point-Free, but there’s still more to say.

We want to explore the heart of this problem from first principles. We’ll start with a vanilla SwiftUI application that has two separate screens, each with their own behavior and functionality, but they also need to share a little bit of functionality. Further, the parent screen that encapsulates the two children also wants to be able to observe the changes in each of the children.

This is a surprisingly subtle interaction to get right, especially in vanilla SwiftUI. The crux of the problem is that we want to be able to bundle up most or all of our application’s behavior into a single object, and then derive new objects from it that contain only a subset of the behavior. So, we could take the root application domain and derive smaller and smaller domains. For example, we take the app-level object and derive an object for just the home screen, and then further derive from that an object for the profile screen, and then further derive from that an object for the settings screen, all the while these derived objects will stay in sync with each other so that the changes in one are instantly reflected in the others.

Now, if you are a user of our Composable Architecture library this all probably sounds very similar to the concept of `Store`s and `.scope`s, and you’re right, but we want to use vanilla SwiftUI as a jumping off point to dig even deeper into those concepts.

### [A moderately complex SwiftUI application](https://www.pointfree.co/episodes/ep146-derived-behavior-the-problem#t148)

In order to understand why `.scope` is so powerful we need to first understand what it’s like to build SwiftUI applications in a world where `.scope` is not available.

So let’s start. We are going to build a simple, toy application. It will be a tab view application, where the first tab holds a simple counter feature with the ability to mark some numbers as our favorites, and then the second tab will show all those favorite numbers with the ability to remove that number from the favorites.

Nothing too complex, but it actually gets at the heart of quite a complex interaction that is difficult to correctly model in SwiftUI applications. What we have here is two domains of an application that operate independently of each other, yet somehow still communicate with one another. A mutation in one screen is instantly shown in the other. If I add a favorite number in the first tab, it instantly shows in the second tab. If I remove that favorite from the second tab, we instantly see the result of that in the first tab.

Let’s rebuild this application in vanilla SwiftUI so that we can understand why this toy application is already quite complex. We’ll start with the view layer, which can be done in a straightforward manner.

We’ll create a view that puts a `TabView` at the root with two views, one for each tab.

```
struct ContentView: View {
  var body: some View {
    TabView {
      Text("Counter")
        .tabItem { Text("Counter") }

      Text("Profile")
        .tabItem { Text("Profile") }
    }
  }
}
```

That already gets us a simple view on the screen if run the preview:

```
struct ContentView_Previews: PreviewProvider {
  static var previews: some View {
  ContentView()
  }
}
```

Now instead of simple `Text` views for each tab we’re going to want something with a fuller feature set. We could implement that view directly in line right inside `AppView`, but it’s far more customary to break out those things into their own standalone views.

We can start with the counter view, which just needs an `HStack` to put the minus button, count label and plus button next to each other, and then all of that wrapped in a `VStack` to put the “Save” button underneath:

```
struct VanillaCounterView: View {
  var body: some View {
    VStack {
      HStack {
        Button("-") { }
        Text("0")
        Button("+") { }
      }
      Button("Save") { }
    }
  }
}
```

We don’t currently have any data to populate the count text or any way of executing behavior in those button action closures, but we’ll get to that soon enough.

Next we could do the profile view, which is just a `List` wrapping a `ForEach` to display a bunch of numbers with a remove button to the right of the number:

```
struct VanillaProfileView: View {
  var body: some View {
    List {
      ForEach(1...10, id: \.self) { number in
        HStack {
          Text("\(number)")
          Spacer()
          Button("Remove") { }
        }
      }
    }
  }
}
```

With some basic view hierarchy in place we could start filling in some behavior for these screens. In SwiftUI one does this by creating a class to conform to `ObservableObject`. This gives you a place to hold onto state so that you can add behavior to that state and so that SwiftUI can observe all changes to the state:

```
class AppViewModel: ObservableObject {

}
```

The only state we care about in the application right now is the current count value and the set of favorites:

```
class AppViewModel: ObservableObject {
  @Published var count = 0
  @Published var favorites: Set<Int> = []
}
```

We’d like to have access to this view model in the counter view:

```
struct VanillaCounterView: View {
  @ObservedObject var viewModel: AppViewModel

  ...
}
```

But that also means the parent must hold onto it in order to pass the view model down to the child view:

```
struct VanillaContentView: View {
  @ObservedObject var viewModel: AppViewModel

  var body: some View {
    TabView {
      VanillaCounterView(viewModel: self.viewModel)

    ...
  }
}
```

And similarly for the profile view:

```
struct VanillaProfileView: View {
  @ObservedObject var viewModel: AppViewModel

  ...
}
```

And it can be passed this view model from the app view:

```
VanillaProfileView(viewModel: self.viewModel)
  .tabItem { Text("Profile") }
```

We just need to pass this view model to `VanillaContentView` in the preview and app entry point.

Now that all of our views have access to the view model we can start implementing behavior and everything should just magically work. For example, we can hook up the increment and decrement buttons in the counter view by just mutating the view model’s `count` field right in the action closure:

```
HStack {
  Button("-") { self.viewModel.count -= 1 }
  Text("\(self.viewModel.count)")
  Button("+") { self.viewModel.count += 1 }
}
```

We can further implement the save and remove functionality for the favorites by doing a little bit of conditional logic to check if the current count is in the favorites, and then determining which type of button should be displayed:

```
if self.viewModel.favorites.contains(self.viewModel.count) {
  Button("Remove") {
    self.viewModel.favorites.remove(self.viewModel.count)
  }
} else {
  Button("Save") {
    self.viewModel.favorites.insert(self.viewModel.count)
  }
}
```

Implementing the behavior of the profile is also quite straightforward:

```
struct VanillaProfileView: View {
  @ObservedObject var viewModel: AppViewModel

  var body: some View {
    List {
      ForEach(self.viewModel.favorites.sorted(), id: \.self) { number in
        HStack {
          Text("\(number)")
          Spacer()
          Button("Remove") {
            self.viewModel.favorites.remove(number)
          }
        }
      }
    }
  }
}
```

Now ideally we probably wouldn’t want to just reach into the view model and mutating it directly from the view because any logic in the view is very difficult test. Either requiring a broad testing tool such as snapshot testing, which is hard to test small, focused parts of your application, or you have to use some big machinery like UI testing, which can be very involved, slow and flakey.

Instead we should probably implement endpoints on the view model, defined as methods, that implement this logic, and then the view can just invoke those methods. That makes the view model testable outside the context of a SwiftUI view, and simplifies the logic of the view because it just needs to invoke a method. However, we’re not going to do that for the purposes of this demo, but we want to mention it.

But, that point aside, the application now fully works. We can add and subtract from the counter, we can add and remove numbers from our favorites, and we can view the favorites from the profile screen. We can even remove favorites from the profile and the counter screen will show that updated state.

Let’s add one more quick feature. Currently the `AppView` isn’t actually using the view model in any real way. It’s just holding onto it so that it can hand it off to the counter and profile screens. Let’s change that by showing the current count and number of favorites in the tab items:

```
var body: some View {
  TabView {
    VanillaCounterView(viewModel: self.viewModel)
      .tabItem { Text("Counter \(self.viewModel.count)") }

    VanillaProfileView(viewModel: self.viewModel)
      .tabItem { Text("Profile \(self.viewModel.favorites.count)") }
  }
}
```

### [Child-parent view model communication](https://www.pointfree.co/episodes/ep146-derived-behavior-the-problem#t848)

Ok, so this is pretty cool. This is obviously a toy application, but it is demonstrating something really powerful. It’s kind of hard to see because SwiftUI is doing so much work for us, but the fact that we have state instantly synchronizing state across two different screens, and really 3 screens if you consider the surrounding tab view a screen in its own right, is really amazing, and hard to do right if you don’t have the right tools.

But now we’re going to throw a wrench into all the niceness SwiftUI gives us. Although what we have built here will work perfectly fine for a simple application, it unfortunately is not long for this world. As soon as your application needs more screens, more functionality and more communication between screens the pattern of putting all your application’s logic into a single gigantic object that is passed around everywhere becomes completely untenable.

Instead what we’d like to be able to do is split up the `AppViewModel` into some logically smaller pieces that can operate in isolation without knowing too much about the other domains while still having the ability to communicate with each other. That may sound a little weird, but it comes with tons of benefits as we mentioned before:

-   You can build, run and test subsets of your application in isolation without needing to understand the entire application at once.

-   If you further split those subsets of the application into their own module you further strengthen the boundary between components, which makes them more independent, more versatile, easier to refactor, and it becomes more clear when a component is doing things it shouldn’t be, such as trying to reach out to global state.

-   And once you accomplish a bit of modularization you will see speed up in compile times because now the Swift compiler can be smarter in how it parallelizes building your application.

And that’s just barely scratching the surface. There are tons of benefits. And every framework in every language tries to solve this problem. Whether you are using Swift and SwiftUI, or JavaScript and React, or Kotlin and Android, everyone wants to break down their applications behavior into sub-objects that can be understood in isolation and pieced back together.

So, let’s try refactoring our application to split up its responsibilities. We’d love if each screen could be powered off of its own view model, and then somehow have them piece together to form the full application.

We’ll begin with the counter view. Let’s introduce a new view model that is only concerned with the domain of the counter:

```
class CounterViewModel: ObservableObject {
  @Published var count = 0
  @Published var favorites: Set<Int> = []
}
```

And we’ll swap out the `AppViewModel` for the new `CounterViewModel`:

```
struct VanillaCounterView: View {
  @ObservedObject var viewModel: CounterViewModel

  ...
}
```

Amazingly everything in this view continues to compile just fine, the only error is up in the `VanillaContentView` because we aren’t passing the right type of view model when constructing the counter view. We’ll get to that in a moment though.

Next let’s do the same for the profile view:

```
class ProfileViewModel: ObservableObject {
  @Published var favorites: Set<Int> = []
}

struct VanillaProfileView: View {
  @ObservedObject var viewModel: ProfileViewModel

  ...
}
```

And again this view compiles, we just have an error up in the app view. So let’s look at that now.

Perhaps the simplest thing we could do is just to hold onto both view models in the `VanillaContentView`:

```
struct VanillaContentView: View {

  @ObservedObject var counterViewModel: CounterViewModel
  @ObservedObject var profileViewModel: ProfileViewModel

  var body: some View {
    TabView {
      VanillaCounterView(viewModel: self.counterViewModel)
        .tabItem { Text("Counter \(self.counterViewModel.count)") }

      VanillaProfileView(viewModel: self.profileViewModel)
        .tabItem { Text("Profile \(self.profileViewModel.favorites.count)") }
    }
  }
}
```

And now this part of the code is compiling, but our SwiftUI preview and app entry point is having problems because we now need to provide both view models there, as well:

```
  VanillaContentView(

    counterViewModel: .init(),
    profileViewModel: .init()
  )
}
```

And now everything compiles, but does it work?

If we run the preview we will see that the counter seems to work, and the the tab item is updating, but if we save some numbers the second tab’s UI doesn’t update at all, nor does it’s tab item.

This shouldn’t be too surprising because we now have two fully separate, independent view models. There’s no coordination between them, and so they are operating on fully distinct pieces of state.

However, even though it doesn’t currently work we have at least achieved some isolation between the screens. If we wanted to we could even move `CounterView` and `CounterViewModel` into their own module, and `ProfileView` and `ProfileViewModel` could go into their own module, all without a single dependency between them. This means we could start building, running and testing them in full isolation without having to worry about all of the state and responsibilities of the full application.

With that said, we need to do a bit of extra work to integrate these two view models so that certain changes in one will be instantly reflected in the other. A place we could attempt this is back in the `AppViewModel`, which currently isn’t being used at all. What if that view model held references to the other two view models?

One way to do this would be to hold onto some `@Published` properties:

```
class AppViewModel: ObservableObject {
  @Published var counter = CounterViewModel()
  @Published var profile = ProfileViewModel()
}
```

However, this isn’t going to work as we would expect. To see why, let’s start using this single view model in our `VanillaContentView` rather than holding onto two view models:

```
struct VanillaContentView: View {
  @ObservedObject var viewModel: AppViewModel

  var body: some View {
    TabView {
      VanillaCounterView(viewModel: self.viewModel.counter)
        .tabItem { Text("Counter \(self.viewModel.counter.count)") }

      VanillaProfileView(viewModel: self.viewModel.profile)
        .tabItem { Text("Profile \(self.viewModel.profile.favorites.count)") }
    }
  }
}
```

And let’s fix the SwiftUI preview and app entry point:

```
VanillaContentView(
  viewModel: .init()
)
```

If we run this in the preview then we will see that although the counter screen seems to work correctly, the tab item count is no longer updating.

The reason this is happening is because `@Published` only captures changes made to the property that are visible to the `willSet` observer, and because `counter` and `profile` are classes the `willSet` is not triggered on changes inside those objects, but rather only if you wholesale replace the entire object. Only value types get the benefits of having `willSet` triggered on any change on the inside of the value.

So, the changes being made inside the child view models are not causing the parent view model to update its view. And if that’s the case, is there any point to marking these fields as `@Published`? Seems more honest to just use `let`s, and we should probably also provide an initializer now:

```
class AppViewModel: ObservableObject {
  let counter: CounterViewModel
  let profile: ProfileViewModel

  init(
    counter: CounterViewModel = .init(),
    profile: ProfileViewModel = .init()
  ) {
    self.counter = counter
    self.profile = profile
  }
}
```

So, this is more honest, but still doesn’t fix any problems for us.

What we need is someway of telling the `VanillaAppView` that the view model has some changes whenever one of the child view models changes. We can actually directly observe whenever a child view model changes from the outside because by virtue of the fact that it is an `ObservableObject` it has an `objectWillChange` property that is a publisher. This is a requirement of the protocol, but we don’t often have to interact with it directly because the `@Published` property wrapper completely hides that detail from us, but there are situations we need to take a peek behind the curtains and interact with it directly.

All we need to do is `.sink` on the child’s `objectWillChange` publisher to be notified when one of its pieces of state is about to change, and then we can ping the `AppViewModel`’s `objectWillChange`, which will then notify SwiftUI:

```
self.counter.objectWillChange
  .sink { self.objectWillChange.send() }
```

> ⚠️ Result of call to ‘sink(receiveValue:)’ is unused

However, `.sink` returns a value, and we’re getting a warning about that now. It returns a cancellable, which we have to hold onto in our view model, otherwise this subscription will be immediately killed and we will never be notified of any updates.

So, let’s add a set of cancellables to the `AppViewModel` for storing these:

```
class AppViewModel: ObservableObject {
  var cancellables: Set<AnyCancellable> = []
  ...
}
```

And then we can store the subscription to `objectWillChange` in that set:

```
self.counter.objectWillChange
  .sink { self.objectWillChange.send() }
  .store(in: &self.cancellables)
```

Now when we run the preview the count on the tab item will update as we increment and decrement the count in the first tab.

But we’ve also accidentally introduced a retain cycle. We have to be careful now that we are manually sinking on publishers and managing cancellable. We can avoid this cycle by capturing `self` weakly in the `.sink`:

```
self.counter.objectWillChange
  .sink { [weak self] self?.objectWillChange.send() }
  .store(in: &self.cancellables)
```

However, we’ve only captured half the story with respect to getting the parent to update when one of the children updates. If we save some favorite numbers from the first tab we will see that the tab item for the profile doesn’t update at all. It seems that we still aren’t propagating some changes to the parent `AppViewModel`. Maybe what we need to do is also subscribe to the profile’s `objectWillChange` in order to notify the `AppViewModel` of changes.

And this is because we aren’t propagating the changes from the profile to the parent `AppViewModel`. To do this we can basically copy and paste what we did for the counter with a few small changes:

```
self.profile.objectWillChange
  .sink { [weak self] self?.objectWillChange.send() }
  .store(in: &self.cancellables)
```

### [Parent-child view model communication](https://www.pointfree.co/episodes/ep146-derived-behavior-the-problem#t1556)

Huh, it’s still not working. What gives?

Well, although we now properly have the child view models telling the parent when they update, we still don’t have anything that is synchronizing the changes between sibling view models.

And this brings us to the most complicated part of integrating multiple observable objects together.

We now need to be able to observe the changes to one particular piece of state in each view model, and then replay those changes in the other view model. In this case the state in question is the array of favorite numbers.

Perhaps the easiest way to do this is to just `.sink` on the `$favorites` publisher that comes for free by virtue of the fact that we are using `@Published` in our view models:

```
self.counter.$favorites
  .sink {
  }
```

Inside this sync we want to replay this change to the `profile` view model:

```
self.counter.$favorites
  .sink {
    self.profile.favorites = $0
  }
```

But this creates a retain cycle because we are capturing `self` strongly in a long-living publisher, so let’s weakify:

```
self.counter.$favorites
  .sink { [weak self] in
    self?.profile.favorites = $0
  }
```

And we have to remember to store the cancellable:

```
self.counter.$favorites
  .sink { [weak self] in
    self?.profile.favorites = $0
  }
  .store(in: &self.cancellables)
```

And with this change we will now see that saving favorite numbers from the first tab causes the number in the tab item to change. And if we even switch to the second tab we will see all of our favorites sitting there. So, it seems like a success! We can even shorten this bit of code because we can use the `.assign` operator on publishers to pipe changes into `@Published` fields:

```
self.counter.$favorites
  .assign(to: &self.profile.$favorites)



```

Unfortunately this is only half the story. If we were to remove some of the favorites from the second tab we will see that the first tab somehow missed out on all the changes. The first tab will continue to say those numbers are in the favorites.

And this is because we need to further subscribe to the `$favorites` field of the `profile` view model and replay those changes to the `counter` view model:

```
self.profile.$favorites
  .assign(to: &self.counter.$favorites)
```

But now we have a crash in our preview:

```
CrashError: Scope crashed

Scope crashed. Check ~/Library/Logs/DiagnosticReports for crash logs from your application.

==================================

|  RemoteHumanReadableError: Failed to update preview.
|
|  The preview process appears to have crashed.
|
|  Error encountered when sending 'previewInstances' message to agent.
```

This is happening because we suddenly have an infinite loop on our hands. When the counter’s favorites change we replay it to the profile, but then when the profile’s favorites change we replay it to the counter, and so on and so on. We need a way to break this cycle.

Perhaps the easiest way to break the retain cycle is to simply tack on a couple of `.removeDuplicates()` calls to the published properties:

```
self.counter.$favorites
  .removeDuplicates()
  .assign(to: &self.profile.$favorites)

self.profile.$favorites
  .removeDuplicates()
  .assign(to: &self.counter.$favorites)
```

This will make it so that when we enter an echo chamber of replays happening back and forth we will cut them off since those replays will be duplicates of previous changes.

If we run the preview again we will see that everything seems to work, which is great, but this solution isn’t ideal even though it is short and succinct. One problem is that we are doing extra work to break this cycle. Not only do we need to do `.removeDuplicates()`, which incurs the cost of keeping around a copy of our data and an equality check, but we are sending more data into these publishers than necessary. We shouldn’t need to send extra data into the publisher just so that it can be filtered out by `.removeDuplicates()`. We should be able to stop that process a little earlier. Further, this technique also requires the state we are observing to be `Equatable`, which may not be possible in practice.

An alternative approach to breaking the infinite cycle is to keep track of when we are in the middle of updating the counter state, and in that case we will short circuit replaying profile changes back to the counter.

We can do this by keeping track of a little mutable boolean that is true during the duration of mutating the `profile` view model:

```
var profileIsUpdating = false

self.counter.$favorites
  .sink { [weak self] in
    profileIsUpdating = true
    defer { profileIsUpdating = false }
    self?.profile.favorites = $0
  }
  .store(in: &self.cancellables)
```

And then before performing the counter mutation we can first make sure we are not already in the middle of performing a profile mutation because then we can skip this:

```
self.profile.$favorites
  .sink { [weak self] in
    guard !profileIsUpdating else { return }
    self?.counter.favorites = $0
  }
  .store(in: &self.cancellables)
```

But of course what we do for one we have to do for the other. We need another boolean to track when we are performing a counter mutation:

```
var counterIsUpdating = false
```

We’ll guard against then when we get a counter update:

```
self.counter.$favorites
  .sink { [weak self] in
    guard !counterIsUpdating else { return }
    profileIsUpdating = true
    defer { profileIsUpdating = false }
    self?.profile.favorites = $0
  }
  .store(in: &self.cancellables)
```

And we’ll update that boolean when we get a profile update:

```
self.profile.$favorites
  .sink { [weak self] in
    guard !profileIsUpdating else { return }
    counterIsUpdating = true
    defer { counterIsUpdating = false }
    self?.counter.favorites = $0
  }
  .store(in: &self.cancellables)
```

If we run the preview now we will see that everything works as we expect. When we make changes in one tab it is instantly reflected in the other, and the info in the tabs updates instantly too. Further, we are doing a lot less work than the `.removeDuplicates()` technique. We aren’t sending unnecessary data through publishers just to remove dupes, and we aren’t incurring potentially costly equability checks.

So this is great, but also the code is really gnarly:

```
init(
  counter: CounterViewModel = .init(),
  profile: ProfileViewModel = .init()
) {
  self.counter = counter
  self.profile = profile

  self.counter.objectWillChange
    .sink { self.objectWillChange.send() }
    .store(in: &self.cancellables)
  self.profile.objectWillChange
    .sink { self.objectWillChange.send() }
    .store(in: &self.cancellables)

  var counterIsUpdating = false
  var profileIsUpdating = false

  self.counter.$favorites
    .sink { [weak self] in
      guard !counterIsUpdating else { return }
      profileIsUpdating = true
      defer { profileIsUpdating = false }
      self?.profile.favorites = $0
    }
    .store(in: &self.cancellables)

  self.profile.$favorites
    .sink { [weak self] in
      guard !profileIsUpdating else { return }
      counterIsUpdating = true
      defer { counterIsUpdating = false }
      self?.counter.favorites = $0
    }
    .store(in: &self.cancellables)
}
```

All of these pieces are unfortunately necessary. None can be omitted. We gotta listen to the `objectWillChange` on each child view model to ping the `AppViewModel`’s `objectWillChange`, and we also have to subscribe to changes to state in each view model so that we can replay those changes in the other view model. If we leave out any of these pieces we will have a half functional `AppViewModel`.

This is a _ton_ of code that didn’t exist when we had a single, gigantic `AppViewModel`. It’s kind of painful to implement, with lots of sharp edges, so it seems likely that you’ll skip this kind of thing altogether and instead just pile more and more into your `AppViewModel`.

There is another tool that SwiftUI gives us to “derive” sub-behavior from an `ObservableObject`, and that is from the fact that observable objects can derive `Binding`s from `@Published` fields.

```
self.$viewModel.count 
```

You could hand these kinds of bindings to other views and objects, which can make mutations to it and be instantly notified when it is mutated elsewhere.

However, this is a very small subset of behavior: it is just state mutations. You can’t take a binding and make API requests or request a user’s location. That kind of behavior needs to live in a view model, since it is a class, which has state and reference semantics.

Further, anywhere you hand a `Binding` off, you are giving unfettered access to mutate this state, which is not ideal.

So, this is the current state of deriving behavior in vanilla SwiftUI applications. If you have a view model that holds a bunch of state and behavior and you want to derive little child view models that contain only a subset of that state and behavior, then you have a bit of manual work ahead of you.

-   First you need to make sure that you ping the parent’s `objectWillChange` publisher whenever any of the children’s state changes.

-   And then you need to observe any state that should be shared so that you can replay those state changes to each child.

Further, as you perform this manual work you have to be extra careful because you are dealing with reference types. This means you run the risk of creating retain cycles and silently mutating objects from a distance which can introduce a lot of complexity to your code base. And that’s only scratching the surface. There are a lot more problems that need to be solved, but we wanted to give everyone just a small taste of what it looks like in vanilla SwiftUI.

And even if you solve all of those problems you still have more work ahead of you. Right now the notification mechanism for telling the parent view model when a child view model changes is very crude. We have no idea what state changed in the child, just that something changed. This means we are going to re-render the parent every time something changes in the child, even if the parent just wants to observe one small piece of state. This is quickly going to lead to the parent view being rendered way more times than it should be, and that can cause performance problems.

We can fix this by observing changes directly on the fields of the observable objects rather than just observing `objectWillChange`. That will allow us to make sure the parent re-renders only when properties it actually cares change. However, this adds additional complexity because every time we want to to use a new piece of state from a child in the parent view you have to remember to start observing that state so that you can ping the parent’s `objectWillChange`.

Now it’s worth mentioning that the techniques we have outlined are just the best we have come up with so far, but there could be better ones out there. We’ve seen some people use singletons for coordinating across many view models, and although that sounds scary it’s certainly worth trying to see if it can be made reasonable. Also, Apple’s WWDC event is happening soon, and so maybe soon we’ll get an official story from Apple on how to handle child view models.

### [Next time: the Composable Architecture](https://www.pointfree.co/episodes/ep146-derived-behavior-the-problem#t2358)

But until the solution is handed to us from on high, we actually already have a really robust solution to this problem…that is, if you’re using the Composable Architecture.

One of the most fundamental concepts in the Composable Architecture is that of a `Store`. It is the runtime that actually powers your application, and it kinda serves a similar purpose as a view model. It is created with the initial state your application starts in, a reducer that implements your application’s logic, and an environment of dependencies that are needed for your application to do its job.

It is possible, and even encouraged, that your application start with one single store at the root of your application. It will hold your entire application’s state and logic all in one cohesive package. That may sound scary at first, but it also unlocks some wonderful abilities and super powers once your application is built off a single source of truth.

However, having a single root store for the entire application can become quite unwieldy. We certainly don’t want to pass around this gigantic object all over the place to any feature that needs access to state or needs to send user actions. That would give each feature access to everything in the application, even if just needs access to a few small things.

Sounds like we need some kind of operator that allows us to derive child stores from an existing store, just like we attempted to do with view models. Well, luckily for us the Composable Architecture ships with such an operator: `.scope`. Scope is the fundamental operation on `Store` that allows you to transform a store that runs a parent domain’s logic into a store that runs a child domain’s logic. So we can take that gigantic root store and scope it to smaller and smaller domains. For example, we could take the app-level store and scope it down to the home screen store, and then scope that down to the store for the profile screen, and then scope that down to the store for the settings screen.

This is an incredibly important concept for understanding the Composable Architecture, but we feel we haven’t spent enough time on the topic. We introduced the concept of scoping in some of our earliest episodes when we were first uncovering the Composable Architecture, and back then we even called it a different name, but we didn’t really dive deep into it. So, we want to spend a little more time with `.scope` and make sure that everyone knows how to wield it correctly, and along the way we will discover some potential performance problems with scope, and then fix them 😅.

Let’s start by rebuilding the application we just explored, but this time using the Composable Architecture…next time!    