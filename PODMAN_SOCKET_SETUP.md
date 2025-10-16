# Podman Socket Setup Instructions

## สำหรับ Windows/WSL2

### 1. ตรวจสอบ Podman Socket บน Host

```bash
# ตรวจสอบว่า Podman socket กำลังรันอยู่
podman system connection list

# เปิด Podman socket (rootless mode)
podman system service --time=0 unix:///run/user/1000/podman/podman.sock

# หรือสำหรับ rootful mode
# sudo podman system service --time=0 unix:///run/podman/podman.sock
```

### 2. หาตำแหน่ง Podman Socket

```bash
# สำหรับ rootless mode (แนะนำ)
ls -la /run/user/$(id -u)/podman/podman.sock

# สำหรับ rootful mode
ls -la /run/podman/podman.sock
```

### 3. เปิด Podman Socket เป็น Service (ถาวร)

```bash
# Enable podman socket service (rootless)
systemctl --user enable podman.socket
systemctl --user start podman.socket

# Check status
systemctl --user status podman.socket

# หรือสำหรับ rootful
# sudo systemctl enable podman.socket
# sudo systemctl start podman.socket
```

### 4. Build และรัน Jenkins Container

```bash
# Build image ใหม่
podman-compose up -d --build

# หรือใช้ docker-compose
docker-compose up -d --build
```

### 5. ทดสอบการเชื่อมต่อ Podman จากภายใน Container

```bash
# เข้าไปใน container
podman exec -it jenkins bash

# ทดสอบ podman command
podman --version
podman ps
podman images
```

## Troubleshooting

### ปัญหา: Permission Denied

```bash
# ตรวจสอบ permissions ของ socket
ls -la /run/user/$(id -u)/podman/podman.sock

# เพิ่ม permissions (ถ้าจำเป็น)
chmod 666 /run/user/$(id -u)/podman/podman.sock
```

### ปัญหา: Socket ไม่มีอยู่

```bash
# Restart podman socket service
systemctl --user restart podman.socket

# หรือรันแบบ manual
podman system service --time=0 unix:///run/user/$(id -u)/podman/podman.sock &
```

## ใช้งานใน Jenkins Pipeline

```groovy
pipeline {
    agent any

    stages {
        stage('Test Podman') {
            steps {
                sh 'podman --version'
                sh 'podman ps'
            }
        }

        stage('Build with Podman') {
            steps {
                sh 'podman build -t myapp:latest .'
            }
        }

        stage('Run Container') {
            steps {
                sh 'podman run -d --name myapp myapp:latest'
            }
        }
    }
}
```

## หมายเหตุ

- ใช้ rootless mode จะปลอดภัยกว่า
- Socket path สำหรับ rootless: `/run/user/1000/podman/podman.sock`
- Socket path สำหรับ rootful: `/run/podman/podman.sock`
- UID 1000 อาจแตกต่างกันไป ใช้ `id -u` เพื่อดู UID ของคุณ
