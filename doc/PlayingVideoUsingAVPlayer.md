# Playing Video Using AVPlayer

## AVAssetResourceLoaderDelegate
- [[UnderstandingAVAssetResourceLoaderDelegate]]

## Generation Video Thumbnails 
- [Generating Video Thumbnails at Runtime in iOS (Swift) | by Paul Wallace | Medium](https://medium.com/@PaulWall43/generating-video-thumnails-at-runtime-in-ios-swift-a2b092301c9a)
```swift
func createThumbnailOfVideoFromRemoteUrl(url: String) -> UIImage? {
    let asset = AVAsset(url: URL(string: url)!)
    let assetImgGenerate = AVAssetImageGenerator(asset: asset)
    assetImgGenerate.appliesPreferredTrackTransform = true
    //Can set this to improve performance if target size is known before hand
    //assetImgGenerate.maximumSize = CGSize(width,height)
    let time = CMTimeMakeWithSeconds(1.0, 600)
    do {
        let img = try assetImgGenerate.copyCGImage(at: time, actualTime: nil)
        let thumbnail = UIImage(cgImage: img)
        return thumbnail
    } catch {
      print(error.localizedDescription)
      return nil
    }
}
```
## Reference 
### AVAssetResourceLoaderDelegate
- [Understanding AVAssetResourceLoaderDelegate | by Eugene Zozulia | Medium](https://medium.com/@EugeneZZI/understanding-avassetresourceloaderdelegate-b90b3fe2c059)
- [AVPlayer practices local Cache functionality | by ZhgChgLi | ZRealm Dev. | Jan, 2021 | Medium](https://medium.com/zrealm-ios-dev/avplayer-%E5%AF%A6%E8%B8%90%E6%9C%AC%E5%9C%B0-cache-%E5%8A%9F%E8%83%BD%E5%A4%A7%E5%85%A8-6ce488898003)
- [SZAVPlayer | LuoYe Blog (caisanze.com)](https://caisanze.com/post/swift-avplayer/)
- [Design and implementation of iOS AVPlayer video caching | 楚权的世界 (chuquan.me)](http://chuquan.me/2019/12/03/ios-avplayer-support-cache/)

----
- tags: #avplayer #cache