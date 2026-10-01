# DTC245160010 - Website Quảng bá Sản phẩm

## 1. Giới thiệu

Đây là project thực hành triển khai Website Quảng bá Sản phẩm sử dụng WordPress và MySQL trên Docker.

Hệ thống được triển khai và quản lý bằng Docker Compose, kết hợp Nginx Reverse Proxy, Prometheus, Grafana, Loki và Promtail.

## 2. Kiến trúc hệ thống

Các thành phần chính:

- WordPress: Website quảng bá sản phẩm
- MySQL 8.0: Cơ sở dữ liệu cho WordPress
- phpMyAdmin: Quản trị cơ sở dữ liệu
- Nginx: Reverse Proxy và Security Headers
- Prometheus: Thu thập metrics
- Grafana: Hiển thị dashboard monitoring
- cAdvisor: Giám sát container
- MySQL Exporter: Giám sát MySQL
- Nginx Exporter: Giám sát Nginx
- Node Exporter: Giám sát hệ thống
- Loki: Lưu trữ log tập trung
- Promtail: Thu thập log Docker

## 3. Cấu trúc thư mục

```text
DTC245160010/
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
