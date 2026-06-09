# Lab 05 - Test Reports & Evidence

## Newman Test Results

### Summary
- **Total Tests:** 19 assertions
- **Passed:** 19
- **Failed:** 0
- **Success Rate:** 100%

### Files
- `newman-lab05-compose.json` - Full test results in JSON format

### Test Coverage

1. **Health Check** ✅
   - Status code 200
   - Response structure validation
   - Service health verification

2. **Create Reading - Valid** ✅
   - Status code 201
   - Response structure validation
   - Reading acceptance verification

3. **Create Reading - High Temperature** ✅
   - Status code 201
   - Warning header presence

4. **Create Reading - Unauthorized** ✅
   - Status code 401
   - Problem+JSON content type
   - Error details validation

5. **Get Latest Readings** ✅
   - Status code 200
   - Items array validation
   - Item structure verification

6. **Get Reading by ID** ✅
   - Status code 200
   - Reading details validation

7. **Get Reading by ID - Not Found** ✅
   - Status code 404
   - Problem+JSON content type
   - Error details validation

## Docker Compose Stack

### Services Status
All services are healthy and running:

| Service | Container | Status | Port |
|---------|-----------|--------|------|
| API | fit4110-api-lab05 | ✅ Healthy | 8000 |
| Database | fit4110-db-lab05 | ✅ Healthy | 5432 (internal) |
| AI Service | fit4110-ai-lab05 | ✅ Healthy | 9000 (internal) |

### Evidence Files
- `docker-compose-logs.txt` - Container logs from all services
- `db-health-check.txt` - PostgreSQL readiness check
- `api-health-check.json` - API health endpoint response

## Lab Completion Status

✅ All requirements met:
- Docker Compose với 3 services
- Health checks configured
- Newman tests pass 100%
- Non-root user trong containers
- Environment variables configured
- Documentation complete
- Evidence collected
