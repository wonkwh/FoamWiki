# Derived Behavior: Composable Architecture

## Introduction
[[Derived BehaviorPart1]]에서 구현했던 예제를 Composable Architecture 를 이용해서 재구현해보자 


One of the most fundamental concepts in the Composable Architecture is that of a `Store`. It is the runtime that actually powers your application, and it kinda serves a similar purpose as a view model. It is created with the initial state your application starts in, a reducer that implements your application’s logic, and an environment of dependencies that are needed for your application to do its job.

It is possible, and even encouraged, that your application start with one single store at the root of your application. It will hold your entire application’s state and logic all in one cohesive package. That may sound scary at first, but it also unlocks some wonderful abilities and super powers once your application is built off a single source of truth.

However, having a single root store for the entire application can become quite unwieldy. We certainly don’t want to pass around this gigantic object all over the place to any feature that needs access to state or needs to send user actions. That would give each feature access to everything in the application, even if just needs access to a few small things.

Sounds like we need some kind of operator that allows us to derive child stores from an existing store, just like we attempted to do with view models. 

Well, luckily for us the Composable Architecture ships with such an operator: `.scope`. Scope is the fundamental operation on `Store` that allows you to transform a store that runs a parent domain’s logic into a store that runs a child domain’s logic. 

So we can take that gigantic root store and scope it to smaller and smaller domains. 
For example, we could take the app-level store and scope it down to the home screen store, and then scope that down to the store for the profile screen, and then scope that down to the store for the settings screen.

This is an incredibly important concept for understanding the Composable Architecture, but we feel we haven’t spent enough time on the topic. 


## Rebuilding in the Composable Architecture

- To build a feature in the Composable Architecture you start with some domain modeling for the state, actions and environment needed to run our application.
-  The state can just hold the current count and set of favorites:

```
struct AppState: Equatable {
  var count = 0
  var favorites: Set<Int> = []
}
```

First thing we’ll notice is that we get to use simple value types! We no longer need to trap our domain inside a reference type just so that we can conform to `ObservableObject` and use the `@Published` property wrapper. This already means that our domain is quite a bit simpler than the vanilla SwiftUI application just by virtue of the fact that value types are so much simpler than reference types.

Next we can model the user actions that can happen in the application. There’s the obvious actions, such as tapping the increment, decrement, save or remove buttons:

```
enum AppAction {
  case incrementButtonTapped
  case decrementButtonTapped
  case saveButtonTapped
  case removeButtonTapped
}
```

But there’s also a user action of removing a favorite from the profile view. Now it may seem like we could just leverage the `.removeButtonTapped` action for this, but removing a favorite from the profile is a little different from removing a favorite on the counter screen. On the counter screen you save or remove the current count from the set of favorites, whereas on the profile you select a specific number to remove from the favorites. For this reason we need to model it as a separate action:

```
case profileRemoveButtonTapped(Int)
```

Next we would typically model the environment for the feature, which would be a struct that holds all of the dependencies the application needs to do its job. This would include things like API clients, location managers, databases, and more. But our feature is very simple right now and doesn’t require any dependencies, so we can just use an empty struct:

```
struct AppEnvironment {
}
```

With the domain defined we can implement a reducer that glues everything up. It will handle each incoming action to figure out how it should mutate state and what effects should be executed. The logic in this reducer will look similar to what we did in the view layer of the vanilla SwiftUI application:

```
let appReducer = Reducer<AppState, AppAction, AppEnvironment> { state, action, _ in
  switch action {
  case .incrementButtonTapped:
    state.count += 1
    return .none
  case .decrementButtonTapped:
    state.count -= 1
    return .none
  case .saveButtonTapped:
    state.favorites.insert(state.count)
    return .none
  case .removeButtonTapped:
    state.favorites.remove(state.count)
    return .none
  case let .profileRemoveButtonTapped(number):
    state.favorites.remove(number)
    return .none
  }
}
```

It’s a little more verbose than what we did in the vanilla SwiftUI version, but because it’s out of the view layer it is now 100% testable. And even better, once our application gets more complex by involving side effects you will instantly be able to test that behavior too.

