# Concise Forms swiftUI

## Form view hierarchy

```swift

struct VanillaSwiftUIFoamView: View {
   var body: some View {
      Form {
          Section(header: Text("Profile")) {
             TextField("Display name", text: .constant(""))
             Toggle("Protect my posts", isOn: .constant(false))
          }
          Section(header: Text("Communication")) {
             Toggle("Send notification", isOn: .constant(false))
             Picker("Top posts digest", selection: .constant(Digest.off )) {
                Text("daily")
                Text("weekly")
                Text("off")
             }
          }
       }.navigationTitle("Setting")
    }
}

  
enum Digest {
   case daily
   case weekly
   case off
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
       NavigationView {
         VanillaSwiftUIFoamView()
       }
    }
}
```

###  result
![](form2.png)
![](form1.png)

## ViewModel Binding
```swift
class SettingsViewModel: ObservableObject {
  @Published var digest = Digest.off
  @Published var displayName = "kwang"
  @Published var protectMyPosts = false
  @Published var sendNotifications = false
  
  func reset() {
    self.digest = .off
    self.displayName = ""
    self.protectMyPosts = false
    self.sendNotifications = false
  }
}

struct VanillaSwiftUIFoamView: View {
  @ObservedObject var viewModel: SettingsViewModel
  
    var body: some View {
      Form {
        Section(header: Text("Profile")) {
          TextField("Display name", text: $viewModel.displayName)
          Toggle("Protect my posts", isOn: $viewModel.protectMyPosts)
        }
        
        Section(header: Text("Communication")) {
          Toggle("Send notification", isOn: $viewModel.sendNotifications)
          
          if self.viewModel.sendNotifications {
            Picker("Top posts digest", selection: $viewModel.digest) {
              ForEach(Digest.allCases, id: \.self) { digest in //Digest.AllCased와 혼동하지 말자.
                Text(digest.rawValue)
              }
          }
//            Text("daily").tag(Digest.daily)
//            Text("weekly").tag(Digest.weekly)
//            Text("off").tag(Digest.off)
          }
        }
        Button("Reset") {
          self.viewModel.reset()
        }
      }.navigationTitle("Setting")
    }
}

enum Digest: String, CaseIterable {
  case daily
  case weekly
  case off
}


struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
      NavigationView {
        VanillaSwiftUIFoamView(viewModel: SettingsViewModel())
      }
    }
}

```

## Advanced form logic