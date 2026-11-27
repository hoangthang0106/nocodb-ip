# IT-MANAGER — NocoDB Portainer Git Stack (18898:18898)

## Port và trùng cổng

Port **18898** là cổng tiếp theo sau Home Assistant **18897**. Trong repo `hoangthang0106/social-analyzer`, `docker-compose.yml` cũng có cấu hình 18898, nhưng **không có bằng chứng Social Analyzer đang chạy** trên host. Không được dùng cùng host port cho hai ứng dụng.

**Trước khi deploy kiểm tra trên Docker host `192.168.20.194`:**

```bash
sudo ss -lntp | grep ':18898' || echo 'No process currently listening on 18898'
docker ps --format 'table {{.Names}}\t{{.Ports}}'
```

Nếu 18898 đã được một container khác dùng, **dừng triển khai, không tự ý xóa container hay đổi mạng**. Nếu muốn giữ Social Analyzer trên cổng này thì chọn port khác cho NocoDB.

## Portainer > Stacks > Add stack > Git repository

| Trường | Giá trị |
|---|---|
| Name | `nocodb-ip-18898` |
| Repository URL | `https://github.com/hoangthang0106/nocodb-ip` |
| Repository reference | `refs/heads/portainer-18898` |
| Compose path | `docker-compose.yml` |

Bấm **Deploy the stack**. Truy cập **`http://192.168.20.194:18898/dashboard`** để tạo tài khoản quản trị lần đầu.

## Kiến trúc và dữ liệu

- Dùng image chính thức `nocodb/nocodb:latest`; **không build source fork**.
- TCP `18898:18898` với `PORT=18898` để ứng dụng lắng nghe trên cùng port container.
- Dùng `network_mode: bridge` của Docker host có sẵn; **không tạo thêm bridge/subnet** để tránh lỗi mất kết nối LAN đã gặp.
- Volume `nocodb_ip_data` tại `/usr/app/data` lưu cấu hình và database metadata SQLite. Không xóa volume khi cập nhật/gỡ Stack.
- SQLite hợp để bắt đầu hoặc dùng thử; dữ liệu quan trọng hoặc nhiều người dùng nên có PostgreSQL và backup riêng.
- `NC_SITE_URL=http://192.168.20.194:18898` được đặt cho đường dẫn ứng dụng.
- Với các ứng dụng cài từ nhánh Portainer khác, các port được ghi trên GitHub là **cấu hình dự kiến**, không phải báo cáo host port đang chiếm.

## Chẩn đoán

```bash
docker ps -a --filter name=nocodb-ip
docker logs --tail 80 nocodb-ip
docker port nocodb-ip
curl -I --max-time 10 http://127.0.0.1:18898/
```

Nếu đã cài Stack cũ `nocodb-ip-18899`, cần **cập nhật chính Stack đó** sang nhánh mới, không deploy cả hai Stack cùng lúc vì trùng container name và volume. Không xóa volume.

## Nguồn chính thức

- https://github.com/nocodb/nocodb
- https://nocodb.com/docs/self-hosting/