With the domain and reducer defined we can start building out the view layer. The view will hold onto what is known as a `Store`, which is the runtime that can actually power a application. However, we only need to hold onto it as a `let`, not as an `@ObservedObject`, and this is because it’s only an encapsulation of the runtime of your application, but it does not actually observe any state changes or allow you to send actions to it:

```
struct TcaContentView: View {
  let store: Store<AppState, AppAction>

  var body: some View {
  }
}
```

The Composable Architecture draws a line between the runtime and observation of the runtime, and so the way you can actually get access to state and actions in the store is via what is known as a `ViewStore`. The simplest way to create one of these in SwiftUI is via the `WithViewStore` helper:

```
struct TcaContentView: View {
  let store: Store<AppState, AppAction>

  var body: some View {
    WithViewStore(self.store) { viewStore in

    }
  }
}
```

`WithViewStore` needs to know when a store’s state changes, and by default uses equality to do so, so we need to conform `AppState` to be equatable.

```
extension AppState: Equatable {
  ...
}
```

This observes everything happening inside the store and allows you to send actions into the system. For example, we can now implement the `TabView` inside here quite easily because we have access to the application’s `count` and `favorites` values:

```
struct TcaContentView: View {
  let store: Store<AppState, AppAction>

  var body: some View {
    WithViewStore(self.store) { viewStore in
      TabView {
        Text("Counter")
          .tabItem { Text("Counter \(viewStore.count)") }

        Text("Profile")
          .tabItem { Text("Profile \(viewStore.favorites.count)") }
      }
    }
  }
}
```

We currently have simple text views stubbed in for the tab content, but what we really want to do is create dedicated views for these screens like we did in the vanilla SwiftUI application.

We can do this by creating another view that holds onto a store as a `let` property:

```
struct TcaCounterView: View {
  let store: Store<AppState, AppAction>

  var body: some View {

  }
}
```

We can implement the body of this similarly to how we did in the vanilla SwiftUI application, except we will use `WithViewStore` to observe state changes, and we will send actions to the view store instead of making mutations directly in the view:

```
var body: some View {
  WithViewStore(self.store) { viewStore in
    VStack {
      HStack {
        Button("-") { viewStore.send(.decrementButtonTapped) }
        Text("\(viewStore.count)")
        Button("+") { viewStore.send(.incrementButtonTapped) }
      }
      if !viewStore.favorites.contains(viewStore.count) {
        Button("Save") { viewStore.send(.saveButtonTapped) }
      } else {
        Button("Remove") { viewStore.send(.removeButtonTapped) }
      }
    }
  }
}
```

And with that view created we can now use it over in the tab view by just passing along the store:

```
TabView {
  TcaCounterView(store: self.store)
    .tabItem { Text("Counter \(viewStore.count)") }

  Text("Profile")
    .tabItem { Text("Profile \(viewStore.favorites.count)") }
}
```

And we now have a half functioning app. If we create a quick Xcode preview we see that the first tab works great, and even the tab items update, but the profile tab is still unimplemented.

The profile can be implemented in much the same way as the counter:

```
struct TcaProfileView: View {
  let store: Store<AppState, AppAction>

  var body: some View {
    WithViewStore(self.store) { viewStore in
      List {
        ForEach(Array(viewStore.favorites.sorted()), id: \.self) { number in
          HStack {
            Text("\(number)")
            Spacer()
            Button("Remove") { viewStore.send(.profileRemoveButtonTapped(number)) }
          }
        }
      }
    }
  }
}
```

And now this view can be used in the tab view:

```
TabView {
  TcaCounterView(store: self.store)
    .tabItem { Text("Counter \(viewStore.count)") }

  TcaProfileView(store: self.store)
    .tabItem { Text("Profile \(viewStore.favorites.count)") }
}
```

And we now have a fully functional application. Both tabs work independently, but also they are sharing the favorites state so that changes in one are instantly reflected in the other.

### Breaking down a large TCA domain

What we’ve done so far doesn’t look too different from the first version of the application we built in vanilla SwiftUI last time. Some code has moved around and we were able to leverage value types instead of references types, but overall the shape looks about the same.

