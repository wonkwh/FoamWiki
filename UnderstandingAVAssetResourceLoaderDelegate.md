# # Understanding AVAssetResourceLoaderDelegate

##  simple save and export video file 
- 비디오 파일을 로컬 disk 에 저장하는 가장 빠르고 쉬운 방법
   - `AVAssetExportSession`  을 이용하면 된다. 
   - quality preset  , _outputURL_ , _outputFileType_ 을 설정 
   - 그 후 `exportAsynchronously()` 호출


```swift
func exportSession(forAsset asset: AVURLAsset) {  
   if !asset.isExportable { return }

   guard let exporter = AVAssetExportSession(asset: composition,                      presetName: AVAssetExportPresetHighestQuality) else {  
      print("Failed to create export session")  
      return  
  }

  let fileName = asset.url.lastPathComponent  
  let documentsDirectory = FileManager.default.urls(for: .documentDirectory,           in: .userDomainMask).last!  
  let outputURL = documentsDirectory.appendingPathComponent(fileName)
   
  if FileManager.default.fileExists(atPath: outputURL.path) {  
     do {  
       try FileManager.default.removeItem(at: outputURL)  
     } catch let error {  
      print("Failed to delete file with error: \\(error)")  
    }  
  }

   exporter.outputURL = outputURL  
   exporter.outputFileType = AVFileType.mp4
   
   exporter.exportAsynchronously {  
      print("Exporter did finish")  
      if let error = exporter.error {  
          print("Error \\(error)")  
      }  
  }  
}
```

- 그러나 유저들은 podcast, streaming app 에서 오늘 보고 pause 하고 내일 다시 play 할수 있다.
- _AVAssetExportSession_.  으로 전체 파일을 저장하는 것만으로 충분하지 않다. 
- `AVAssetResourceLoaderDelegate` 을 이용하자

##  Intercepting original AVAssetResourceLoadingRequests
- _videoURL_ to the remote file.
- _AVPlayer_ to play content in the App. We use it with the wrapper _MediaPlayer_ to make the life a bit easier.
*   And `SimpleResourceLoaderDelegate` which we need to implement. It will do all the magic for us.

View controller has a separate method to create and setup _MediaPlayer_.

```swift
  func createPlayer() -> MediaPlayer {
    guard let url = URL(string: "https://www.quirksmode.org/html5/videos/big\_buck\_bunny.mp4") else {
      fatalError("Wrong video url.")
    }
    
    self.loaderDelegate = SimpleResourceLoaderDelegate(withURL: url)
    let videoAsset = AVURLAsset(url: self.loaderDelegate!.streamingAssetURL)
    videoAsset.resourceLoader.setDelegate(self.loaderDelegate, queue: DispatchQueue.main)
    self.loaderDelegate?.completion = { localFileURL in
      if let localFileURL = localFileURL {
        print("Media file saved to: \\(localFileURL)")
      } else {
        print("Failed to download media file.")
      }
    }
    
    let player = MediaPlayer(withAsset: videoAsset)
    player.delegate = self
    player.playerView = self.playerView
    
    return player
  }
```


- First, we create a custom loader delegate.  
- we can create _AVURLAsset_ with our videoURL. 
- Attention, we should not pass the original URL like `https://www.some.com/short.mp4`, but a bit modified `https-demoloader://www.some.com/short.mp4` if we want to our SimpleResourceLoaderDelegate be invoked.  
- After we can assign _loaderDelegate_ to _videoAsset_.  
- And finally, create _AVPlayer_ (in our case wrapper _MediaPlayer_) with the _videoAsset_.

Let’s go to the _AVAssetResourceLoaderDelegate_ implementation and look what’s going on in it. Right now we are interested in just two methods:

*   First one invokes when some data should be loaded
```swift
func resourceLoader(_ resourceLoader: AVAssetResourceLoader,                                    shouldWaitForLoadingOfRequestedResource loadingRequest:                       AVAssetResourceLoadingRequest) -> Bool
```
*   Second invokes when some previous request was canceled
```swift
func resourceLoader(_ resourceLoader: AVAssetResourceLoader, 
             didCancel loadingRequest: AVAssetResourceLoadingRequest)
```

- When the App starts playing the video first method invokes when the player asks about a new batch of data. 
- With our delegate, we intercept requests and load all data with our self and provide back to the origin `AVAssetResourceLoadingRequest` data which we have. 
- All provided data is our responsibility from now.  


Here’s a diagram of `AVAssetResourceLoadingRequests`:

![Image for post](AVAssetResourceLoadingRequests1.jpeg)

- As we can see `AVAssetResourceLoader` sends few requests to get the info about the file. 
- And junks of data with _requestedOffset_ and _requestedLength_.
- We need to save all valid _AVAssetResourceLoadingRequest_ somewhere. 
- Let’s use for simple Array for that

```swift
private var loadingRequests = [AVAssetResourceLoadingRequest]()
```

Also, we need to handle request cancellation. In this case, we will just remove the saved origin request from the array.

```swift
if let index = self.loadingRequests.firstIndex(of: loadingRequest) {  
    self.loadingRequests.remove(at: index)  
}
```

## Downloading real data
- Now we need to download all the required data for the video. 
- We can create `URLSession` with the single `URLSessionDataTask` to keep everything simple in this example. 
- Also, we need to implement a few methods from `URLSessionTaskDelegate` and `URLSessionDataDelegate`.  

We need these methods:

