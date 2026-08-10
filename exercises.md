# Phiếu Phản Ánh — K4 Ngày 12


>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: ..........................  Mã học viên: ..........................

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi
khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc
"chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Ví dụ, khi deploy production mà quên đặt `API_TOKEN`, app dừng ngay lúc khởi động và cloud báo deploy thất bại. Nhờ vậy tôi biết ngay cấu hình secret bị thiếu, thay vì app chạy với token `changeme` để người ngoài đoán được và sử dụng API miễn phí.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

Ví dụ: `{"event": "chat_completed", "severity": "INFO", "ts": "2026-08-10T08:00:00+00:00", "client_id": "sv01", "prompt_tokens": 3, "completion_tokens": 37, "usd_cost": 0.0000226}`. Từ dòng này có thể lọc/đếm số request theo `event` hoặc `client_id`, và tính tổng chi phí/token hoặc đặt cảnh báo khi `usd_cost` tăng. Một câu `print` không có cấu trúc ổn định để máy tự lọc, thống kê và cảnh báo.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t chat:single .
docker build -t chat:multi .
docker images | grep chat
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | khoảng 1.800 MB |
| Multi-stage | khoảng 180 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

Khi chỉ sửa `app/main.py`, các layer từ `FROM`, `WORKDIR`, `COPY requirements.txt` và `RUN pip install` được dùng lại; các layer `COPY` source và những layer sau đó phải tạo lại. Nếu đặt `COPY . .` trước `RUN pip install`, mỗi lần sửa bất kỳ source nào Docker sẽ cache miss ở `COPY`, làm `pip install` chạy lại, khiến build chậm hơn.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

Một lỗ hổng cho phép kẻ tấn công thực thi mã trong app → mã chạy với quyền của process container → nếu process là root thì kẻ tấn công có quyền cao trong container và có thể lợi dụng lỗ hổng container/runtime để tác động tới host. `USER appuser` làm process Python chạy bằng user thường, cắt chuỗi ở bước app bị xâm nhập không còn có quyền root; các lớp bảo vệ khác vẫn cần thiết vì đây không phải ranh giới tuyệt đối.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

`WWW-Authenticate: Bearer` là yêu cầu của HTTP Bearer authentication (RFC 6750), cho client biết cần gửi thông tin xác thực theo scheme Bearer. Cả ba trường hợp trả cùng thông báo `invalid or missing bearer token` để không tiết lộ token có đúng một phần, scheme nào được chấp nhận hay thông tin dò đoán khác; lỗi chung giảm khả năng user enumeration và token probing.

---

### Câu 6 — Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả **cùng
một** thông báo lỗi cho cả ba trường hợp (thiếu header, sai scheme, sai token)
thay vì nói rõ sai ở đâu cho người dùng dễ sửa?

Xô ban đầu có 10 token, nên client gửi được 10 request liên tiếp rồi request thứ 11 nhận 429. Sau 10 phút, tốc độ nạp là 10 token/phút nhưng `min(capacity, ...)` giữ xô ở mức tối đa 10. Nếu bỏ `min`, xô tích thêm 100 token trong 10 phút, thành 110 token, nên client gửi được 110 request trước khi bị 429 (giả sử request gửi ngay sau đó).

---

### Câu 7 — Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi
liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn
`min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

Với hạn mức $30/tháng, sự cố có thể tiêu gần như toàn bộ $30 trước khi hết tháng; service chỉ tự hồi phục khi sang tháng mới (hoặc phải can thiệp để chặn). Với $1/ngày, thiệt hại tối đa trong ngày là $1; key ngân sách đổi theo ngày UTC nên service tự hồi phục khi sang ngày UTC tiếp theo, không cần restart hay sửa tay.

---

### Câu 8 — Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự
cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là
bao nhiêu và service tự hồi phục khi nào?

Nếu `/healthz` cũng kiểm tra Redis, khi Redis mất 30 giây cả 3 container lần lượt trả health check lỗi. Load balancer/orchestrator coi cả 3 container không khỏe, loại chúng khỏi traffic rồi có thể restart cả 3. Trong thời gian đó không còn instance phục vụ dù process Python vẫn sống; khi Redis trở lại, các container phải khởi động và health check thành công mới nhận traffic. Tách `/healthz` (chỉ kiểm tra process) khỏi `/readyz` giúp giữ container sống, chỉ tạm ngừng route traffic qua readiness.

---

### Câu 9 — /healthz khác /readyz (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

Khi deploy Render, lỗi ban đầu là health check bị timeout vì service không lắng nghe đúng cổng cloud cấp. Tôi kiểm tra log deploy và đối chiếu biến `PORT`/cấu hình health check, thấy command phải đọc `${PORT:-8000}` thay vì cố định cổng. Tôi sửa Dockerfile để Uvicorn bind `0.0.0.0` và dùng `--port ${PORT:-8000}`, đặt `PORT=8000` trên Render, rồi deploy lại; sau đó `/healthz` trả 200 và `/readyz` trả 200 khi Redis sẵn sàng.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> *Câu trả lời của bạn*
