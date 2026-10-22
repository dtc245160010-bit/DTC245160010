# ĐỀ 3 – WEBSITE QUẢNG BÁ SẢN PHẨM SỬ DỤNG WORDPRESS
#### Học phần:	Triển khai và Quản trị Hệ thống Phần mềm
#### Sinh viên:	Vũ Thị Hà
#### Mã sinh viên:	DTC245160010
#### GitHub:	DTC245160010-bit (https://github.com/dtc245160010-bit/DTC245160010)
---
# 1. Giới thiệu
Dự án triển khai website quảng bá sản phẩm bằng WordPress trên môi trường Docker Compose. Hệ thống có các thành phần phục vụ website, quản trị dữ liệu, giám sát và tập trung log.
Mục tiêu
Triển khai website WordPress và cơ sở dữ liệu MySQL.
Sử dụng phpMyAdmin để quản trị cơ sở dữ liệu.
Sử dụng Nginx làm reverse proxy và áp dụng cấu hình bảo mật HTTP theo cấu hình thực tế.
Giám sát hệ thống bằng Prometheus và Grafana.
Thu thập, tập trung và truy vấn log bằng Promtail, Loki và LogQL.
Trình bày các biện pháp hardening đã thực sự cấu hình và kiểm chứng.
# 2. Thành phần hệ thống
Thành phần	Vai trò	Cách kiểm tra
WordPress	Website quảng bá sản phẩm	Mở trang web và kiểm tra nội dung
MySQL	Lưu dữ liệu WordPress	Kiểm tra container và database
phpMyAdmin	Quản trị database qua giao diện web	Đăng nhập và xem các bảng
Nginx	Reverse proxy và/hoặc HTTP security headers	Kiểm tra cấu hình và phản hồi HTTP
Prometheus	Thu thập metrics	Kiểm tra mục Status → Targets
Grafana	Dashboard giám sát và giao diện truy vấn log	Mở dashboard và Explore
Loki	Lưu trữ log tập trung	Thực hiện truy vấn LogQL
Promtail	Thu thập và gửi log đến Loki	Kiểm tra trạng thái dịch vụ và log
Exporters	Cung cấp metrics cho Prometheus, nếu có	Kiểm tra Prometheus Targets
> **Lưu ý:** Tên dịch vụ, cổng và các thành phần phải khớp với file Compose thực tế trong repository. Chỉ mô tả những thành phần đã được cấu hình.
# 3. Kiến trúc hệ thống
## 3.1. Luồng website
```text
Trình duyệt
    |
    v
  Nginx
    |
    v
 WordPress
    |
    v
  MySQL

phpMyAdmin ---> MySQL
```
Nginx tiếp nhận yêu cầu web và chuyển tiếp đến WordPress theo cấu hình. WordPress đọc/ghi dữ liệu trong MySQL. phpMyAdmin cung cấp giao diện quản trị cơ sở dữ liệu.
## 3.2. Luồng giám sát
```text
Exporters / targets
        |
        v
   Prometheus
        |
        v
     Grafana
```
Prometheus thu thập metrics từ các target đã cấu hình. Grafana truy vấn Prometheus và hiển thị số liệu trên dashboard. Cần kiểm tra trạng thái target và dữ liệu thực tế trước khi demo.
## 3.3. Luồng logging
```text
Docker/container logs
        |
        v
     Promtail
        |
        v
       Loki
        |
        v
 Grafana Explore
```
Promtail thu thập log theo cấu hình và gửi đến Loki. Grafana Explore sử dụng LogQL để lọc log theo nhãn và nội dung.
# 4. Yêu cầu môi trường
Linux/Ubuntu VM hoặc môi trường có Docker Engine.
Docker Compose plugin.
Git.
Trình duyệt web.
Kiểm tra công cụ:
```bash
docker --version
docker compose version
git --version
```
# 5. Hướng dẫn cài đặt và chạy
## 5.1. Clone repository (chỉ cần khi chưa có mã nguồn)
```bash
git clone https://github.com/dtc245160010-bit/DTC245160010.git
cd DTC245160010
```
Nếu mã nguồn đã có trên máy, chuyển đến thư mục dự án hiện tại thay vì clone lại.
## 5.2. Kiểm tra file dự án
```bash
ls -la
find . -maxdepth 3 -type f | sort
```
Xác định file Compose thực tế, thường là `docker-compose.yml` hoặc `compose.yaml`.
## 5.3. Cấu hình biến môi trường
Nếu repository có file `.env.example`, tạo file `.env`:
```bash
cp .env.example .env
nano .env
```
Điền các biến theo hướng dẫn và theo những biến được tham chiếu trong file Compose. Nếu không có `.env.example`, kiểm tra cấu hình thực tế trước khi tạo biến.
Bảo mật: Không commit mật khẩu thật, token hoặc file `.env` có thông tin bí mật lên GitHub. Chỉ đưa file mẫu không chứa thông tin nhạy cảm vào repository.
## 5.4. Kiểm tra cấu hình và khởi chạy
Chạy trong thư mục chứa file Compose:
```bash
docker compose config
docker compose up -d
docker compose ps
```
Xem log:
```bash
docker compose logs --tail 50
```
Xem log một dịch vụ cụ thể:
```bash
docker compose logs --tail 50 <ten-dich-vu>
```
Thay `<ten-dich-vu>` bằng tên dịch vụ có trong file Compose. Nếu dự án dùng nhiều file Compose, hãy chỉ định đúng các file đang tồn tại bằng tùy chọn `-f`.
## 5.5. Dừng hệ thống
```bash
docker compose down
```
> Không thêm `-v` nếu chưa sao lưu và chưa hiểu tác động, vì có thể xóa volume chứa dữ liệu WordPress/MySQL.
# 6. Địa chỉ truy cập
Lấy IP hiện tại của Ubuntu:
```bash
hostname -I
```
Các cổng dưới đây là giá trị mẫu từng dùng trong môi trường dự án. Xác minh lại bằng `docker compose ps` và phần `ports` trước khi truy cập.
Dịch vụ	Địa chỉ mẫu
WordPress qua Nginx	`http://<IP-UBUNTU>:8080`
WordPress trực tiếp (nếu đã publish cổng)	`http://<IP-UBUNTU>:8083`
phpMyAdmin	`http://<IP-UBUNTU>:8081`
Grafana	`http://<IP-UBUNTU>:3000`
Prometheus	`http://<IP-UBUNTU>:9090`
Thay `<IP-UBUNTU>` bằng IP thực tế của máy ảo. Chỉ sử dụng địa chỉ ứng với cổng đã được cấu hình.
# 7. Kiểm tra website và database
## 7.1. WordPress
Mở địa chỉ WordPress qua Nginx.
Kiểm tra trang chủ và nội dung website.
Mở một số trang hoặc sản phẩm đã tạo.
Chụp ảnh minh chứng có thanh địa chỉ trình duyệt.
## 7.2. MySQL và phpMyAdmin
Mở phpMyAdmin.
Đăng nhập bằng tài khoản được cấu hình trong dự án.
Xác nhận database WordPress tồn tại.
Mở danh sách bảng để chứng minh dữ liệu đã được tạo.
Không chụp hoặc công khai mật khẩu.
# 8. Nginx reverse proxy và bảo mật HTTP
Kiểm tra file cấu hình Nginx thực tế và trình bày:
Cổng Nginx đang lắng nghe.
Địa chỉ dịch vụ WordPress mà Nginx chuyển tiếp đến.
Các HTTP security headers đã cấu hình, nếu có.
HTTPS và chứng chỉ self-signed, nếu đã triển khai.
Kiểm tra phản hồi HTTP:
```bash
curl -I http://<IP-UBUNTU>:8080
```
Thay địa chỉ bằng URL thực tế. Chỉ ghi HTTPS hoặc security headers là hoàn thành khi đã kiểm tra được cấu hình và kết quả. Nếu hệ thống chỉ dùng HTTP, hãy mô tả đúng trạng thái thực tế.
# 9. Prometheus và Grafana – giám sát
## 9.1. Prometheus Targets
Mở giao diện Prometheus.
Chọn Status → Targets.
Kiểm tra các target đã cấu hình.
Chụp ảnh thể hiện tên target và trạng thái `UP`/`DOWN`.
Trạng thái `UP` cho biết target có thể được scrape tại thời điểm kiểm tra, không có nghĩa toàn bộ ứng dụng chắc chắn không có lỗi.
## 9.2. Grafana dashboard
Đăng nhập Grafana.
Mở dashboard giám sát đã tạo.
Chọn khoảng thời gian có dữ liệu, ví dụ Last 24 hours.
Kiểm tra các panel có dữ liệu thực tế cho:
CPU, bộ nhớ và network của container.
Nginx/web server.
MySQL/database.
Nếu panel trống, kiểm tra nguồn dữ liệu, truy vấn, thời gian và trạng thái target.
## 9.3. Ảnh minh chứng 
Prometheus Targets.  

