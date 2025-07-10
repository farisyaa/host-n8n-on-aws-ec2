# 🔧 n8n Setup on AWS EC2 (macOS)

A simple, less than 10 steps guide to set up your own [n8n](https://n8n.io) automation workspace on an AWS EC2 instance using macOS.


## 🧩 Part 1: Launch EC2 Instance on AWS

### ✅ Step 1: Create an AWS Account
Sign up at [aws.amazon.com](https://aws.amazon.com/).

### ✅ Step 2: Launch a New EC2 Instance

1. Go to EC2 Dashboard → click **Launch Instance**
2. **Name**: e.g., `n8n-server`
3. **Application and OS Image**: Select **Amazon Linux 2023 (x86_64)**
4. **Instance Type**: `t2.micro` (Free Tier eligible)
5. **Key Pair**: Create a new key pair and download it in `.pem` format
6. **Network Settings**: 
   - Add SSH (port 22)
   - Add Custom TCP (port 5678)
7. Leave everything else as default and launch the instance.

### ✅ Step 3: Connect to Your Instance via SSH (macOS)

#### 1. Set permissions for your `.pem` key:
```bash
chmod 400 ~/Downloads/my-key.pem
```

#### 2. Get your instance’s Public IPv4 or DNS from the EC2 Console to connect on terminal:
```bash
ssh -i ~/Downloads/my-key.pem ec2-user@<your-public-ip>
```

## ⚙️ Part 2: Configure n8n on EC2
### ✅ Step 1: Update the system
```bash
sudo yum update -y
```

### ✅ Step 2: Install Node.js & npm
```bash
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs

To verify command:
node -v
npm -v
```

### ✅ Step 3: Install n8n Globally
```bash
sudo npm install -g n8n
n8n --version
```

### ✅ Step 4: Test your n8n
```bash
http://<your-public-ip>:5678
```

### ✅ Step 5: Add Port 5678 to Security Group
Go to EC2 Console → Security Groups
1. Edit Inbound Rules
2. Add:
   - Type: Custom TCP
   - Port: 5678
   - Source: 0.0.0.0/0 (or limit to your IP)

### ✅ Step 6: Run n8n as a Background Service
#### 1. Create a systemd service file:
```bash
sudo nano /etc/systemd/system/n8n.service
````

#### 2. Paste
```bash
[Unit]
Description=n8n workflow automation tool
After=network.target

[Service]
ExecStart=/usr/bin/n8n
Restart=always
User=ec2-user
Environment=HOME=/home/ec2-user
Environment=DATA_FOLDER=/home/ec2-user/.n8n

[Install]
WantedBy=multi-user.target
```

#### 3. Start the service
```bash
sudo systemctl daemon-reload
sudo systemctl enable n8n
sudo systemctl start n8n
sudo systemctl status n8n
```

### ✅ Step 7: Enable HTTPS with SSL
#### 1. Install OpenSSL & generate certs
```bash
sudo yum install -y openssl
sudo mkdir /etc/n8n-certs
sudo openssl req -x509 -newkey rsa:4096 -keyout /etc/n8n-certs/n8n-key.pem -out /etc/n8n-certs/n8n-cert.pem -days 365 -nodes
```

#### 2. Set permissions
```bash
sudo chown ec2-user:ec2-user /etc/n8n-certs/n8n-*.pem
```

### ✅ Step 8: Enable HTTPS with SSL
#### 1. Edit the systemd service
```bash
sudo nano /etc/systemd/system/n8n.service
```
Replace
```bash
[Unit]
Description=n8n secure service
After=network.target

[Service]
Type=simple
User=ec2-user
ExecStart=/usr/bin/n8n
Environment=N8N_HOST=0.0.0.0
Environment=N8N_PORT=5678
Environment=N8N_PROTOCOL=https
Environment=N8N_SSL_KEY=/etc/n8n-certs/n8n-key.pem
Environment=N8N_SSL_CERT=/etc/n8n-certs/n8n-cert.pem
Environment=N8N_BASIC_AUTH_ACTIVE=true
Environment=N8N_BASIC_AUTH_USER=your_username
Environment=N8N_BASIC_AUTH_PASSWORD=your_password
Restart=always

[Install]
WantedBy=multi-user.target
```

#### 2. Restart the service
```bash
sudo systemctl daemon-reload
sudo systemctl restart n8n
```

## 🧠⚙️ You're Done!
Access your n8n securely!!!
```bash
https://<your-public-ip>:5678
```




















