ĐỀ 3 – WEBSITE QUẢNG BÁ SẢN PHẨM SỬ DỤNG WORDPRESS
Học phần: Triển khai và Quản trị Hệ thống Phần mềm
Sinh viên: Vũ Thị Hà
Mã sinh viên: DTC245160010
Repository GitHub: https://github.com/dtc245160010-bit/DTC245160010.git

1. Giới thiệu
Dự án triển khai website quảng bá sản phẩm sử dụng WordPress trên môi trường Docker Compose. Hệ thống tích hợp cơ sở dữ liệu MySQL, công cụ quản trị cơ sở dữ liệu phpMyAdmin, Nginx làm reverse proxy, Prometheus và Grafana để giám sát, cùng Loki và Promtail để tập trung log.

Mục tiêu là triển khai các thành phần bằng Docker Compose, hướng dẫn cách khởi chạy và kiểm tra hệ thống, đồng thời trình bày các biện pháp hardening đã cấu hình và kiểm chứng.
2. Thành phần hệ thống
Thành phần	Vai trò	Cách kiểm tra
WordPress	Website quảng bá sản phẩm	Mở địa chỉ website được công bố ở mục 5
MySQL	Lưu dữ liệu WordPress	Kiểm tra container và kết nối từ WordPress/phpMyAdmin
phpMyAdmin	Quản trị cơ sở dữ liệu qua giao diện web	Mở địa chỉ phpMyAdmin ở mục 5
Nginx	Reverse proxy và/hoặc áp dụng HTTP security headers theo cấu hình	Kiểm tra cấu hình Nginx và phản hồi HTTP
Prometheus	Thu thập và lưu trữ metrics	Mở giao diện Prometheus, kiểm tra Targets
Grafana	Hiển thị dashboard giám sát và truy vấn log	Mở dashboard, kiểm tra các panel
Loki	Lưu trữ log tập trung	Truy vấn log từ Grafana Explore
Promtail	Thu thập và gửi log đến Loki	Kiểm tra trạng thái dịch vụ và dữ liệu log
Exporters (nếu được khai báo trong Compose)	Cung cấp metrics cho Prometheus	Kiểm tra Targets trong Prometheus


Lưu ý: Chỉ giữ các thành phần thực sự được khai báo trong các file Compose của repository. Tên dịch vụ, cổng và đường dẫn bên dưới cần được đối chiếu với cấu hình hiện tại trước khi demo.

3. Kiến trúc và luồng hoạt động
3.1. Luồng website
Trình duyệt → Nginx → WordPress → MySQL

Nginx tiếp nhận yêu cầu HTTP/HTTPS theo cấu hình thực tế rồi chuyển tiếp đến WordPress. WordPress đọc/ghi dữ liệu nội dung trong MySQL. phpMyAdmin cung cấp giao diện để quản trị cơ sở dữ liệu.
3.2. Luồng giám sát
Các exporter/dịch vụ được cấu hình → Prometheus → Grafana

Prometheus thu thập metrics từ các target đã khai báo. Grafana truy vấn nguồn dữ liệu Prometheus và hiển thị các chỉ số trên dashboard. Cần xác nhận các target ở trạng thái UP trước khi chụp ảnh minh chứng.
3.3. Luồng logging
Docker/container logs → Promtail → Loki → Grafana Explore

Promtail thu thập log theo cấu hình và gửi đến Loki. Trong Grafana Explore, sử dụng LogQL để lọc log theo nhãn và nội dung. Kết quả truy vấn phụ thuộc vào nhãn thực tế, thời gian được chọn và việc có log phù hợp hay không.
4. Yêu cầu môi trường
- Máy Linux/Ubuntu VM hoặc môi trường có Docker Engine.
- Docker Compose plugin.
- Git.
- Trình duyệt web để truy cập các giao diện quản trị.
Kiểm tra cài đặt:
docker --version
docker compose version
git --version
5. Tải mã nguồn và khởi chạy
5.1. Clone repository
git clone https://github.com/dtc245160010-bit/DTC245160010.git
cd DTC245160010
Nếu thư mục dự án thực tế có tên hoặc vị trí khác, chuyển đến thư mục chứa file Compose trước khi chạy các lệnh tiếp theo.
5.2. Kiểm tra cấu trúc repository
ls -la
find . -maxdepth 3 -type f | sort
Xác định tên file Compose thực tế, ví dụ docker-compose.yml hoặc compose.yaml, cùng các thư mục cấu hình Nginx, Prometheus, Loki và Promtail nếu có.
5.3. Cấu hình biến môi trường
Nếu dự án có file .env.example, sao chép thành .env:p
cp .env.example .env
nano .env
Điền các giá trị cần thiết theo biến được tham chiếu trong file Compose. Nếu repository không có .env.example, hãy kiểm tra docker-compose.yml/compose.yaml và tài liệu đi kèm để biết các biến cần thiết; không tạo biến tùy ý.

