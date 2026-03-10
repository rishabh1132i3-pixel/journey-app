# 🛤️ Jerney — Blog Platform

A Gen-Z vibe blog platform built with a 3-tier architecture. Write posts, drop comments, and share your hot takes with the world.

![Tech Stack](https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react)
![Tech Stack](https://img.shields.io/badge/Node.js-20-339933?style=flat-square&logo=node.js)
![Tech Stack](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat-square&logo=postgresql)
![Tech Stack](https://img.shields.io/badge/Nginx-Latest-009639?style=flat-square&logo=nginx)

## ✨ Features

- 📝 **Create** blog posts with emoji vibes
- ✏️ **Edit** your existing posts
- 🗑️ **Delete** posts you're not feeling anymore
- 💬 **Comment** on posts — interact with the community
- 🎨 **Gen-Z UI** — dark mode, glass morphism, gradient vibes

## 🏗️ Architecture

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│   Frontend   │────▶│   Backend    │────▶│  Database    │
│   (React)    │◀────│  (Node.js)   │◀────│ (PostgreSQL) │
│   Port 80    │     │  Port 5000   │     │  Port 5432   │
└─────────────┘     └──────────────┘     └──────────────┘
     Nginx            Express API           pg driver
```

## 📁 Project Structure

```
Jerney/
├── frontend/            # React (Vite) frontend
│   ├── src/
│   │   ├── components/  # Navbar, PostCard, CommentSection, ConfirmModal
│   │   ├── pages/       # Home, PostDetail, CreatePost, EditPost
│   │   ├── api.js       # Axios API client
│   │   ├── App.jsx      # Router setup
│   │   ├── main.jsx     # Entry point
│   │   └── index.css    # Gen-Z styles
│   ├── index.html
│   ├── vite.config.js
│   └── package.json
├── backend/             # Node.js Express API
│   ├── src/
│   │   ├── index.js     # Express server entry
│   │   ├── db.js        # PostgreSQL connection + table init
│   │   └── routes/
│   │       ├── posts.js     # CRUD for posts
│   │       └── comments.js  # CRUD for comments
│   ├── .env
│   └── package.json
├── deploy/              # Deployment configs
│   ├── setup.sh         # Automated EC2 setup script
│   └── jerney-nginx.conf # Nginx configuration
├── .gitignore
└── README.md
```

---

## 🚀 Deploy on AWS EC2 (Step-by-Step)

### Step 1: Launch an EC2 Instance

1. Log into the **AWS Console** → Go to **EC2** → Click **Launch Instance**
2. Configure:
   - **Name**: `Jerney-Blog`
   - **AMI**: Ubuntu Server 22.04 LTS (or 24.04 LTS)
   - **Instance type**: `t2.micro` (free tier) or `t2.small`
   - **Key pair**: Create or select an existing key pair (e.g., `jerney-key.pem`)
   - **Security Group** — Allow inbound rules:
     | Type  | Port | Source    |
     |-------|------|-----------|
     | SSH   | 22   | Your IP   |
     | HTTP  | 80   | 0.0.0.0/0|
3. Click **Launch Instance**

### Step 2: Connect to Your EC2 Instance

```bash
# Make your key file secure
chmod 400 jerney-key.pem

# Connect via SSH
ssh -i jerney-key.pem ubuntu@<YOUR_EC2_PUBLIC_IP>
```

### Step 3: Transfer Project Files to EC2

**Option A: Using SCP (from your local machine)**

```bash
# From your local machine, run:
scp -i jerney-key.pem -r /path/to/Jerney ubuntu@<YOUR_EC2_PUBLIC_IP>:~/Jerney
```

**Option B: Using Git**

```bash
# On EC2 instance:
sudo apt update && sudo apt install -y git
git clone <YOUR_REPO_URL> ~/Jerney
```

### Step 4: Run the Automated Setup Script

```bash
# On EC2 instance:
cd ~/Jerney
chmod +x deploy/setup.sh
./deploy/setup.sh
```

This script will automatically:
- ✅ Install Node.js 20.x
- ✅ Install and configure PostgreSQL
- ✅ Create the database and user
- ✅ Install and configure Nginx as a reverse proxy
- ✅ Install PM2 (process manager)
- ✅ Build the React frontend
- ✅ Start the backend server
- ✅ Set up everything to auto-start on reboot

### Step 5: Access Your Blog

Open your browser and go to:

```
http://<YOUR_EC2_PUBLIC_IP>
```

That's it! 🎉 Jerney is live.

---

## 🔧 Manual Setup (If You Prefer Step-by-Step)

If you'd rather set things up manually instead of running the script:

### 1. Install Dependencies

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Node.js 20.x
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Install PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# Install Nginx
sudo apt install -y nginx

# Install PM2
sudo npm install -g pm2
```

### 2. Configure PostgreSQL

```bash
sudo -u postgres psql
```

In the PostgreSQL shell:

```sql
CREATE USER jerney_user WITH PASSWORD 'jerney_pass_2026';
CREATE DATABASE jerney_db OWNER jerney_user;
GRANT ALL PRIVILEGES ON DATABASE jerney_db TO jerney_user;
\c jerney_db
GRANT ALL ON SCHEMA public TO jerney_user;
\q
```

### 3. Set Up the Backend

```bash
cd ~/Jerney/backend

# Install dependencies
npm install --production

# Verify .env file has correct settings:
cat .env
# PORT=5000
# DB_USER=jerney_user
# DB_PASSWORD=jerney_pass_2026
# DB_HOST=localhost
# DB_PORT=5432
# DB_NAME=jerney_db

# Start with PM2
pm2 start src/index.js --name jerney-backend
pm2 save
```

### 4. Build the Frontend

```bash
cd ~/Jerney/frontend
npm install
npm run build
```

### 5. Set Up the Project Directory

```bash
sudo mkdir -p /var/www/jerney
sudo cp -r ~/Jerney/* /var/www/jerney/
sudo chown -R $USER:$USER /var/www/jerney
```

### 6. Configure Nginx

```bash
sudo cp ~/Jerney/deploy/jerney-nginx.conf /etc/nginx/sites-available/jerney
sudo ln -sf /etc/nginx/sites-available/jerney /etc/nginx/sites-enabled/jerney
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl restart nginx
sudo systemctl enable nginx
```

### 7. Enable Auto-Start on Reboot

```bash
pm2 startup systemd -u $USER --hp /home/$USER
# Copy and run the command it outputs
pm2 save
```

---

## 🛠️ Useful Commands

| Command | Description |
|---------|-------------|
| `pm2 status` | Check if backend is running |
| `pm2 logs` | View backend logs in real-time |
| `pm2 restart jerney-backend` | Restart the backend |
| `pm2 stop jerney-backend` | Stop the backend |
| `sudo systemctl restart nginx` | Restart Nginx |
| `sudo systemctl status nginx` | Check Nginx status |
| `sudo -u postgres psql -d jerney_db` | Access the database |

## 🔒 Security Recommendations (Production)

1. **Change the database password** in `backend/.env` and PostgreSQL
2. **Set up HTTPS** with Let's Encrypt:
   ```bash
   sudo apt install certbot python3-certbot-nginx
   sudo certbot --nginx -d yourdomain.com
   ```
3. **Use environment variables** instead of `.env` files
4. **Restrict SSH access** to your IP only in the security group
5. **Enable UFW firewall**:
   ```bash
   sudo ufw allow OpenSSH
   sudo ufw allow 'Nginx Full'
   sudo ufw enable
   ```

## 🐛 Troubleshooting

### Backend won't start
```bash
# Check logs
pm2 logs jerney-backend

# Verify PostgreSQL is running
sudo systemctl status postgresql

# Test database connection
sudo -u postgres psql -d jerney_db -c "SELECT 1;"
```

### Frontend shows blank page
```bash
# Ensure build completed successfully
ls -la /var/www/jerney/frontend/dist/

# Check Nginx config
sudo nginx -t
sudo tail -20 /var/log/nginx/error.log
```

### Can't connect to the site
```bash
# Verify Nginx is running
sudo systemctl status nginx

# Check if backend is running
pm2 status

# Verify EC2 security group allows port 80
```

### Database connection refused
```bash
# Check PostgreSQL status
sudo systemctl status postgresql

# Verify user can connect
psql -U jerney_user -d jerney_db -h localhost -c "SELECT 1;"
```

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Health check |
| GET | `/api/posts` | Get all posts |
| GET | `/api/posts/:id` | Get single post with comments |
| POST | `/api/posts` | Create a new post |
| PUT | `/api/posts/:id` | Update a post |
| DELETE | `/api/posts/:id` | Delete a post |
| GET | `/api/comments/post/:postId` | Get comments for a post |
| POST | `/api/comments` | Create a comment |
| DELETE | `/api/comments/:id` | Delete a comment |

---

Built with 💜 by the Jerney team. No cap, this blog platform hits different. 🛤️
