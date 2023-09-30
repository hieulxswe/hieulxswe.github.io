---
title: "Long Weekend: Cafe & tối ưu hệ thống P.1"
date: 2023-09-30T00:00:00+00:00
author: Hieu Xuan Leu
layout: post
permalink: /long-weekend-cafe-toi-uu-he-thong-p-1/
thumbnail: /assets/images/toiuuhethong.png
categories: backend
tags: [backend, optimzesystem]
---

Sydney, 30 Sep 2023

Xin chào mọi người, mình là Hiếu (aka. Brian) – hiện tại đang sinh sống – học tập & làm việc tại Sydney. Nếu đây là lần đầu bạn ghé thăm blog, hãy lựa chọn tab “Home” để có thể xem những bài viết/chuyên mục mà mình đã chia sẻ trước đây. Và đừng quên “Add blog to your favorites” để có thể cập nhật những bài viết mới nhất từ một người “Xa nhà, gần máy tính” nhé. Cảm ơn các bạn!

Hôm nay là một ngày cuối tuần đẹp trời. Mình dậy từ 5am “hít” khí trời, vệ sinh cá nhân, ăn sáng. Sau đó sẽ “lên đường” tới thư viện để học & làm việc (Từ khi sang Úc, mình có thói quen là khi rảnh sẽ lên thư viện ở các Suburb khác nhau). Nay mình ngồi ở Five Dock Library (một thư viện mà mình yêu thích 😊).

