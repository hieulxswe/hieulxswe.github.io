---
title: "[vn] Thiết kế kiến trúc iOS app: tách UI, business logic và data layer"
date: 2025-08-21 11:00:00 +1100
author: Hieu Xuan Leu
layout: post
permalink: /ios-app-architecture-presentation-domain-data-layer/
categories: ios
tags: [ios, swift, swiftui, architecture, mvvm, software engineering]
published: true
---

Khi một iOS app còn nhỏ, việc viết toàn bộ logic trong `ViewController` hoặc `View` có thể vẫn chạy ổn.

Một màn hình gọi API, parse response, update UI, handle loading, show error, navigate sang màn hình khác. Tất cả nằm trong một file. Ban đầu nhìn có vẻ nhanh và đơn giản.

Nhưng khi app lớn dần, cách viết này thường tạo ra nhiều vấn đề:

- UI layer chứa quá nhiều logic.
- Code khó test.
- Business logic bị lặp lại ở nhiều màn hình.
- Network code bị trộn với presentation logic.
- Một thay đổi nhỏ dễ ảnh hưởng tới nhiều phần không liên quan.
- File `ViewController` hoặc `ViewModel` ngày càng lớn và khó maintain.

Vì vậy, một iOS app production thường nên được chia thành nhiều layer rõ ràng hơn.

## Một kiến trúc phổ biến

Một cách tổ chức khá phổ biến trong iOS là chia app thành các layer sau:

```text
Presentation Layer
↓
Domain Layer
↓
Data Layer
```

Mỗi layer có một trách nhiệm riêng.

## Presentation Layer

Presentation layer là nơi xử lý UI và state hiển thị.

Với SwiftUI, layer này thường bao gồm:

```text
View
ViewModel
UI State
Navigation State
```

Ví dụ:

```swift
struct UserProfileView: View {
    @StateObject private var viewModel: UserProfileViewModel

    var body: some View {
        Group {
            switch viewModel.state {
            case .loading:
                ProgressView()

            case .loaded(let user):
                VStack {
                    Text(user.name)
                    Text(user.email)
                }

            case .failed(let message):
                Text(message)
            }
        }
        .task {
            await viewModel.loadUser()
        }
    }
}
```

`View` chỉ nên tập trung vào việc render UI dựa trên state.

Nó không nên biết API endpoint là gì, request được tạo như thế nào, response decode ra sao, hoặc dữ liệu được cache ở đâu.

Những phần đó nên nằm ở layer khác.

## ViewModel

`ViewModel` chịu trách nhiệm chuyển dữ liệu từ domain layer thành state mà UI có thể render.

Ví dụ:

```swift
@MainActor
final class UserProfileViewModel: ObservableObject {
    enum State {
        case loading
        case loaded(UserProfileViewData)
        case failed(String)
    }

    @Published private(set) var state: State = .loading

    private let getUserProfile: GetUserProfileUseCase

    init(getUserProfile: GetUserProfileUseCase) {
        self.getUserProfile = getUserProfile
    }

    func loadUser() async {
        state = .loading

        do {
            let user = try await getUserProfile.execute()
            state = .loaded(
                UserProfileViewData(
                    name: user.name,
                    email: user.email
                )
            )
        } catch {
            state = .failed("Unable to load user profile.")
        }
    }
}
```

Ở đây, `ViewModel` không gọi trực tiếp `URLSession`.

Thay vào đó, nó gọi một use case: `GetUserProfileUseCase`.

Điều này giúp `ViewModel` dễ test hơn, vì mình có thể mock use case trong unit test.

## Domain Layer

Domain layer chứa business logic chính của app.

Layer này không nên phụ thuộc vào SwiftUI, UIKit, URLSession, Core Data, hay bất kỳ framework cụ thể nào nếu không thật sự cần thiết.

Ví dụ:

```swift
struct User {
    let id: String
    let name: String
    let email: String
}

protocol UserRepository {
    func getProfile() async throws -> User
}

final class GetUserProfileUseCase {
    private let repository: UserRepository

    init(repository: UserRepository) {
        self.repository = repository
    }

    func execute() async throws -> User {
        try await repository.getProfile()
    }
}
```

Điểm quan trọng ở đây là `GetUserProfileUseCase` chỉ biết tới `UserRepository` protocol.

Nó không quan tâm dữ liệu đến từ API, database, cache hay mock data.

Đây là dependency inversion: domain layer định nghĩa interface, data layer implement interface đó.

## Data Layer

Data layer chịu trách nhiệm lấy dữ liệu từ bên ngoài app hoặc từ local storage.

Ví dụ:

```swift
final class RemoteUserRepository: UserRepository {
    private let apiClient: APIClient

    init(apiClient: APIClient) {
        self.apiClient = apiClient
    }

    func getProfile() async throws -> User {
        let response: UserProfileResponse = try await apiClient.request(
            endpoint: UserEndpoint.profile
        )

        return User(
            id: response.id,
            name: response.name,
            email: response.email
        )
    }
}
```

Data layer có thể chứa:

```text
API Client
DTO / Response Model
Repository Implementation
Local Cache
Database
Keychain Storage
```

Một lỗi phổ biến là dùng trực tiếp response model từ API trong UI.

Ví dụ:

```swift
struct UserProfileResponse: Decodable {
    let id: String
    let fullName: String
    let emailAddress: String
}
```

Nếu UI dùng trực tiếp `UserProfileResponse`, UI sẽ bị phụ thuộc vào format của backend.

Khi backend đổi field từ `fullName` sang `name`, nhiều phần UI có thể bị ảnh hưởng.

