# SwiftUI Form Tutorial 

- [SwiftUI Form tutorial for beginners](https://programmingwithswift.com/swiftui-form-beginners-guide/)


##  1: Create the Form
- Update code in your `ContentView` to look like the below:

```swift
struct ContentView: View {
    var body: some View {
        NavigationView {
            Form {
                // Form fields go here
            }.navigationBarTitle(Text("Profile"))
        }
    }
}
```

All that we done here is remove the default "hello world" Text view, and replace it with a NavigationView so that we can set a .navigationBarTitle, and a Form view which is needed for our form fields in the steps that follow.

## 2: Add First and Last name TextFields to the Form
We can now add some of our first form fields. We will add two TextField views for Firstname and Lastname.

Before we add the TextField, lets add our `@State`  properties that we will use for the Firstname and Lastname.

Add the following code above var body: some View :
```swift
@State private var firstname = ""
@State private var lastname = ""
```

We need these so that we can bind them to our TextField views.

Next we need to add those TextField views to our Form. To do that, update your body property code to look like this:
```swift
var body: some View {
    NavigationView {
        Form {
            TextField("Firstname",
                      text: $firstname)
            TextField("Lastname",
                      text: $lastname)
        }.navigationBarTitle(Text("Profile"))
    }
}
```
If you build and run the app, you will see the following:


## 3: Add location Picker
We want the user to be able to select a location from a provided data source. This could come from an api request, but for now we will use a struct that has a static property which will contain all the locations.

Let's first create the Location struct. Add the following outside of the main ContentView:
```swift
struct Location {
	static let allLocations = [
		"New York",
		"London",
		"Tokyo",
		"Berlin",
		"Paris"
	]
}
```
As said before, this is just a static property which a String array which contains all the locations that we need for our Picker view.

Next we need to have another @State property which can store the selected location. To do this, add a new @State property below the lastname property we created before. This is what the property will look like:

```swift
@State private var location = ""
```

Now that we have our data source and location state, let's add our Picker view below the TextField views we added in our previous step.

To do that, add the following code below our Lastname TextField:
```swift
Picker(selection: $location,
       label: Text("Location")) {
        ForEach(Location.allLocations, id: \.self) { location in
            Text(location).tag(location)
        }
}
```
We bind the Picker to our location state. When the Picker's value changes, it will update the location property.

We also have a ForEach that will loop through all the locations that we have in our data source.

If you build and run the app now, you will see this:


If you tap on the Picker view, you will see the following:


When we tap on one of the choices it will update our location property and navigate back to the Profile view. Because the location property has been updated, it will show on the Picker view next to the Location text.


Our next step is to allow the user to accept our terms and conditions.

##  4: Add Terms and Conditions Toggle view
As with all the other fields that we have added, we need to add some @State so that we can bind the Toggle view to it.

Let's add a termsAccepted property below the location property we added in the previous step:

	@State private var termsAccepted = false
	
This needs to be false because we cannot accept the terms and conditions on behalf of our users.

We now need to add our Toggle view. This will go below the Picker view from the previous step.

To add the Toggle, add the following code:
```swift
Toggle(isOn: $termsAccepted,
       label: {
           Text("Accept terms and conditions")
})
```
As with the previous steps, we bind the view to a property, in this case termsAccepted. We then need to add a label, this is a simple Text view.

If you build and run the app, it will look like this:


##  5: Add a Stepper view for Age
This is the last input field for this section of Form. Once again, we need to add some state, so lets do that by adding a new property called age

Add the following code below the termsAccepted property:

	@State private var age = 20
	
I have just defaulted this to 20, but the age range will be from 18-100, we will add that in with the next bit of code:
```swift
Stepper(value: $age, 
        in: 18...100, 
        label: {
    Text("Current age: \(self.age)")
})
```
We bind the Stepper view to our age property, we then provide an age range, in this case we will allow the ages from 18 to 100, and lastly we add a Text view that will display the current age value.

If you run the app, you will see the following:


If you click the - and + the age will decrement and increment within the range that we provided.

##  6: Add "Update Profile" Button
This Button won't really do anything besides print that the profile has been updated.

Add the following code below the Stepper we added previously:
```swift
Button(action: {
    print("Updated profile")
}, label: {
    Text("Update Profile")
})
```
That was simple enough. But we don't want this button to be visible unless our Form is valid, so let's add a validation method to our ContentView.

For our validation we want to ensure that the Firstname and Lastname fields are not empty, we also want to make sure that the user has accepted our terms and conditions and lastly the user needs to select a location.

To do this add the following code below the body property:
```swift
private func isUserInformationValid() -> Bool {
    if firstname.isEmpty {
        return false
    }
    
    if lastname.isEmpty {
        return false
    }
    
    if !termsAccepted {
        return false
    }
    
    if location.isEmpty {
        return false
    }
    
    return true
}
```
If any of the fields are not valid according to the above rules, we will return false, otherwise we will return true.

Let's wrap our Button so that it only shows when our Form is valid.

Update the Button code to look like this:
```swift
if self.isUserInformationValid() {
    Button(action: {
        print("Updated profile")
    }, label: {
        Text("Update Profile")
    })
}
```
Every time one of the fields change, this will check to see if our Form is valid and if it is, it will show the Button otherwise it will not be rendered.

If you build and run the app, you will see the following:


Once you fill in all the fields you will see the "Update Profile" button, like this:


## 7: Add a Section to the Form
Before we add the reset password section to the Form we need to wrap all the above fields in a Section.

To do this, replace the body property with the following code:
```swift
var body: some View {
    NavigationView {
        Form {
            Section(header: Text("User Details")) {
                TextField("Firstname",
                          text: $firstname)
                TextField("Lastname",
                          text: $lastname)
                Picker(selection: $location,
                       label: Text("Location")) {
                        ForEach(Location.allLocations, id: \.self) { location in
                            Text(location).tag(location)
                        }
                }
                
                Toggle(isOn: $termsAccepted,
                       label: {
                        Text("Accept terms and conditions")
                })
                
                Stepper(value: $age,
                        in: 18...100,
                        label: {
                    Text("Current age: \(self.age)")
                })
                
                if self.isUserInformationValid() {
                    Button(action: {
                        print("Updated profile")
                    }, label: {
                        Text("Update Profile")
                    })
                }
            }
        }.navigationBarTitle(Text("Profile"))
    }
}
```
All we have done now is add the Section view with a header of User Details. So when we run the app now, we will see the following:


It is a very subtle difference, but there is now a User Details heading just below the Profile title.

## 8: Add the reset password Section
Just below the above Section that we added, add a new Section with the heading "Password". The code will look like this:
```swift
Section(header: Text("Password")) {
    // Fields go here
}
```
We now need to add three @State properties and one local constant.

Add the following properties below the age property that we added in Step 5:
```swift
private let oldPasswordToConfirmAgainst = "12345"
@State private var oldPassword = ""
@State private var newPassword = ""
@State private var confirmedPassword = ""
```
We will bind the three @State properties to SecureField views, and we will use the oldPasswordToConfirmAgainst to mock validate that the oldPassword is our "previous" password.

Inside the new password Section, add the following code:

```swift
SecureField("Enter old password", text: $oldPassword)
SecureField("New Password", text: $newPassword)
SecureField("Confirm New Password", text: $confirmedPassword)
```
Each of the SecureField views have been bound to one of the @State properties that we added.

If you build and run the app now, you will see the following:


The next thing that we need to add is our "Update Password" Button. To do that, add the following code below the SecureField views we just added:

```swift
Button(action: {
    print("Updated password")
}, label: {
    Text("Update password")
})
```
Once again, this Button does nothing besides print "Updated Password". If you build and run the app now you will see the button, but we don't want the button to show unless the password fields are valid.

Let's add the password validation method so that the "Update password" Button only shows when the password fields are valid.

Add the following validation code below the `isUserInformationValid` method that we added for the User Details validation:
```swift
private func isPasswordValid() -> Bool {    
    if oldPassword != oldPasswordToConfirmAgainst {
        return false
    }
    
    if !newPassword.isEmpty && newPassword == confirmedPassword {
        return true
    }
    
    return false
}
```
To ensure that the password fields are valid, we check to see if the oldPassword is not equal to the oldPasswordToConfirmAgainst. If it is not the same we return false.

We will then make sure that newPassword is not empty, and we check to see if newPassword and confirmedPassword is the same, if they are the same, we return true. Otherwise we return false.

Let's update the Button to only show when the password fields are valid. To do that, replace the current Button that you have with the following:
```swift
if self.isPasswordValid() {
    Button(action: {
        print("Updated password")
    }, label: {
        Text("Update password")
    })
}
```
If you build and run the app now, nothing will be visually different from the previous time we you ran the app.

We now need to type "12345" into the "Enter old password" field, and then we need to enter the same password in the "New Password" and "Confirm Password" SecureField views.

Once you have done that, you will see the following:


##  9: Move text fields when keyboard shows
Ok, in the previous step I did not have a keyboard showing up as I was using my mac keyboard. Unfortunately, doing this leaves a massive flaw in this form. The flaw is that if the user is on a device the keyboard will cover the text fields, so let's fix that.

The first thing we need to do is to wrap the NavigationView in a VStack. I am not going to show all the code for this, but this is the structure now:
```swift
var body: some View {
    VStack {
        NavigationView {
            Form {
                ...
            }
        }
    }
 }
 ```
Now that we have the VStack added we add in the code that will allow us to see the textfields when the keyboard is showing.

To move the text fields up we are going to need to get the keyboard height, for this functionality we need to add the following property below confirmedPassword:
```swift
@State private var keyboardOffset: CGFloat = 0
```
Now let's set the offset on the NavigationView. To do this, add the offset modifier to the NavigationView:

Your code should look like this:
```swift
var body: some View {
    VStack {
        NavigationView {
            Form {
                ...
            }
        }.offset(y: -self.keyboardOffset)
    }
 }
 ```
As you can see, the offset modifier uses the keyboardOffset property that we added.

We have two more things to do, firstly we need to add the code that will listen for the notification that the keyboard sends when it shows and hides.

We will add listeners for these in the onAppear modifier, update your code to look like this following:
```swift
var body: some View {
    VStack {
        NavigationView {
            Form {
                ...
            }
        }.offset(y: -self.keyboardOffset)
        .onAppear {
                NotificationCenter.default.addObserver(forName: UIResponder.keyboardDidShowNotification,
                                                       object: nil,
                                                       queue: .main) { (notification) in
                                                        NotificationCenter.default.addObserver(
                                                            forName: UIResponder.keyboardDidShowNotification,
                                                            object: nil,
                                                            queue: .main) { (notification) in
                                                                guard let userInfo = notification.userInfo,
                                                                    let keyboardRect = userInfo[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect else { return }
                                                                
                                                                self.keyboardOffset = keyboardRect.height
                                                        }
                                                        
                                                        NotificationCenter.default.addObserver(
                                                            forName: UIResponder.keyboardDidHideNotification,
                                                            object: nil,
                                                            queue: .main) { (notification) in
                                                                self.keyboardOffset = 0
                                                        }
                }
            }
    }
 }
 ```
This looks like a lot of code, but it really isn't. We are adding observers for two notifications. The first being for when the keyboard shows, UIResponder.keyboardDidShowNotification, and the second for when it hides, UIResponder.keyboardDidHideNotification.

When we listen for UIResponder.keyboardDidShowNotification, we are doing more than when we listen for the keyboard hiding. When the keyboard shows we need to get the userInfo, if that is not nil we will try and get the value for UIResponder.keyboardFrameEndUserInfoKey, which, at the same time we will try and cast as a CGRect. This key holds the rectangle information for the keyboard.

If this data is not nil and we can cast it, we set the keyboardOffset to the height that we get from keyboardRect.

When we listen to see when the keyboard closes, all we need to do is set keyboardOffset to 0.

Lastly we need to change the background color of the VStack. To do that, add the following modifier to the VStack:

```swift
.background(Color(UIColor.systemGray6))
```

This will stop there being a white rectangle showing up just above the keyboard.

## 추가 링크
- [SwiftUI Build a Form from scratch](https://www.youtube.com/watch?v=jrAA9Gt-jqw)
- [Building forms with SwiftUI](https://swiftwithmajid.com/2019/06/19/building-forms-with-swiftui/)

---
- tags: #swiftui 