# Using SwiftLint, SwiftFormat
## Install

      bew install swiftlint
      brew install swiftformat
간단히 brew 로 설치할 수 있다. 

## xcode config
- Build Phases. Click the + and select "New Run Script Phase". Insert the following as the script:
```
if which swiftlint >/dev/null; then
  swiftlint
else
  echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
fi
```


 
## Reference
- [Using SwiftLint to decrease code smell on Xcode. | by Ricardo Santos | Feb, 2021 | Medium](https://ricardojpsantos.medium.com/using-swiftlint-to-decrease-code-smell-on-xcode-e1dd49258f22)
- [wonkwh/swiftBestPractice (github.com)](https://github.com/wonkwh/swiftBestPractice)
----
- tags #swiftLang #swiftlint #lint 