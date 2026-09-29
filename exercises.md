# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Văn Giáp  Mã học viên: 2A202602903

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

Việc không để giá trị mặc định giúp tránh trường hợp quên cài đặt API Key đặc biệt là trong trường hợp deploy lên production khi mà việc quên set biến API Key có thể gây ra lỗi bảo mật hoặc chết tính năng.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

{"timestamp":"2026-09-29T05:00:00.000000Z","level":"info","event":"ask_completed","user_id":"sv-test","tokens_in":3,"tokens_out":35,"cost_usd":0.00002145}
2 Việc có thể làm với dòng log:
- Các hệ thống giám sát/log aggregator (như Datadog, Grafana Loki, ELK) có thể parse trường dữ liệu tự động để vẽ biểu đồ và alert theo ngưỡng cost_usd hoặc tokens.
- Cho phép lọc và truy vấn chính xác theo user_id hoặc event mà không cần dùng regex phức tạp như text log thông thường.


---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.19GB |
| Multi-stage | 185 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

Phần dung lượng chênh lệch (~1GB) bao gồm các công cụ build/compiler (như gcc, build-essential, python dev headers), cache tải về của pip và các file tạm trong quá trình cài đặt. Bản multi-stage nhỏ gọn hơn nhiều vì chỉ copy các package đã cài đặt sang stage chạy (runtime), loại bỏ hoàn toàn trình biên dịch và file rác không cần thiết cho quá trình chạy. 

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

- Khi sửa `app/main.py`: Các layer từ đầu đến trước lệnh copy mã nguồn (base image, `COPY requirements.txt`, `RUN pip install`) đều được dùng lại từ cache (`CACHED`). Chỉ có layer copy mã nguồn `app/` và các layer sau đó mới phải chạy lại.
- Nếu đặt `COPY . .` lên trước `RUN pip install`: Mỗi khi sửa bất kỳ file mã nguồn nào, cache của layer `COPY . .` bị vô hiệu lực, kéo theo layer `RUN pip install` bắt buộc phải chạy lại từ đầu. Kết quả là việc tải và cài đặt toàn bộ dependencies bị lặp lại, khiến thời gian build lâu hơn rất nhiều.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

- Chuỗi sự kiện:
  1. Ứng dụng Python xuất hiện lỗ hổng bảo mật (ví dụ: RCE, command injection, path traversal).
  2. Kẻ tấn công kích hoạt mã độc bên trong container. Do container chạy mặc định bằng `root` (UID 0), kẻ tấn công chiếm toàn quyền root trong môi trường container.
  3. Từ quyền root container, kẻ tấn công khai thác lỗ hổng kernel của máy host hoặc container breakout để leo thang chiếm quyền root trên hệ điều hành máy host.
- Lệnh `USER appuser` cắt đứt chuỗi tấn công ngay tại bước 2: Khi tiến trình Python chạy dưới quyền user giới hạn (non-root), kẻ tấn công dù có RCE cũng chỉ có quyền của user thường trong container, không thể ghi/sửa các file hệ thống, không thể tương tác namespace cấp cao hay breakout ra máy host.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

- Số request tối đa có thể gửi trong 2 giây liên tiếp là: **20 request**.
- Cách đạt được: Người dùng gửi 10 request vào giây cuối cùng của phút thứ nhất (10:00:59, dùng hết hạn mức 10/phút). Đúng 1 giây sau, khi bước sang giây 10:01:00 (hoặc 10:01:01), bộ đếm phút đồng hồ tự động reset về 0 cho phút mới, người dùng gửi tiếp ngay 10 request nữa. Tổng cộng có 20 request được gửi thành công chỉ trong 2 giây liên tiếp mà vẫn không vi phạm hạn mức của từng phút đồng hồ.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

- Điểm khác nhau:
  - **Rate limit**: Giới hạn *tần suất / tốc độ* request trong một khoảng thời gian ngắn (ví dụ: 10 request/phút) để chống nghẽn mạng, spam và bảo vệ server khỏi bị quá tải.
  - **Cost guard**: Giới hạn *tổng chi phí / số tiền* tiêu thụ tích lũy trong chu kỳ dài (ví dụ: 10 USD/tháng) để bảo vệ ngân sách chi trả cho nhà cung cấp mô hình AI/LLM.
- Tình huống Rate limit cho qua nhưng Cost guard chặn: Người dùng chỉ gửi 1 request trong 10 phút (rất thấp, rate limit cho qua), nhưng request này chứa câu hỏi và ngữ cảnh khổng lồ làm tiêu tốn 100.000 token, vượt quá ngân sách tháng còn lại -> Cost guard chặn (HTTP 402).
- Tình huống Cost guard cho qua nhưng Rate limit chặn: Người dùng mới tiêu 0.01$ trong ngân sách 10$ của tháng (ngân sách còn nhiều, cost guard cho qua), nhưng gửi dồn dập 15 request chỉ trong vòng vài giây -> Rate limit chặn (HTTP 429).

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

Thứ tự các sự kiện xảy ra khi Redis mất kết nối 30 giây:
1. Redis gặp sự cố mất kết nối trong 30 giây.
2. Bộ điều phối (orchestrator) thực hiện liveness probe định kỳ tới cả 3 container agent.
3. Do endpoint gộp chung có kiểm tra Redis, cả 3 container đồng loạt trả về lỗi/unhealthy.
4. Orchestrator nhận định process của cả 3 container đã bị hỏng/treo, quyết định đồng thời dừng và khởi động lại (restart) cả 3 container.
5. Khi khởi động lại, các container mới vẫn chưa thể kết nối tới Redis (đang trong 30s mất kết nối), tiếp tục probe thất bại và rơi vào vòng lặp restart liên tục (CrashLoopBackOff).
6. Toàn bộ hệ thống sập hoàn toàn (100% downtime), không còn container nào phục vụ người dùng kể cả khi Redis bắt đầu kết nối lại bình thường.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

Nếu lịch sử được lưu trong dict Python trong bộ nhớ RAM của từng process:
Vì load balancer phân phối các request tới 3 container khác nhau (A, B, C), mỗi container chỉ giữ một dict riêng trong RAM của nó. Bạn sẽ thấy `history_length` thay đổi thất thường, không tăng dần đều liên tục mà nhảy lộn xộn hoặc ngẫu nhiên quay về 0 tùy thuộc vào request kế tiếp được điều phối tới container nào (ví dụ: request 1 vào container A -> history=0; request 2 vào container B -> history=0; request 3 vào container A -> history=1...). Agent sẽ liên tục "mất trí nhớ" về câu hỏi vừa nói trước đó.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

Lỗi gặp khi deploy trên cloud là sai REDIS_URL, app thông báo lỗi 500 server error, tôi tìm ra nguên nhân bằng cách xem log, xem gợi ý từ test checkpoint -> nhận ra thiếu REDIS_URL từ Railway và add lại là xong
