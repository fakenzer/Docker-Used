# Hướng dẫn Docker từ cơ bản đến nâng cao

## Mục lục
1. [Docker là gì?](#docker-là-gì)
2. [Cài đặt Docker](#cài-đặt-docker)
3. [Các khái niệm cơ bản](#các-khái-niệm-cơ-bản)
4. [Các lệnh Docker cơ bản](#các-lệnh-docker-cơ-bản)
5. [Ví dụ thực hành](#ví-dụ-thực-hành)
6. [Dockerfile](#dockerfile)
7. [Docker Compose](#docker-compose)
8. [Best Practices](#best-practices)

## Docker là gì?

Docker là một nền tảng containerization cho phép đóng gói ứng dụng và tất cả dependencies của nó vào một container nhẹ, portable và self-sufficient. Container này có thể chạy trên bất kỳ máy nào có Docker được cài đặt.

### Lợi ích của Docker:
- **Consistency**: Ứng dụng chạy giống nhau trên mọi môi trường
- **Isolation**: Các ứng dụng được cô lập với nhau
- **Portability**: Dễ dàng di chuyển giữa các môi trường
- **Efficiency**: Sử dụng tài nguyên hiệu quả hơn VM
- **Scalability**: Dễ dàng scale up/down

## Cài đặt Docker

### Windows/macOS:
1. Tải Docker Desktop từ [docker.com](https://www.docker.com/products/docker-desktop)
2. Cài đặt và khởi động Docker Desktop
3. Kiểm tra cài đặt: `docker --version`

### Ubuntu/Linux:
```bash
# Cập nhật package index
sudo apt update

# Cài đặt các package cần thiết
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release

# Thêm Docker GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Thêm Docker repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Cài đặt Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Kiểm tra cài đặt
sudo docker --version
```

## Các khái niệm cơ bản

### Image
- **Định nghĩa**: Template chỉ đọc để tạo container
- **Đặc điểm**: Immutable, có thể version, có thể share

### Container
- **Định nghĩa**: Instance đang chạy của một image
- **Đặc điểm**: Mutable, có lifecycle, isolated

### Registry
- **Định nghĩa**: Nơi lưu trữ và phân phối Docker images
- **Ví dụ**: Docker Hub, AWS ECR, Google Container Registry

### Dockerfile
- **Định nghĩa**: File text chứa instructions để build image
- **Mục đích**: Tự động hóa việc tạo image

## Các lệnh Docker cơ bản

### Quản lý Images
```bash
# Liệt kê images
docker images
docker image ls

# Tải image từ registry
docker pull ubuntu:20.04

# Xóa image
docker rmi ubuntu:20.04

# Build image từ Dockerfile
docker build -t myapp:v1.0 .

# Tag image
docker tag myapp:v1.0 myapp:latest
```

### Quản lý Containers
```bash
# Chạy container
docker run ubuntu:20.04

# Chạy container với tên
docker run --name my-ubuntu ubuntu:20.04

# Chạy container interactively
docker run -it ubuntu:20.04 /bin/bash

# Chạy container trong background
docker run -d nginx

# Liệt kê containers đang chạy
docker ps

# Liệt kê tất cả containers
docker ps -a

# Dừng container
docker stop container_name

# Khởi động lại container
docker restart container_name

# Xóa container
docker rm container_name

# Xem logs
docker logs container_name

# Exec vào container đang chạy
docker exec -it container_name /bin/bash
```

### Quản lý Networks và Volumes
```bash
# Liệt kê networks
docker network ls

# Tạo network
docker network create my-network

# Liệt kê volumes
docker volume ls

# Tạo volume
docker volume create my-volume

# Mount volume
docker run -v my-volume:/data ubuntu:20.04
```

## Ví dụ thực hành

### Ví dụ 1: Chạy web server đơn giản với Nginx

```bash
# Tải Nginx image
docker pull nginx

# Chạy Nginx container
docker run -d --name my-nginx -p 8080:80 nginx

# Kiểm tra container đang chạy
docker ps

# Truy cập web server tại http://localhost:8080

# Dừng và xóa container
docker stop my-nginx
docker rm my-nginx
```

### Ví dụ 2: Chạy database MySQL

```bash
# Chạy MySQL container với environment variables
docker run -d \
  --name my-mysql \
  -e MYSQL_ROOT_PASSWORD=mypassword \
  -e MYSQL_DATABASE=mydb \
  -p 3306:3306 \
  mysql:8.0

# Kết nối đến database
docker exec -it my-mysql mysql -u root -p

# Trong MySQL shell
USE mydb;
CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(100));
INSERT INTO users (name) VALUES ('John Doe');
SELECT * FROM users;
```

### Ví dụ 3: Chạy ứng dụng Node.js

Tạo file `app.js`:
```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('Hello Docker!');
});

app.listen(port, () => {
  console.log(`App listening at http://localhost:${port}`);
});
```

Tạo file `package.json`:
```json
{
  "name": "docker-node-app",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

Chạy với Docker:
```bash
# Chạy Node.js container
docker run -d \
  --name my-node-app \
  -p 3000:3000 \
  -v $(pwd):/app \
  -w /app \
  node:16 \
  sh -c "npm install && npm start"

# Truy cập ứng dụng tại http://localhost:3000
```

## Dockerfile

### Cấu trúc cơ bản của Dockerfile

```dockerfile
# Base image
FROM node:16-alpine

# Metadata
LABEL maintainer="your-email@example.com"
LABEL version="1.0"

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Default command
CMD ["npm", "start"]
```

### Ví dụ Dockerfile cho ứng dụng Python Flask

```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Copy requirements first for better caching
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN useradd --create-home --shell /bin/bash app
USER app

EXPOSE 5000

CMD ["python", "app.py"]
```

### Build và chạy từ Dockerfile

```bash
# Build image
docker build -t my-app:v1.0 .

# Chạy container từ image vừa build
docker run -d --name my-app -p 3000:3000 my-app:v1.0
```

## Docker Compose

Docker Compose cho phép định nghĩa và chạy multi-container applications.

### Ví dụ docker-compose.yml

```yaml
version: '3.8'

services:
  # Web application
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    depends_on:
      - db
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped

  # Database
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: myapp
      MYSQL_USER: appuser
      MYSQL_PASSWORD: apppassword
    volumes:
      - db_data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3306:3306"
    restart: unless-stopped

  # Redis cache
  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"
    restart: unless-stopped

  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - web
    restart: unless-stopped

volumes:
  db_data:

networks:
  default:
    driver: bridge
```

### Các lệnh Docker Compose

```bash
# Khởi động tất cả services
docker-compose up

# Khởi động trong background
docker-compose up -d

# Build và khởi động
docker-compose up --build

# Dừng tất cả services
docker-compose down

# Dừng và xóa volumes
docker-compose down -v

# Xem logs
docker-compose logs

# Xem logs của service cụ thể
docker-compose logs web

# Scale service
docker-compose up --scale web=3

# Exec vào service
docker-compose exec web /bin/bash
```

## Best Practices

### 1. Dockerfile Best Practices

```dockerfile
# Sử dụng multi-stage builds
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:16-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
CMD ["npm", "start"]

# Minimize layers
RUN apt-get update && apt-get install -y \
    curl \
    vim \
    && rm -rf /var/lib/apt/lists/*

#  Use specific tags
FROM node:16.14.2-alpine

#  Don't run as root
RUN adduser -D appuser
USER appuser
```

### 2. Security Best Practices

```bash
# Scan image for vulnerabilities
docker scan my-app:latest

# Use secrets for sensitive data
docker run -d --name app \
  --secret db_password \
  my-app:latest

# Limit container resources
docker run -d --name app \
  --memory="512m" \
  --cpus="1.0" \
  my-app:latest
```

### 3. Performance Best Practices

```dockerfile
# Use .dockerignore
# .dockerignore
node_modules
.git
.gitignore
README.md
Dockerfile
.dockerignore

#  Cache dependencies
COPY package*.json ./
RUN npm ci --only=production
COPY . .

#  Use alpine images when possible
FROM node:16-alpine
```

### 4. Development Workflow

```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    command: npm run dev
```

### 5. Production Deployment

```yaml
# docker-compose.prod.yml
version: '3.8'
services:
  web:
    image: my-app:${APP_VERSION}
    restart: unless-stopped
    environment:
      - NODE_ENV=production
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## Monitoring và Debugging

### Xem resource usage
```bash
# Container stats
docker stats

# System info
docker system info

# Disk usage
docker system df

# Clean up
docker system prune
```

### Debug containers
```bash
# Inspect container
docker inspect container_name

# View processes
docker top container_name

# Copy files from/to container
docker cp file.txt container_name:/path/
docker cp container_name:/path/file.txt ./
```

## Kết luận

Docker là một công cụ mạnh mẽ giúp đơn giản hóa việc phát triển, triển khai và quản lý ứng dụng. Với những kiến thức cơ bản này, bạn có thể bắt đầu sử dụng Docker cho các dự án của mình. Hãy thực hành thường xuyên và tìm hiểu thêm các tính năng nâng cao như Docker Swarm, Kubernetes integration, và CI/CD workflows.
