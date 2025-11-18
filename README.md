# 🚀 Node.js Heroku Sample – Dockerized & Deployed on AWS EC2 with Nginx

This project is a Node.js web application (Heroku sample app) packaged into a Docker image and deployed on an AWS EC2 Ubuntu instance.
Nginx is used as a reverse proxy to route external traffic (port 80) to the running Docker container (port 5006).

---

## 📦 Project Overview

This repository contains:

* A Node.js application (Express + EJS)
* Dockerfile for containerizing the app
* Nginx configuration for production deployment
* Jenkins pipeline (optional) for CI/CD
* Instructions for local and server deployment

---

## 🏗 Architecture

```
User → Nginx (Port 80) → Docker Container → Node.js App (Port 5006)
```

* **Nginx** serves as the public-facing web server.
* **Docker** isolates the Node application.
* **EC2** hosts both Nginx and Docker.

---

## 🐳 Docker Setup

### **Build Docker Image**

```bash
docker build -t kishangollamudi/nodeapp:latest .
```

### **Run the Container (Host 3000 → Container 5006)**

```bash
docker run -d --name nodeapp -p 3000:5006 kishangollamudi/nodeapp:latest
```

### **Check Logs**

```bash
docker logs -f nodeapp
```

You should see:

```
Listening on 5006
```

---

## 🖥 Deploying on AWS EC2 (Ubuntu)

### **1. Install Docker**

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
```

### **2. Pull & Run the Container**

```bash
docker pull kishangollamudi/nodeapp:latest

docker run -d --name nodeapp -p 3000:5006 kishangollamudi/nodeapp:latest
```

### **3. Install Nginx**

```bash
sudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
```

---

## 🌐 Nginx Reverse Proxy Configuration

Create an Nginx site file:

```bash
sudo tee /etc/nginx/sites-available/nodeapp <<'EOF'
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
EOF
```

Enable the site:

```bash
sudo ln -sf /etc/nginx/sites-available/nodeapp /etc/nginx/sites-enabled/nodeapp
sudo rm -f /etc/nginx/sites-enabled/default
```

Reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔐 AWS Security Group Rules

Allow inbound:

| Port | Purpose             | Source       |
| ---- | ------------------- | ------------ |
| 80   | HTTP (Nginx)        | 0.0.0.0/0    |
| 3000 | App test (optional) | 0.0.0.0/0    |
| 22   | SSH                 | Your IP only |

---

## 🧪 Verification

### From EC2:

```bash
curl http://localhost:3000
```

### From Browser:

```
http://<EC2-PUBLIC-IP>
```

If the page loads, deployment is successful 🎉

---

## 🛠 Troubleshooting

### **502 Bad Gateway**

* Container not running → `docker ps`
* App not listening → `docker logs nodeapp`
* Wrong proxy port → verify Nginx `proxy_pass`
* Host port not mapped → `docker inspect nodeapp`

### **Nginx default page still showing**

```bash
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl reload nginx
```

### **App works on :3000 but not on port 80**

* Nginx configuration incorrect
* Security group missing port 80 rule

---

## 🚀 Future Enhancements

* HTTPS (Certbot + Let's Encrypt)
* Systemd service for auto-start
* Jenkins automated deployment
* Docker Compose for multi-service architecture

---

## 👤 Author

**Kishan Gollamudi**
Docker | Jenkins | AWS | DevOps Engineer in Progress ⚡

---

If you want, I can also create:

✅ A **deployment.sh** automation script
✅ A **Jenkins CD stage** that deploys the container to EC2
✅ A **docker-compose.yml** for local development

Just tell me!
