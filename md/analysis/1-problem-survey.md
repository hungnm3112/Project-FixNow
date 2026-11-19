# BƯỚC 1: KHẢO SÁT & HIỂU BÀI TOÁN (5 CÂU HỎI KHẢO SÁT NGHIỆP VỤ)

> **Đầu vào:** [Đề bài & Yêu cầu gốc (`md/requirement.md`)](../requirement.md)
> **Chuẩn áp dụng:** [1_problem_survey_5_questions.md](../../.agents/rules/2-analysis/1_problem_survey_5_questions.md) & [SystemsThinking.md](../../.agents/rules/2-analysis/SystemsThinking.md)
> **Mục đích:** Bóc tách gốc rễ bài toán điều phối đơn đặt lịch tại nhà cho **kỹ thuật viên nội bộ 24h Group**, xử lý đặc thù dữ liệu đầu vào chỉ có **địa chỉ chữ** từ Hệ thống 24h đẩy sang, xác lập ranh giới hệ thống và chốt quyết định nghiệp vụ làm đầu vào cho **Bước 2 (`md/analysis/2-actor-identification.md`)**.

---

## 1. Mô hình Chuyển hóa Yêu cầu Hệ thống

```
Pain Points (4 Nỗi đau trong khâu phân bổ đơn tại nhà hiện tại)
  ➔ Business Goals (4 Mục tiêu hệ thống OBJ-01 .. OBJ-04)
  ➔ System Boundary (Ranh giới từ lúc nhận đơn từ 24h ➔ Geocoding ➔ Bắn đơn 30s ➔ Trả kết quả về 24h)
  ➔ Acceptance Criteria (Tiêu chí nghiệm thu Geocoding 2 lớp + Xoay vòng 30s + Fallback về Cơ sở)
```

---

## 2. Năm Câu hỏi Khảo sát & Hiểu bài toán gốc

### Câu 1: Tại sao cần xây dựng hệ thống này? *(Business Value & System Boundary)*

#### 1.1. Giá trị Kinh doanh Cốt lõi (Business Value)
1. **Tự động hóa khâu Điều phối đơn đặt lịch tại nhà từ Hệ thống 24h:** Khi các kênh App/Website (đổ thẳng vào 24h) và FB/Zalo/Hotline/Caresoft (CSKH xác nhận tạo trên 24h) đẩy đơn đặt lịch tại nhà sang, phân hệ mới tự động phân bổ ngay cho kỹ thuật viên nội bộ mà không cần người điều phối gọi điện hay nhắn Zalo thủ công.
2. **Giải quyết bài toán định vị từ địa chỉ chữ & Khai thác dữ liệu vị trí nhân viên sẵn có:** Vì dữ liệu đơn đặt lịch tại nhà từ Hệ thống 24h cơ bản chỉ có **địa chỉ bằng chữ**, hệ thống tự động chuyển đổi địa chỉ sang tọa độ GPS (`Geocoding`) để đối chiếu với tính năng theo dõi vị trí nhân viên sẵn có, chọn ra KTV nội bộ gần khách hàng nhất.
3. **Chống ngâm đơn / bỏ sót đơn:** Đảm bảo mọi đơn đều luân chuyển theo nhịp **30 giây/KTV** (`KTV A ➔ KTV B ➔ ...`) và tự động rơi về **Cơ sở (Chi nhánh)** gần nhất nếu không có KTV nào nhận (hoặc khi địa chỉ quá mơ hồ không xác định được GPS).

#### 1.2. Ranh giới Hệ thống (System Boundary)
- **Bên trong phạm vi hệ thống (`In-Scope`):**
  - Nhận webhook/API đơn đặt lịch tại nhà mới từ Hệ thống 24h (kèm địa chỉ chữ của khách hàng và khu vực/quận huyện nếu có).
  - Đồng bộ/đọc dữ liệu `read-only` từ Hệ thống 24h gồm: tài khoản KTV, phân quyền, danh sách Cơ sở và dữ liệu vị trí GPS nhân viên.
  - Tự động Geocoding địa chỉ chữ sang tọa độ GPS (kèm cache/fallback 2 lớp khi không phân giải được tọa độ).
  - Quét danh sách KTV nội bộ khả dụng trong bán kính cấu hình, xếp hạng từ gần đến xa và bắn thông báo đẩy xoay vòng **30 giây/người** trên app nội bộ.
  - Xử lý thao tác **Nhận đơn / Từ chối** trong 30 giây của KTV; tự động chuyển đơn về **Cơ sở** nếu hết danh sách KTV không ai nhận (hoặc lỗi Geocoding).
  - Cho phép Quản lý Cơ sở tiếp nhận đơn chuyển về Cơ sở và giám sát KTV trực thuộc; cho phép Admin cấu hình tham số điều phối động và xem thống kê.
  - Đồng bộ kết quả gán đơn (thuộc KTV nào hoặc thuộc Cơ sở nào) trả về cho Hệ thống 24h.