- To process the response with file info and fill with it info request from origin `AVAssetResourceLoadingRequest`
```swift
func urlSession(_ session: URLSession, 
                task: URLSessionTask,
                didCompleteWithError error: Error?)
```
 - To process data from our video file. We will store all the data in memory for now. 
 - It’s not the best solution, but it is enough for a short video from our example.

```swift
func urlSession(_ session: URLSession, 
                dataTask: URLSessionDataTask, 
                didReceive data: Data)
```
 - To process error from task loading or successful completion of the loading
```swift
func urlSession(_ session: URLSession, 
                      task: URLSessionTask,                     didCompleteWithError error: Error?)
```
Now we can create our `URLSession` with code

```swift
func createURLSession() -> URLSession {  
   let config = URLSessionConfiguration.default  
   let operationQueue = OperationQueue()  
   operationQueue.maxConcurrentOperationCount = 1  
   return URLSession(configuration: config, delegate: self, delegateQueue: operationQueue)  
 }
```

 - All received data we will save in the variable

```swift
private lazy var mediaData = Data()
```

 - And then we need to create our data task when we will receive the first `AVAssetResourceLoadingRequest`

```swift
if self.urlSession == nil {  
   self.urlSession = self.createURLSession()  
   let task = self.urlSession!.dataTask(with: self.url)  
   task.resume()  
}
```

- So we have intercepted all origin requests from the resource loader and have created our data task to download video data. 
- So now we can send chunks of downloaded data to the saved _`AVAssetResourceLoadingDataRequest`. 
- But don’t forget about `AVAssetResourceLoadingContentInformationRequest` too. 
- When we received the `URLResponse` from the `URLSessionDataTask` we should fill with it the `AVAssetResourceLoadingContentInformationRequest`.

 
It looks like

```swift
func fillInfoRequest(request: inout AVAssetResourceLoadingRequest, 
                     response: URLResponse) {  
   request.contentInformationRequest?.isByteRangeAccessSupported = true  
   request.contentInformationRequest?.contentType = response.mimeType  
   request.contentInformationRequest?.contentLength = response.expectedContentLength  
 }
```

And every time when we receive new data from data task we can check our saved origin _loadingRequests_ and fill them with available data. And maybe finish some of them.

- For example, we have this case. Some data already downloaded. `AVAssetResourceLoadingDataRequest` has some _requestedOffset_ and _requestedLength_. 
- Also, we already sent some data to it, so _currentOffset_ is not 0 too. Looks like on the diagram

![Image for post](AVAssetResourceLoadingDataRequest2.jpg)

Now we need to calculate which pease of data available for the origin request.  
Logic looks like

![Image for post](flowDownloadVideo.jpeg)

Required steps:

*   Check with the length of downloaded data we can pass to the request
*   Respond with a chunk of data
*   Check if we sent all required data to the request, and finish it if so

In code
```swift
  func checkAndRespond(forRequest dataRequest: AVAssetResourceLoadingDataRequest) -> Bool {
    let downloadedData          = self.mediaData
    let downloadedDataLength    = Int64(downloadedData.count)
    
    let requestRequestedOffset  = dataRequest.requestedOffset
    let requestRequestedLength  = Int64(dataRequest.requestedLength)
    let requestCurrentOffset    = dataRequest.currentOffset
    
    if downloadedDataLength < requestCurrentOffset {
      return false
    }
    
    let downloadedUnreadDataLength  = downloadedDataLength - requestCurrentOffset
    let requestUnreadDataLength     = requestRequestedOffset + requestRequestedLength - requestCurrentOffset
    let respondDataLength           = min(requestUnreadDataLength, downloadedUnreadDataLength)
    
    dataRequest.respond(with: downloadedData.subdata(in: Range(NSMakeRange(Int(requestCurrentOffset), Int(respondDataLength)))!))
    
    let requestEndOffset = requestRequestedOffset + requestRequestedLength
    
    return requestCurrentOffset >= requestEndOffset
  }
```

##  Finishing loading
When we finish with downloading the data, we can save it to a local file for example.

```swift
  func saveMediaDataToLocalFile() -> URL? {
    guard let docFolderURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first else {
      return nil
    }
    
    let fileName = self.url.lastPathComponent
    let fileURL = docFolderURL.appendingPathComponent(fileName)
    
    if FileManager.default.fileExists(atPath: fileURL.path) {
      do {
        try FileManager.default.removeItem(at: fileURL)
      } catch let error {
        print("Failed to delete file with error: \\(error)")
      }
    }
    
    do {
      try self.mediaData.write(to: fileURL)
    } catch let error {
      print("Failed to save data with error: \\(error)")
      return nil
    }
    
    return fileURL
  }
```

- That’s all with the basics of the `AVAssetResourceLoaderDelegate`. 
- 

## Reference
- [Understanding AVAssetResourceLoaderDelegate | by Eugene Zozulia | Medium](https://medium.com/@EugeneZZI/understanding-avassetresourceloaderdelegate-b90b3fe2c059)
- [AVPlayer practices local Cache functionality/AVQueuePlayer with AVURLAsset… | by ZhgChgLi | ZRealm Dev. | Jan, 2021 | Medium](https://medium.com/zrealm-ios-dev/avplayer-%E5%AF%A6%E8%B8%90%E6%9C%AC%E5%9C%B0-cache-%E5%8A%9F%E8%83%BD%E5%A4%A7%E5%85%A8-6ce488898003)
- [Jared Sinclair | Implementing AVAssetResourceLoaderDelegate: a How-To Guide](https://jaredsinclair.com/2016/09/03/implementing-avassetresourceload.html)
- 
----
- tags : #avplayer 