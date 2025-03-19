Yes! You can use **Docker** instead of a local installation, and I’ll show you how to ensure **your AI containers always run**, with persistent data stored at **`/home/skynet/data`**.

---

## **🚀 1. Setup Docker for AI Models (Ollama, Whisper, Mistral, DeepSeek)**
Instead of installing models manually, **we'll run them inside Docker containers**.

### **🔹 Step 1: Install Docker & Docker Compose**
On your **AI Server** (Linux-based system):

```sh
sudo apt update && sudo apt install -y docker.io docker-compose
```
On **Windows/Mac**, install **Docker Desktop** from: [Docker Official Site](https://www.docker.com/).

Ensure Docker runs at boot:
```sh
sudo systemctl enable --now docker
```

---

## **🚀 2. Setup AI Models in Docker**
### **🔹 Step 2: Create a Persistent Data Directory**
We will store all **AI models and logs** under:
```sh
mkdir -p /home/skynet/data
```

---

## **📜 3. Create a Docker Compose File**
To manage all AI services efficiently, create a **Docker Compose file** (`docker-compose.yml`) in `/home/skynet/data`:

```yaml
version: "3.8"

services:
  ollama:
    image: ollama/ollama
    container_name: ollama-ai
    restart: always
    ports:
      - "11434:11434"
    volumes:
      - /home/skynet/data/ollama:/root/.ollama

  whisper:
    image: ghcr.io/ggerganov/whisper.cpp:latest
    container_name: whisper-stt
    restart: always
    ports:
      - "9000:9000"
    volumes:
      - /home/skynet/data/whisper:/data
    command: ["/bin/sh", "-c", "./server -m models/ggml-base.en.bin -p 9000"]

  nginx:
    image: nginx:latest
    container_name: nginx-proxy
    restart: always
    ports:
      - "8000:8000"
    volumes:
      - /home/skynet/data/nginx:/etc/nginx/conf.d
```

📌 **Explanation**:
- ✅ **Ollama** runs AI models like **DeepSeek & Mistral**.
- ✅ **Whisper STT** transcribes speech and serves **STT at `http://YOUR_AI_SERVER_IP:9000`**.
- ✅ **Nginx** is optional for **proxying authentication**.

---

## **🚀 4. Pull AI Models & Run Containers**
### **🔹 Step 1: Pull & Prepare AI Models**
Before running, **download the AI models into Ollama**:

```sh
docker run --rm -v /home/skynet/data/ollama:/root/.ollama ollama/ollama pull deepseek-chat
docker run --rm -v /home/skynet/data/ollama:/root/.ollama ollama/ollama pull mistral
```
For **smaller models** (faster on CPUs):
```sh
docker run --rm -v /home/skynet/data/ollama:/root/.ollama ollama/ollama pull deepseek-chat:Q4_K_M
docker run --rm -v /home/skynet/data/ollama:/root/.ollama ollama/ollama pull mistral:Q4_K_M
```

### **🔹 Step 2: Start the Containers**
Run all containers using:
```sh
cd /home/skynet/data
docker-compose up -d
```

**📌 What happens?**
- Ollama AI **(DeepSeek, Mistral)** runs on **port 11434**.
- Whisper **(Speech-to-Text)** runs on **port 9000**.
- Nginx (Optional) **secures access**.

✅ **All AI models run persistently and restart automatically**.

---

## **🚀 5. Configure Home Assistant to Use Remote AI Server**
Now, set **Home Assistant** to send **AI requests** to the remote machine.

Edit **`configuration.yaml`**:

```yaml
assist_pipeline:
  - name: "DeepSeek Assistant (Remote)"
    language: "en"
    conversation_agent:
      type: "custom"
      url: "http://YOUR_AI_SERVER_IP:11434/api/generate"
      model: "deepseek-chat"

stt:
  - platform: custom
    url: "http://YOUR_AI_SERVER_IP:9000"

tts:
  - platform: google_translate
```
📌 **What This Does:**
- 🔹 **Home Assistant connects to DeepSeek** for AI responses.
- 🔹 **Speech-to-Text (STT) uses Whisper** running on the remote machine.

---

## **🚀 6. Automate AI Model Management**
### **🔹 Step 1: Ensure Containers Restart on Boot**
```sh
sudo systemctl enable docker
```

### **🔹 Step 2: Auto-Pull AI Updates**
Create a **cron job** to check for model updates every day:
```sh
crontab -e
```
Add:
```
0 3 * * * docker-compose pull && docker-compose up -d
```
📌 **This updates models at 3 AM daily**.

---

## **🚀 7. Secure AI Services**
### **🔹 Step 1: Firewall Rules**
Allow only **Home Assistant** to access AI services:
```sh
sudo ufw allow from YOUR_HOME_ASSISTANT_IP to any port 11434
sudo ufw allow from YOUR_HOME_ASSISTANT_IP to any port 9000
```
Deny everything else:
```sh
sudo ufw deny 11434
sudo ufw deny 9000
```

### **🔹 Step 2: Add Authentication (Optional)**
If you want to **require authentication**, modify Nginx (`/home/skynet/data/nginx/ollama.conf`):

```nginx
server {
    listen 8000;
    location / {
        proxy_pass http://localhost:11434;
        auth_basic "Restricted";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
```
Create a password file:
```sh
sudo apt install apache2-utils
sudo htpasswd -c /home/skynet/data/nginx/.htpasswd homeassistant
```
Restart Nginx:
```sh
docker restart nginx-proxy
```
📌 **Now Home Assistant must authenticate before using AI.**

---

## **🎯 Final Summary**
✅ **Run AI models in Docker containers** for **better management & persistence**.  
✅ **Data stored in `/home/skynet/data`** for easy access.  
✅ **Home Assistant connects to AI server over HTTP**.  
✅ **Firewall & authentication secure AI access**.

---

### **🚀 Next Steps**
Would you like a **debugging script** to test Home Assistant’s connection? 🔥
