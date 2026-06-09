# Readiness Checklist – Lab 05

Đây là danh sách kiểm tra (checklist) để đảm bảo stack Docker Compose của bạn đã sẵn sàng trước khi gửi bài. Hãy tick vào mỗi mục sau khi hoàn thành.

- [x] **Database ready:** container DB đã chạy và phản hồi `pg_isready`. Kiểm tra bằng `docker exec -it fit4110-db-lab05 pg_isready -U lab05`.
- [x] **AI service ready:** container AI service trả về `200` cho endpoint `/health` và `/predict` hoạt động.
- [x] **API ready:** container API trả `200` cho `/health` và có thể tạo/lấy readings khi token hợp lệ.
- [x] **Environment variables:** `.env` đã được thiết lập đúng (APP_PORT, POSTGRES_USER, AUTH_TOKEN,…). Không sử dụng secret thật; lưu secret vào `.env` cục bộ, commit `.env.example`.
- [x] **Network & Ports:** mạng `team-internal` hoạt động; API gọi được AI bằng hostname `ai-service`; ports 8000 (API), 9000 (AI) và 5432 (DB) được map đúng.
- [x] **Image tags:** Image được build với tag phù hợp và sẵn sàng để push lên registry (ghcr.io hoặc Docker Hub).

## Chi tiết kiểm tra

### 1. Database Ready
- Container: `fit4110-db-lab05`
- Port: 5432 (internal)
- Healthcheck: `pg_isready -U lab05`
- Database: `iotdb`
- User: `lab05`
- Status: ✅ Ready

### 2. AI Service Ready  
- Container: `fit4110-ai-lab05`
- Port: 9000 (internal)
- Endpoints:
  - GET `/health` → 200 OK
  - POST `/predict` → 200 OK (returns dummy objects)
- Status: ✅ Ready

### 3. API Ready
- Container: `fit4110-api-lab05`
- Port: 8000 (mapped to host)
- Endpoints tested:
  - GET `/health` → 200 OK
  - POST `/readings` (with valid token) → 201 Created
  - GET `/readings/latest` → 200 OK
  - GET `/readings/{id}` → 200 OK / 404 Not Found
- Authentication: Bearer token required
- Status: ✅ Ready

### 4. Environment Variables
- File `.env` created from `.env.example`
- All required variables configured:
  - `APP_HOST=0.0.0.0`
  - `APP_PORT=8000`
  - `AUTH_TOKEN=local-dev-token`
  - `SERVICE_NAME=iot-ingestion`
  - `SERVICE_VERSION=0.5.0`
  - `POSTGRES_USER=lab05`
  - `POSTGRES_PASSWORD=lab05pass`
  - `POSTGRES_DB=iotdb`
- Secret management: ✅ Using local dev token, not committed to repo
- Status: ✅ Configured

### 5. Network & Ports
- Networks:
  - `team-internal` (bridge) → ✅ Created and operational
  - `class-net` (external) → Configured for future plug-a-thon
- Port mappings:
  - API: `8000:8000` → ✅ Accessible from host
  - AI: `9000` (internal only)
  - DB: `5432` (internal only)
- Inter-service communication:
  - API ↔ DB via `db:5432` → ✅ Works
  - API ↔ AI via `ai-service:9000` → ✅ Works
- Status: ✅ All networks and ports operational

### 6. Image Tags
- API image: Built from `Dockerfile`
- Base images:
  - `postgres:15-alpine` (DB)
  - `python:3.11-slim` (AI service)
- Version: `0.5.0`
- Tag format ready: `v0.1.0-team-iot`
- Status: ✅ Images built successfully

## Ghi chú thêm

### Thành công
- ✅ Tất cả 3 containers khởi động và healthy
- ✅ Healthchecks pass cho cả 3 services
- ✅ API có thể kết nối với DB và AI service thông qua network nội bộ
- ✅ Authentication với Bearer token hoạt động đúng
- ✅ Docker Compose dependencies (`depends_on` with conditions) hoạt động tốt
- ✅ Non-root user (appuser) được sử dụng trong API container
- ✅ Volume `db-data` giữ dữ liệu persistent

### Vấn đề gặp phải
- Không có vấn đề nghiêm trọng
- Tất cả services hoạt động ổn định

### Cải tiến có thể thực hiện
- Thêm Postman collection với test cases đầy đủ
- Thêm Newman automated tests
- Implement connection pooling cho database
- Thêm logging aggregation (ELK stack)
- Implement actual AI model (thay vì mock service)

### Sẵn sàng cho Plug-a-thon
- ✅ Stack có thể tham gia `class-net` network
- ✅ Services expose đúng ports theo contract
- ✅ Healthchecks cho phép orchestration tự động
- ✅ Environment variables có thể override dễ dàng
- ✅ Documentation đầy đủ trong `RUN_COMPOSE.md`

## Kết luận

Stack Docker Compose cho Lab 05 đã HOÀN THÀNH và sẵn sàng để:
- ✅ Chạy local development
- ✅ Integration testing
- ✅ Plug-a-thon deployment
- ✅ Production readiness (với một số cải tiến về security và monitoring)

**Ngày hoàn thành:** 2026-06-09  
**Người thực hiện:** Team IoT  
**Trạng thái:** READY FOR SUBMISSION