But now we are in a position to exercise two super powers of the Composable Architecture: `pullback` for breaking down an application’s logic into independent pieces, and `scope` for deriving child stores from parent stores. These two operators together instantly give us access to the functionality that we were really grappling with in the vanilla SwiftUI world. We no longer have to worry about manually notifying parent domains when child domains change, or figuring out how to synchronize state between domains. If you get all the pieces into place and it compiles, things tend to just “work”.

Let’s refactor the application so that the counter and profile views are better compartmentalized. We can start by defining new domains just for the individual screens, rather than trying to model the full application’s state all at once.

The counter domain holds the `count` and `favorites`, has most of the actions that we already modeled, and also has an empty environment (for now):

```
struct CounterState: Equatable {
  var count = 0
  var favorites: Set<Int> = []
}
enum CounterAction {
  case incrementButtonTapped
  case decrementButtonTapped
  case saveButtonTapped
  case removeButtonTapped
}
struct CounterEnvironment {}
```

The reducer also looks pretty similar to what we’ve done before. We can even copy and paste a lot of it over:

```
let counterReducer = Reducer<CounterState, CounterAction, CounterEnvironment> { state, action, _ in
  switch action {
  case .incrementButtonTapped:
    state.count += 1
    return .none
  case .decrementButtonTapped:
    state.count -= 1
    return .none
  case .saveButtonTapped:
    state.favorites.insert(state.count)
    return .none
  case .removeButtonTapped:
    state.favorites.remove(state.count)
    return .none
  }
}
```

Now we can update the `TcaCounterView` to hold onto a store of just the specific counter domain, and not all of `AppState` and `AppAction`s:

```
struct TcaCounterView: View {
  let store: Store<CounterState, CounterAction>


  ...
}
```

This view still compiles because we have everything in the counter domain that the view needs. Other parts of the application are not compiling, but we will get to that soon enough.

Let’s do the same for the profile. We can define its domain, which is quite small compared to the full app state:

```
struct ProfileState: Equatable {
  var favorites: Set<Int> = []
}
enum ProfileAction {
  case removeButtonTapped(Int)
}
struct ProfileEnvironment {}
```

And the reducer is simple since it’s only handling one action, which we can even take from from the previous app reducer:

```
let profileReducer = Reducer<ProfileState, ProfileAction, ProfileEnvironment> { state, action, _ in
  switch action {
  case let .removeButtonTapped(number):
    state.favorites.remove(number)
    return .none
  }
}
```

Now we can exchange the store that knows about all of app state and actions for one that just understand profile state and actions:

```
struct TcaProfileView: View {

  let store: Store<ProfileState, ProfileAction>

  ...
}
```

This view still compiles, but of course we have compiler errors in other spots of the application, which we will get into a moment.

But before moving on it’s worth noting that now each of these views is fully standalone. The views can be built in full isolation without the changes of one affecting the other. We could even separate each of these views and their associated domain into their own modules so that we would have even stronger guarantees that these features are fully isolated.

Now let’s take a look at those compiler errors and see what’s going on:

```
TabView {
  
  TcaCounterView(store: self.store)
    .tabItem { Text("Counter \(viewStore.count)") }

  
  TcaProfileView(store: self.store)
    .tabItem { Text("Profile \(viewStore.favorites.count)") }
}
```

This is happening because the `TcaContentView` currently holds onto a store of all app state and action, but we need to hand stores off to `TcaCounterView` and `TcaProfileView` that hold onto only a subset of that domain. To handle this properly we need to slightly refactor the app’s domain so that it is properly composed from the child domains.

For example, the current app state holds a `count` and a set of `favorites`:

```
struct AppState: Equatable {
  var count = 0
  var favorites: Set<Int> = []
}
```

The counter feature wants all of this state and the profile feature only wants a subset of this state, so we want a way to be able to hand those features bits of this state so that they can use it, but in such a way that any changes they make will automatically be played back to us. We can do this by just adding some computed properties to derive counter and profile state from `AppState`, and by supplying a setter we get automatic sharing of state.

For example, a writable computed property to derive `CounterState` from `AppState` looks like this:

```
var counter: CounterState {
  get {
    .init(count: self.count, favorites: self.favorites)
  }
  set {
    self.count = newValue.count
    self.favorites = newValue.favorites
  }
}
```

