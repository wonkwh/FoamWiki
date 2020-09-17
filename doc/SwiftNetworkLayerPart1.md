# Swift Network Layer, Part 1: Designing The API
- [Link](https://blog.malcolmk.com/swift-network-layer-part-1-designing-the-api-ckewm4w9p002dggs1hqfm03lm)

## Part 0
 - [Part0](https://blog.malcolmk.com/swift-network-layer-series-ckecgql5s00ko9as121oy6hdu)
  ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1598513873066/8kVP01zev.png?auto=format&q=60)
  
## API Design
```swift
	let client = Client<ITunesApiRoute>()
	let publisher = client.request(SearchResult.self, from: .searchResult)

```
----
## Code
### Folder Structure
![](https://cdn.hashnode.com/res/hashnode/image/upload/v1599717007227/BZGqLB2b3.png?auto=format&q=60)
### Model
- `"https://itunes.apple.com/search?term=\(searchTerm)&offset=0&limit=20"`


```swift
struct SearchResult: Decodable {
    let resultCount: Int
    let results: [Result]
}

struct Result: Decodable {
    let trackId: Int
    let trackName: String
    let primaryGenreName: String
    var averageUserRating: Float?
    var screenshotUrls: [String]?
    let artworkUrl100: String // app icon
    var formattedPrice: String?
    var description: String?
    var releaseNotes: String?
    var artistName: String?
    var collectionName: String?
}
```

### Client
```swift
protocol HTTPClient {
    associatedtype Route: RouteProvider
    func request<T: Decodable>(_ model: T.Type,
                               from route: Route,
                               urlSession: URLSession) -> AnyPublisher<HTTPResponse<T>, HTTPError>
}

final class Client<Route: RouteProvider>: HTTPClient {

    func request<T>(_ model: T.Type,
                    from route: Route,
                    urlSession: URLSession = .shared) -> AnyPublisher<HTTPResponse<T>, HTTPError> {
        fatalError("Not implemented yet")
    }

}

```


### RouteProvider
```swift
protocol RouteProvider {
    var baseURL: URL { get }
    var path: String { get }
    var method: HTTPMethod { get }
    var headers: [String: String] { get }
}

```
#### PlaceHolderRoute
```swift
enum ITunesApiRoute {
    case searchResult
}


extension ITunesApiRoute: RouteProvider {
    var baseURL: URL {
        guard let url = URL(string: "https://itunes.apple.com") else {
            fatalError("Base URL could not be configured.")
        }
        return url
    }

    var path: String {
        switch self {
        case .searchResult: return "/search"
        }
    }

    var method: HTTPMethod {
        switch self {
        //"https://itunes.apple.com/search?term=\(searchTerm)&offset=0&limit=20"
        case .searchResult:
            return .get(["term": "IU", "offset": "0", "limit": "20"])
        }
    }

    var headers: [String : String] {
        return [:]
    }
}

```

### HTTPMethod
```swift
enum HTTPMethod {
    case get([String: String]? = nil)
    case put(HTTPContentType)
    case post(HTTPContentType)
    case patch(HTTPContentType)
    case delete

    public var rawValue: String {
        switch self {
        case .get: return "GET"
        case .put: return "PUT"
        case .post: return "POST"
        case .patch: return "PATCH"
        case .delete: return "DELETE"
        }
    }
}
```
- The GET case will enable us to pass in URL query parameters with our request. 
- We can construct use  `.get(["page": 2])` to construct a URL like: 
	`http://api.test.com/content?page=2`

### HTTPContentType
```swift
enum HTTPContentType {
    case json(Encodable?)
    case urlEncoded(EncodableDictionary)

    var headerValue: String {
        switch self {
        case .json: return "application/json"
        case .urlEncoded: return "application/x-www-form-urlencoded"
        }
    }
}
```
### HTTPError
```swift
struct HTTPError: Error {
    private enum Code: Int {
        case unknown                        = -1
        case networkUnreachable             = 0
        case unableToParseResponse          = 1
        case badRequest                     = 400
        case internalServerError            = 500
    }

    let route: RouteProvider?
    let response: HTTPURLResponse?
    let error: Error?

    var message: String {
        switch Code(rawValue: response?.statusCode ?? -1) {
        case .unknown:
            return "Something went wrong"
        case .networkUnreachable:
            return "Please check your internet connectivity"
        default:
            return systemMessage
        }
    }

    private var systemMessage: String {
        HTTPURLResponse.localizedString(forStatusCode: response?.statusCode ?? 0)
    }
}
```
### Encodable Dictionary
- URL encodes Dictionary keys and values and returns them as Data. 
	- Input: [“Name”: “kwanghyun W”, “Emoji”: “🍩”]
	- Output: Name=kwanghyun%20W&Emoji=%F0%9F%8D%A9 (string representation of data)

```swift

protocol EncodableDictionary {
    var asURLEncodedString: String? { get }
    var asURLEncodedData: Data? { get }
}

extension Dictionary: EncodableDictionary {

    var asURLEncodedString: String? {
        var pairs: [String] = []
        for (key, value) in self {
            pairs.append("\(key)=\(value)")
        }
        return pairs
            .joined(separator: "&")
            .addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed)
    }

    var asURLEncodedData: Data? { asURLEncodedString?.data(using: .utf8) }
}



```
----
## Resource
- Swift Enum Documentation
- [HTTP in Swift](https://davedelong.com/blog/2020/06/27/http-in-swift-part-1/)
- [URLs in Swift](https://www.avanderlee.com/swift/url-components/) by Antoine van der Lee
- [Why Swift enums with associated values can’t have a rawValue](https://medium.com/@PhiJay/why-swift-enums-with-associated-values-cannot-have-a-raw-value-21e41d5ec11) by Mischa Hildebrand

---
- tags : #networking #combine 