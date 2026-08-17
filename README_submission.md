# Lab 17 - Multi-Memory Agent voi Zep

**Họ và tên:** Hồ Quang Minh
**Mã sinh viên:** 2A202601906

## 1. Phân tích Benchmark
- **Layer có hit rate thấp nhất**: Trong cấu hình no-memory, layer có hit rate thấp nhất là `long_term`, `episodic` và `semantic` (hit rate 0%), do Agent chỉ nhớ được các trao đổi gần nhất trong short-term memory (E01, E10). 
- **Query retrieve nhiều token nhất**: Thường là các query được route vào layer `episodic` và `semantic` (chẳng hạn E06, E11) vì các layer này trả về các đoạn nguyên bản (raw text/episodes) khá dài nhằm giữ nguyên được các literal marker.
- **Case mixed (E07)**: Cần kết hợp giữa **Long-term memory** (để biết preference ngôn ngữ ưu tiên của Minh là Python) và **Semantic memory** (để biết rule hướng dẫn retry payment). Các evidence bắt buộc cần lấy ra được là `Python` và `Idempotency-Key`.
- **Token reduction**: Ở baseline no-memory, số token giảm tuyệt đối (gần 100% so với việc phải nhét toàn bộ lịch sử vào prompt) nhưng đổi lại hit rate tụt xuống 0% đối với các thông tin cũ. Context Budget Manager (10/4/3/3) giúp ta vẫn đạt hit rate xuất sắc mà giữ được kích thước prompt tối ưu (không bị loãng).

## 2. Các câu hỏi lý thuyết
- **Layer quan trọng nhất trong bộ test này**: Đó là **Long-term layer**. Nó giải quyết số lượng case nhiều nhất trong bộ đánh giá (E02, E03, E08, E09 và 1 phần E07). Nó giúp phân biệt được sự ưu tiên của các user khác nhau (như Lan và Minh - E09) và tự động nhận diện cập nhật ưu tiên mới nhất (recency - E08).
- **Trade-off giữa Context Block (Zep) và Redis+Qdrant**: Zep cung cấp giải pháp managed memory toàn diện, tự động trích xuất sự kiện, xử lý xung đột cũ-mới (recency) và tạo Context Block tóm tắt thông minh, nhưng đổi lại cần có API Key và phải phụ thuộc dịch vụ ngoài. Ngược lại, Redis+Qdrant (Local) hoàn toàn miễn phí, độc lập, nhưng lập trình viên phải "tự bơi" viết toàn bộ logic vector embedding, quản lý knowledge graph và xử lý xung đột dữ liệu thủ công.
- **Guardrail chống memory poisoning**: Yếu tố then chốt là việc phân quyền nghiêm ngặt theo không gian người dùng (user-scoped namespace), ví dụ dữ liệu của `minh-lab17` thì user khác tuyệt đối không thể truy xuất. Đối với loại Semantic memory dùng chung (standalone graph), ta không thể đưa trực tiếp raw input của người dùng vào, mà cần có cơ chế (compiled/curated/approval) kiểm duyệt cẩn thận trước khi cho index để tránh "ô nhiễm" tri thức của bot.
