# Hướng dẫn chạy Docker Compose cho Lab 05

Tài liệu này hướng dẫn chi tiết cách chạy stack Docker Compose cho Lab 05 - FIT4110.

## Yêu cầu hệ thống

Trước khi bắt đầu, đảm bảo bạn đã cài đặt:

- **Docker Desktop** 20.10+ hoặc Docker Engine với Docker Compose v2
- **Git** để clone repository
- **curl** hoặc **Postman** để test API (tùy chọn)
- **Node.js 20.x LTS** và **npm** nếu muốn chạy Newman test (tùy chọn)

Kiểm tra phiên bản:

```bash
docker --version
docker compose version
git --version
```

## Bước 1: Clone repository

```bash
git clone <repository-url>
cd lab-05-minhson11
```

## Bước 2: Chuẩn bị file môi trường

Tạo file `.env` từ template `.env.example`:

```bash
# Windows CMD
copy .env.example .env

# Windows PowerShell
Copy-Item .env.example .env

# Linux/macOS
cp .env.example .env
```

Mở file `.env` và kiểm tra các biến môi trường. Giá trị mặc định đã đủ để chạy local:

```env
APP_HOST=0.0.0.0
APP_PORT=8000
AUTH_TOKEN=local-dev-token
SERVICE_NAME=iot-ingestion
SERVICE_VERSION=0.5.0
ENV=local

POSTGRES_USER=lab05
POSTGRES_PASSWORD=lab05pass
POSTGRES_DB=iotdb
```

**Lưu ý:** Không commit file `.env` vào Git. File này chứa cấu hình local và có thể chứa secret.

## Bước 3: Build và chạy Docker Compose stack

Sử dụng Docker Compose để build và khởi động tất cả services:

```bash
docker compose up -d --build
```

Lệnh này sẽ:
- Build Docker image cho API service
- Pull image PostgreSQL 15-alpine
- Pull image Python 3.11-slim cho AI service
- Tạo network `team-internal` và volume `db-data`
- Khởi động 3 containers: `db`, `ai-service`, và `api`

## Bước 4: Kiểm tra trạng thái containers

Xem danh sách containers đang chạy:

```bash
docker compose ps
```

Kết quả mong đợi:

```
NAME                    STATUS          PORTS
fit4110-api-lab05       Up (healthy)    0.0.0.0:8000->8000/tcp
fit4110-db-lab05        Up (healthy)    5432/tcp
fit4110-ai-lab05        Up (healthy)    9000/tcp
```

Theo dõi logs của tất cả services:

```bash
docker compose logs -f
```

Hoặc xem log của từng service riêng lẻ:

```bash
docker compose logs -f api
docker compose logs -f db
docker compose logs -f ai-service
```

## Bước 5: Kiểm tra health của từng service

### API Service (port 8000)

```bash
curl http://localhost:8000/health
```

Kết quả mong đợi:

```json
{
  "status": "ok",
  "service": "iot-ingestion",
  "version": "0.5.0"
}
```

### AI Service (port 9000)

```bash
curl http://localhost:9000/health
```

Kết quả mong đợi:

```json
{
  "status": "ok",
  "service": "ai-service",
  "version": "0.5.0"
}
```

### Database Service

```bash
docker exec -it fit4110-db-lab05 pg_isready -U lab05
```

Kết quả mong đợi:

```
/var/run/postgresql:5432 - accepting connections
```

## Bước 6: Test API với curl

### Test endpoint POST /readings (tạo sensor reading)

```bash
curl -X POST http://localhost:8000/readings \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer local-dev-token" \
  -d '{
    "device_id": "ESP32-LAB-A01",
    "metric": "temperature",
    "value": 31.5,
    "unit": "celsius",
    "timestamp": "2026-05-13T08:30:00+07:00"
  }'
```

Kết quả mong đợi (201 Created):