Bảo mật: Không đưa mật khẩu thật, token hoặc thông tin nhạy cảm vào GitHub. Không commit file .env chứa thông tin bí mật. Chỉ commit file mẫu với giá trị minh họa không dùng trong thực tế. Docker khuyến nghị quản lý biến môi trường cẩn thận và không dùng chúng để truyền bí mật nếu có thể sử dụng cơ chế secrets phù hợp. Tham khảo Docker Docs – Environment variables.
5.4. Kiểm tra cấu hình và khởi chạy
Đứng tại thư mục chứa file Compose:
docker compose config
docker compose up -d
docker compose ps
Nếu dự án dùng nhiều file Compose, cần chỉ định đúng các file bằng -f, theo cấu trúc thực tế, ví dụ:
docker compose -f docker-compose.yml -f docker-compose.monitoring.yml config
docker compose -f docker-compose.yml -f docker-compose.monitoring.yml up -d
Chỉ dùng ví dụ nhiều file trên nếu repository thực sự có các file với tên tương ứng. Nếu lệnh docker compose config báo lỗi, sửa cấu hình trước khi tiếp tục.

Xem log dịch vụ:
docker compose logs --tail 50
Xem log của một dịch vụ cụ thể, thay <service> bằng tên dịch vụ trong file Compose:
docker compose logs --tail 50 <service>
5.5. Dừng hệ thống
docker compose down
Lệnh trên dừng và gỡ các container/network do Compose quản lý. Không thêm -v nếu chưa chắc chắn, vì tùy cấu hình, việc xóa volume có thể làm mất dữ liệu WordPress/MySQL.
6. Địa chỉ truy cập dịch vụ
Địa chỉ và cổng phụ thuộc vào phần ports trong file Compose. Lấy địa chỉ IP của Ubuntu VM bằng hostname -I, sau đó thay <IP-UBUNTU> trong bảng dưới bằng IP thực tế.
Dịch vụ	Địa chỉ mẫu	Nội dung cần xác minh
WordPress qua Nginx	http://<IP-UBUNTU>:8080	Trang website tải thành công
WordPress trực tiếp (nếu có publish cổng)	http://<IP-UBUNTU>:8083	Chỉ dùng nếu cổng này được khai báo trong Compose
phpMyAdmin	http://<IP-UBUNTU>:8081	Đăng nhập được và xem được database
Grafana	http://<IP-UBUNTU>:3000	Đăng nhập được và mở dashboard
Prometheus	http://<IP-UBUNTU>:9090	Mở được giao diện và kiểm tra Targets


Đây là các cổng mẫu dựa trên môi trường triển khai đã sử dụng trước đó. Hãy xác minh bằng docker compose ps và phần ports trong cấu hình hiện tại; không coi các địa chỉ này là cố định nếu cấu hình đã thay đổi.