Dashboard container monitoring.  
 
Panel Nginx.  
 
Panel MySQL.
# 10. Loki, Promtail và LogQL
Mở Grafana → Explore, chọn nguồn dữ liệu Loki và chọn khoảng thời gian phù hợp. Dùng Label browser để xác nhận tên nhãn và giá trị nhãn thực tế.
Các truy vấn sau là ví dụ dành cho môi trường có nhãn `container` và tên container tương ứng. Nếu nhãn khác, thay bằng nhãn thực tế.
Truy vấn 1 – Log Nginx
```logql
{container="de3_nginx"}
```
Lọc log có nhãn container là `de3_nginx`.
Truy vấn 2 – Log Nginx chứa mã 499
```logql
{container="de3_nginx"} |= "499"
```
Lọc các dòng log Nginx có chứa chuỗi `499`. Nếu không có kết quả, hãy kiểm tra khoảng thời gian và nhãn; không tạo log giả để làm minh chứng.
Truy vấn 3 – Log WordPress
```logql
{container="de3_wordpress"}
```
Lọc log có nhãn container là `de3_wordpress`.
Cách chứng minh logging hoạt động
Chọn khoảng thời gian có log.
Chạy 2–3 truy vấn và chụp ảnh kết quả.
Nếu không có log, xác minh nhãn trong Label browser và kiểm tra Promtail/Loki.
Giải thích luồng Docker logs → Promtail → Loki → Grafana Explore.
## 11. Hardening – biện pháp bảo mật
Mật khẩu mạnh	Cấu hình đã che thông tin bí mật; không công khai mật khẩu  