This will make sure that any mutations the counter feature makes to state is automatically replayed back to the parent.

Deriving `ProfileState` from `AppState` can be done similarly:

```
var profile: ProfileState {
  get {
    .init(favorites: self.favorites)
  }
  set {
    self.favorites = newValue.favorites
  }
}
```

So, this is how we can share application state with each of the child features. How do we do something similar with actions?

Well, that’s quite a bit easier. Since actions for each feature are completely disjoint and distinct, we can just model them as an enum with a case for each child feature:

```
enum AppAction {
  case counter(CounterAction)
  case profile(ProfileAction)
}
```

With these changes we now have compiler errors in the `appReducer`, but this is to be expected because we changed the domain quite a bit. For starters, we now only have two top level actions to handle, each of which lead to more actions:

```
let appReducer = Reducer<AppState, AppAction, AppEnvironment> { state, action, _ in
  switch action {
  case let .counter(counterAction):

  case let .profile(profileAction):
  }
}
```

Inside these cases we could just call out to the `counterReducer` and `profileReducer`. It takes a bit of manual work because we have to make sure to map any effects produced by the child reducers so that their outputs go into `AppAction`:

```
let appReducer = Reducer<AppState, AppAction, AppEnvironment> { state, action, _ in
  switch action {
  case let .counter(counterAction):
    return counterReducer.run(&state.counter, counterAction, CounterEnvironment())
      .map(AppAction.counter)

  case let .profile(profileAction):
    return profileReducer.run(&state.profile, profileAction, ProfileEnvironment())
      .map(AppAction.profile)
  }
}
```

This is pretty messy and accident prone, and luckily there’s a better way. We can make use of a super power of the Composable Architecture known as `pullback`. It allows you to transform reducers that work on local domains into reducers that work on global domains by basically doing the work we are manually doing above. Rather than switching on actions to find the one you are interested in and manually invoking the reducer on a piece of sub-state, and then `map` the resulting effect, you can instead concentrate on providing the high-level transformations that describe how to transform the global domain into the local domain. Then the operator uses that information to run the local reducer in the global domain.

For example, to `pullback` the `counterReducer` so that it runs on all of the app domain we just have to provide 3 pieces of information:

```
counterReducer
  .pullback(
    state: <#WritableKeyPath<GlobalState, CounterState>#>,
    action: <#CasePath<GlobalAction, CounterAction>#>,
    environment: <#(GlobalEnvironment) -> CounterEnvironment#>
  )
```

We first need to describe how to extract counter state from the app state and how one can plug new counter state back into the app state. We can use writable key paths to do this:

```
counterReducer
  .pullback(
    state: \AppState.counter,
    action: ???,
    environment: ???
  )
```

Next we need to describe how to extract counter actions from app actions and how one can plug counter actions into app actions. To do this we need an analogous notion of key paths but for enums, which is a concept we developed in the past and called them “case paths.” You can write these things from scratch, or you can use a little prefix operator that we provide to auto-derive it from the case of an enum:

```
counterReducer
  .pullback(
    state: \AppState.counter,
    action: /AppAction.counter,
    environment: ???
  )
```

And finally we need to describe how to derive a counter environment from an app environment. Currently our environments don’t hold onto anything, and so it’s as simple as this:

```
counterReducer
  .pullback(
    state: \AppState.counter,
    action: /AppAction.counter,
    environment: { (_: AppEnvironment) in CounterEnvironment() }
  )
```

But in the future we would pluck off the dependencies from `AppEnvironment` that the counter needs and pass them along.

And so with these few lines we have described how to embed the `counterReducer`’s functionality into the domain of the full app state. We can do the same for the `profileReducer`, but even better we can use another operator called `combine` to smash them together so that we get one single reducer that has all of the functionality from both the counter and the profile, but it is all operating on the level of the app domain:

```
let appReducer = Reducer.combine(
  counterReducer
    .pullback(
      state: \AppState.counter,
      action: /AppAction.counter,
      environment: { (_: AppEnvironment) in CounterEnvironment() }
    ),

  profileReducer
    .pullback(
      state: \AppState.profile,
      action: /AppAction.profile,
      environment: { (_: AppEnvironment) in ProfileEnvironment() }
    )
)
```

