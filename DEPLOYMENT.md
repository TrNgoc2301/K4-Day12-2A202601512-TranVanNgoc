# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|---|---|
| Họ và tên | Trần Văn Ngọc |
| Mã học viên | 2A202601512 |
| Repo | https://github.com/TrNgoc2301/K4-Day12-2A202601512-TranVanNgoc |

## Service

| Mục | Nội dung |
|---|---|
| Public URL | https://day12-chat-bbvs.onrender.com |
| Platform | Render Blueprint |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

| Biến | Đã set | Ghi chú |
|---|---|---|
| `PORT` | ✅ | 8000 |
| `API_TOKEN` | ✅ | đặt trong Render dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Render Key Value qua `fromService` |
| `BUCKET_CAPACITY` | ✅ | 10 |
| `REFILL_PER_MINUTE` | ✅ | 10 |
| `DAILY_BUDGET_USD` | ✅ | 1.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Kết Quả Chạy Thật

- `GET /health` trả `200` với trạng thái `ok`.
- `GET /ready` trả `200` với Redis sẵn sàng.
- `POST /ask` không có API key trả `401` cùng header `WWW-Authenticate: Bearer`.
- `POST /ask` kèm `X-API-Key` hợp lệ trả `200` và JSON có câu trả lời.

## Ảnh Chụp Màn Hình

Service đã được triển khai qua Render Blueprint với web service `day12-chat`
và Key Value `day12-chat-redis`.
