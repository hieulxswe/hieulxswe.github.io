---
title: "[vn] Tối ưu hóa Fetching Data giữa Firebase và ứng dụng iOS"
date: 2025-04-25T00:00:00+00:00
author: Hieu Xuan Leu
layout: post
permalink: /fetching-data-firebase-and-ios/
categories: ios
tags: [fetching data, ios, firebase, swift, swiftui]
image: ../assets/images/swiftandfirebase.png
---

Trong thời đại số hóa hiện nay, việc phát triển ứng dụng di động đòi hỏi không chỉ giao diện đẹp mắt mà còn khả năng quản lý dữ liệu hiệu quả. Firebase của Google đã trở thành một trong những giải pháp backend phổ biến nhất cho các nhà phát triển iOS, cung cấp hàng loạt công cụ để xây dựng, quản lý và mở rộng ứng dụng. Tuy nhiên, việc tối ưu hóa cách ứng dụng iOS của bạn tương tác với Firebase có thể tạo ra sự khác biệt lớn về hiệu suất, trải nghiệm người dùng và chi phí vận hành.

Trong bài viết này, tôi sẽ chia sẻ những bài học kinh nghiệm về việc tối ưu hóa fetching data giữa Firebase và iOS app, dựa trên trải nghiệm xây dựng một ứng dụng loyalty cho một chuỗi cửa hàng cà phê. Những kỹ thuật và giải pháp được trình bày ở đây đều đã được triển khai và kiểm nghiệm trong môi trường thực tế, giúp nâng cao hiệu suất và trải nghiệm người dùng đáng kể.

## Nội dung bài viết

