# Three Column Navigation
## Link
- [kean/ThreeColumnNavigation: A minimal example of three-column navigation for iPad and macOS using SwiftUI (github.com)](https://github.com/kean/ThreeColumnNavigation)
- [sidebar-navigation](https://gavinw.me/swift-macos/?utm_campaign=iOS%252BDev%252BWeekly&utm_medium=email&utm_source=iOS%252BDev%252BWeekly%252BIssue%252B516#sidebar-navigation)
- [SideBarToggle](https://gavinw.me/swift-macos/?utm_campaign=iOS%252BDev%252BWeekly&utm_medium=email&utm_source=iOS%252BDev%252BWeekly%252BIssue%252B516#sidebar-toggle)
- [NavigationView in SwiftUI | SwiftOnTap](https://swiftontap.com/navigationview)
- [LostMoa - Double-Column Navigation Split View in SwiftUI](https://lostmoa.com/blog/DoubleColumnNavigationSplitViewInSwiftUI/)
- [Using Sidebar in SwiftUI Without a NavigationView | by Imthathullah | Better Programming](https://betterprogramming.pub/using-sidebar-in-swiftui-without-a-navigationview-94f4181c09b)

##
```swift


import SwiftUI

#if canImport(UIKit)
import UIKit
#endif

struct ContentView: View {
  
  var body: some View {
    NavigationView {
      Sidebar()
      Text("No Sidebar Selection")
      Text("No Message Selection")
    }
    .onAppear {
#if canImport(UIKit)
      // Setting up iPad layout to match macOS behaviour
      UIApplication.shared.setFirstSplitViewPreferredDisplayMode(.twoBesideSecondary)
#endif
    }
  }
}

struct Sidebar: View {
  @State private var isDefaultItemActive = true
  
  var body: some View {
    let list = List {
      Text("Favorites")
        .font(.caption)
        .foregroundColor(.secondary)
      NavigationLink(destination: IndoxView(), isActive: $isDefaultItemActive) {
        Label("Inbox", systemImage: "tray.2")
      }
      NavigationLink(destination: SentView()) {
        Label("Sent", systemImage: "paperplane")
      }
    }
    .listStyle(SidebarListStyle())
    
#if os(macOS)
    list.toolbar {
      Button(action: toggleSidebar) {
        Image(systemName: "sidebar.left")
      }
    }
#else
    list
#endif
  }
}

#if os(macOS)
private func toggleSidebar() {
  NSApp.keyWindow?.firstResponder?
    .tryToPerform(#selector(NSSplitViewController.toggleSidebar(_:)), with: nil)
}
#endif

struct IndoxView: View {
  var body: some View {
    List(Array(0...100).map(String.init), id: \.self) { message in
      NavigationLink(destination: MessageDetailsView(message: message)) {
        Text(message)
      }
    }
    .navigationTitle("Inbox")
    .toolbar {
      Button(action: { /* Open filters */ }) {
        Image(systemName: "line.horizontal.3.decrease.circle")
      }
    }
  }
}

struct SentView: View {
  var body: some View {
    Text("No Sent Messages")
      .navigationTitle("Sent")
      .toolbar {
        Button(action: {}) {
          Image(systemName: "line.horizontal.3.decrease.circle")
        }
      }
  }
}

struct MessageDetailsView: View {
  let message: String
  
  var body: some View {
    Text("Details for \(message)")
      .toolbar {
        Button(action: {}) {
          Image(systemName: "square.and.arrow.up")
        }
      }
  }
}

struct ContentView_Previews: PreviewProvider {
  static var previews: some View {
    ContentView()
  }
}

// MARK: - Consistency with iPad

#if canImport(UIKit)
private extension UIApplication {
  func setFirstSplitViewPreferredDisplayMode(_ preferredDisplayMode: UISplitViewController.DisplayMode) {
    var splitViewController: UISplitViewController? {
      UIApplication.shared.firstSplitViewController
    }
    
    // Sometimes split view is not available instantly
    if let splitViewController = splitViewController {
      splitViewController.preferredDisplayMode = preferredDisplayMode
    } else {
      DispatchQueue.main.async {
        splitViewController?.preferredDisplayMode = preferredDisplayMode
      }
    }
  }
  
  private var firstSplitViewController: UISplitViewController? {
    windows.first { $0.isKeyWindow }?
      .rootViewController?.firstSplitViewController
  }
}

private extension UIViewController {
  var firstSplitViewController: UISplitViewController? {
    self as? UISplitViewController
    ?? children.lazy.compactMap { $0.firstSplitViewController }.first
  }
}
#endif

```
## 



----
- tags: #swiftui #macDev 