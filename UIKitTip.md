# UIKit Tip

- getting-version-and-build-info-with-swift
	- [ios - Getting Version and Build Info with Swift - Stack Overflow](https://stackoverflow.com/questions/24501288/getting-version-and-build-info-with-swift)
	
	```swift
extension UIApplication {

    func applicationVersion() -> String {

        return NSBundle.mainBundle().objectForInfoDictionaryKey("CFBundleShortVersionString") as! String
    }

    func applicationBuild() -> String {

        return NSBundle.mainBundle().objectForInfoDictionaryKey(kCFBundleVersionKey as String) as! String
    }

    func versionBuild() -> String {

        let version = self.applicationVersion()
        let build = self.applicationBuild()

        return "v\(version)(\(build))"
    }
}


  ```
  
  - get device model
	- [ios - How to determine the current iPhone/device model? - Stack Overflow](https://stackoverflow.com/questions/26028918/how-to-determine-the-current-iphone-device-model)


```swift
func modelIdentifier() -> String {
    if let simulatorModelIdentifier = ProcessInfo().environment["SIMULATOR_MODEL_IDENTIFIER"] { return simulatorModelIdentifier }
    var sysinfo = utsname()
    uname(&sysinfo) // ignore return value
    return String(bytes: Data(bytes: &sysinfo.machine, count: Int(_SYS_NAMELEN)), encoding: .ascii)!.trimmingCharacters(in: .controlCharacters)
}
```