7. Kiểm tra website và cơ sở dữ liệu
7.1. Website WordPress
- Mở địa chỉ WordPress qua Nginx.
- Kiểm tra trang chủ hiển thị nội dung.
- Mở một số trang hoặc sản phẩm đã tạo để chứng minh website hoạt động.
- Chụp ảnh màn hình có thanh địa chỉ trình duyệt.
7.2. MySQL và phpMyAdmin
- Mở phpMyAdmin.
- Đăng nhập bằng tài khoản được cấu hình trong dự án.
- Xác nhận database WordPress tồn tại.
- Mở danh sách bảng để chứng minh dữ liệu đã được tạo.
- Không chụp hoặc công khai mật khẩu trong ảnh minh chứng.
8. Nginx reverse proxy và bảo mật HTTP
Kiểm tra file cấu hình Nginx thực tế trong repository. Khi demo, trình bày:
1. Nginx lắng nghe trên cổng nào.
2. Nginx chuyển tiếp yêu cầu đến dịch vụ WordPress nào.
3. Các HTTP security headers có trong cấu hình, nếu đã bật.
4. HTTPS và chứng chỉ self-signed, nếu đã triển khai.
Có thể kiểm tra phản hồi HTTP bằng lệnh sau, thay URL bằng địa chỉ thực tế:
curl -I http://<IP-UBUNTU>:8080
Nếu hệ thống đã cấu hình HTTPS, kiểm tra bằng URL https://... tương ứng. Chỉ báo cáo HTTPS là hoàn thành khi đã cấu hình và kiểm thử được; nếu mới áp dụng security headers qua HTTP, hãy mô tả đúng trạng thái đó.
9. Prometheus và Grafana – giám sát
9.1. Kiểm tra Prometheus
1. Mở giao diện Prometheus.
2. Vào mục Status → Targets.
3. Kiểm tra các target đã cấu hình.
4. Chụp ảnh thể hiện target và trạng thái UP/DOWN.
Target UP cho biết Prometheus có thể scrape target đó tại thời điểm kiểm tra; không tự nó chứng minh toàn bộ ứng dụng hoạt động hoàn hảo.
9.2. Kiểm tra Grafana
1. Đăng nhập Grafana.
2. Mở dashboard giám sát đã tạo.
3. Chọn khoảng thời gian phù hợp, ví dụ Last 24 hours.
4. Xác minh các panel có dữ liệu cho những thành phần thực sự được cấu hình, chẳng hạn:
   - CPU, bộ nhớ và network của container.
   - Metrics của Nginx/web server.
   - Metrics của MySQL/database.
5. Nếu panel trống, kiểm tra nguồn dữ liệu, truy vấn, khoảng thời gian và trạng thái target trước khi chụp ảnh.
9.3. Minh chứng cần có
- Ảnh Prometheus Targets.
- Ảnh dashboard container monitoring.
- Ảnh panel Nginx.
- Ảnh panel MySQL.
- Các ảnh phải thể hiện dữ liệu thực tế, không chỉ là tên panel.
10. Loki, Promtail và truy vấn LogQL
Mở Grafana → Explore, chọn nguồn dữ liệu Loki và đặt khoảng thời gian có log. Trước tiên, mở Label browser để xác nhận nhãn và giá trị nhãn thực tế. Các truy vấn dưới đây là ví dụ cho môi trường đã dùng nhãn container; nếu nhãn của hệ thống khác, thay theo dữ liệu thực tế.
Truy vấn 1 – Log của Nginx
{container="de3_nginx"}
Ý nghĩa: Lọc log có nhãn container là de3_nginx.
Truy vấn 2 – Log Nginx có mã 499
{container="de3_nginx"} |= "499"
Ý nghĩa: Lọc các dòng log Nginx chứa chuỗi 499. Nếu không có kết quả, điều đó có thể đơn giản là trong khoảng thời gian chọn không có dòng log phù hợp; không nên tạo log giả để thay thế minh chứng.
Truy vấn 3 – Log WordPress
{container="de3_wordpress"}
Ý nghĩa: Lọc log có nhãn container là de3_wordpress.
Cách chứng minh logging hoạt động
- Chọn khoảng thời gian có log.
- Chạy từng truy vấn và chụp ảnh kết quả.
- Nếu truy vấn không trả về log, xác nhận nhãn bằng Label browser và kiểm tra Promtail/Loki.
- Trình bày luồng Docker logs → Promtail → Loki → Grafana Explore.
- Ghi rõ tên nhãn dựa trên kết quả thực tế của hệ thống.
11. Hardening – biện pháp bảo mật
Phần này cần đối chiếu trực tiếp với cấu hình dự án. Chỉ đánh dấu hoàn thành các biện pháp đã triển khai và có thể chứng minh.
Biện pháp	Bằng chứng cần trình bày
Mật khẩu mạnh, không dùng mật khẩu mẫu khi demo	Cấu hình đã che thông tin bí mật; không công khai mật khẩu
Không commit .env hoặc secret lên GitHub	.gitignore, file mẫu không chứa bí mật và kiểm tra repository
Phân tách mạng Docker theo vai trò	Các khai báo networks trong Compose và giải thích dịch vụ nào cần giao tiếp
Hạn chế publish cổng không cần thiết	Phần ports/expose trong Compose
Giới hạn quyền tài khoản database	Cấu hình tài khoản và quyền MySQL thực tế
HTTP security headers hoặc HTTPS	Cấu hình Nginx và kết quả kiểm tra bằng curl -I/truy cập HTTPS
Chạy dịch vụ non-root, nếu hỗ trợ và đã cấu hình	Trường user, Dockerfile hoặc kết quả kiểm tra user trong container


