# Derived Behavior: The Problem

## Introduction

There’s a large complex problem that we all grapple with when making applications, but it’s not often addressed head on and given a systematic study. Our applications are built as large blobs of state, which represents the actual data of the application, and behavior to evolve that state over time. Now this big blob of data and behavior is extremely difficult to understand all at once. It’s of course all there, even if some parts are hidden away in little implicit chunks of state and behavior, but ideally we could cook up tools and techniques that allow us to understand only a small part of the application at once.

Doing this comes with huge benefits. Things like improved compile times, allowing yourself to build and run subsets of your application in isolation, strengthening the boundaries between components to make them more versatile, and more. We’ve definitely harped on these concepts over and over on Point-Free, but there’s still more to say.

We want to explore the heart of this problem from first principles. We’ll start with a vanilla SwiftUI application that has two separate screens, each with their own behavior and functionality, but they also need to share a little bit of functionality. Further, the parent screen that encapsulates the two children also wants to be able to observe the changes in each of the children.

This is a surprisingly subtle interaction to get right, especially in vanilla SwiftUI. The crux of the problem is that we want to be able to bundle up most or all of our application’s behavior into a single object, and then derive new objects from it that contain only a subset of the behavior. So, we could take the root application domain and derive smaller and smaller domains. For example, we take the app-level object and derive an object for just the home screen, and then further derive from that an object for the profile screen, and then further derive from that an object for the settings screen, all the while these derived objects will stay in sync with each other so that the changes in one are instantly reflected in the others.

Now, if you are a user of our Composable Architecture library this all probably sounds very similar to the concept of `Store`s and `.scope`s, and you’re right, but we want to use vanilla SwiftUI as a jumping off point to dig even deeper into those concepts.



##  UI 구현 

```swift
      

//

// ContentView.swift

// derivedBehavior

//

// Created by kwanghyun won on 2021/06/11.

//

  

import SwiftUI

  

struct ContentView: View {

 var body: some View {

 TabView {

 CountView()

 .tabItem { Text("Counter") }

 ProfileView()

 .tabItem { Text("Profile") }

 }

 }

}

  

struct CountView: View {

 var body: some View {

 VStack {

 HStack {

 Button("-") {

 }

 Text("0")

 Button("+") {

 }

 }

 Button("Save") {

 }

 }

 }

}

  

struct ProfileView: View {

 var body: some View {

 List {

 ForEach(1...10, id: \.self) { number in

 HStack{

 Text("\(number)")

 Spacer()

 Button("Removed") {

 }

 }

 }

 }

 }

}

  

struct ContentView_Previews: PreviewProvider {

  static var previews: some View {

    ContentView()

 }
}
```

## viewModel 구현
- AppViewModel 로 구현


```swift
import SwiftUI

class AppViewModel: ObservableObject {
  @Published var count = 0
  @Published var favorited: Set<Int> = []
}

struct ContentView: View {
  @ObservedObject var viewModel: AppViewModel
    var body: some View {
      TabView {
        CountView(viewModel: viewModel)
          .tabItem { Text("Profile \(self.viewModel.count)")  }
        ProfileView(viewModel: viewModel)
          .tabItem { Text("Profile \(self.viewModel.favorited.count)")  }
      }
    }
}

struct CountView: View {
  @ObservedObject var viewModel: AppViewModel
  
  var body: some View {
    VStack {
      HStack {
        Button("-") {
          self.viewModel.count -= 1
        }
        Text("\(self.viewModel.count)")
        Button("+") {
          self.viewModel.count += 1
        }
      }
      
      if self.viewModel.favorited.contains(self.viewModel.count) {
        Button("Remove") {
          self.viewModel.favorited.remove(self.viewModel.count)
        }
      } else {
        Button("Save") {
          self.viewModel.favorited.insert(self.viewModel.count)
        }
      }
    }
  }
}

struct ProfileView: View {
  @ObservedObject var viewModel: AppViewModel
  
  var body: some View {
    List {
      ForEach(self.viewModel.favorited.sorted(), id: \.self) { number in
        HStack{
          Text("\(number)")
          Spacer()
          Button("Removed") {
            self.viewModel.favorited.remove(number)
          }
        }
      }
    }
  }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
      ContentView(viewModel: .init())
    }
}

```


### Child-parent view model communication
- `CountViewModel`, `ProfileViewMode`로 분리 

```swift
import SwiftUI

class AppViewModel: ObservableObject {
  @Published var count = 0
  @Published var favorited: Set<Int> = []
}


struct ContentView: View {
  @ObservedObject var counterViewModel: CounterViewModel
  @ObservedObject var profileViewModel: ProfileViewModel
    var body: some View {
      TabView {
        CountView(viewModel: counterViewModel)
          .tabItem { Text("Profile \(self.counterViewModel.count)")  }
        ProfileView(viewModel: profileViewModel)
          .tabItem { Text("Profile \(self.profileViewModel.favorited.count)")  }
      }
    }
}

class CounterViewModel: ObservableObject {
  @Published var count = 0
  @Published var favorited: Set<Int> = []
}

struct CountView: View {
  @ObservedObject var viewModel: CounterViewModel
  
  var body: some View {
    VStack {
      HStack {
        Button("-") {
          self.viewModel.count -= 1
        }
        Text("\(self.viewModel.count)")
        Button("+") {
          self.viewModel.count += 1
        }
      }
      
      if self.viewModel.favorited.contains(self.viewModel.count) {
        Button("Remove") {
          self.viewModel.favorited.remove(self.viewModel.count)
        }
      } else {
        Button("Save") {
          self.viewModel.favorited.insert(self.viewModel.count)
        }
      }
    }
  }
}

class ProfileViewModel: ObservableObject {
  @Published var count = 0
  @Published var favorited: Set<Int> = []
}

struct ProfileView: View {
  @ObservedObject var viewModel: ProfileViewModel
  
  var body: some View {
    List {
      ForEach(self.viewModel.favorited.sorted(), id: \.self) { number in
        HStack{
          Text("\(number)")
          Spacer()
          Button("Removed") {
            self.viewModel.favorited.remove(number)
          }
        }
      }
    }
  }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
      ContentView(
        counterViewModel: .init(),
        profileViewModel: .init()
      )
    }
}
elf) { number in
        HStack{
          Text("\(number)")
          Spacer()
          Button("Removed") {
            self.viewModel.favorited.remove(number)
          }
        }
      }
    }
  }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
      ContentView(
        counterViewModel: .init(),
        profileViewModel: .init()
      )
    }
}

```

- `Profile` tab이 정상적으로 동작하지 않는다. 
    - 당연히 CountViewModel과 ProfileViewModel 이 전혀 연결되어 있지 않은 viewModel이므로 당연한 결과 



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

### Parent-child view model communication

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

## Reference
- [@StateObject and @ObservedObject in SwiftUI | Matt Moriarity](https://www.mattmoriarity.com/2020-07-03-stateobject-and-observableobject-in-swiftui/)
- [https://developer.apple.com/videos/play/wwdc2020/10040/](https://developer.apple.com/videos/play/wwdc2020/10040/)
- [https://rhonabwy.com/2021/02/13/nested-observable-objects-in-swiftui/](https://rhonabwy.com/2021/02/13/nested-observable-objects-in-swiftui/)
- [https://twitter.com/Oh_Its_Daniel/status/1277187721304342529](https://twitter.com/Oh_Its_Daniel/status/1277187721304342529)

---
tags: #swiftui #tca_archtecture 