So this `appReducer` should now work exactly as it used to, except each feature has been quarantined into its own little sub-reducer that work only on the domain they care about. Those features can get as complex as they want to without ever letting that complexity spill over.

The full application is still not compiling, but these few lines right here accomplish most of what we were grappling with over in the vanilla SwiftUI world. We have defined two seemingly independent domains for the counter and profile, and implemented the functionality for each of those domains in complete isolation. But at the same time we have allowed ourselves to very easily glue those domains together to form the full domain of the application. Sharing of state between the domains happens automatic by virtue of the fact that we are using writable key paths to derive the child domains. There’s no need to listen for changes in one domain and replay them in the other.

###  Breaking down a large TCA runtime

But there’s one final step we need to take to complete the refactor. We’ve broken down the functionality of the application into two isolated domains, but we haven’t broken down the actual runtime that powers the behavior of the application. That’s the `Store`, and currently we’re trying to pass a store of the full application domain to views that only want a small portion of the domain:

```
TabView {
  
  TcaCounterView(store: self.store)
    .tabItem { Text("Counter \(viewStore.count)") }

  
  TcaProfileView(store: self.store)
    .tabItem { Text("Profile \(viewStore.favorites.count)") }
}
```

There’s an operator on stores that allow you to derive child runtimes from parent runtimes, and its analogous to the `pullback` operator on reducers, which allows us to transform child reducers into parent reducers. The two operators work in different directions, one goes from parent to child and the other goes from child to parent, yet they serve the same purpose: to break down domains into smaller domains that glue together.

The operator for stores is called `scope` in the Composable Architecture, although when we first discovered this operator in the early episodes covering the architecture we called it `view`. This kinda made sense as we are forming a “view” into a subset of the domain, but it was also confusing since we deal with other things called “view” in SwiftUI and UIKit. So, ultimately we renamed to `scope`.

The scope operator takes two arguments:

```
TcaCounterView(
  store: self.store.scope(
    state: <#T##(AppState) -> LocalState#>,
    action: <#T##(LocalAction) -> AppAction#>
  )
)
```

You need to describe how to extract local state from the app state, and you need to describe how to embed local actions into the app actions. It may seem a little strange that this transformations go different directions, but it’s exactly how things must be. In the counter view you want to access counter state, and so the scope store must know how to transform app state into counter state so that it can be accessed in the counter view. Further, the counter view also wants to send counter actions, and so when the scoped store receives one of those local actions it must know how to embed it in an app action so that it can then forward it to the main app store.

Filling in these requirements is straightforward, we can pluck out counter state with the `.counter` computed property, and we can embed counter actions into `AppAction` via the `counter` case:

```
TcaCounterView(
  store: self.store.scope(
    state: \.counter,
    action: AppAction.counter
  )
)
```

And we can do the same for the `TcaProfileView`:

```
TcaProfileView(
  store: self.store.scope(
    state: \.profile,
    action: AppAction.profile
  )
    )
```

And the whole application is now compiling for the first time in a while, and if we run the preview we see that everything works exactly as it did before.

### Examples of scoping

So that’s how `pullback` and `scope` work in the Composable Architecture, and as we’ve said before it is a super power of the library. Not only can we peel away a small bit of behavior from the larger, global behavior of the entire app, but we can also do so in a lightweight way. We don’t have to jump through hoops or get annoyed whenever we want to try this out.

And if you’ve been using the Composable Architecture yourself for awhile you may take for granted how easy it is to `.scope` a store to a child domain and then pass it along to a child view. It becomes second nature right away. However, in the vanilla SwiftUI world you have to do a lot more work to achieve this functionality. As we saw last episode we needed a bunch of extra code so that we could notify the parent anytime a child domain changes, and extra code to synchronize shared state between siblings. And even all that glue code wasn’t perfect. It wasn’t as efficient as it should be and it was fraught, and makes you start to wonder whether the solution is worse than the problem.

But when deriving child behavior from parent behavior is as easy as just calling `.scope` and providing a couple of arguments, there’s no reason not to do it. You should feel free to scope to smaller and smaller domains, helping split your application into lots of tiny pieces that are very easy to understand in isolation.