Không commit `.env` hoặc secret	`.gitignore` và file mẫu không chứa bí mật  

Phân tách mạng Docker	Phần `networks` trong Compose và giải thích luồng giao tiếp  

Hạn chế publish cổng	Kiểm tra `ports`/`expose` trong Compose   

Giới hạn quyền tài khoản database	Cấu hình tài khoản và quyền MySQL thực tế  

Security headers hoặc HTTPS	Cấu hình Nginx và kết quả `curl -I`/truy cập HTTPS  

Chạy non-root (nếu hỗ trợ và đã cấu hình)	`user`, Dockerfile hoặc kết quả kiểm tra user trong container  

Không tuyên bố biện pháp nào đã hoàn thành nếu chưa cấu hình và kiểm chứng. Trong báo cáo, ghi rõ biện pháp, cách triển khai, cách kiểm tra và ảnh minh chứng.
# 12. GitHub và lịch sử commit
#### commit theo giai đoạn:  

feat: deploy WordPress with MySQL and Nginx   

feat: add Prometheus and Grafana monitoring  

feat: add Loki and Promtail logging  

docs: improve README and deployment evidence  

#### Kiểm tra trạng thái Git:
```bash
git status
git log --oneline --decorate -5
git diff --check
git diff -- README.md
```
#### Sau khi chỉnh sửa README:
```bash
git add README.md
git commit -m "docs: improve README formatting and demo guide"
git push origin main
```
Nếu nhánh chính không phải `main`, sử dụng tên nhánh thực tế. Nếu Git báo `nothing to commit`, không cần tạo commit mới.
# 13. Danh sách ảnh minh chứng
#### Nên chuẩn bị các ảnh sau cho báo cáo và demo:
 01_github_repository.png – Repository và cấu trúc mã nguồn.  
 
 02_git_commits.png – Lịch sử commit.  

 03_wordpress_homepage.png – Trang chủ WordPress.  
 
 04_phpmyadmin_database.png – Database và bảng WordPress.  
 
 05_nginx_config_or_headers.png – Cấu hình Nginx hoặc HTTP headers.  
 
 06_prometheus_targets.png – Trạng thái Prometheus Targets.  
 
 07_grafana_container_monitoring.png – Metrics container.  
 
 08_grafana_nginx_metrics.png – Metrics Nginx.  
 
 09_grafana_mysql_metrics.png – Metrics MySQL.  
 
 10_loki_nginx_logs.png – Log Nginx trong Grafana Explore.  
 
 11_logql_query.png – Một truy vấn LogQL có kết quả thực tế.  
 
 12_hardening_evidence.png – Bằng chứng hardening.  
 
Nếu truy vấn mã 499 không có kết quả trong khoảng thời gian kiểm tra, hãy dùng truy vấn log khác có dữ liệu thực tế và ghi chú rõ trong báo cáo.
# 14. Kịch bản demo
Mở GitHub, giới thiệu repository, README và lịch sử commit.  

Mở website WordPress.  

Mở phpMyAdmin và chỉ ra database/bảng WordPress.  

Trình bày Nginx reverse proxy và cấu hình bảo mật đã triển khai.  

Mở Prometheus Targets.  

Mở Grafana dashboard, giải thích metrics container, Nginx và MySQL.  

Mở Grafana Explore và chạy 2–3 truy vấn LogQL.  

Trình bày các biện pháp hardening cùng ảnh minh chứng.  

# 15. Kết luận
Dự án kết hợp triển khai website WordPress với công cụ quản trị dữ liệu, giám sát và logging trên Docker Compose. Kết quả cần được xác nhận bằng việc khởi chạy thành công, kiểm tra các dịch vụ, truy vấn metrics/log thực tế và lưu ảnh minh chứng. Tên dịch vụ, cổng, file cấu hình và biện pháp bảo mật trong tài liệu cần khớp với cấu hình thực tế của repository.