1. [Kiến trúc quản lý dữ liệu trong ứng dụng iOS-Firebase](#1-kiến-trúc-quản-lý-dữ-liệu-trong-ứng-dụng-ios-firebase)
2. [Các thách thức khi làm việc với Firestore](#2-các-thách-thức-khi-làm-việc-với-firestore)
3. [Chiến lược tối ưu hóa fetching data](#3-chiến-lược-tối-ưu-hóa-fetching-data)
4. [Xử lý offline và đồng bộ hóa dữ liệu](#4-xử-lý-offline-và-đồng-bộ-hóa-dữ-liệu)
5. [Monitoring và debugging](#5-monitoring-và-debugging)
6. [Kết luận](#6-kết-luận)

## 1. Kiến trúc quản lý dữ liệu trong ứng dụng iOS-Firebase

### 1.1 Mô hình quản lý dữ liệu trong ứng dụng iOS

Trong dự án của chúng tôi, chúng tôi đã thiết kế kiến trúc quản lý dữ liệu theo mô hình ba lớp:

1. **Presentation Layer**: SwiftUI views (`DashboardView`, `ProfileView`, `QRCodeView`, v.v...)
2. **Business Logic Layer**: `UserManager` class
3. **Data Access Layer**: `FirebaseService` singleton

Mỗi lớp có trách nhiệm rõ ràng:
- **Presentation Layer**: Hiển thị dữ liệu và tương tác với người dùng
- **Business Logic Layer**: Xử lý logic nghiệp vụ, quản lý trạng thái ứng dụng
- **Data Access Layer**: Giao tiếp với Firestore, xử lý truy vấn dữ liệu

Thiết kế này mang lại nhiều lợi ích:
- **Separation of concerns**: Mỗi lớp có nhiệm vụ riêng biệt
- **Testability**: Dễ dàng kiểm thử từng lớp một cách độc lập
- **Maintainability**: Dễ dàng bảo trì và mở rộng mã nguồn

Hãy xem code của lớp `FirebaseService` làm ví dụ:

```swift
class FirebaseService {
    private let db = Firestore.firestore()
    static let shared = FirebaseService()
    
    private init() {}
    
    func fetchUserProfile(userId: String) async throws -> UserProfile? {
        print("Fetching user profile for userId: \(userId)")
        
        let snapshot = try await db.collection("users")
            .whereField("appleUserId", isEqualTo: userId)
            .getDocuments()
        
        guard let document = snapshot.documents.first else {
            print("No user found with ID: \(userId)")
            return nil
        }
        
        // Xử lý và trả về dữ liệu user
        // ...
    }
    
    // Các phương thức khác...
}
```

### 1.2 Đồng bộ hóa dữ liệu giữa các lớp

Điểm quan trọng trong kiến trúc này là cách đồng bộ hóa dữ liệu giữa tầng giao diện và tầng dữ liệu. Chúng tôi sử dụng kết hợp:

- **Combine framework** thông qua `@Published` properties trong `UserManager`
- **Swift Concurrency** (async/await) trong `FirebaseService`
- **EnvironmentObject** để truyền state xuống view hierarchy

Điều này giúp giao diện người dùng tự động cập nhật khi dữ liệu thay đổi, đồng thời xử lý bất đồng bộ một cách hiệu quả.

## 2. Các thách thức khi làm việc với Firestore

### 2.1 Limitasi của Firebase Firestore

Khi xây dựng ứng dụng với Firebase Firestore, chúng tôi đã gặp phải một số thách thức đáng chú ý:

1. **Giới hạn số lượng query**: Firestore tính phí dựa trên số lượng đọc/ghi
2. **Độ trễ mạng**: Fetching data từ cloud có thể chậm trong điều kiện mạng không tốt
3. **Cấu trúc dữ liệu phẳng**: Firestore không phải là cơ sở dữ liệu quan hệ, nên việc mô hình hóa dữ liệu khác với SQL truyền thống

### 2.2 Ảnh hưởng tới trải nghiệm người dùng

Những thách thức này có thể ảnh hưởng trực tiếp đến trải nghiệm người dùng:

- **Thời gian chờ**: Người dùng phải đợi dữ liệu tải về
- **Sử dụng dữ liệu lỗi thời**: Nếu không cập nhật kịp thời từ server
- **Tăng chi phí**: Quá nhiều truy vấn không cần thiết làm tăng chi phí vận hành

## 3. Chiến lược tối ưu hóa fetching data

### 3.1 Sử dụng caching thông minh

Một trong những chiến lược hiệu quả nhất là sử dụng caching. Trong `UserManager`, chúng tôi đã triển khai caching hai lớp:

```swift
// UserDefaults cho dữ liệu không nhạy cảm
userDefaults.set(user.name, forKey: "userName_\(user.id)")
userDefaults.set(self.points, forKey: "userPoints_\(user.id)")

// KeychainHelper cho dữ liệu nhạy cảm
keychainHelper.save(data, service: "appIdentifier", account: "userIdentifier")
```

Việc phân chia dữ liệu giữa UserDefaults và Keychain mang lại nhiều lợi ích:

1. **Bảo mật phân tầng**: Dữ liệu nhạy cảm được bảo vệ bởi Keychain
2. **Hiệu suất tối ưu**: UserDefaults nhanh hơn cho dữ liệu thường xuyên truy cập
3. **Độ bền của dữ liệu**: Dữ liệu được duy trì qua các lần khởi động ứng dụng

Chúng tôi cũng thiết lập cơ chế hết hạn cho dữ liệu cache:

```swift
// Thêm timestamp cho dữ liệu cache
userDefaults.set(Date().timeIntervalSince1970, forKey: "lastCacheUpdate_\(user.id)")

// Kiểm tra hết hạn trước khi sử dụng
let lastUpdate = userDefaults.double(forKey: "lastCacheUpdate_\(user.id)")
let currentTime = Date().timeIntervalSince1970
let cacheLifetime: TimeInterval = 3600 // 1 giờ

if currentTime - lastUpdate > cacheLifetime {
    // Cache đã hết hạn, cần refresh từ server
    refreshUserData()
}
```

### 3.2 Tối ưu queries Firestore

Chúng tôi áp dụng một số nguyên tắc tối ưu khi truy vấn Firestore:

1. **Chỉ lấy dữ liệu cần thiết**: Sử dụng query để lọc dữ liệu phía server
2. **Sử dụng indexes**: Tạo index cho các trường thường xuyên truy vấn
3. **Batch operations**: Gom nhóm các thao tác liên quan
4. **Pagination**: Chia nhỏ kết quả thành các trang khi làm việc với dữ liệu lớn
5. **Query throttling**: Hạn chế tần suất truy vấn để giảm chi phí

#### 3.2.1 Sử dụng `.select()` để lấy chỉ các trường cần thiết

```swift
// Thay vì lấy toàn bộ document
let snapshot = try await db.collection("users").document(userId).getDocument()

// Chỉ lấy các trường cần thiết
let snapshot = try await db.collection("users").document(userId)
    .select(["name", "points", "lastUpdated"])
    .getDocument()
```

#### 3.2.2 Sử dụng transactions cho các cập nhật phức tạp

Trong phương thức `redeemReward`, chúng tôi sử dụng transaction để đảm bảo tính nhất quán dữ liệu:

```swift
// Bắt đầu transaction
try await db.runTransaction { transaction, errorPointer in
    // Cập nhật điểm và phần thưởng đã đổi trong profile người dùng
    transaction.updateData(
        [
            "points": newPoints,
            "rewardsRedeemed": newRewardsRedeemed,
            "lastUpdated": Timestamp(date: Date())
        ],
        forDocument: self.db.collection("users").document(userDocId)
    )
    
    // Thêm giao dịch điểm
    let transactionData: [String: Any] = [
        "userId": userId,
        "points": -10,
        "timestamp": Timestamp(date: Date()),
        "type": "redeemed",
        "staffId": NSNull()
    ]
    
    let transactionRef = self.db.collection("pointTransactions").document()
    transaction.setData(transactionData, forDocument: transactionRef)
    
    return nil
}
```

### 3.3 Tối ưu mô hình dữ liệu Firestore

Chúng tôi đã thiết kế cấu trúc dữ liệu Firestore để tối ưu cho các trường hợp sử dụng phổ biến:

- **Denormalization**: Lưu trữ một số dữ liệu trùng lặp để giảm số lượng query
- **Subdocuments**: Sử dụng subdocuments cho dữ liệu liên quan chặt chẽ
- **Collection Group Queries**: Cho phép truy vấn cùng một collection ở nhiều nơi
- **Flat structure**: Tránh nested data quá sâu để dễ dàng truy vấn
- **References**: Sử dụng document references để tạo quan hệ giữa các entities

#### 3.3.1 Nguyên tắc thiết kế cấu trúc Firestore

1. **Thiết kế theo truy vấn**: Tổ chức dữ liệu dựa trên cách bạn sẽ truy vấn
2. **Avoid high write rates**: Tránh các document có tần suất ghi cao
3. **Distribute workload**: Phân tán tải đều trên các document
4. **Balance document size**: Duy trì kích thước document hợp lý (< 1MB)

#### 3.3.2 Ví dụ về mô hình dữ liệu:

```swift
struct UserProfile: Codable, Identifiable {
    @DocumentID var id: String?
    let appleUserId: String
    var name: String
    var points: Int
    var rewardsRedeemed: Int
    var lastUpdated: Date
    
    // Những trường tùy chọn khác có thể thêm sau
    var email: String?
}
```

### 3.4 Sử dụng Swift Concurrency

Swift Concurrency (async/await) đã cách mạng hóa cách chúng tôi xử lý các tác vụ bất đồng bộ. Thay vì sử dụng completion handlers hoặc combine, chúng tôi đã chuyển hoàn toàn sang async/await vì những lợi ích nổi bật:

1. **Code dễ đọc hơn**: Cấu trúc tuần tự, giống như code đồng bộ
2. **Xử lý lỗi tốt hơn**: Sử dụng try-catch thay vì error parameters
3. **Task management**: Dễ dàng hủy và quản lý các tác vụ
4. **Structured concurrency**: Quản lý các tác vụ con và phụ thuộc

#### 3.4.1 Ví dụ sử dụng async/await với Firestore:

```swift
func refreshUserData() {
    guard let user = currentUser else { return }
    
    DispatchQueue.main.async {
        self.isLoading = true
    }
    
    Task {
        do {
            if let profile = try await FirebaseService.shared.fetchUserProfile(userId: user.id) {
                DispatchQueue.main.async {
                    self.points = profile.points
                    self.userDefaults.set(profile.points, forKey: "userPoints_\(user.id)")
                    
                    // Đảm bảo cập nhật userDocId nếu chưa có
                    if self.userDocId == nil {
                        self.userDocId = profile.id
                    }
                    
                    // Cập nhật tên nếu đã thay đổi
                    if profile.name != user.name {
                        self.currentUser = User(id: user.id, name: profile.name)
                        self.userDefaults.set(profile.name, forKey: "userName")
                        self.userDefaults.set(profile.name, forKey: "userName_\(user.id)")
                    }
                    
                    self.isLoading = false
                }
            } else {
                DispatchQueue.main.async {
                    self.isLoading = false
                    self.errorMessage = "Could not find user data on server."
                }
            }
        } catch {
            DispatchQueue.main.async {
                self.isLoading = false
                self.errorMessage = "Could not connect to server."
            }
        }
    }
}
```

## 4. Xử lý offline và đồng bộ hóa dữ liệu

### 4.1 Chiến lược Offline-First

Trong ứng dụng của chúng tôi, việc áp dụng chiến lược "offline-first" đã cho thấy những cải thiện đáng kể về trải nghiệm người dùng:

1. **Ưu tiên dữ liệu local**: Hiển thị dữ liệu từ cache ngay lập tức
2. **Cập nhật nền**: Đồng bộ hóa với server khi có kết nối
3. **Xử lý xung đột**: Chiến lược giải quyết xung đột dữ liệu

```swift
private func loadUserData() {
    // Khôi phục trạng thái staff mode nếu có
    if userDefaults.bool(forKey: "isStaffLoggedIn") {
        self.isStaffLoggedIn = true
        self.staffName = userDefaults.string(forKey: "staffName") ?? "Staff"
        return 
    }
    
    // Khôi phục dữ liệu user thông thường
    if userDefaults.bool(forKey: "isLoggedIn"),
       let id = userDefaults.string(forKey: "userId") {
        
        let name = userDefaults.string(forKey: "userName_\(id)") ??
                   userDefaults.string(forKey: "userName") ??
                   "Coffee Lover"
        
        self.currentUser = User(id: id, name: name)
        self.isLoggedIn = true
        self.points = userDefaults.integer(forKey: "userPoints_\(id)")
        
        // Đồng bộ với Firebase
        refreshUserData()
    }
}
```

### 4.2 Xử lý lỗi thông minh

Xử lý lỗi là một phần quan trọng của chiến lược offline-first. Chúng tôi đã triển khai một hệ thống xử lý lỗi nhiều lớp:

1. **Lớp network**: Phát hiện và phản ứng với lỗi kết nối
2. **Lớp data**: Xử lý lỗi khi đọc/ghi dữ liệu
3. **Lớp UI**: Hiển thị thông báo phù hợp với người dùng

Chiến lược này có ưu điểm:
- Cải thiện trải nghiệm người dùng bằng cách xử lý lỗi một cách mềm dẻo
- Giảm thiểu tác động của lỗi đối với dữ liệu
- Tự động khôi phục khi có thể

Ví dụ về cách chúng tôi xử lý lỗi khi thêm điểm:

```swift
func addPoint() {
    guard let user = currentUser else { return }
    
    // Cập nhật UI ngay lập tức
    DispatchQueue.main.async {
        self.points += 1
        self.lastPointChange = Date()
        
        // Cập nhật local cache
        self.userDefaults.set(self.points, forKey: "userPoints_\(user.id)")
    }
    
    // Cập nhật lên Firebase
    Task {
        do {
            let newPoints = try await FirebaseService.shared.addPointToUser(userId: user.id)
            
            // Cập nhật lại UI nếu cần (nếu số điểm từ server khác với local)
            DispatchQueue.main.async {
                if newPoints != self.points {
                    self.points = newPoints
                    self.userDefaults.set(newPoints, forKey: "userPoints_\(user.id)")
                }
            }
        } catch {
            print("Error adding point to Firebase: \(error.localizedDescription)")
            // UI đã được cập nhật, nên không cần xử lý lỗi ở đây
        }
    }
}
```

## 5. Monitoring và debugging

### 5.1 Logging hiệu quả

Logging có cấu trúc đã trở thành một công cụ không thể thiếu trong quá trình phát triển và bảo trì ứng dụng của chúng tôi. Thay vì sử dụng `print()` đơn giản, chúng tôi đã xây dựng một hệ thống logging có cấu trúc:

```swift
enum LogLevel {
    case debug, info, warning, error, fatal
}

class Logger {
    static func log(_ level: LogLevel, _ message: String, file: String = #file, function: String = #function, line: Int = #line) {
        #if DEBUG
        let fileName = URL(fileURLWithPath: file).lastPathComponent
        let prefix: String
        
        switch level {
        case .debug: prefix = "🔍 DEBUG"
        case .info: prefix = "ℹ️ INFO"
        case .warning: prefix = "⚠️ WARNING"
        case .error: prefix = "❌ ERROR"
        case .fatal: prefix = "☠️ FATAL"
        }
        
        let logMessage = "\(prefix) [\(fileName):\(line) \(function)] - \(message)"
        print(logMessage)
        #endif
    }
}

// Usage
Logger.log(.info, "Fetching user profile for userId: \(userId)")
// ...sau đó...
Logger.log(.info, "Successfully fetched user profile: \(profile.name), docID: \(profile.id ?? "unknown")")
```

Hệ thống logging này mang lại nhiều lợi ích:

1. **Rõ ràng về ngữ cảnh**: Biết chính xác đoạn code nào đang ghi log
2. **Phân loại mức độ nghiêm trọng**: Dễ dàng lọc log theo mức độ
3. **Tối ưu cho sản phẩm**: Logs sẽ chỉ xuất hiện trong môi trường DEBUG
4. **Cải thiện khả năng phát hiện lỗi**: Format nhất quán giúp dễ phân tích

### 5.2 Sử dụng Firebase Analytics và Performance Monitoring

Firebase Analytics và Performance Monitoring là những công cụ mạnh mẽ giúp chúng tôi hiểu rõ hơn về cách người dùng tương tác với ứng dụng và phát hiện các vấn đề về hiệu suất.

#### 5.2.1 Firebase Analytics

Firebase Analytics giúp chúng tôi theo dõi:
- Hiệu suất ứng dụng
- Hành vi người dùng
- Tần suất sử dụng các tính năng
- Tỷ lệ chuyển đổi
- Phân đoạn người dùng

Chúng tôi đã thiết lập các sự kiện tùy chỉnh để theo dõi các hành động quan trọng:

```swift
// Track khi người dùng đăng nhập
Analytics.logEvent("user_login", parameters: [
    "login_method": "apple_id",
    "user_type": isNewUser ? "new" : "existing"
])

// Track khi người dùng hoàn thành một tương tác quan trọng
Analytics.logEvent("reward_redeemed", parameters: [
    "points_used": 10,
    "reward_type": "free_coffee",
    "user_total_points": userPoints
])
```

#### 5.2.2 Performance Monitoring

Firebase Performance Monitoring giúp chúng tôi theo dõi và tối ưu hiệu suất ứng dụng:

```swift
// Đo thời gian của một thao tác quan trọng
let trace = Performance.startTrace(name: "fetch_user_profile")
do {
    let profile = try await fetchUserProfile(userId: userId)
    trace?.stop()
    return profile
} catch {
    trace?.stop()
    throw error
}

// Đo thời gian mạng
let metric = HTTPMetric(url: url, httpMethod: .get)
metric.start()
let (data, response) = try await URLSession.shared.data(from: url)
metric.responseCode = (response as? HTTPURLResponse)?.statusCode ?? 0
metric.stop()
```

Nhờ Performance Monitoring, chúng tôi đã xác định được các bottleneck và tối ưu hóa các tác vụ quan trọng, giảm thời gian phản hồi trung bình xuống 40%.

### 5.3 Theo dõi và tối ưu chi phí Firebase

Firebase Firestore tính phí theo số lượng đọc/ghi và lượng dữ liệu, do đó việc theo dõi và tối ưu chi phí là một phần quan trọng trong quy trình phát triển và vận hành:

#### 5.3.1 Theo dõi chi phí

Chúng tôi đã thiết lập hệ thống giám sát chi phí Firebase:
- Giám sát số lượng đọc/ghi thông qua bảng điều khiển Firebase
- Thiết lập cảnh báo ngân sách để nhận thông báo khi chi phí tăng đột biến
- Phân tích chi tiết việc sử dụng tài nguyên theo tính năng và nhóm người dùng

#### 5.3.2 Chiến lược tối ưu chi phí

1. **Giảm số lượng reads**:
   ```swift
   // Thay vì truy vấn mỗi khi cần
   func getUserData() async throws -> UserProfile {
       return try await db.collection("users").document(userId).getDocument().data()
   }
   
   // Sử dụng cache và chỉ truy vấn khi cần
   func getUserData() async throws -> UserProfile {
       // Kiểm tra cache
       if let cachedData = userDataCache[userId], !isCacheExpired(cachedData) {
           return cachedData.profile
       }
       
       // Nếu không có trong cache hoặc đã hết hạn, truy vấn từ Firestore
       let profile = try await db.collection("users").document(userId).getDocument().data()
       userDataCache[userId] = (profile: profile, timestamp: Date())
       return profile
   }
   ```

2. **Gom nhóm writes**:
   ```swift
   // Thực hiện batch update thay vì nhiều updates riêng lẻ
   let batch = db.batch()
   
   // Cập nhật user profile
   let userRef = db.collection("users").document(userId)
   batch.updateData(["points": newPoints, "lastUpdated": Date()], forDocument: userRef)
   
   // Thêm transaction history
   let transactionRef = db.collection("transactions").document()
   batch.setData(transactionData, forDocument: transactionRef)
   
   // Cập nhật stats
   let statsRef = db.collection("stats").document("daily")
   batch.updateData(["totalTransactions": FieldValue.increment(1)], forDocument: statsRef)
   
   // Thực hiện tất cả trong một lần gọi
   try await batch.commit()
   ```

3. **Tối ưu dung lượng dữ liệu**:
   - Sử dụng các tên trường ngắn gọn
   - Lưu trữ timestamp dưới dạng số thay vì string
   - Áp dụng nén dữ liệu khi cần thiết

4. **Rate limiting và caching**:
   - Giới hạn tần suất cập nhật dữ liệu theo thời gian thực
   - Áp dụng caching cho dữ liệu ít thay đổi

## 6. Kết luận

Fetching data hiệu quả giữa Firebase và iOS app đòi hỏi sự cân bằng giữa hiệu suất, trải nghiệm người dùng và chi phí. Qua dự án ứng dụng loyalty của chúng tôi, chúng tôi đã rút ra được nhiều bài học quan trọng:

1. **Thiết kế kiến trúc đúng** giúp quản lý dữ liệu hiệu quả
2. **Chiến lược caching thông minh** cải thiện trải nghiệm người dùng
3. **Tối ưu hóa queries** giảm chi phí và tăng hiệu suất
4. **Xử lý offline-first** đảm bảo ứng dụng hoạt động trong mọi điều kiện mạng
5. **Monitoring liên tục** giúp phát hiện và khắc phục sự cố sớm

Bằng cách áp dụng những nguyên tắc này, bạn có thể xây dựng ứng dụng iOS với Firebase hiệu quả, tiết kiệm chi phí và mang lại trải nghiệm tuyệt vời cho người dùng.

---

Nếu bạn có câu hỏi hoặc muốn thảo luận thêm về chủ đề này, hãy để lại bình luận bên dưới. Và đừng quên chia sẻ kinh nghiệm của bạn khi làm việc với Firebase và iOS - chúng tôi luôn mong muốn học hỏi thêm từ cộng đồng!
