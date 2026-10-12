\# Bài 4 - Custom Network \& Named Volume



\## 1. Mục tiêu



Tạo Docker Custom Bridge Network để các container giao tiếp với nhau bằng hostname và sử dụng Named Volume để lưu trữ dữ liệu PostgreSQL bền vững.



\## 2. Thành phần



\- Network: `app-net`

\- Volume: `db-data`

\- Database Container: `db-server`

\- Backend Container: `app-container`

\- Database: PostgreSQL 15 Alpine

\- Backend Image: `hw-backend:v1`



\## 3. Tạo Network



```bash

docker network create --driver bridge app-net

```



\## 4. Tạo Named Volume



```bash

docker volume create db-data

```



\## 5. Chạy PostgreSQL



```bash

docker run -d \\

&#x20; --name db-server \\

&#x20; --network app-net \\

&#x20; -e POSTGRES\_USER=appuser \\

&#x20; -e POSTGRES\_PASSWORD=apppass \\

&#x20; -e POSTGRES\_DB=appdb \\

&#x20; -v db-data:/var/lib/postgresql/data \\

&#x20; postgres:15-alpine

```



Volume `db-data` được mount vào:



```text

/var/lib/postgresql/data

```



\## 6. Chạy Backend



Backend được kết nối vào cùng network `app-net` và sử dụng hostname:



```text

db-server

```



Docker DNS cho phép backend phân giải hostname `db-server` thành địa chỉ IP nội bộ của database container.



\## 7. Kiểm tra Network



```bash

docker network inspect app-net

```



Kiểm tra DNS:



```bash

docker exec app-container getent hosts db-server

```



\## 8. Kiểm tra Data Persistence



Tạo bảng và thêm dữ liệu:



```sql

CREATE TABLE users (

&#x20;   id SERIAL PRIMARY KEY,

&#x20;   name VARCHAR(100)

);



INSERT INTO users (name) VALUES ('Chien');



SELECT \* FROM users;

```



Sau đó xóa container:



```bash

docker rm -f db-server

```



Volume `db-data` vẫn được giữ lại.



Khởi chạy lại database container với cùng volume:



```bash

docker run -d \\

&#x20; --name db-server \\

&#x20; --network app-net \\

&#x20; -e POSTGRES\_USER=appuser \\

&#x20; -e POSTGRES\_PASSWORD=apppass \\

&#x20; -e POSTGRES\_DB=appdb \\

&#x20; -v db-data:/var/lib/postgresql/data \\

&#x20; postgres:15-alpine

```



Kiểm tra:



```sql

SELECT \* FROM users;

```



Dữ liệu vẫn còn sau khi container database bị xóa và tạo lại.



\## 9. Kết luận



Custom Bridge Network cho phép các container giao tiếp nội bộ bằng tên service/container thông qua Docker DNS. Named Volume giúp dữ liệu PostgreSQL tồn tại độc lập với vòng đời của container.



Do đó, khi `db-server` bị xóa và tạo lại, dữ liệu vẫn được khôi phục từ volume `db-data`.