- **Bên ngoài phạm vi hệ thống (`Out-of-Scope`):**
  - **Không xây dựng lại** màn hình đặt lịch cho Khách hàng (App/Web) hay màn hình CSKH tiếp nhận FB/Zalo/Hotline/Caresoft (các phần này thuộc Hệ thống 24h hiện hữu).
  - **Không bao gồm** các nghiệp vụ phía sau khi đã nhận đơn (di chuyển, kiểm tra lỗi, báo giá, xuất linh kiện, sửa chữa, thanh toán, bảo hành, đánh giá).
  - **Không quản trị mới** tài khoản KTV, phân quyền hay danh mục Cơ sở (chỉ đồng bộ `read-only` từ Hệ thống 24h) và không tuyển thợ tự do bên ngoài.

---

### Câu 2: Hiện tại quy trình đang làm như thế nào? *(As-Is Process)*

Trước khi có phân hệ tự động bắn đơn theo vị trí, luồng xử lý đơn đặt lịch tại nhà diễn ra như sau:
1. **Hình thành đơn trên Hệ thống 24h từ 2 luồng:**
   - *Luồng App / Website:* Khách hàng đặt lịch trực tiếp trên App hoặc các Website ➔ đơn đi thẳng vào **Hệ thống 24h**.
   - *Luồng CSKH đa kênh:* Khách hàng nhắn/gọi qua **Facebook, Zalo, Hotline, Caresoft (chat web)** ➔ bộ phận **CSKH** xác nhận thông tin, địa chỉ bằng chữ với khách rồi tạo đơn trên **Hệ thống 24h**.
2. **Phân đơn thủ công sau khi có đơn trên Hệ thống 24h:**
   - Bộ phận điều phối / Tổng đài hoặc Quản lý Cơ sở xem đơn mới (chỉ có địa chỉ chữ của khách), tự ước lượng khu vực rồi gọi điện hoặc nhắn tin vào nhóm Zalo hỏi xem KTV nội bộ nào đang rảnh gần khu vực đó để giao đơn.
   - Nếu không tìm được KTV rảnh ngay thì để đơn treo tại Cơ sở chờ Quản lý Cơ sở tự phân công thủ công cho KTV đi xử lý.

---

### Câu 3: Vấn đề (Nỗi đau) hiện tại là gì? *(Pain Points ➔ System Objectives)*

| Vấn đề hiện tại | Hệ quả | Mục tiêu hệ thống tương ứng |
|---|---|---|
| **1. Phân đơn thủ công qua điện thoại / nhóm Zalo sau khi có đơn trên Hệ thống 24h** | Tốn thời gian chờ của khách hàng; tổng đài và quản lý cơ sở mất công dò hỏi thủ công từng KTV đang rảnh. | **`OBJ-01` — Tự động hóa tiếp nhận từ 24h & gán đơn:** Ngay khi Hệ thống 24h đẩy đơn mới sang, phân hệ Điều phối tự động chạy luồng tìm và gán đơn không cần thao tác tay. |
| **2. Đơn đặt lịch tại nhà chỉ có địa chỉ bằng chữ, chưa khai thác được dữ liệu vị trí nhân viên sẵn có** | Dù hệ thống đã có vị trí GPS của nhân viên, nhưng đơn khách hàng chỉ là địa chỉ chữ nên không tự động tính được khoảng cách để chọn KTV gần nhất. | **`OBJ-02` — Tự động Geocoding 2 lớp & Ghép KTV theo khoảng cách GPS:** Chuyển đổi địa chỉ chữ thành `(lat, lng)` qua Geocoding API (kèm cache & fallback), sau đó xếp hạng KTV khả dụng từ gần đến xa. |
| **3. Khi KTV bận hoặc bỏ lỡ thông báo, đơn bị ngâm tại chỗ** | Không có đồng hồ đếm ngược tự động chuyển tiếp, dẫn tới đơn bị treo mà không sang được KTV khác hoặc chuyển về Cơ sở kịp thời. | **`OBJ-03` — Xoay vòng 30s & Tự động chuyển về Cơ sở:** Thiết lập bộ đếm **30 giây/người** (`KTV A ➔ KTV B`) và tự động đẩy đơn về Cơ sở gần nhất nếu hết KTV không ai nhận (hoặc không xác định được tọa độ). |
| **4. Thiếu đồng bộ trạng thái điều phối 2 chiều với Hệ thống 24h & Giám sát Cơ sở** | Khó kiểm soát đơn nào đã có KTV nhận, đơn nào bị từ chối/quá hạn 30s rơi về Cơ sở, và thiếu số liệu đánh giá tỷ lệ nhận đơn. | **`OBJ-04` — Đồng bộ 2 chiều với Hệ thống 24h & Giám sát điều phối:** Trả ngay kết quả gán đơn cho Hệ thống 24h xử lý tiếp, đồng thời cung cấp màn hình tiếp nhận đơn fallback và thống kê điều phối. |

