# Hướng dẫn triển khai n8n với Nginx

## Yêu cầu hệ thống
- Docker và Docker Compose
- Nginx
- Domain đã trỏ DNS về server
- SSL certificate (Let's Encrypt hoặc các loại khác)

## Các bước triển khai

### 1. Chuẩn bị môi trường

Tạo thư mục cho dữ liệu n8n:
```bash
mkdir -p n8n_data
```

### 2. Cấu hình môi trường

Tạo file `.env` với nội dung:
```env
DOMAIN_NAME=your-domain.com
SUBDOMAIN=n8n
GENERIC_TIMEZONE=Asia/Ho_Chi_Minh
```

### 3. Cấu hình Nginx

Tạo file cấu hình Nginx (ví dụ: `/etc/nginx/conf.d/n8n.conf`):
```nginx
server {
    listen 80;
    server_name n8n.your-domain.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name n8n.your-domain.com;

    # SSL configuration
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://127.0.0.1:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Tăng timeout
        proxy_connect_timeout 600;
        proxy_send_timeout 600;
        proxy_read_timeout 600;
        send_timeout 600;
    }
}
```

### 4. Khởi động n8n

```bash
# Khởi động n8n
docker-compose up -d

# Kiểm tra logs
docker-compose logs -f
```

### 5. Kiểm tra hoạt động

- Truy cập https://n8n.your-domain.com
- Đăng nhập với thông tin mặc định (nếu là lần đầu tiên)

## Cấu trúc thư mục

```
.
├── docker-compose.yml    # File cấu hình Docker Compose
├── .env                 # File biến môi trường
└── n8n_data            # Thư mục chứa dữ liệu n8n
```

## Bảo trì và nâng cấp

### Backup dữ liệu
```bash
# Backup thư mục n8n_data
tar -czf n8n_backup_$(date +%Y%m%d).tar.gz n8n_data/
```

### Cập nhật n8n
```bash
# Pull image mới nhất
docker-compose pull

# Khởi động lại với image mới
docker-compose up -d
```

### Xem logs
```bash
docker-compose logs -f
```

## Xử lý sự cố

### 1. Không thể truy cập n8n
- Kiểm tra status của container: `docker-compose ps`
- Kiểm tra logs: `docker-compose logs -f`
- Kiểm tra cấu hình Nginx
- Kiểm tra SSL certificate

### 2. Lỗi WebSocket
- Kiểm tra cấu hình WebSocket trong Nginx
- Đảm bảo các header WebSocket đã được cấu hình đúng

### 3. Timeout khi chạy workflow dài
- Kiểm tra các giá trị timeout trong cấu hình Nginx
- Tăng giá trị timeout nếu cần thiết

## Lưu ý bảo mật

1. Luôn sử dụng HTTPS
2. Giới hạn truy cập vào cổng 5678 chỉ từ localhost
3. Thường xuyên cập nhật n8n lên phiên bản mới nhất
4. Backup dữ liệu định kỳ
5. Sử dụng mật khẩu mạnh cho tài khoản admin

## Hỗ trợ

Nếu gặp vấn đề, vui lòng:
1. Kiểm tra logs của Docker: `docker-compose logs -f`
2. Kiểm tra logs của Nginx: `tail -f /var/log/nginx/error.log`
3. Tạo issue trên GitHub repository này