We’d like to now show a few examples of how scoping is used in real world code bases to pass around small bits of behavior to child views. We’ll start with one of the example applications that we developed for the Composable Architecture repository, so let’s open up the workspace that comes with the library:

We are going to look at the TicTacToe application. If you haven’t checked out this application before it’s just a very simple game that has a bit of a contrived complexity added onto it to make it more interesting. We model the idea of having to go through a logic flow, with a possible additional two factor flow, before you finally land on the “start game” screen.

One interesting thing we’ve done with this example is built it in both UIKit and SwiftUI in order to show off the libraries capabilities in both frameworks. You can even do some silly stuff like open one version of the application, like say the SwiftUI, perform a few actions, and then close the modal and open the UIKit version, and you will find that you are restored to exactly where you left off.

This shows the power of driving your application off of state. It makes it trivial to resume your application in any state you want, including the full navigation stack.

This application heavily uses `.scope` in order to build each screen of this application in isolation. We can see the beginning of this if we hop over to `AppSwiftView` in the `TicTacToe` project to see a use of `.scope`:

```
IfLetStore(self.store.scope(state: \.login, action: AppAction.login)) { store in
  ...
}
```

Now we haven’t talked about `IfLetStore` views yet, but we will soon. All you need to know about it for now is that it is a means to transform stores of optional state into stores of non-optional state, much like how `if let` statements allow us to safely unwrap optionals in regular Swift code.

So what we are doing here is scoping on the store in order to narrow down to just the `login` domain, which is optional since you are not logged in when the app first launches, and then using `IfLetStore` to unwrap that login state, and then handing that store down to the `LoginView`.

Then, the `LoginView` only has to worry about its domain:

```
public struct LoginView: View {
  let store: Store<LoginState, LoginAction>

  ...
}
```

It doesn’t need to know anything about the greater application domain, such as the game. In fact, it couldn’t even if it wanted to. The `LoginView` is in a completely different module from the game with no dependency between them.

The `LoginView` does something similar to transform its behavior and pass it along down the line. It uses `.scope` in order to transform its store into one that understands the two factor domain:

```
IfLetStore(
  self.store.scope(state: \.twoFactor, action: LoginAction.twoFactor),
  then: TwoFactorView.init(store:)
)
```

And this store is handed to the `TwoFactorView`, which also only knows about its own local domain:

```
public struct TwoFactorView: View {
  let store: Store<TwoFactorState, TwoFactorAction>

  ...
}
```

It is also in its own module and so it knows nothing about the login domain or the game domain.

If back up and look at the `AppSwiftView` file again we will see there was another `.scope` in there:

```
IfLetStore(self.store.scope(state: \.newGame, action: AppAction.newGame)) { store in
  ...
}
```

This one shows the “new game” screen when the `.newGame` state becomes non-nil, which happens once you login successfully. The “new game” screen is where you enter the names of each player, and then where you start the game. It only depends on the state it needs to know about:

```
public struct NewGameView: View {
  let store: Store<NewGameState, NewGameAction>

  ...
}
```

In particular it doesn’t know anything about the login domain, and it too is in its own module separate from the others.

Inside this file there’s another usage of `.scope` to transform this store into one that knows only about the game domain:

```
IfLetStore(
  self.store.scope(state: \.game, action: NewGameAction.game),
  then: GameView.init(store:)
)
```

Which allows us to create a `GameView` that only knows about the game domain:

```
public struct GameView: View {
  let store: Store<GameState, GameAction>

  ...
}
```

So just in this small toy application we have seen 4 uses of `.scope` in order to successively chisel away the root app store into smaller and smaller domains that correspond to how we drill down deeper into the application’s screens. If sometime soon there are additional screens to navigate to from the game screen we would just scope on the game store and pass it along.

And every step of the way all of the state is fully synchronized and we never worry about child domains notifying parent domains of changes. All of that is baked directly into the `.scope` operation. All of the intermediate stores fully work as if they were standalone stores in their own right, but really they were all derived from the same root store.

Let’s take a look at one more real world example, this time in isowords, a word game we built in SwiftUI and open sourced a few months ago. In the previous example we started at the root and then explored how we scope stores in order to reach leaf views in the application. This time we are going to go in reverse. We are going to start with a particular view, and work our way backwards to trace the chain of `.scope`s that lead us back to the root application store.