```json
{
  "reading_id": "R-20260609-0001",
  "device_id": "ESP32-LAB-A01",
  "metric": "temperature",
  "accepted": true,
  "created_at": "2026-06-09T..."
}
```

### Test endpoint GET /readings/latest

```bash
curl http://localhost:8000/readings/latest?limit=5 \
  -H "Authorization: Bearer local-dev-token"
```

### Test AI service prediction

```bash
curl -X POST http://localhost:9000/predict \
  -H "Content-Type: application/json"
```

Kết quả mong đợi:

```json
{
  "objects": ["person", "bicycle"],
  "confidence": [0.98, 0.85]
}
```

## Bước 7: Test với Postman (tùy chọn)

1. Mở Postman Desktop hoặc Postman Web
2. Import collection từ `postman/collections/` (nếu có)
3. Import environment từ `postman/environments/FIT4110_lab05_local.postman_environment.json`
4. Chọn environment `FIT4110_lab05_local`
5. Chạy các request trong collection

## Bước 8: Chạy Newman test (tùy chọn)

Nếu bạn muốn chạy automated test với Newman:

```bash
# Cài đặt dependencies
npm install

# Chạy Newman test (nếu có collection)
npx newman run postman/collections/<collection-name>.json \
  -e postman/environments/FIT4110_lab05_local.postman_environment.json \
  --reporters cli,html \
  --reporter-html-export reports/newman-lab05-compose.html
```

## Bước 9: Dừng stack

Để dừng tất cả containers nhưng giữ lại data:

```bash
docker compose stop
```

Để dừng và xóa containers (giữ volumes):

```bash
docker compose down
```

Để dừng, xóa containers VÀ xóa volumes (mất data):

```bash
docker compose down -v
```

## Bước 10: Khởi động lại stack

Để khởi động lại stack đã được build:

```bash
docker compose up -d
```

Để rebuild lại images trước khi chạy:

```bash
docker compose up -d --build
```

## Sử dụng Makefile (nếu có)

Repository cung cấp Makefile với các lệnh tiện lợi:

```bash
# Build và chạy compose stack
make compose-up

# Dừng và xóa stack
make compose-down

# Theo dõi logs
make logs

# Chạy Newman test (nếu có)
make test-compose
```

## Xử lý sự cố (Troubleshooting)

### Container không healthy

```bash
# Xem logs của container bị lỗi
docker compose logs <service-name>

# Restart container
docker compose restart <service-name>
```

### Port đã được sử dụng

Nếu port 8000 hoặc 9000 đã được sử dụng, sửa trong file `.env`:

```env
APP_PORT=8001  # Thay đổi port
```

Sau đó chạy lại:

```bash
docker compose down
docker compose up -d --build
```

### Database connection error

Kiểm tra DB đã sẵn sàng chưa:

```bash
docker compose logs db
docker exec -it fit4110-db-lab05 pg_isready -U lab05
```

### Xóa tất cả và chạy lại từ đầu

```bash
docker compose down -v
docker compose up -d --build
```

## Network và Communication

Stack sử dụng 2 networks:

1. **team-internal** (bridge): Cho giao tiếp nội bộ giữa `api`, `db`, và `ai-service`
2. **class-net** (external): Cho kết nối với các service khác trong lớp (sẽ được sử dụng trong plug-a-thon)

Services có thể gọi nhau bằng tên container:
- API gọi DB: `postgresql://lab05:lab05pass@db:5432/iotdb`
- API gọi AI: `http://ai-service:9000/predict`

## Kiểm tra readiness

Xem file `checklists/readiness-checklist.md` để kiểm tra tất cả các mục readiness trước khi nộp bài.

## Hỗ trợ

Nếu gặp vấn đề, kiểm tra:
1. Docker Desktop đang chạy
2. File `.env` đã được tạo từ `.env.example`
3. Không có service nào khác đang chiếm dụng port 8000, 9000 hoặc 5432
4. Log của containers: `docker compose logs -f`

## Tài liệu tham khảo

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
