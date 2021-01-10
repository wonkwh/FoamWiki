# MimTok Legacy Refactoring

## Souce 분석
- [[MimTokSourceanalysys]]
## Refactoring
###  File Structure & Architecture 
- Project Folder 구조가 일관된 규칙이 없고 너무 난잡하다 
	- service, model, networking, ui_scene, utility, extension으로 재구성
- Object-c 코드 100% 
		- Apple 은 Object-C의 지원을 끝냈으므로 Swift로의 변환필요 
- 기본적은 MVC architecture 
	- mvc 의 전형적인 단점들 그대로 가지고 있음 
	- Massive View Controller : 모든것을 ViewController에 다 때려넣어 비대해진다. 
		- 예) `YBLookVideoVC.m` class file , 1213Line
		- 유지 보스가 어려울뿐더러 code변경시 side effect 심각하게 우려 
	- MVVM + Coordinator, Redux architecture, Clean Swift Architecture 선택하여 변경해야 함
		-  RxSwift 혹은 Combine 이용하여 view, view- model Binding, notification 처리
- TestCode가 1도 없다
	- Refactoring하기 전 최소한의 BusinessLogic에 대한 UnitTest가 필요함
- 2020/12/29 현재 compiler warning 208 개
	- critical 한 warning이 있는지 쉽게 파악이 안됨
### Localization
- 현재 First Language로 중국어
- `InfoPList.strings` file 에 정의 
- 현재 key 값도 중국어다! 
	- key  는 모두 영어로 변경
	- default language도 변경
- https://poeditor.com 등의 사용도 고려해서 CI, CD 서비스와 통합
### Perfomance Issue
- 현재 video List scrolling issue 원인을 파악해보면 
	- ios 10에서 부터 지원하는 list prefeching 기술을 이용하지 않음 
		- 해당기술 적용하여 `YBLookVideoVC` class refactoring 
		- 아니면 open source library( texture library )
	- Downloading Video While Streaming
		- https://medium.com/@EugeneZZI/understanding-avassetresourceloaderdelegate-b90b3fe2c059
	- - Memory Cache and Disk Cache 적용
	- [[ObjectiveCToSwift]]
- Memory Leak Detect Library 설치 
	- https://github.com/krzysztofzablocki/LifetimeTracker
	- https://github.com/didi/DoraemonKit/blob/master/README_EN.md
		- http://xingyun.xiaojukeji.com/docs/dokit_en#/intro
		- app에서 app언어를 영어로 바꾸면 중국어에서 영어로 바뀐다. 
	


- UserDefault(Local File)
	- 너무나도 과도하게 데어터를 file로 저정하고 있다. 
		- performance 저하 --> DB(CoreData, Sqllite, RealmDB) 로 migration 필요 
- image 의 width, height를 UserDefault 저장하고 있다 !!
		- Nuke, Kingsfisher 등의 image cache library를 사용하여 개선필요
		- Thumbnail images의 scales down 을 pre-processor로 처리필요

- [ ] Resource 가 모두 png파일로 폴더로 분산 
	- [x] `图片` 라는 이름의 폴더 --> 모두 `Resource` 폴더로 변경 완료
	- [x] 사용하지 않는 쓸모없는 리소스들이 다수있다 
	- [x] vector graphics file (pdf file) 로 모두 변경하고 asetLibrary 로 통합
- [[PreviewVideoScrollPerformanceRefactgoring]]
### DevOps
- Crash debug Log 받아볼수 없음 
	- `firebase crashlytics` service 추가 
- CI, CD service 부재
	- release, test 배포등이 자동화 되어 있지 않음. 
	- fastlane, github action, JetBrainSpace CI service 활용하여 추가 
- CodingConvention이 룰이 없는듯이 보이며 일관성이 없다. 
	- 접두어 `YB`로 시작되는 파일은 `YB007` 이라는 userID의 원소스 
	- 그외 `camelCase` file 명도 혼재되어 있다 
	- 소스코드 이해및 유지보스를 위해 CodingConvention및 NamingConvention을 문서, Tool 을 통해 통일할 필요가 있다 
		- SwiftLint, SwiftFormat tool 사용
	- git branching 이 규칙이 없다. 
		- 현재 `dev_master` 가 외주 개발사가 했던 branch 
		- 대강의 히스토리는 파악할수 있으나 tag, release branch를 사용하지 않아 version 관리가 되고 있지 않다
		- [x] 현재 Git-flow 로 변경 
			- 참조 - [우린 Git-flow를 사용하고 있어요](https://woowabros.github.io/experience/2017/10/30/baemin-mobile-git-branch-strategy.html)
		- issue, feature구현, pull request 등의 규칙이 필요
	- source formatting 
		- swift file은 swiftlint, swiftformat을 사용하면 되나. 
		- #Objective-C 는 clang-format 사용하자 
			- [[ObjectiveC-SourceFormatting]]

## 3rdParty Library
- 제거해야할 중국관련 Library, Framework
	- QCloud, Qiniu
		- Cloud Storage 관련
			- `YBStorage` 관련 --> 제거 가능 
	- Baidu Voice
		- Speech recogination 관련
		- iOS 10 이후 speech recognition API 가 추가되었다. 중국어 인식을 쓰지 않는다면 지워도 될듯.
	- QMap
		- Tencent Map SDK
		- `AppDelegate.m` 파일에 `sendLocation()` 함수에서 key 등록외에는 없음 
		- 제거 완료

- 그외 제거 할수 없는 필수 Media Filter, AR 관련 
	- MHBeautySDK
		- https://ext.dcloud.net.cn/plugin?id=1933
		- Camera Beauty Filter 
		- 대체 가능한 소스들은 refrence 참조
	- TXLiteAVSDK_Professional
		- https://cloud.tencent.com/product/mlvb
		- https://cloud.tencent.com/document/product/647
		- Mobile Live, Brodcast, Editing Library 
		- 어떤 패키지를 구매했는지? 그것에 따라 비용과 제한있음. 
		- 필수적인 기능들이라 대체가 안되고 새로 구현시 비용이 크다. 
			- 비용은 [mimtokFeature](mimtokFeature.md) 참조
	- H5 
		- Hybrid WebView 관련

- Media 관련


## Feature 구현 estimation
[[mimtokFeature]]

[[MimTok source reference]]
----
- tags: #mimtok