Không cần tuyên bố tất cả biện pháp trong bảng đều đã được triển khai. Hãy nêu rõ biện pháp nào đã làm, cách cấu hình, cách kiểm tra và ảnh minh chứng. Nếu chưa triển khai biện pháp nào, hãy hoàn thiện cấu hình rồi mới ghi là đạt yêu cầu.
12. GitHub và lịch sử commit
Repository cần có mã nguồn, file cấu hình, README và lịch sử commit có ý nghĩa. Có thể tổ chức các mốc commit theo nội dung công việc, ví dụ:
1. feat: deploy WordPress with MySQL and Nginx
2. feat: add Prometheus and Grafana monitoring
3. feat: add Loki and Promtail logging
4. docs: improve README and deployment evidence
Đây là tên commit gợi ý, không phải bằng chứng rằng các commit đó đã tồn tại. Không tạo commit chỉ để làm đẹp lịch sử; nội dung commit phải khớp với thay đổi thực tế.

Kiểm tra trước khi đẩy:
git status
git log --oneline --decorate -5
git diff --check
git diff -- README.md
Sau khi lưu README và kiểm tra thay đổi:
git add README.md
git commit -m "docs: update deployment README and demo checklist"
git push origin main
Nếu nhánh chính không phải main, hãy dùng tên nhánh thực tế.
13. Danh sách minh chứng cho báo cáo và demo
Nên chuẩn bị ít nhất các ảnh sau, đặt tên dễ hiểu và chèn vào báo cáo:
1. 01_github_repository.png – Repository và cấu trúc mã nguồn.
2. 02_git_commits.png – Lịch sử commit.
3. 03_wordpress_homepage.png – Trang chủ website.
4. 04_phpmyadmin_database.png – Database và bảng WordPress.
5. 05_nginx_config_or_headers.png – Cấu hình Nginx hoặc kết quả kiểm tra headers.
6. 06_prometheus_targets.png – Trạng thái các target.
7. 07_grafana_container_monitoring.png – Metrics container.
8. 08_grafana_nginx_metrics.png – Metrics web server/Nginx.
9. 09_grafana_mysql_metrics.png – Metrics database/MySQL.
10. 10_loki_nginx_logs.png – Log Nginx trong Grafana Explore.
11. 11_logql_query_499.png – Truy vấn LogQL lọc mã 499, nếu có kết quả thực tế.
12. 12_wordpress_logs_and_hardening.png – Log WordPress và/hoặc bằng chứng hardening.
Nếu truy vấn 499 không có dữ liệu trong khoảng thời gian kiểm tra, thay ảnh đó bằng một truy vấn log thực tế có kết quả và ghi chú rõ trong báo cáo.
14. Kịch bản demo ngắn
1. Mở GitHub và README để giới thiệu mã nguồn, hướng dẫn chạy và lịch sử commit.
2. Mở website WordPress.
3. Mở phpMyAdmin, chỉ ra database và bảng WordPress.
4. Trình bày Nginx reverse proxy và cấu hình bảo mật đã triển khai.
5. Mở Prometheus Targets và xác nhận các target.
6. Mở Grafana dashboard, giải thích metrics container, Nginx và MySQL.
7. Mở Grafana Explore, chạy 2–3 truy vấn LogQL.
8. Trình bày các biện pháp hardening cùng bằng chứng cấu hình/kiểm tra.
15. Kết luận
Dự án hướng đến việc triển khai website WordPress và các thành phần quản trị, giám sát, logging bằng Docker Compose. Kết quả cuối cùng cần được xác nhận bằng việc khởi chạy thành công, kiểm tra từng dịch vụ, truy vấn metrics/log thực tế và lưu ảnh minh chứng. README này là hướng dẫn triển khai và demo; các cổng, tên dịch vụ, file cấu hình và biện pháp bảo mật phải được đối chiếu với trạng thái thực tế của repository trước khi nộ