The view we are going to start with is known as the `CalendarView` in the code base, and it’s a view that displays a little calendar in order to show how your daily challenges ranks over the past 30 days. Let’s fire up the application in the simulator so that we can see what it looks like:

If we hop over to `CalendarView.swift` we will see that the view needs a store to do its job, but that the store only needs a small bit of domain:

```
struct CalendarView: View {
  let store: Store<DailyChallengeResultsState, DailyChallengeResultsAction>

  ...
}
```

The state only holds a little bit of information for the results of the past daily challenges, and the action enum only has a few actions. It doesn’t need to know anything else about the rest of the application, which means it is very easy to develop this feature in isolation. We can run it in a preview very easily and test it.

Let’s trace the creation of this view all the way back to the root of the application. If we search for where `CalendarView` is invoked we will see that it comes from `DailyChallengeResults.swift`. It looks like the `DailyChallengeResultsView` creates the calendar, but doesn’t actually do any scoping to create it.

So, let’s see who creates `DailyChallengeResultsView`. Looks like it’s created in the `DailyChallengeView`, and is done via an `IfLetStore` and a `.scope`:

```
IfLetStore(
  self.store.scope(
    state: (\DailyChallengeState.route)
      .appending(path: /DailyChallengeState.Route.results)
      .extract(from:),
    action: DailyChallengeAction.dailyChallengeResults
  ),
  then: DailyChallengeResultsView.init(store:)
)
```

This looks pretty intense, but let’s not focus on what is actually happening in the code and instead just recognize that we are indeed scoping the store and handed it down to the `DailyChallengeResultsView`.

Now if we search for who creates `DailyChallengeView` we will find it in `DailyChallengeHeaderView`:

```
IfLetStore(
  self.store.scope(
    state: (\HomeState.route).appending(path: /HomeRoute.dailyChallenge).extract(from:),
    action: HomeAction.dailyChallenge
  ),
  then: DailyChallengeView.init(store:)
)
```

Again we have an `IfLetStore` and a `.scope`.

Next we see who creates the `DailyChallengeHeaderView` we will find it in the `HomeView`:

```
DailyChallengeHeaderView(store: self.store)
  .screenEdgePadding(.horizontal)
```

And then who creates `HomeView`s? That happens in `AppView`, which is the root view of the entire application:

```
HomeView(store: self.store.scope(state: \.home, action: AppAction.home))
```

So, we’ve traced ourselves back to the very root of the application. In fact, this `self.store` is of type `Store<AppState, AppAction>`, which means it holds onto the domain for the entire application. And from this gigantic blob of state and behavior we were able to scope over and over and over until we got all the way down to a tiny bit of state that powers the calendar. In fact, we scoped about 5 times, and really we did even more than that because the `IfLetStore` view uses `.scope` under the hood.

### Next time: scoping and performance

So, what we’ve seen is that the Composable Architecture comes with some tools out of the box that allow you to “derive behavior”, which means that you can take a big blob of behavior, such as the store that controls your entire application, and derive new stores that focus in on just a subset of that behavior. This is crucially important if you want to build large, complex applications that can be split apart into isolated modules.

Unfortunately SwiftUI does not give us these tools out of the box. Currently it is on us to manually implement the integration points between child view models and parent view models. We have to make sure that anytime the child changes we notify the parent so that it can do any work necessary, and we need to listen for changes to particular pieces of state in each child so that we can replay them to the other siblings. We also have to do extra work in order to make sure we don’t accidentally create infinite loops between children, such as when child A updates child B which causes child B to update child A, and so on.

However, all is not sunshine and rainbows in the Composable Architecture world. When you start building long chains of scoping and observations you run the risk of introducing performance problems if not done in the right way. Some of this can be solved in user land by being more vigilant with what parts of state need observing and what parts do not, and other things can be solved by the library itself. It turns out that some of the code in the Composable Architecture that handles scoping is not as efficient as it could be, and we want to take a moment to fix those problems.

Let’s start by first seeing what tools the Composable Architecture ships with that allows us to fine tune the performance of our applications, and show what the corresponding story looks like for vanilla SwiftUI applications…next time!