---

### Câu 4: Ai là người tham gia sử dụng? *(Target Users thô)*

| Nhóm đối tượng (thô) | Căn cứ trong thực tế 24h | Nhu cầu chính |
|---|---|---|
| **1. Kỹ thuật viên nội bộ 24h (`ACT`)** | Nhân sự kỹ thuật cơ hữu sử dụng app nội bộ để đi sửa tại nhà. | Bật/tắt trạng thái sẵn sàng; nhận Push Notification khi có đơn gần nhất và bấm **Nhận đơn** hoặc **Từ chối** trong 30 giây. |
| **2. Quản lý Cơ sở (`ACT`)** | Nhân sự quản lý tại các chi nhánh 24h Group tiếp nhận đơn fallback. | Nhận thông báo và tiếp nhận xử lý các đơn tự động chuyển về Cơ sở; giám sát danh sách đơn & vị trí KTV thuộc cơ sở trên bản đồ. |
| **3. Quản trị viên Hệ thống (`ACT`)** | Quản trị vận hành nền tảng điều phối. | Cấu hình các tham số động (bán kính quét GPS, thời gian đếm ngược 30s, độ trễ GPS tối đa) và xem báo cáo thống kê / nhật ký điều phối *(tài khoản, phân quyền, danh sách Cơ sở đọc từ Hệ thống 24h)*. |
| **4. Hệ thống Vận hành 24h hiện hữu (`EXT`)** | Nền tảng gốc nơi đơn App/Web & đơn CSKH (FB, Zalo, Hotline, Caresoft) đổ về và xử lý hậu kỳ. | Đẩy đơn đặt lịch tại nhà (kèm địa chỉ chữ) sang phân hệ Điều phối; đồng bộ dữ liệu nguồn `read-only` (KTV, Phân quyền, Cơ sở, Vị trí nhân viên) và nhận lại kết quả gán đơn để xử lý tiếp. |
| **5. Dịch vụ Bản đồ & Push Notification (`EXT`)** | Hạ tầng bên thứ 3 phục vụ chuyển đổi địa chỉ chữ & thông báo đẩy. | Chuyển đổi địa chỉ bằng chữ sang tọa độ GPS `(lat, lng)` (`Geocoding API`) và bắn thông báo đẩy 30 giây tới thiết bị của KTV / Cơ sở. |

*(Lưu ý: **Khách hàng** và **Bộ phận CSKH** thao tác tạo đơn ở App/Web và Hệ thống 24h bên ngoài, không đăng nhập hay tương tác trực tiếp vào phân hệ Điều phối nên thuộc dữ liệu đầu vào của đơn đặt lịch, không phải Actor của hệ thống này).*

---

### Câu 5: Kết quả mong muốn là gì? *(Acceptance Criteria)*

