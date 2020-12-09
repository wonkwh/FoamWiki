# project generation tool 
- xcodeGen, tuist 등 xcode project file `proj` 파일 생성툴들
## Tuist
- https://tuist.io/docs/usage/getting-started/
- xcodegen과의 차이점
	- yml 파일이 아닌 swift source file로 config file 구성
	- project generation뿐 아니라 build 까지 지원


## 첫 실행
- install 
```bash
	bash <(curl -Ls https://install.tuist.io)
```

bootstrap

	tuist init --platform ios

위와 같이 실행하면 

	.rw-r--r--@ 1.2k wonkwh  9 12 11:40 .gitignore
	.rw-r--r--@  987 wonkwh  9 12 11:40 Project.swift
	drwxr-xr-x     - wonkwh  9 12 11:40 Targets
	drwxr-xr-x     - wonkwh  9 12 11:40 Tuist

위와 같이 생성된다.  `Project.swift` 파일이 config 파일이라 할수 있다. 
xcode project 파일을 생성하려면 

	tuist generate 
	
실행하면 

	.rw-r--r--@ 1.2k wonkwh  9 12 11:40 .gitignore
	drwxr-xr-x     - wonkwh  9 12 11:42 Derived
	drwxr-xr-x     - wonkwh  9 12 11:42 MyTuistTempleate.xcodeproj
	drwxr-xr-x     - wonkwh  9 12 11:42 MyTuistTempleate.xcworkspace
	.rw-r--r--@  987 wonkwh  9 12 11:40 Project.swift
	drwxr-xr-x     - wonkwh  9 12 11:40 Targets
	drwxr-xr-x     - wonkwh  9 12 11:40 Tuist

`MyTuistTempleate`은 폴더명으로 생성되며 workdpace file까지 같이 생성.  `Derived` folder도 추가 생성된다. 
project file만 생성하려면 `tuist generate --project-only` 와 같이 flag를 붙인다 

CLI에서 project 를 build하려면 

	tuist build 

를 실행하면 된다. 

## Edit Project.swift

	tuist edit 

  `Manifests` project를 생성하여 xcode 가 실행된다. 
  
  ### script build action 추가 
  - swiftlint, swiftformat, R.swift 과 같은 라이브러리를 추가하고 pre build script 를 추가

### 3rd party library
- spm 을 이용해서 라이브러리 추가
- Debug build에만 woodpeck library 를 추가
	- https://www.woodpeck.cn/manuallink.html 

### add Feature Project
custom project name, custom feature project를   [요기](https://github.com/alexanderwe/ios-starter) 에서는 `cookiecutter` tool을 사용
- https://tuist.io/docs/commands/scaffold/

## result repository


## resource link
- [👋 안녕 XcodeGen](https://pilgwon.github.io/post/hello-xcodegen)
- [tuist-project-generation](https://developers.soundcloud.com/blog/tuist-project-generation)
- [first-touches-of-using-tuist-for-xcode-project-generation](https://medium.com/swlh/first-touches-of-using-tuist-for-xcode-project-generation-f46c630bc29b)
- https://github.com/alexanderwe/ios-starter
----
- tags : #xcode 