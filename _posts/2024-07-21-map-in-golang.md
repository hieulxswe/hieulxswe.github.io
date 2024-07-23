---
title: "[vn] Cách sử dụng Map trong Golang"
date: 2024-07-21T00:00:00+00:00
author: Hieu Xuan Leu
layout: post
permalink: /map-in-golang/
categories: golang
tags: [golang, map]
image: ../assets/images/map_in_go.png
---

Map trong Golang là một cấu trúc dữ liệu rất mạnh mẽ và linh hoạt, cho phép lưu trữ các cặp `key-value` (khóa-giá trị). Nó rất hữu ích trong nhiều tình huống lập trình khi bạn cần tra cứu giá trị dựa trên khóa. Trong bài viết này, chúng ta sẽ tìm hiểu cách sử dụng map, những lợi ích và một số kỹ thuật tối ưu hóa hiệu suất khi làm việc với map trong Golang.

## 1. Giới thiệu về Map

Map là một tập hợp các cặp `key-value`, trong đó mỗi khóa `(key)` là duy nhất và ánh xạ tới một giá trị `(value)`. Map là một cách tuyệt vời để tra cứu dữ liệu nhanh chóng và dễ dàng. Trong Golang, map được triển khai như một cấu trúc dữ liệu nội tại với cú pháp đơn giản và dễ hiểu.

### Tạo một Map

Để khai báo một map, bạn cần sử dụng từ khóa `make` hoặc cú pháp trực tiếp. Dưới đây là một ví dụ về cách tạo một map:

```go
ages := make(map[string]int)
```

Hoặc bạn cũng có thể khởi tạo map với các giá trị ban đầu:

```go
ages := map[string]int{
    "Brian": 23,
    "Alie": 27,
}
```

Trong ví dụ trên, chúng ta đã tạo một map `ages` với các khóa là kiểu `string` và các giá trị là kiểu `int`.

### Thêm, truy vấn & xoá phần tử

#### Thêm phần tử

Để thêm hoặc cập nhật một phần tử trong map, bạn chỉ cần gán giá trị cho một khóa cụ thể:

```go
ages["Liam"] = 4
```

#### Truy xuất phần tử

Để truy xuất giá trị từ map, bạn sử dụng khóa tương ứng:

```go
age := ages["Alie"]
fmt.Println(age)  // Output: 27
```

#### Xóa phần tử

Để xóa một phần tử từ map, bạn sử dụng từ khóa `delete`:

```go
delete(ages, "Liam")
```

### Kiểm tra sự tồn tại của khoá

Khi truy xuất giá trị từ map, nếu khóa không tồn tại, giá trị trả về sẽ là giá trị `zero` của loại giá trị đó (ví dụ: `0` cho `int`, `""` cho `string`). Để kiểm tra xem khóa có tồn tại hay không, bạn có thể sử dụng cú pháp sau:

```go
age, exists := ages["Liam"]
if exists {
    fmt.Println("Liam's age is", age)
} else {
    fmt.Println("Liam's age is not in the map")
}
```

### Lặp qua các phần tử trong Map

Bạn có thể sử dụng vòng lặp `for range` để lặp qua tất cả các cặp `key-value` trong map:

```go
for name, age := range ages {
    fmt.Printf("%s is %d years old\n", name, age)
}
```

## 2. Một số ví dụ

### Ví dụ 1: Đếm số lần xuất hiện của các từ trong một đoạn văn bản

Dưới đây là một ví dụ về cách sử dụng map để đếm số lần xuất hiện của các từ trong một đoạn văn bản:

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    text := "this is a sample text with several words this is a sample"
    words := strings.Fields(text)

    wordCount := make(map[string]int)
    for _, word := range words {
        wordCount[word]++
    }

    for word, count := range wordCount {
        fmt.Printf("%s: %d\n", word, count)
    }
}
```

### Ví dụ 2: Sử dụng Map trong Struct

Map cũng có thể được sử dụng bên trong các struct. Dưới đây là một ví dụ về cách sử dụng map trong struct để quản lý thông tin sinh viên:

```go
package main

import "fmt"

type Student struct {
    Name string
    Age  int
    Grades map[string]string
}

func main() {
    student := Student{
        Name: "Brian Matthews",
        Age:  23,
        Grades: map[string]string{
            "Math":    "A",
            "Science": "B",
            "History": "A",
        },
    }

    fmt.Println("Student:", student.Name)
    fmt.Println("Age:", student.Age)
    for subject, grade := range student.Grades {
        fmt.Printf("%s: %s\n", subject, grade)
    }
}
```

### Ví dụ 3: Sử dụng Map để nhóm các phần tử theo một tiêu chí

Dưới đây là một ví dụ về cách sử dụng map để nhóm các phần tử theo một tiêu chí nhất định, chẳng hạn như nhóm các tên theo chữ cái đầu tiên:

```go
package main

