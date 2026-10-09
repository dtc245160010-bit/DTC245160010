# DTC245160010 - Website Quảng bá Sản phẩm

## 1. Giới thiệu

Dự án triển khai website quảng bá sản phẩm sử dụng WordPress và MySQL trên Docker. Hệ thống được quản lý bằng Docker Compose, kết hợp Nginx Reverse Proxy, Prometheus, Grafana, Loki và Promtail để giám sát và thu thập nhật ký hoạt động.

## 2. Công nghệ sử dụng

- WordPress: xây dựng website quảng bá sản phẩm.
- MySQL 8.0: lưu trữ dữ liệu website.
- phpMyAdmin: quản trị cơ sở dữ liệu.
- Docker và Docker Compose: triển khai, quản lý container.
- Nginx: Reverse Proxy và cấu hình Security Headers.
- Prometheus: thu thập metrics.
- Grafana: trực quan hóa dữ liệu giám sát.
- Node Exporter, cAdvisor: giám sát hệ thống và container.
- MySQL Exporter, Nginx Exporter: cung cấp metrics cho MySQL và Nginx.
- Loki và Promtail: tập trung hóa và truy vấn log.

## 3. Kiến trúc hệ thống

WordPress sử dụng MySQL để lưu trữ dữ liệu. Người dùng truy cập website thông qua Nginx Reverse Proxy. phpMyAdmin hỗ trợ quản trị cơ sở dữ liệu. Prometheus thu thập metrics từ các exporter và cAdvisor; Grafana hiển thị dashboard. Promtail thu thập log container và gửi đến Loki để truy vấn bằng LogQL.

## 4. Cấu trúc thư mục

```text
de3-wordpress/
├── docker-compose.yml
├── nginx/
│   └── default.conf
├── monitoring/
│   ├── docker-compose.yml
│   └── prometheus.yml
├── logging/
│   ├── docker-compose.yml
│   └── promtail-config.yml
├── .gitignore
└── README.md
```

## 5. Yêu cầu môi trường

- Ubuntu Linux.
- Docker.
- Docker Compose.
- Trình duyệt web để truy cập website và các giao diện quản trị.

Kiểm tra phiên bản công cụ:

```bash
docker --version
docker-compose --version
```

## 6. Triển khai và khởi động

Di chuyển vào thư mục dự án:

```bash
cd ~/de3-wordpress
```

Khởi động các dịch vụ website:

```bash
docker-compose up -d
```

Kiểm tra trạng thái container:

```bash
docker ps
```

Các dịch vụ monitoring và logging có thể được quản lý bằng những file Compose riêng trong thư mục `monitoring/` và `logging/`. Chạy chúng theo cấu hình hiện có của môi trường triển khai.

## 7. Địa chỉ truy cập

Thay `192.168.59.130` bằng địa chỉ IP Ubuntu VM nếu IP thay đổi.

- Website: http://192.168.59.130:8080/
- Trang quản trị WordPress: http://192.168.59.130:8080/wp-admin
- phpMyAdmin: http://192.168.59.130:8081/
- Prometheus: http://192.168.59.130:9090/
- Grafana: http://192.168.59.130:3000/

Địa chỉ chỉ truy cập được khi máy ảo đang chạy, các container tương ứng hoạt động và cổng dịch vụ được phép truy cập.

## 8. Kiểm tra hệ thống

Kiểm tra container:

```bash
docker ps
```

Kiểm tra phản hồi HTTP của website:

```bash
curl -I http://192.168.59.130:8080/
```

Kiểm tra cấu hình Prometheus nếu công cụ `promtool` có sẵn:

```bash
promtool check config monitoring/prometheus.yml
```

Trong Prometheus, kiểm tra trang Targets để xác nhận các mục tiêu giám sát. Trong Grafana, kiểm tra dashboard monitoring và Explore với datasource Loki để truy vấn log bằng LogQL.

## 9. Bảo mật

- Sử dụng mật khẩu mạnh cho các tài khoản dịch vụ.
- Không đưa file `.env`, `.my.cnf`, khóa riêng hoặc thông tin bí mật lên GitHub.
- Sử dụng `.gitignore` để loại trừ các file chứa thông tin nhạy cảm.
- Cấu hình Nginx Security Headers.
- Tách mạng backend và giới hạn quyền tài khoản giám sát cơ sở dữ liệu.
- Chỉ mở các cổng cần thiết cho việc truy cập và quản trị.

## 10. Git và quản lý phiên bản

Kiểm tra lịch sử commit:

```bash
git log --oneline --all
```

Kiểm tra trạng thái repository:

```bash
git status
```

Đẩy thay đổi lên GitHub sau khi kiểm tra:

```bash
git add README.md
git commit -m "docs: complete project README"
git push origin main
```

Chỉ chạy ba lệnh cuối khi README đã được lưu và kiểm tra kỹ.

## 11. Repository

GitHub: https://github.com/dtc245160010-bit/DTC245160010