![alt](https://hieulxswe.com/wp-content/uploads/2023/09/IMG_3691-1-1536x1152.png)

Chia sẻ ngắn về bản thân vậy thôi, bây giờ chúng ta cùng đi vào chi tiết bài viết nhé! Let’s goooooo …

Ở trong bài viết này, mình sẽ chia sẻ đôi chút kinh nghiệm cá nhân & tham khảo từ các nguồn về việc thiết kế & tối ưu hệ thống “phục vụ hàng triệu người dùng”.

Như chúng ta đã biết, việc thiết kế hệ thống để phục vụ hàng triệu người dùng là một thử thách khó & không có câu trả lời chính xác (nó giống như việc: “Dev tán gái”, “Phụ nữ đến tháng”, …. 🤣). Vì vậy, chúng ta cùng đọc – tìm hiểu & góp ý cho nhau nhé!

## Tại sao khó?
Để thiết kế một hệ thống cho 5-10 người sử dụng thì vô cùng đơn giản. Tuy nhiên, khi scope lên hàng trăm, nghìn, trăm nghìn … thậm trí cả triệu người dùng thì lại là một bài toán lớn & vô cùng phức tạp.

Công ty mình có rất nhiều portals & roles khác nhau (Manager, Staff, Agent, Client …). Không những thế, hệ thống của công ty mình còn có “nhiệm vụ” tiếp nhận hồ sơ từ khách hàng, tính toán những “thông số” cho khách hàng, …. (Nói chung là NẶNG, nhưng vì lý do bảo mật nên mình không tiện chia sẻ ở đây). Và tất nhiên sẽ không dừng ở những tác vụ này, chúng tôi sẽ luôn luôn scale-up 👌

Ví dụ: Một server bình thường có thể tiếp nhận và chịu tải được 100 request/s. Khi lượng request tăng lên khoảng 1000 thì chúng ta có thể nâng cấp server có cấu hình cao & tốt hơn. Nhưng khi lượng request lên vài trăm ngàn hoặc cả triệu request thì việc nâng cấp server là điều “gần như không thể”, lúc này bắt buộc chúng phải thiết kế & tối ưu sao cho nhiều server chạy cùng lúc

## Một số yếu tố quan trọng của một hệ thống
Khi thiết kế hệ thống, có 3 yếu tố quan trọng mà chúng ta cần để ý đó là: Performance, Availability và Scalability.

![alt](https://hieulxswe.com/wp-content/uploads/2023/09/image.png)

* Performance (Hiệu suất): Tốc độ phản hồi của hệ thống. Được tính bằng đơn vị thời gian (s hoặc ms).
* Availability (Khả dụng): đề cập đến khả năng của hệ thống hoặc dịch vụ để hoạt động và sẵn sàng sử dụng khi cần thiết, mà không gặp sự cố hoặc gián đoạn lớn.
* Scalability (Mở rộng): Khả năng mở rộng của hệ thống.

Performance đặc biệt quan trọng và cần thiết để đảm bảo người dùng của bạn có thể có trải nghiệm tốt nhất trên cả những thiết bị điện thoại (có CPU và power thấp). Việc số lượng người dùng điện thoại ngày càng nhiều và số lượng người dùng truy cập website thông qua di động thì những website cần được tối ưu tốt hơn để đáp ứng lượng người dùng này.

## Cân bằng tải với Load Balancer
Bộ cân bằng tải (Load Balancer) phân phối đồng đều lưu lượng truy cập giữa các web server được xác định trong bộ cân bằng tải. Xem hình cách hoạt động của bộ cân bằng tải.

![alt](https://hieulxswe.com/wp-content/uploads/2023/09/image-1.png)

Ở hình trên, người dùng kết nối trực tiếp với địa chỉ IP public của bộ cân bằng tải. Với thiết lập trên, web server không thể tiếp cận trực tiếp bởi client. Để bảo mật tốt hơn, IP private được dùng để giao tiếp giữa các server. Một IP private là một địa chỉ IP có thể tiếp cận giữa các server trong cùng một mạng nhưng không thể tiếp cận từ internet bên ngoài. Bộ cân bằng tải giao tiếp với các web server thông qua IP private.

Sau bộ cân bằng tải là hai web server, như vậy là ta đã giải quyết được vấn đề chuyển đổi tự động và cải thiện tính khả dụng của web.
* Nếu server 1 ngoại tính, truy cập sẽ được chuyển sang server 2. Điều này ngăn chặn việc website sập. Ta cũng sẽ thêm một web server khoẻ mạnh mới vào để cân bằng tải.
* Nếu lưu lượng truy cập web tăng mạnh, hai server là không đủ để xử lý, bộ cân bằng tải có thể xử lý vấn đề này một cách gọn gàng. Bạn chỉ cần thêm server vào nhóm web server bộ cận bằng tải sẽ tự đổi gửi yêu cầu đến nó.

## Phân tán dữ liệu với Content Delivery Network (CDN)
Content delivery network (CDN) là một nhóm server đặt tại nhiều vị trí khác nhau để hỗ trợ nội dung được trải dài ở nhiều khu vực vị trí địa lý khác nhau.

Bằng cách phân tán hệ thống trên một khu vực rộng lớn, website có thể giảm thiểu lượng băng thông tiêu thụ và thời gian tải trang, đồng thời có khả năng xử lý được nhiều request đồng thời.

![alt](https://hieulxswe.com/wp-content/uploads/2023/09/image-2.png)

## Caching
Là một kỹ thuật tăng độ truy xuất dữ liệu và giảm tải cho hệ thống. Cache là nơi lưu tập hợp các dữ liệu, thường có tính chất nhất thời, cho phép sử dụng lại dữ liệu đã lấy hoặc tính toán trước đó, nên sẽ giúp tăng tốc cho việc truy xuất dữ liệu ở những lần sau.

![alt](https://hieulxswe.com/wp-content/uploads/2023/09/image-3.png)

## Database Replication
Database Replication là một kỹ thuật trong lĩnh vực quản trị cơ sở dữ liệu (Database Management), được sử dụng để tạo ra và duy trì các bản sao của cơ sở dữ liệu gốc (source database) trên các máy chủ (servers) khác nhau. Mục tiêu của Database Replication là cung cấp sự sao lưu dữ liệu, tăng tính sẵn sàng (availability), cải thiện khả năng chịu lỗi (fault tolerance), và tối ưu hóa hiệu suất (performance) của hệ thống cơ sở dữ liệu.

Thông thường nó sẽ là mối quan hệ Master-Slave giữa bản gốc (Master) và bản sao (Slave).

Master chỉ hỗ trợ thao tác ghi. Còn các Slave lấy dữ liệu sao chép từ cơ sở dữ liệu Master và chỉ cung cấp thao tác đọc. Tất cả lệnh chỉnh sửa dữ liệu như INSERT, DELETE và UPDATE sẽ được gửi vào Master DB. Phần lớn ứng dụng đều yêu cầu truy cập đọc nhiều hơn ghi, do đó số lượng cơ sở dữ liệu Slave trong hệ thống nhiều hơn cơ sở dữ liệu Master.

![alt](https://hieulxswe.com/wp-content/uploads/2023/09/image-4.png)

Lợi ích của mối quan hệ Master/Slave bao gồm:
* Sao lưu dữ liệu: Slave lưu trữ bản sao dữ liệu, giúp bảo vệ dữ liệu khỏi mất mát trong trường hợp sự cố hoặc hỏng máy chủ Master.
* Phân phối tải công việc: Slave có thể được sử dụng để đọc dữ liệu, giảm áp lực cho máy chủ Master và cải thiện hiệu suất hệ thống.
* Tính nhất quán: Slave đồng bộ hóa dữ liệu từ Master, đảm bảo tính nhất quán giữa các bản sao.

## Kết
Tạm kết P.1, do thời gian có hạn nên bài viết này mình xin tạm dừng ở đây. Những kỹ thuật bên trên chỉ mang tính chất tham khảo (còn phụ thuộc vào mô hình hoạt động của hệ thống). Nếu các bạn có góp ý hoặc có các kỹ thuật hay hơn, hãy bình luận bên dưới để cùng nhau học hỏi nhé! Have a nice day ~everyone ✌️


