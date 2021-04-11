# replace MHBeauty with ARGEAR SDK


## 시어스랩 요청사항

- 카메라 관련 기존 코드를 이용하기 위해서 추가 사항

```
/**
  * 
  * @param texture texture ID
  * @param width The width of the texture
  * @param height The height of the texture
  * @return returns the texture to the SDK
  * Note: The texture type is GL_TEXTURE_2D, and . The texture format is GL_RGBA
  */ 
- (GLuint)onPreProcessTexture:(GLuint)texture width:(CGFloat)width height:(CGFloat)height
```
- 위 형식의 protocol 을 구현해야 하므로 기존 ARGear sdk에서 
- 
- `ARGSession` 에 기존 아래와 같은 method로 camera data를 받아 처리
```
- (BOOL)updateSampleBuffer:(CMSampleBufferRef)sampleBuffer
                fromConnection:(AVCaptureConnection* __nullable)connection;
```

- 아래와 유사한 형태의 function 을 `ARGSession` 에 추가 가능한지 요청
```
/**
  * 
  * @param texture texture ID
  * @param width The width of the texture
  * @param height The height of the texture
  * Note: The texture type is GL_TEXTURE_2D, and . The texture format is GL_RGBA
  */ 
- (BOOL)updateGlTexture:(GLuint)texture width:(CGFloat)width height:(CGFloat)height;
```

## 외주 업체 요구사항
### case 1
   시어스랩이 추가 API 제공시
-  `MHBeautySDK.framework` 를 제거하고 `ARGear.framework`로 교체
   -  `TXLiteAVSDK` 는 그대로 재사용.
   -  `TCVideoRecordController` 에서 제거후 `TXVideoCustomProcessDelegate` 을 구현
   -  ARGear 에서 제공하는 추가 필터및 ar sticker, 등의 기능이 정상동작하여야 한다. 
-  `MHSDK` 에 포함되어 있던 `MHUI` 제거 후 재개발
   -  현 zeplin UI/UX 디자인 가이드에 맞게 UI 수정
-  기존 camera record 기능, 합성, 편집기능에 side effect 가 발생하면 안된다. 

-  task 별로 feature  branch 생성후 github pull request 기능을 이용하여 code review 후 toypudding ios dev team에서 merge 진행
   -  coding convention 문서및 `swiftlint`, `swiftformat` config file 공유
   -  `github action` CI service를 이용해 build, warning, lint warning 제거
   -  모든 추가 구현은 swift 5.0 Language 사용
   -  ios 13 이상이므로 SwiftUI, Combine 사용가능
   -  objective-c 사용불가


### case 2
- 추가 API 사용하지 않고 sample code로만 

-  `MHBeautySDK.framework` 를 제거하고 `ARGear.framework`로 교체
   -  `TXLiteAVSDK` 에서 `TXUGCRecord.h` 의 기능을 제거 하고 ARGear sample 을 참조하여 camera recording 기능 추가개발
      -  sample에서 제공하지 않는 recordSpeed 기능, setBGM 기능 추가구현 필요
   -  ARGear 에서 제공하는 추가 필터및 ar sticker, 등의 기능이 정상동작하여야 한다. 
-  `MHSDK` 에 포함되어 있던 `MHUI` 제거 후 재개발
   -  현 zeplin UI/UX 디자인 가이드에 맞게 UI 수정
-  기존 camera record 기능, 합성, 편집기능에 side effect 가 발생하면 안된다. 

-  task 별로 feature  branch 생성후 github pull request 기능을 이용하여 code review 후 toypudding ios dev team에서 merge 진행
   -  coding convention 문서및 `swiftlint`, `swiftformat` config file 공유
   -  `github action` CI service를 이용해 build, warning, lint warning 제거
   -  모든 추가 구현은 swift 5.0 Language 사용
   -  ios 13 이상이므로 SwiftUI, Combine 사용가능
   -  objective-c 사용불가


----
- tags: #mimtok 