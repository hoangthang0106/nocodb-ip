# IT-MANAGER — Cài NocoDB trên Portainer (18899:18899)

## Vì sao dùng 18899 thay vì 18898?

Theo cấu hình GitHub `hoangthang0106/social-analyzer/docker-compose.yml`, cổng **18898** đã dành cho Social Analyzer; chưa xác minh trạng thái runtime Docker host. **18899** được chọn làm ứng viên tiếp theo. Trong cấu hình NetAlertX cũ, Grafana 18899 chỉ thuộc tệp monitoring tùy chọn, không nằm trong Stack mặc định; **vẫn phải kiểm tra host trước khi cài**:

```bash
sudo ss -lntp | grep ':18899'
docker ps --format 'table {{.Names}}\t{{.Ports}}'
```

Nếu thấy 18899 đang sử dụng, KHÔNG deploy hoặc xóa ứng dụng đang dùng cổng; chọn cổng trống khác rồi sửa Compose.

## Portainer Git Stack

- **Name**: `nocodb-ip-18899`
- **Repository URL**: `https://github.com/hoangthang0106/nocodb-ip`
- **Repository reference**: `refs/heads/portainer-18899`
- **Compose path**: `docker-compose.yml`
- Chọn **Deploy the stack**.
- Truy cập `http://192.168.20.194:18899/dashboard` và tạo tài khoản admin lần đầu.

## Docker networking

- Compose dùng `network_mode: bridge` (Docker system bridge có sẵn, trước đây là `172.17.0.0/16` trên Portainer). Không cấp subnet `192.168.x.x`, không tạo bridge mới; tránh lặp lại sự cố route xảy ra khi dùng mạng Compose tự cấp.
- Chỉ publish `18899:18899/tcp`, vì NocoDB hỗ trợ biến `PORT=18899`; không dùng `18899:8080`.
- Không thay IP LAN Docker host `192.168.20.194`.

## Lưu dữ liệu và giới hạn

- Image chính thức `nocodb/nocodb:latest`, **không biên dịch** mã nguồn fork `develop`; nhánh triển khai chỉ chứa Compose và hướng dẫn để Portainer clone nhanh.
- Volume `nocodb_ip_data` mount tại `/usr/app/data`, lưu thông tin NocoDB SQLite và file NocoDB lưu cục bộ. Không chọn xóa volume khi gỡ/cập nhật Stack.
- Cách này phù hợp dùng thử hoặc quy mô nhỏ. Với database quan trọng hoặc nhiều người dùng, nên chuyển metadata DB sang PostgreSQL, triển khai backup và kiểm soát phiên bản trước khi sử dụng sản xuất.
- `NC_SITE_URL` đặt URL mà trình duyệt dùng để tạo link/invite.
- Tạo tài khoản quản trị bằng mật khẩu mạnh ngay khi chạy; chỉ mở web ra LAN đáng tin cậy, nên có HTTPS hoặc reverse proxy nếu mở Internet.

## Kiểm tra nhanh

```bash
docker ps -a --filter name=nocodb-ip
docker logs --tail 100 nocodb-ip
docker port nocodb-ip
docker inspect nocodb-ip --format '{{.HostConfig.NetworkMode}} | {{range $n,$v := .NetworkSettings.Networks}}{{$n}}={{$v.IPAddress}} {{end}}'
curl -I --max-time 10 http://127.0.0.1:18899/
```

Nếu cài rồi mất kết nối toàn bộ LAN, gỡ **riêng** Stack mới, không xóa volumes; không dùng `docker network prune` hoặc đổi subnet ngẫu nhiên.

## Nguồn

- https://github.com/hoangthang0106/nocodb-ip
- https://nocodb.com/docs/self-hosting/environment-variables
- https://nocodb.com/docs/self-hosting/installation/quickstart