| Mục tiêu hệ thống (từ Câu 3) | Tiêu chí đạt cụ thể (kiểm chứng được) |
|---|---|
| **`OBJ-01` — Tự động hóa tiếp nhận & khóa đơn chống trùng** | • Hệ thống tiếp nhận tự động 100% đơn đặt lịch tại nhà từ Hệ thống 24h đẩy sang qua API.<br>• Khi một KTV bấm **Nhận đơn** hợp lệ, đơn được khóa ngay lập tức (ngăn chặn 2 KTV nhận trùng cùng 1 đơn). |
| **`OBJ-02` — Geocoding 2 lớp linh hoạt + Fallback** | • Nếu đơn từ 24h đã kèm `(lat, lng)` thì dùng luôn; nếu chỉ có **địa chỉ bằng chữ** (mặc định thực tế), Backend tự động kiểm tra bộ nhớ đệm địa chỉ rồi gọi **Geocoding API** để phân giải ra `(lat, lng)`.<br>• Nếu địa chỉ quá mơ hồ không xác định được tọa độ tin cậy ➔ Tự động chuyển thẳng đơn về **Cơ sở** theo Quận/Huyện của đơn, không làm treo luồng.<br>• Sau khi có tọa độ, quét toàn bộ KTV nội bộ đang sẵn sàng trong bán kính cấu hình và xếp hạng chính xác **từ gần đến xa**. |
| **`OBJ-03` — Xoay vòng 30s & Tự động chuyển về Cơ sở** | • Bắn thông báo tuần tự từng KTV với thời gian chờ **30 giây/người**: hết 30s không phản hồi hoặc KTV bấm từ chối ➔ tự động thu hồi lượt cũ và bắn ngay sang KTV gần tiếp theo.<br>• Khi toàn bộ danh sách KTV khả dụng trong bán kính đều không nhận (hoặc ngay từ đầu không có KTV nào trong bán kính) ➔ tự động gán đơn về **Cơ sở gần vị trí khách nhất** và gửi thông báo cho Quản lý Cơ sở. |
| **`OBJ-04` — Đồng bộ 2 chiều với Hệ thống 24h & Giám sát** | • Ngay khi đơn được gán cho KTV hoặc chuyển về Cơ sở, kết quả được đồng bộ chính xác về Hệ thống 24h để xử lý nghiệp vụ phía sau.<br>• Quản lý Cơ sở xem được đơn fallback & vị trí KTV thuộc cơ sở; Admin cấu hình được tham số điều phối động và xem tỷ lệ nhận đơn / quá hạn 30s / fallback. |

---

## 3. Quản lý Giả định & Chốt Quyết định Nghiệp vụ / Kiến trúc

| Mã | Hạng mục Quyết định | Nội dung Đã Chốt & Trạng thái |
|:---:|---|---|
| **`DEC-01`** | **Đầu mối Tiếp nhận Đơn (Upstream)** | **`[Đã xác nhận]`** 100% đơn đặt lịch tại nhà (từ App/Web lẫn do CSKH tạo từ FB, Zalo, Hotline, Caresoft) đi qua **Hệ thống 24h** rồi đẩy tự động sang phân hệ Điều phối qua API. |
| **`DEC-02`** | **Xác định Tọa độ Khách hàng (Cách 1 — Hybrid Geocoding + Fallback 2 lớp)** | **`[Đã xác nhận]`** Dữ liệu từ Hệ thống 24h cơ bản chỉ có **địa chỉ bằng chữ** (hỗ trợ nhận kèm `lat, lng` nếu có). Backend Điều phối tự động tra Cache / gọi **Geocoding API** để đổi địa chỉ chữ thành `(lat, lng)`; nếu địa chỉ quá mơ hồ không phân giải được tọa độ thì **tự động chuyển thẳng đơn về Cơ sở** theo Quận/Huyện. |
| **`DEC-03`** | **Phạm vi Actor & Ranh giới 24h** | **`[Đã xác nhận]`** Chỉ giữ **3 Actor con người nội bộ** (`Kỹ thuật viên nội bộ`, `Quản lý Cơ sở`, `Quản trị viên Hệ thống`) và **2 Hệ thống ngoài** (`Hệ thống Vận hành 24h`, `Dịch vụ Bản đồ/Push`). Tài khoản KTV, phân quyền, danh sách Cơ sở & vị trí nhân viên đồng bộ `read-only` từ Hệ thống 24h; toàn bộ nghiệp vụ sau khi gán đơn do Hệ thống 24h xử lý tiếp. |
| **`DEC-04`** | **Thuật toán Quét KTV & Chuyển tiếp 30 giây** | **`[Đã xác nhận]`** Quét KTV nội bộ đang sẵn sàng trong bán kính cấu hình, xếp hạng **từ gần đến xa**, bắn thông báo tuần tự **30 giây/người** (`KTV A ➔ KTV B`); không ai nhận thì **tự động chuyển về Cơ sở gần nhất**. |
| **`DEC-05`** | **Tham số Điều phối Cấu hình Động** | **`[Đã xác nhận / Cấu hình bởi Admin]`** Các ngưỡng gồm: bán kính quét GPS mặc định (km), thời gian đếm ngược mỗi lượt (mặc định `30s`), độ trễ tín hiệu GPS tối đa cho phép của KTV được cấu hình động trên hệ thống quản trị. |

---

> **Đầu ra chuyển tiếp sang Bước 2:** Toàn bộ nhóm đối tượng ở **Câu 4**, quy trình ở **Câu 2**, tiêu chí nghiệm thu ở **Câu 5** và các quyết định `DEC-01`..`DEC-05` là đầu vào trực tiếp cho **Bước 2: Truy vết & Bóc tách Tác nhân (`md/analysis/2-actor-identification.md`)**.
