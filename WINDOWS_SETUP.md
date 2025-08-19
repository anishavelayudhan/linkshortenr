# LinkShortenr Windows Setup Guide

This guide will help you set up your link shortener application on a Windows machine using the exact same configuration as your Mac setup.

## 🎯 What We're Setting Up

- **Spring Boot Backend**: Java application running on port 8080
- **PostgreSQL Database**: Database running on port 5433
- **Nginx Reverse Proxy**: Running on port 80 (standard HTTP)
- **Custom Domain**: `linkshortenr.anisha.cc` pointing to your local machine

## 📋 Prerequisites

### Required Software
- [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)
- [Git for Windows](https://git-scm.com/download/win)
- [Visual Studio Code](https://code.visualstudio.com/) (optional, for editing files)

### System Requirements
- Windows 10/11 (64-bit)
- At least 4GB RAM
- Docker Desktop enabled
- WSL2 enabled (Docker Desktop will handle this)

## 🚀 Step-by-Step Setup

### 1. Install Docker Desktop

1. Download Docker Desktop from [docker.com](https://www.docker.com/products/docker-desktop/)
2. Run the installer and follow the setup wizard
3. **Important**: Enable WSL2 when prompted
4. Restart your computer if required
5. Start Docker Desktop and wait for it to be ready (green icon in system tray)

### 2. Clone Your Project

Open Command Prompt or PowerShell as Administrator and run:

```cmd
cd C:\Users\%USERNAME%\Documents
git clone https://github.com/yourusername/linkshortenr.git
cd linkshortenr
```

### 3. Create Environment File

Create a `.env` file in your project root with these exact settings:

```bash
DB_HOST=localhost
DB_PORT=5433
DB_NAME=link_shortener
DB_USERNAME=myuser
DB_PASSWORD=mypassword123
FRONTEND_URL=https://linkshortenr.vercel.app
API_BASE_URL=https://linkshortenr.anisha.cc/api
```

**Save this file as `.env` (no file extension) in your project root.**

### 4. Update Windows Hosts File

1. Open Notepad as Administrator
2. Open file: `C:\Windows\System32\drivers\etc\hosts`
3. Add this line at the end:
   ```
   127.0.0.1    linkshortenr.anisha.cc
   ```
4. Save and close the file

### 5. Install Nginx for Windows

#### Option A: Using Chocolatey (Recommended)
1. Install Chocolatey first: [chocolatey.org](https://chocolatey.org/install)
2. Open PowerShell as Administrator and run:
   ```powershell
   choco install nginx
   ```

#### Option B: Manual Download
1. Download Nginx for Windows from [nginx.org](http://nginx.org/en/download.html)
2. Extract to `C:\nginx`
3. Add `C:\nginx` to your system PATH

### 6. Configure Nginx

1. Navigate to your Nginx config directory:
   - **Chocolatey**: `C:\tools\nginx\conf`
   - **Manual**: `C:\nginx\conf`

2. **Backup the original config:**
   ```cmd
   copy nginx.conf nginx.conf.backup
   ```

3. **Replace the entire `nginx.conf` content with:**

```nginx
#user  nobody;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include sites-enabled/*.conf;
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    # Commented out default server block to avoid conflicts
    # server {
    #     listen       8080;
    #     server_name  localhost;
    #     location / {
    #         root   html;
    #         index  index.html index.htm;
    #     }
    # }

    include servers/*;
}
```

4. **Create the sites directory structure:**
   ```cmd
   mkdir sites-available
   mkdir sites-enabled
   ```

5. **Create `sites-available\linkshortenr.conf`:**

```nginx
server {
    listen 80;
    server_name linkshortenr.anisha.cc;

    # Forward all API requests to your local Spring Boot app
    location /api/ {
        proxy_pass http://localhost:8080/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # CORS headers
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
        add_header Access-Control-Allow-Headers "Content-Type, Authorization";
        
        # Handle preflight requests
        if ($request_method = 'OPTIONS') {
            add_header Access-Control-Allow-Origin *;
            add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
            add_header Access-Control-Allow-Headers "Content-Type, Authorization";
            add_header Content-Length 0;
            add_header Content-Type text/plain;
            return 200;
        }
    }

    # Health check endpoint
    location /health {
        proxy_pass http://localhost:8080/actuator/health;
        proxy_set_header Host $host;
    }

    # Root endpoint
    location / {
        return 200 "LinkShortenr API is running!";
        add_header Content-Type text/plain;
    }
}
```

6. **Create the symlink:**
   ```cmd
   mklink sites-enabled\linkshortenr.conf sites-available\linkshortenr.conf
   ```

### 7. Start Your Application

1. **Open Command Prompt in your project directory:**
   ```cmd
   cd C:\Users\%USERNAME%\Documents\linkshortenr
   ```

2. **Start the backend and database:**
   ```cmd
   docker-compose up --build -d
   ```

3. **Verify containers are running:**
   ```cmd
   docker-compose ps
   ```

   You should see:
   - `linkshortenr_backend` running on port 8080
   - `linkshortenr_db` running on port 5433

### 8. Start Nginx

1. **Test Nginx configuration:**
   ```cmd
   nginx -t
   ```

2. **Start Nginx:**
   ```cmd
   nginx
   ```

3. **If Nginx is already running, reload it:**
   ```cmd
   nginx -s reload
   ```

## 🧪 Testing Your Setup

### Test Backend Directly
```cmd
curl -I http://localhost:8080/actuator/health
```
Expected: `HTTP/1.1 200`

### Test Nginx Proxy
```cmd
curl -I http://localhost:80
```
Expected: `HTTP/1.1 200`

### Test Custom Domain
```cmd
curl http://linkshortenr.anisha.cc
```
Expected: `LinkShortenr API is running!`

### Test Health Endpoint
```cmd
curl http://linkshortenr.anisha.cc/health
```
Expected: `{"status":"UP"}`

## 🔧 Troubleshooting

### Port Already in Use
If you get "port already in use" errors:

1. **Check what's using the port:**
   ```cmd
   netstat -ano | findstr :8080
   netstat -ano | findstr :80
   ```

2. **Kill the process:**
   ```cmd
   taskkill /PID <PID_NUMBER> /F
   ```

### Docker Issues
1. **Restart Docker Desktop** if containers won't start
2. **Check Docker logs:**
   ```cmd
   docker-compose logs backend
   docker-compose logs postgres
   ```

### Nginx Issues
1. **Check Nginx error logs:**
   ```cmd
   tail -f C:\nginx\logs\error.log
   ```

2. **Restart Nginx:**
   ```cmd
   nginx -s stop
   nginx
   ```

## 🌐 Production Deployment

### Option 1: Keep on Windows Mini PC
1. **Configure Windows Firewall** to allow ports 80 and 8080
2. **Set up port forwarding** on your router (port 80 → your mini PC)
3. **Update DNS** to point `linkshortenr.anisha.cc` to your home IP

### Option 2: Deploy to Cloud
1. **Use the same Docker setup** on a cloud server
2. **Point your domain DNS** to the cloud server IP
3. **Configure SSL certificates** for HTTPS

## 📁 File Structure

Your Windows setup should look like this:
```
C:\Users\%USERNAME%\Documents\linkshortenr\
├── .env                          # Environment variables
├── docker-compose.yml            # Docker services
├── backend\                      # Spring Boot application
│   ├── Dockerfile
│   ├── pom.xml
│   └── src\
├── frontend\                     # React/Next.js frontend
└── WINDOWS_SETUP.md             # This file
```

## 🎉 Success!

Once everything is working, you'll have:
- ✅ **Backend API**: `http://localhost:8080`
- ✅ **Database**: PostgreSQL on port 5433
- ✅ **Reverse Proxy**: Nginx on port 80
- ✅ **Custom Domain**: `http://linkshortenr.anisha.cc`
- ✅ **API Endpoint**: `http://linkshortenr.anisha.cc/api/*`

## 📞 Need Help?

If you encounter issues:
1. Check the troubleshooting section above
2. Verify all ports are free
3. Ensure Docker Desktop is running
4. Check that all configuration files are correct

Your link shortener is now ready to run on Windows! 🚀