Cách tốt hơn là map response model sang domain model:

```swift
extension UserProfileResponse {
    func toDomain() -> User {
        User(
            id: id,
            name: fullName,
            email: emailAddress
        )
    }
}
```

Sau đó app dùng `User`, không dùng trực tiếp `UserProfileResponse`.

## Vì sao nên tách layer?

Việc tách layer giúp app dễ maintain hơn.

Thứ nhất, UI thay đổi không làm ảnh hưởng tới data layer.

Ví dụ, nếu chuyển từ UIKit sang SwiftUI, domain layer và data layer có thể được giữ lại phần lớn.

Thứ hai, business logic dễ test hơn.

Use case có thể được test mà không cần render UI hoặc gọi API thật.

```swift
final class MockUserRepository: UserRepository {
    var result: Result<User, Error>

    init(result: Result<User, Error>) {
        self.result = result
    }

    func getProfile() async throws -> User {
        try result.get()
    }
}
```

Unit test:

```swift
func testGetUserProfileSuccess() async throws {
    let expectedUser = User(
        id: "1",
        name: "Hieu",
        email: "hieu@example.com"
    )

    let repository = MockUserRepository(result: .success(expectedUser))
    let useCase = GetUserProfileUseCase(repository: repository)

    let user = try await useCase.execute()

    XCTAssertEqual(user.id, "1")
    XCTAssertEqual(user.name, "Hieu")
}
```

Thứ ba, code dễ thay đổi hơn.

Nếu sau này muốn thêm cache, mình có thể thay đổi repository implementation mà không cần sửa ViewModel.

```swift
final class CachedUserRepository: UserRepository {
    private let remoteRepository: UserRepository
    private let cache: UserCache

    init(remoteRepository: UserRepository, cache: UserCache) {
        self.remoteRepository = remoteRepository
        self.cache = cache
    }

    func getProfile() async throws -> User {
        if let cachedUser = await cache.getUser() {
            return cachedUser
        }

        let user = try await remoteRepository.getProfile()
        await cache.save(user)
        return user
    }
}
```

UI vẫn gọi cùng một use case, use case vẫn gọi cùng một protocol.

Phần thay đổi chỉ nằm ở data layer.

## Dependency Injection

Để các layer hoạt động độc lập, app cần dependency injection.

Ví dụ đơn giản:

```swift
final class AppContainer {
    lazy var apiClient = APIClient()

    lazy var userRepository: UserRepository = RemoteUserRepository(
        apiClient: apiClient
    )

    lazy var getUserProfileUseCase = GetUserProfileUseCase(
        repository: userRepository
    )

    @MainActor
    func makeUserProfileViewModel() -> UserProfileViewModel {
        UserProfileViewModel(
            getUserProfile: getUserProfileUseCase
        )
    }
}
```

Trong app nhỏ, manual dependency injection như trên thường là đủ.

Không nhất thiết phải dùng dependency injection framework nếu app chưa đủ phức tạp.

Điều quan trọng là object không tự tạo dependency bên trong nó.

Ví dụ nên tránh:

```swift
final class UserProfileViewModel: ObservableObject {
    private let apiClient = APIClient()
}
```

Cách này khiến `ViewModel` khó test, vì không thể thay `APIClient` bằng mock.

Nên inject dependency từ bên ngoài:

```swift
final class UserProfileViewModel: ObservableObject {
    private let getUserProfile: GetUserProfileUseCase

    init(getUserProfile: GetUserProfileUseCase) {
        self.getUserProfile = getUserProfile
    }
}
```

## Khi nào architecture trở nên quá mức?

Không phải app nào cũng cần Clean Architecture đầy đủ.

Nếu app rất nhỏ, ít màn hình, ít business logic, việc tạo quá nhiều layer có thể làm code phức tạp không cần thiết.

Một số dấu hiệu cho thấy nên tách architecture rõ hơn:

- App có nhiều màn hình dùng chung business logic.
- App cần unit test nghiêm túc.
- App có nhiều data source: API, local database, cache.
- App có logic phức tạp hơn CRUD đơn giản.
- Team có nhiều developer cùng làm trên một codebase.
- Feature thay đổi thường xuyên.

Architecture tốt không phải là có nhiều folder hoặc nhiều protocol.

Architecture tốt là khi code dễ hiểu, dễ test, dễ thay đổi, và giảm coupling giữa các phần không liên quan.

## Kết luận

Trong iOS development, viết app chạy được chỉ là bước đầu.

Để app có thể phát triển lâu dài, codebase cần được tổ chức rõ ràng.

Một cách tiếp cận thực tế là tách app thành:

```text
Presentation Layer: View, ViewModel, UI State
Domain Layer: Entity, Use Case, Repository Protocol
Data Layer: API, DTO, Repository Implementation, Cache
```

Cách tách này giúp UI không phụ thuộc trực tiếp vào API, business logic dễ test hơn, và data source có thể thay đổi mà không ảnh hưởng tới toàn bộ app.

Không cần áp dụng architecture một cách máy móc.

Nhưng khi app bắt đầu lớn, việc tách trách nhiệm rõ ràng sẽ giúp codebase dễ maintain hơn rất nhiều.

Một iOS app tốt không chỉ cần UI đẹp và feature đầy đủ.

Nó còn cần một nền tảng code đủ rõ ràng để team có thể tiếp tục phát triển, debug, test và thay đổi trong thời gian dài.

#iOS #Swift #SwiftUI #Architecture #MVVM #SoftwareEngineering
