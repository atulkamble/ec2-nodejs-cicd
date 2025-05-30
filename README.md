Complete Guide to **install Node.js on EC2 and configure CI/CD with GitHub** using a simple approach — ideal for projects that don’t use AWS CodePipeline but still want auto-deployment via GitHub.

References: 
1. https://ngrok.com/downloads/linux?tab=snap
2. https://dashboard.ngrok.com/signup
3. https://dashboard.ngrok.com/get-started/your-authtoken
---

## ✅ **Part 1: Launch & Connect to EC2**

1. Launch EC2 (Amazon Linux 2023 or Ubuntu)
2. Allow ports **22 (SSH)**, **3000**, and/or **80** in Security Group
3. Connect via SSH:
```bash
cd Downloads
chmod 400 nodejs.pem
ssh -i "nodejs.pem" ec2-user@ec2-54-210-164-87.compute-1.amazonaws.com
```

---

## 📦 **Part 2: Install Node.js and Git**

### For **Amazon Linux 2023**
```bash
sudo dnf update -y
sudo dnf install -y nodejs npm git
```

Check:
```bash
node -v
npm -v
git --version
```

---

## 🗂️ **Part 3: Clone GitHub Repo and Setup App**

1. Clone your app repo:
```bash
git clone https://github.com/atulkamble/ec2-nodejs-cicd.git
cd ec2-nodejs-cicd
```

2. Install dependencies:
```bash
npm init
npm install
```

3. Test your app:
```bash
node app.js
# Or whatever your start command is
```

---

## 🔁 **Part 4: Setup PM2 to Keep App Running**

```bash
sudo npm install -g pm2
pm2 start app.js     # or npm start or whatever runs your app
pm2 save
pm2 startup
```

---

## ⚙️ **Part 5: Configure CI/CD via GitHub Webhooks**

You’ll auto-deploy the app whenever a push is made to GitHub.

### 🛠️ Step 1: Setup a Deployment Script
Create a script to pull the latest code and restart the app:

```bash
nano deploy.sh
```

Paste:
```bash
#!/bin/bash
cd /home/ec2-user/your-nodejs-app
git pull origin main
npm install
pm2 restart all
```

Make it executable:
```bash
chmod +x deploy.sh
```

---

### 🌐 Step 2: Install `ngrok` or Configure Nginx + SSL (Optional for HTTPS)

For GitHub Webhooks to work, you need an HTTPS endpoint. If you don’t have a domain with SSL, you can temporarily use `ngrok`:

```bash
npm install ngrok
npm fund
npm install 

npm install nodemon-ngrok-webpack-plugin
npm fund
npm audit fix

sudo wget -O /etc/yum.repos.d/snapd.repo https://bboozzoo.github.io/snapd-amazon-linux/al2023/snapd.repo
sudo dnf install snapd -y
snap install ngrok

// sudo ngrok config add-authtoken 2v847WRCBUgH3HltVzyLizJfJOO_6Paqx88oRHn7qZqT9u8u5

sudo ngrok http 3000

// Cntrl+C
```

Use the HTTPS URL it gives for webhook in GitHub.

---

### 🔐 Step 3: Create a GitHub Webhook

1. Go to your GitHub repo → **Settings** → **Webhooks**
2. Add a webhook:
   - Payload URL: `http://your-ec2-ip:4000/webhook` (or your `ngrok` HTTPS URL)

http://54.210.164.87:4000/webhook
https://e3e6-54-210-164-87.ngrok-fr

                              
   - Content type: `application/json`
   - Secret: *(optional)*
   - Event: `Just the push event`

---

### 🧠 Step 4: Create a Webhook Listener (using Express)

Install Express:
```bash
npm install express body-parser
```

Create `webhook.js`:
```js
const express = require('express');
const bodyParser = require('body-parser');
const { exec } = require('child_process');

const app = express();
app.use(bodyParser.json());

app.post('/webhook', (req, res) => {
    console.log('GitHub Webhook triggered');
    exec('sh ~/deploy.sh', (error, stdout, stderr) => {
        if (error) {
            console.error(`Exec error: ${error}`);
            return res.sendStatus(500);
        }
        console.log(stdout);
        res.sendStatus(200);
    });
});

app.listen(4000, () => {
    console.log('Webhook listener running on port 4000');
});
```

Run it:
```bash
node webhook.js
```

You can also run this with PM2:
```bash
pm2 start webhook.js
```

---

## 🎉 That’s It!

Now every time you push code to GitHub, it will:
- Trigger the webhook
- Pull the latest code
- Reinstall dependencies (if needed)
- Restart the app with PM2

---

Would you like this as a **full GitHub repo template**, or want to use **AWS CodePipeline** instead for a managed CI/CD setup?
