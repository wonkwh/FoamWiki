# How to use SwiftUI in Swift Playgrounds

- https://www.hackingwithswift.com/articles/203/how-to-use-swiftui-in-swift-playgrounds


```swift
import PlaygroundSupport
import SwiftUI


struct ContentView: View {
    var body: some View{
        HStack(spacing: 0) {
            Rectangle().fill(Color.red)
            Rectangle().fill(Color.green)
            Rectangle().fill(Color.blue)
        }.frame(width: 300, height: 100)
    }
}

PlaygroundPage.current.setLiveView(ContentView())


```

- tags: #swiftPlayground #swiftui 