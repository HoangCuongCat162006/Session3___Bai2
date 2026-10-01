# Hướng Dẫn Thực Hiện & Kiểm Tra Bài 2: Cấu Hình Trang Lỗi Tùy Chỉnh (404.html)

## 1. Yêu cầu đề bài
- **Tệp nguồn:** [404.html](file:///c:/IT_209/Session3/Session3__Bai2/404.html) đặt tại thư mục web root `/var/www/my-web/html/404.html`.
- **Cấu hình Nginx:** 
  - Khai báo chỉ thị: `error_page 404 /404.html;`
  - Khai báo block bảo mật:
    ```nginx
    location = /404.html {
        root /var/www/my-web/html;
        internal;
    }
    ```
- **Mục đích của chỉ thị `internal`:** Ngăn chặn người dùng từ internet truy cập trực tiếp bằng URL `http://<IP>/404.html`. Khi truy cập trực tiếp, Nginx sẽ từ chối và trả về mã lỗi 404 Not Found.

---

## 2. Các bước triển khai trên Server (Droplet Ubuntu)

### Bước 1: Đảm bảo thư mục web root tồn tại
```bash
sudo mkdir -p /var/www/my-web/html
```

### Bước 2: Tạo hoặc sao chép tệp `404.html` vào thư mục web root
Mở nano để dán nội dung [404.html](file:///c:/IT_209/Session3/Session3__Bai2/404.html):
```bash
sudo nano /var/www/my-web/html/404.html
```
*(Dán toàn bộ nội dung trong tệp `404.html`, sau đó nhấn `Ctrl + O` -> `Enter` để lưu, `Ctrl + X` để thoát).*

Phân quyền chuẩn cho thư mục web:
```bash
sudo chown -R www-data:www-data /var/www/my-web/html
sudo chmod -R 755 /var/www/my-web/html
```

### Bước 3: Cập nhật cấu hình Server Block Nginx
Mở tệp cấu hình server block (thường là `/etc/nginx/sites-available/my-web` hoặc `/etc/nginx/sites-available/default`):
```bash
sudo nano /etc/nginx/sites-available/my-web
```

Điền nội dung cấu hình chuẩn:
```nginx
server {
    listen 80;
    listen [::]:80;

    server_name _;

    root /var/www/my-web/html;
    index index.html index.htm;

    # Cấu hình trang lỗi 404 tùy biến
    error_page 404 /404.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # Chặn truy cập trực tiếp URL /404.html bằng chỉ thị internal
    location = /404.html {
        root /var/www/my-web/html;
        internal;
    }
}
```

Nếu chưa kích hoạt site (chưa có symlink trong `sites-enabled`), tạo symlink:
```bash
sudo ln -sf /etc/nginx/sites-available/my-web /etc/nginx/sites-enabled/
```

### Bước 4: Kiểm tra cú pháp và reload Nginx
Kiểm tra tính chính xác của file cấu hình:
```bash
sudo nginx -t
```
*Kết quả hiển thị `syntax is ok` và `test is successful` là đạt.*

Tải lại cấu hình Nginx:
```bash
sudo systemctl reload nginx
```

---

## 3. Kiểm tra kết quả (Testing)

### Lệnh 1: Truy cập một đường dẫn không tồn tại
```bash
curl -I http://<IP_ADDRESS_DROPLET>/invalid-path-demo
```
**Kết quả mong đợi:**
- Trả về mã phản hồi: `HTTP/1.1 404 Not Found`
- Nếu dùng trình duyệt truy cập `http://<IP_ADDRESS_DROPLET>/invalid-path-demo`, hiển thị giao diện trang HTML lỗi tùy biến đẹp mắt.

### Lệnh 2: Thử truy cập trực tiếp vào tệp `/404.html`
```bash
curl -I http://<IP_ADDRESS_DROPLET>/404.html
```
**Kết quả mong đợi:**
- Vẫn trả về mã phản hồi: `HTTP/1.1 404 Not Found`
- *Giải thích:* Nhờ chỉ thị `internal`, Nginx không cho phép request từ client gọi trực tiếp đến `/404.html`.

---

## 4. Hướng dẫn nộp bài lên GitHub
Theo cấu trúc đề bài: `homework/session_03/ex2/`
Học viên copy 2 file:
1. `404.html`
2. `my-web` (hoặc `my-web.conf` tùy theo cách đặt tên Server Block của bạn)

Vào thư mục `homework/session_03/ex2/` của repo GitHub rồi tiến hành commit & push:
```bash
git add homework/session_03/ex2/
git commit -m "feat(session_03): complete exercise 2 - custom 404 error page with internal directive"
git push origin main
```
