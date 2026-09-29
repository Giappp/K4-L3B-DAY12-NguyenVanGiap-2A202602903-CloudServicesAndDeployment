# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Văn Giáp |
| Mã học viên | 2A202602903 |
| Repo | https://github.com/Giappp/K4-L3B-DAY12-NguyenVanGiap-2A202602903-CloudServicesAndDeployment.git |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-production-fda1.up.railway.app |
| Platform | Railway |
| Ngày deploy | 29/09/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis add-on của platform |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-agent-production-fda1.up.railway.app/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://day12-agent-production-fda1.up.railway.app/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://day12-agent-production-fda1.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://day12-agent-production-fda1.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
giap@static:~$ curl -i https://day12-agent-production-fda1.up.railway.app/health
HTTP/2 200 
content-type: application/json
date: Tue, 29 Sep 2026 04:57:17 GMT
server: railway-hikari
x-railway-request-id: c5lhzu-xRMmmCH0Jjq4OvQ
content-length: 57
x-hikari-trace: hkg1.hn7d
x-railway-edge: hkg1

{"status":"ok","service":"day12-agent","version":"1.0.0"}


giap@static:~$ curl -i https://day12-agent-production-fda1.up.railway.app/ready
HTTP/2 200 
content-type: application/json
date: Tue, 29 Sep 2026 04:57:40 GMT
server: railway-hikari
x-railway-request-id: nPn9x6W-RNm-QqSJ6WHkDg
content-length: 31
x-hikari-trace: hkg1.hn7d
x-railway-edge: hkg1

{"status":"ready","redis":true}

giap@static:~$ curl -i -X POST https://day12-agent-production-fda1.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'
HTTP/2 401 
content-type: application/json
date: Tue, 29 Sep 2026 04:58:06 GMT
server: railway-hikari
x-railway-request-id: me7s_QSrRWqbt1SrxtoGcA
content-length: 39
x-hikari-trace: hkg1.hn7d
x-railway-edge: hkg1

{"detail":"invalid or missing API key"}

(.venv) giap@fedora:~/Desktop/Workspace/Vin AI/K4-L3B-DAY12-NguyenVanGiap-2A202602903-CloudServicesAndDeployment$ curl -i -X POST https://day12-agent-production-fda1.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'
HTTP/2 200 
content-type: application/json
date: Tue, 29 Sep 2026 05:00:00 GMT
server: railway-hikari
x-railway-request-id: -AfGd_DfS0-N-qpJlt7tkg
content-length: 279
x-hikari-trace: hkg1.hn7d
x-railway-edge: hkg1
vary: accept-encoding

{"answer":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud.","user_id":"sv-test","history_length":0,"cost_usd":2.145e-05,"tokens":{"in":3,"out":35}}

(.venv) giap@fedora:~/Desktop/Workspace/Vin AI/K4-L3B-DAY12-NguyenVanGiap-2A202602903-CloudServicesAndDeployment$ for i in $(seq 1 15); do   curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-agent-production-fda1.up.railway.app/ask     -H "Content-Type: application/json"     -H "X-API-Key: $AGEN
T_API_KEY"     -H "X-User-Id: sv-test"     -d '{"question":"test"}'; done; echo
200 200 200 200 200 200 200 200 200 429 429 429 200 429 429 
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

## Nếu Dùng Phương Án Dự Phòng

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:

```
```
