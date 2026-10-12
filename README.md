# Student Management RESTful API

Hệ thống RESTful API quản lý sinh viên được xây dựng bằng **Spring Boot 3.x**, kết nối **MySQL**, sử dụng build tool **Gradle**, và được đóng gói toàn diện bằng **Docker** & **Docker Compose**.

---

## 1. Công nghệ & Kiến trúc
- **Ngôn ngữ**: Java 21
- **Framework**: Spring Boot 3.3.5
- **Build tool**: Gradle 8.x / 9.x
- **Database**: MySQL 8.4
- **ORM / Data Access**: Spring Data JPA & Hibernate
- **Kiến trúc Layered Architecture**:
  - `entity/`: Thực thể JPA (`Student`, `Gender`)
  - `dto/`: Request/Response objects tách biệt hoàn toàn (`StudentRequest`, `StudentResponse`, `ApiResponse<T>`, `PageResponse<T>`)
  - `repository/`: Kế thừa `JpaRepository` & `JpaSpecificationExecutor`
  - `specification/`: `StudentSpecification` hỗ trợ dynamic filtering theo từ khóa & khoảng GPA
  - `service/`: Gồm `StudentService` interface và `StudentServiceImpl`
  - `controller/`: REST API endpoints với `@Valid`
  - `exception/`: Custom exceptions (`ResourceNotFoundException`, `DuplicateResourceException`) & `@RestControllerAdvice` (`GlobalExceptionHandler`)

---

## 2. Cấu hình biến môi trường (`.env`)

File `.env` nằm tại thư mục gốc dự án:
```env
APP_PORT=8080
DB_HOST=mysql-db
DB_PORT=3306
DB_NAME=student_db
DB_USER=student_user
DB_PASSWORD=student_password
MYSQL_ROOT_PASSWORD=root_secure_password
```

File `src/main/resources/application.yml` tự động đọc các biến này kèm giá trị fallback mặc định.

---

## 3. Hướng dẫn khởi chạy

### Cách 1: Chạy toàn bộ hệ thống bằng Docker Compose (Khuyên dùng)
Chỉ với 1 lệnh duy nhất, Docker Compose sẽ khởi tạo MySQL và build/chạy ứng dụng Spring Boot:
```bash
docker compose up -d --build
```

Kiểm tra trạng thái containers:
```bash
docker compose ps
```

Xem log ứng dụng:
```bash
docker compose logs -f student-app
```

Dừng hệ thống:
```bash
docker compose down
```

---

### Cách 2: Chạy trực tiếp qua Gradle (Local)
Yêu cầu đã có MySQL đang chạy tại cổng `3306` (có thể chỉ chạy riêng MySQL container bằng `docker compose up -d mysql-db`):
```bash
# Build và chạy kiểm thử
./gradlew test

# Chạy ứng dụng Spring Boot
./gradlew bootRun
```

---

## 4. Chi tiết RESTful API Endpoints

Chuẩn hóa Response wrapper:
```json
{
  "status": 200,
  "message": "Thông báo kết quả",
  "data": { ... },
  "timestamp": "2026-10-12T08:50:00"
}
```

### 1. Tạo mới sinh viên
- **Method**: `POST`
- **URL**: `http://localhost:8080/api/v1/students`
- **Headers**: `Content-Type: application/json`
- **Request Body**:
```json
{
  "studentCode": "SV001",
  "fullName": "Nguyen Van A",
  "dateOfBirth": "2003-05-15",
  "gender": "MALE",
  "phoneNumber": "0912345678",
  "gpa": 3.65
}
```
- **Response**: `201 Created`

---

### 2. Xem chi tiết sinh viên theo ID
- **Method**: `GET`
- **URL**: `http://localhost:8080/api/v1/students/1`
- **Response**: `200 OK` (hoặc `404 Not Found` nếu không tồn tại)

---

### 3. Cập nhật thông tin sinh viên
- **Method**: `PUT`
- **URL**: `http://localhost:8080/api/v1/students/1`
- **Headers**: `Content-Type: application/json`
- **Request Body**:
```json
{
  "studentCode": "SV001",
  "fullName": "Nguyen Van A (Cap nhat)",
  "dateOfBirth": "2003-05-15",
  "gender": "MALE",
  "phoneNumber": "0987654321",
  "gpa": 3.85
}
```
- **Response**: `200 OK`

---

### 4. Xóa sinh viên theo ID
- **Method**: `DELETE`
- **URL**: `http://localhost:8080/api/v1/students/1`
- **Response**: `200 OK`

---

### 5. Danh sách sinh viên (Phân trang, Sắp xếp, Tìm kiếm & Lọc)
- **Method**: `GET`
- **URL**: `http://localhost:8080/api/v1/students`
- **Query Parameters**:
  - `keyword`: Tìm kiếm tương đối theo `fullName` hoặc `studentCode` (ví dụ: `?keyword=Nguyen` hoặc `?keyword=SV001`)
  - `minGpa`: Lọc sinh viên có GPA >= `minGpa` (ví dụ: `?minGpa=3.2`)
  - `maxGpa`: Lọc sinh viên có GPA <= `maxGpa` (ví dụ: `?maxGpa=3.8`)
  - `page`: Trang hiện tại (mặc định: `0`)
  - `size`: Số phần tử trên mỗi trang (mặc định: `10`)
  - `sortBy`: Trường sắp xếp (mặc định: `id`, có thể là `fullName`, `gpa`, `studentCode`, ...)
  - `sortDir`: Chiều sắp xếp (`asc` hoặc `desc`, mặc định: `asc`)

**Ví dụ URL đầy đủ**:
```text
http://localhost:8080/api/v1/students?keyword=Nguyen&minGpa=3.0&maxGpa=4.0&page=0&size=10&sortBy=gpa&sortDir=desc
```
