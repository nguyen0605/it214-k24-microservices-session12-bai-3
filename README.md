# Báo cáo phân tích và cấu hình Circuit Breaker

## Phần 1 – Báo cáo phân tích lý do Circuit Breaker không mở

Trong tình huống đêm hôm đó, chỉ có 5 khách hàng thực hiện thanh toán và cả 5 người đều gặp lỗi (tỷ lệ lỗi 100%), nhưng Circuit Breaker vẫn không chuyển sang trạng thái **OPEN**. Nguyên nhân chính là do **minimumNumberOfCalls** (Số lượng cuộc gọi tối thiểu):

1. **Cơ chế hoạt động của Sliding Window trong Resilience4j**: Theo mặc định hoặc cấu hình ngầm định, nếu không chỉ định `minimumNumberOfCalls`, giá trị này thường bằng với `slidingWindowSize` (hoặc một giá trị lớn được định nghĩa sẵn tùy phiên bản). 
2. **Xung đột với số lượng request thực tế**: Hệ thống chỉ ghi nhận 5 request (`5 calls`) trong khoảng thời gian đó. Trong khi đó, yêu cầu cấu hình `slidingWindowSize = 20` và `minimumNumberOfCalls` mặc định có thể lớn hơn 5 (ví dụ bằng 20). 
3. **Kết luận**: Circuit Breaker **chỉ tính toán tỷ lệ lỗi (Failure Rate)** khi số lượng request thực tế trong cửa sổ trượt đạt tối thiểu bằng `minimumNumberOfCalls`. Vì mới chỉ có 5 request (chưa đạt ngưỡng tối thiểu), Resilience4j bỏ qua việc tính toán tỷ lệ lỗi, dẫn đến việc cầu dao không mở dù tỷ lệ lỗi lên tới 100%.

Để khắc phục, cần phải cấu hình tường minh thuộc tính `minimumNumberOfCalls` sao cho phù hợp với đặc thù lưu lượng thấp (hoặc tuân thủ nguyên tắc `minimumNumberOfCalls >= permittedNumberOfCallsInHalfOpenState`).

## Phần 2 – Triển khai

File cấu hình `application.yml` hoàn chỉnh được đặt trong thư mục dự án để giải quyết vấn đề trên.