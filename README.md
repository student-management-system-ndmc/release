# Student Management System

Hệ thống quản lý học sinh được xây dựng với NestJS backend và frontend, sử dụng PostgreSQL database.

## 🚀 Cách chạy dự án với Docker

### Yêu cầu hệ thống

- Docker và Docker Compose
- Node.js 22.14.0+ (để development)

### Chạy dự án

1. **Clone repository và di chuyển đến thư mục release:**

- backend

```bash
git clone https://github.com/student-management-system-ndmc/backend.git
```

- frontend

```bash
git clone https://github.com/student-management-system-ndmc/frontend.git
```

- release

```bash
git clone https://github.com/student-management-system-ndmc/release.git
```

```bash
cd release
```

2. **Tạo file .env từ template:**

```bash
cp .env.example .env  # sửa lại .env
```

#### Hoặc tạo file .env mới với nội dung:

```bash
# Tạo file .env với các biến môi trường cần thiết
# Frontend
FRONTEND_PORT=3001
VITE_API_URL=http://localhost:3000

# Backend
PORT=3000

DATABASE_HOST=postgres
DATABASE_PORT=5432
DATABASE_USERNAME=root
DATABASE_PASSWORD=1234
DATABASE_NAME=teen_up
```

3. **Build và chạy các containers:**

```bash
# Hoặc chạy trong background
docker compose up -d --build
```

### Lưu ý: Mỗi lần down và up lại container thì nó sẽ tự động CD và sẽ chạy tự động migrations

4. **Truy cập ứng dụng:**

- Backend API: `http://localhost:3000`
- API Documentation (Swagger): `http://localhost:3000/api/docs`
- Frontend: `http://localhost:3001`

## 🗄️ Database Schema

### Cấu trúc database

Hệ thống sử dụng PostgreSQL với TypeORM và bao gồm 5 bảng chính:

#### 1. **parents**

```sql
- id (Primary Key)
- name (string) - Tên phụ huynh
- phone (string) - Số điện thoại
- email (string) - Email liên hệ
```

#### 2. **students**

```sql
- id (Primary Key)
- name (string) - Tên học sinh
- dob (date) - Ngày sinh
- gender (string) - Giới tính
- current_grade (string) - Lớp hiện tại
- parent_id (Foreign Key) - ID phụ huynh
```

#### 3. **classes**

```sql
- id (Primary Key)
- name (string) - Tên lớp học
- subject (string) - Môn học
- day_of_week (string) - Thứ trong tuần
- time_slot (string) - Khung giờ học
- teacher_name (string) - Tên giáo viên
- max_students (number) - Số học sinh tối đa
```

#### 4. **subscriptions**

```sql
- id (Primary Key)
- student_id (Foreign Key) - ID học sinh
- package_name (string) - Tên gói học
- start_date (date) - Ngày bắt đầu
- end_date (date) - Ngày kết thúc
- total_sessions (number) - Tổng số buổi học
- used_sessions (number) - Số buổi đã sử dụng
```

#### 5. **class_registrations**

```sql
- id (Primary Key)
- class_id (Foreign Key) - ID lớp học
- student_id (Foreign Key) - ID học sinh
```

### Mối quan hệ

- Một phụ huynh có thể có nhiều học sinh (1:N)
- Một học sinh có thể đăng ký nhiều lớp học (N:M qua class_registrations)
- Một học sinh có thể có nhiều gói subscription (1:N)

## 📡 API Endpoints

### Base URL: `http://localhost:3000`

### 👥 **Parents Management**

```http
GET    /api/parents          # Lấy danh sách tất cả phụ huynh
GET    /api/parents/:id      # Lấy thông tin phụ huynh theo ID
POST   /api/parents          # Tạo phụ huynh mới
```

**Ví dụ tạo phụ huynh:**

```bash
curl -X POST http://localhost:3000/api/parents \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Nguyễn Văn An",
    "phone": "0901234567",
    "email": "nva@email.com"
  }'
```

### 🎓 **Students Management**

```http
GET    /api/students         # Lấy danh sách tất cả học sinh
GET    /api/students/:id     # Lấy thông tin học sinh theo ID
POST   /api/students         # Tạo học sinh mới
```

**Ví dụ tạo học sinh:**

```bash
curl -X POST http://localhost:3000/api/students \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Nguyễn Minh Khoa",
    "dob": "2010-05-15",
    "gender": "Male",
    "current_grade": "Lớp 8",
    "parent_id": 1
  }'
```

### 📚 **Classes Management**

```http
GET    /api/classes          # Lấy danh sách tất cả lớp học
GET    /api/classes?day=Monday # Lấy lớp học theo thứ
GET    /api/classes/:id      # Lấy thông tin lớp học theo ID
POST   /api/classes          # Tạo lớp học mới
POST   /api/classes/:id/register # Đăng ký học sinh vào lớp
```

**Ví dụ tạo lớp học:**

```bash
curl -X POST http://localhost:3000/api/classes \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Toán 8A",
    "subject": "Toán học",
    "day_of_week": "Monday",
    "time_slot": "14:00-16:00",
    "teacher_name": "Cô Nguyễn Thị Hoa",
    "max_students": 15
  }'
```

**Ví dụ đăng ký học sinh vào lớp:**

```bash
curl -X POST http://localhost:3000/api/classes/1/register \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": 1
  }'
```

### 📦 **Subscriptions Management**

```http
GET    /api/subscriptions              # Lấy danh sách tất cả gói subscription
GET    /api/subscriptions/:id          # Lấy thông tin subscription theo ID
GET    /api/subscriptions/student/:studentId # Lấy subscription theo học sinh
POST   /api/subscriptions              # Tạo subscription mới
PATCH  /api/subscriptions/:id/use      # Sử dụng 1 buổi học
PATCH  /api/subscriptions/:id/extend   # Gia hạn thêm buổi học
```

**Ví dụ tạo subscription:**

```bash
curl -X POST http://localhost:3000/api/subscriptions \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": 1,
    "package_name": "Gói 20 buổi",
    "start_date": "2024-01-01",
    "end_date": "2024-06-30",
    "total_sessions": 20
  }'
```

**Ví dụ sử dụng buổi học:**

```bash
curl -X PATCH http://localhost:3000/api/subscriptions/1/use
```

**Ví dụ gia hạn subscription:**

```bash
curl -X PATCH http://localhost:3000/api/subscriptions/1/extend \
  -H "Content-Type: application/json" \
  -d '{
    "additional_sessions": 5
  }'
```

## 🛠️ Development

### Chạy backend riêng lẻ

```bash
cd backend

# Install dependencies
pnpm install

# Chạy development mode
pnpm run start:dev

# Build production
pnpm run build
pnpm run start:prod
```

### Database Migration

```bash
cd backend

# Generate migration
pnpm run migration:generate MigrationName

# Run migrations
pnpm run migration:run
```

### API Documentation

Truy cập Swagger UI tại: `http://localhost:3000/api/docs` để xem chi tiết tất cả endpoints và test API trực tiếp.

## 🐛 Troubleshooting

### Lỗi database connection

- Kiểm tra các biến môi trường trong file `.env`
- Đảm bảo PostgreSQL container đã chạy
- Kiểm tra port database không bị conflict

### Lỗi port đã được sử dụng

```bash
# Thay đổi port trong file .env
PORT=3001
FRONTEND_PORT=3002
```

### Reset database

```bash
docker-compose down -v  # Xóa volumes
docker-compose up --build
```