import "fmt"

func main() {
    names := []string{"Alie", "Brian", "Charlie", "David", "Eve", "Frank"}

    nameGroups := make(map[string][]string)
    for _, name := range names {
        firstLetter := string(name[0])
        nameGroups[firstLetter] = append(nameGroups[firstLetter], name)
    }

    for letter, group := range nameGroups {
        fmt.Printf("%s: %v\n", letter, group)
    }
}
```

### Ví dụ 4: Đồng bộ hoá truy cập Map trong các tình huống Concurrent

Khi làm việc với map trong môi trường concurrent, bạn cần đảm bảo rằng truy cập vào map được đồng bộ hóa. Dưới đây là một ví dụ sử dụng `sync.Mutex` để đồng bộ hóa truy cập vào map:

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var mu sync.Mutex
    safeMap := make(map[string]int)

    wg := sync.WaitGroup{}
    wg.Add(2)

    go func() {
        defer wg.Done()
        mu.Lock()
        safeMap["Alie"] = 27
        mu.Unlock()
    }()

    go func() {
        defer wg.Done()
        mu.Lock()
        safeMap["Brian"] = 23
        mu.Unlock()
    }()

    wg.Wait()
    mu.Lock()
    fmt.Println(safeMap)
    mu.Unlock()
}
```

### Ví dụ 5: Sử dụng sync.Map cho các tình huống Concurrent

Package `sync` cung cấp một loại map an toàn cho các tình huống concurrent gọi là `sync.Map`. Dưới đây là một ví dụ sử dụng `sync.Map`:

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var smap sync.Map

    wg := sync.WaitGroup{}
    wg.Add(2)

    go func() {
        defer wg.Done()
        smap.Store("Alie", 27)
    }()

    go func() {
        defer wg.Done()
        smap.Store("Brian", 23)
    }()

    wg.Wait()

    smap.Range(func(key, value interface{}) bool {
        fmt.Printf("%s: %d\n", key, value)
        return true
    })
}
```

## 3. Tối ưu hoá hiệu suất với Map

### Chọn kiểu dữ liệu thích hợp cho khoá

Khóa trong map có thể là bất kỳ loại dữ liệu nào có thể so sánh được (các loại dữ liệu có thể sử dụng toán tử `==` để so sánh). Tuy nhiên, bạn nên chọn loại dữ liệu phù hợp để tối ưu hóa hiệu suất. Ví dụ, sử dụng `string` hoặc `int` làm khóa thường mang lại hiệu suất tốt hơn.

### Dự đoán kích thước Map

Nếu bạn biết trước kích thước của map, bạn có thể sử dụng `make` với tham số thứ hai để cấp phát bộ nhớ một cách hiệu quả hơn:

```go
ages := make(map[string]int, 100)  // Dự đoán map sẽ có khoảng 100 phần tử
```

### Tối ưu hoá hiệu suất khi lặp qua Map

Khi lặp qua map, thứ tự của các phần tử không được đảm bảo và có thể thay đổi. Nếu thứ tự là quan trọng, bạn có thể lưu các khóa vào một slice rồi sắp xếp slice đó trước khi lặp.

```go
var keys []string
for k := range ages {
    keys = append(keys, k)
}
sort.Strings(keys)
for _, k := range keys {
    fmt.Printf("%s is %d years old\n", k, ages[k])
}
```

## 4. Tránh các sai lầm thường gặp

### Tránh truy xuất phần tử không tồn tại

Truy xuất phần tử không tồn tại trong map sẽ trả về giá trị zero của loại giá trị đó, điều này có thể gây nhầm lẫn. Hãy luôn kiểm tra sự tồn tại của khóa trước khi sử dụng giá trị trả về.

### Không thể sử dụng Slice, Map & Function làm khoá

Trong Golang, bạn không thể sử dụng các loại dữ liệu không thể so sánh (như slice, map, và function) làm khóa trong map. Hãy sử dụng các loại dữ liệu có thể so sánh như `int`, `string`, hoặc các struct có định nghĩa phương thức `==`.

## 5. Kết Luận

Map là một cấu trúc dữ liệu mạnh mẽ và tiện dụng trong Golang, cung cấp các tính năng tra cứu và thao tác với dữ liệu hiệu quả. Tuy nhiên, để sử dụng maps một cách hiệu quả và tối ưu, bạn cần hiểu rõ về cách hoạt động của chúng cũng như các kỹ thuật tối ưu hóa phù hợp. Các ví dụ trên hy vọng đã giúp bạn có cái nhìn rõ ràng hơn về cách sử dụng và tối ưu hóa map trong Golang.

---

Nếu bạn có bất kỳ câu hỏi hay cần thêm thông tin chi tiết về Golang, hãy để lại bình luận dưới bài viết nhé. Thanks!