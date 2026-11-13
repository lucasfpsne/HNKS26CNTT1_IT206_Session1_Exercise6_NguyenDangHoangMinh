# Context Window và quản lý bộ nhớ hội thoại

## Phần 1. Giải thích hình tượng “Chiếc cốc nước”

### 1. Context Window là gì?

Context Window là lượng thông tin tối đa mà mô hình có thể xử lý trong một lần phản hồi, được tính bằng token.

Có thể hình dung Context Window như một **chiếc cốc có dung tích giới hạn**:

- **Dung tích cốc:** Giới hạn token của mô hình.
- **Nước trong cốc:** Các chỉ dẫn, tin nhắn, câu trả lời và dữ liệu được đưa vào ngữ cảnh.
- **Đổ thêm nước:** Tiếp tục gửi tin nhắn và tích lũy câu trả lời.
- **Cốc đầy:** Không còn đủ chỗ để giữ toàn bộ nội dung và tạo câu trả lời tiếp theo.

Context Window giống “RAM làm việc” theo nghĩa ẩn dụ, không phải RAM vật lý của máy tính hay kho lưu trữ toàn bộ lịch sử chat.

### 2. Vì sao AI quên thông tin ở lượt 1?

Ở lượt 1, khách hàng cung cấp:

> “Tôi tên Trần Văn B, mã đơn hàng #MTX-9988, mua tại MartX Q1.”

Lúc này, thông tin vẫn nằm trong “cốc”, nên AI có thể gọi đúng tên và nhắc đúng mã đơn.

Sau khoảng 40 lượt trao đổi, nội dung tư vấn và các câu trả lời dài làm lượng token tăng lên. Theo giả định của đề bài, ngữ cảnh đã chạm giới hạn khoảng 32.000 token.

Để tiếp tục xử lý, hệ thống cắt bỏ một phần nội dung cũ. Hiện tượng này gọi là **Truncation — cắt bớt dữ liệu khỏi ngữ cảnh đang được sử dụng**.

Nếu lượt 1 bị loại bỏ mà không được giữ trong bản tóm tắt hoặc đưa lại vào ngữ cảnh, AI không còn trực tiếp thấy tên khách, mã đơn và nơi mua.

**Điểm quan trọng:** Tin nhắn có thể vẫn hiển thị trên giao diện, nhưng không còn nằm trong phần thông tin mô hình được đọc ở lượt hiện tại.

### 3. Phân tích phản hồi sai ở lượt 42

AI trả lời:

> “Chào chị Mai, chị cho em xin lại mã đơn hàng với ạ?”

Có hai vấn đề:

- **Hỏi lại mã đơn:** Phù hợp với tình huống thông tin ban đầu đã bị loại khỏi ngữ cảnh.
- **Gọi khách là “chị Mai”:** Là thông tin không có căn cứ trong nhật ký được cung cấp. Việc mất ngữ cảnh không khiến tên khách tự động biến thành tên khác; đây là lỗi suy đoán hoặc nhầm lẫn của AI.

Khi thiếu dữ liệu, phản hồi phù hợp hơn là:

> “Anh/chị vui lòng xác nhận lại mã đơn hàng để em kiểm tra chính sách hoàn tiền và các khoản phí liên quan.”

AI không nên tự đặt tên khách hoặc khẳng định điều kiện hoàn tiền khi chưa có dữ liệu.

## Phần 2. Hai phương án quản lý bộ nhớ

### Phương án 1: Người dùng tóm tắt và chuyển sang phiên chat mới

**Cách thực hiện:**

1. Yêu cầu AI trả lời ngắn gọn, tránh nhắc lại thông tin không cần thiết.
2. Khi hội thoại bắt đầu dài, yêu cầu tạo bản tóm tắt bàn giao.
3. Kiểm tra bản tóm tắt với thông tin gốc, đặc biệt là tên và mã đơn.
4. Mở phiên chat mới, dán bản tóm tắt đã kiểm tra rồi tiếp tục tư vấn.

**Prompt yêu cầu tóm tắt:**

> Tóm tắt phiên tư vấn thành các mục: thông tin khách hàng, mã đơn, nơi mua, vấn đề đang xử lý, nội dung đã xác nhận và việc cần làm tiếp. Giữ nguyên tên và mã đơn. Thông tin chưa có phải ghi “chưa xác nhận”, không tự suy đoán.

**Ví dụ bản bàn giao:**

- Khách hàng: Trần Văn B.
- Mã đơn hàng: #MTX-9988.
- Nơi mua: MartX Q1.
- Nội dung đã trao đổi: Chi tiết các mặt hàng rau củ.
- Yêu cầu hiện tại: Hỏi về điều kiện hoàn tiền và phí liên quan.
- Điều kiện áp dụng cho đơn này: Chưa xác nhận.
- Việc tiếp theo: Kiểm tra thông tin đơn và chính sách phù hợp.

**Tác dụng:** Thay lịch sử dài bằng bản tóm tắt ngắn, tạo thêm dung lượng cho cuộc trao đổi tiếp theo.

**Hạn chế:** Tóm tắt có thể bỏ sót hoặc ghi sai thông tin, nên nhân viên cần kiểm tra trước khi sử dụng.

### Phương án 2: Hệ thống lưu dữ liệu ngoài hội thoại và quản lý ngân sách token

**Cách thực hiện:**

1. Lưu thông tin quan trọng vào CRM hoặc cơ sở dữ liệu, gắn với đúng khách hàng và phiên tư vấn.
2. Trước mỗi lần AI phản hồi, truy xuất các dữ liệu cần thiết và đưa vào ngữ cảnh.
3. Chỉ giữ các lượt trao đổi gần nhất cùng bản tóm tắt phần hội thoại cũ.
4. Theo dõi số token đầu vào và dành dung lượng cho câu trả lời.
5. Chủ động tóm tắt hoặc bỏ nội dung không liên quan trước khi chạm giới hạn.

**Dữ liệu cần lưu riêng:**

| Trường dữ liệu | Giá trị trong tình huống |
| --- | --- |
| Tên khách hàng | Trần Văn B |
| Mã đơn hàng | #MTX-9988 |
| Nơi mua | MartX Q1 |
| Yêu cầu hiện tại | Kiểm tra điều kiện hoàn tiền và phí liên quan |
| Kết quả kiểm tra chính sách | Chưa xác nhận |

Thông tin đã được xác thực phải được ưu tiên hơn nội dung AI tự suy đoán. Nếu thiếu hoặc mâu thuẫn dữ liệu, hệ thống yêu cầu xác nhận thay vì tự điền.

**Tác dụng:** Thông tin quan trọng không phụ thuộc hoàn toàn vào việc tin nhắn đầu tiên còn nằm trong Context Window hay không.

**Hạn chế:** Cần xây dựng cơ chế lưu trữ, truy xuất đúng hồ sơ và kiểm soát dữ liệu đưa vào mô hình.

## Kết luận

AI “quên” trong tình huống này vì thông tin cũ không còn nằm trong ngữ cảnh được sử dụng, dù lịch sử chat có thể vẫn hiển thị.

Hai giải pháp phù hợp là:

1. **Người dùng:** Tóm tắt, kiểm tra và chuyển sang phiên chat mới.
2. **Hệ thống:** Lưu thông tin quan trọng bên ngoài hội thoại, truy xuất lại khi cần và kiểm soát số token.
