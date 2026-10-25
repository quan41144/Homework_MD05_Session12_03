## 1. Cấu hình Nginx hoàn chỉnh

[`nginx.conf`](./nginx.conf)

```nginx
server {
    listen 443 ssl;
    server_name secure.example.com;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    # 1. Chống Clickjacking: Chỉ cho phép hiển thị trong iframe cùng domain
    add_header X-Frame-Options "SAMEORIGIN" always;

    # 2. Chống MIME-sniffing: Ngăn trình duyệt đoán sai định dạng file
    add_header X-Content-Type-Options "nosniff" always;

    # 3. Ép buộc sử dụng HTTPS (HSTS) trong vòng 1 năm (31536000 giây) bao gồm cả subdomains
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 2. Giải thích chi tiết các Security Headers

| Header | Giá trị | Mục đích & Nguy cơ ngăn chặn |
|---|---|---|
| **`X-Frame-Options`** | `SAMEORIGIN` | Ngăn chặn tấn công **Clickjacking**. Khi đặt `SAMEORIGIN`, trang web chỉ có thể được nhúng vào thẻ `<iframe>`, `<frame>`, `<embed>`, `<object>` từ các trang có cùng nguồn gốc (origin) với nó, ngăn chặn kẻ xấu nhúng website vào trang độc hại để lừa người dùng click. |
| **`X-Content-Type-Options`** | `nosniff` | Ngăn chặn tấn công **MIME Confusion / MIME-sniffing**. Buộc trình duyệt phải tuân thủ đúng MIME type được khai báo trong header `Content-Type`, ngăn trình duyệt tự ý đoán (sniff) loại file (ví dụ: thực thi file ảnh `.jpg` chứa mã độc JS như một script). |
| **`Strict-Transport-Security`** (HSTS) | `max-age=31536000; includeSubDomains` | Ép buộc trình duyệt **chỉ kết nối qua HTTPS** trong thời gian được chỉ định (`31536000` giây = 1 năm) và áp dụng cho toàn bộ domain con (`includeSubDomains`). Giúp ngăn chặn tấn công **SSL Stripping** và **Man-in-the-middle (MITM)**. |
