# ĐỀ BÀI & YÊU CẦU BÀI TOÁN: PROJECT-FIXNOW

## 1. Bối cảnh thực tế

**24h Group** đang vận hành hệ thống sửa chữa với các cơ sở và đội ngũ kỹ thuật viên nội bộ. Đơn khách hàng đặt lịch sửa chữa tại nhà phát sinh từ nhiều kênh khác nhau:
- Khách hàng tự đặt lịch trên **App / Website** ➔ đơn đi thẳng vào **Hệ thống 24h**.
- Khách hàng liên hệ qua **Facebook, Zalo, Hotline, Caresoft (chat web)...** ➔ bộ phận **CSKH** xác nhận với khách và tạo đơn trên **Hệ thống 24h**.

Hiện nay, **Hệ thống 24h** đã có sẵn danh mục cơ sở, tài khoản nhân sự, tính năng **theo dõi vị trí nhân viên kỹ thuật** và đảm nhận toàn bộ các bước nghiệp vụ phía sau (kiểm tra, báo giá, sửa chữa, thanh toán, bảo hành). Tuy nhiên, khâu phân bổ đơn đặt lịch sửa tại nhà mới cho kỹ thuật viên nội bộ vẫn đang làm thủ công: dữ liệu đơn từ Hệ thống 24h chỉ có **địa chỉ bằng chữ** của khách hàng, chưa có cơ chế tự động chuyển đổi ra tọa độ GPS để đối chiếu với vị trí nhân viên kỹ thuật nhằm bắn đơn tự động và xoay vòng khi không có người nhận.

---

## 2. Yêu cầu bài toán

Xây dựng phân hệ **Tiếp nhận & Điều phối đơn đặt lịch tại nhà tự động** tích hợp bổ sung vào hệ thống/app nội bộ của 24h Group với phạm vi tập trung:
- Tiếp nhận dữ liệu đơn đặt lịch sửa chữa tại nhà từ **Hệ thống 24h** đẩy sang (thông tin vị trí đầu vào cơ bản là **địa chỉ bằng chữ** của khách hàng).
- Tự động phân giải **địa chỉ bằng chữ** của khách hàng sang tọa độ GPS (Geocoding) thông qua dịch vụ bản đồ (có cơ chế dự phòng nếu địa chỉ không phân giải được tọa độ thì chuyển thẳng đơn về Cơ sở).
- Dựa vào tọa độ vị trí của khách hàng và **vị trí hiện tại của nhân viên kỹ thuật nội bộ** (từ tính năng theo dõi vị trí nhân viên sẵn có) để bắn thông báo nhận đơn cho kỹ thuật viên phù hợp ở gần nhất.
- Áp dụng cơ chế chuyển tiếp tự động **30 giây**: nếu **Kỹ thuật viên A** không nhận đơn thì sau **30 giây** tự động chuyển đơn sang **Kỹ thuật viên B**. Nếu không có kỹ thuật viên nào nhận đơn thì hệ thống **tự động chuyển đơn đó về Cơ sở**.
- Đồng bộ kết quả gán đơn (đã giao cho Kỹ thuật viên nào hoặc đã chuyển về Cơ sở nào) trả về **Hệ thống 24h** để xử lý tiếp toàn bộ các nghiệp vụ phía sau.

---

## 3. Luồng nghiệp vụ tổng thể

1. **Hệ thống 24h** đẩy đơn đặt lịch sửa tại nhà mới (kèm địa chỉ chữ của khách hàng) sang phân hệ Điều phối.
2. Phân hệ Điều phối tự động phân giải địa chỉ chữ sang tọa độ GPS (`Geocoding`), sau đó quét vị trí hiện tại của các kỹ thuật viên nội bộ gần nhất để bắn thông báo nhận đơn cho **Kỹ thuật viên A** *(nếu địa chỉ mơ hồ không xác định được tọa độ thì chuyển thẳng đơn về Cơ sở)*.
3. Nếu sau **30 giây** Kỹ thuật viên A không nhận đơn (hoặc từ chối), hệ thống tự động chuyển tiếp thông báo sang **Kỹ thuật viên B** gần tiếp theo; nếu hết lượt mà không ai nhận thì **tự động chuyển đơn về Cơ sở**.
4. Khi đơn đã được gán cho Kỹ thuật viên hoặc chuyển về Cơ sở, kết quả được đồng bộ lại cho **Hệ thống 24h hiện hữu** để tiếp tục xử lý các nghiệp vụ phía sau.
