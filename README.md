# n8n-build-sample
# n8n macOS 自架完整教學

## ✅ 環境需求

* macOS
* 安裝 [Docker Desktop](https://www.docker.com/products/docker-desktop/)

---

## 📁 專案資料夾結構

```
~/n8n/
├── .env
├── docker-compose.yml
└── n8n_data/   ← Docker 執行後自動建立
```

---

## 🔧 .env 設定檔

```env
GENERIC_TIMEZONE=Asia/Taipei
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=yourStrongPassword
N8N_SECURE_COOKIE=false
N8N_HOST=0.0.0.0
N8N_PORT=5678
```

---

## 🔧 docker-compose.yml

```yaml
version: '3.7'

services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - GENERIC_TIMEZONE=${GENERIC_TIMEZONE}
      - N8N_BASIC_AUTH_ACTIVE=${N8N_BASIC_AUTH_ACTIVE}
      - N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}
      - N8N_SECURE_COOKIE=${N8N_SECURE_COOKIE}
      - N8N_HOST=${N8N_HOST}
      - N8N_PORT=${N8N_PORT}
    volumes:
      - ./n8n_data:/home/node/.n8n
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G
        reservations:
          memory: 512M
```

啟動方式：

```bash
docker-compose --compatibility up -d
```

---

## 🌐 區網存取設定

查詢本機區網 IP：

```bash
ipconfig getifaddr en0
```

區網內其他人可透過：

```
http://<你的 IP>:5678
```

---

## ❗ 錯誤解法：Secure Cookie 問題

錯誤訊息：

```
Your n8n server is configured to use a secure cookie...
```

✅ 解法：在 `.env` 加上：

```env
N8N_SECURE_COOKIE=false
```

並確保 `docker-compose.yml` 有載入該變數。

---

## 📧 Gmail SMTP 設定教學（寄送通知）

### Gmail 應用程式密碼設定

1. 開啟： [https://myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)
2. 建立「郵件」+「n8n」應用
3. 複製產生的 16 碼密碼

### n8n SMTP Credential 設定範例：

| 欄位       | 值                                                   |
| -------- | --------------------------------------------------- |
| Host     | smtp.gmail.com                                      |
| Port     | 465                                                 |
| Secure   | ✅（true）                                             |
| User     | [your.email@gmail.com](mailto:your.email@gmail.com) |
| Password | 剛才的應用程式密碼                                           |

Client Host Name 可留空或填 `localhost`

---

## 🛠️ Hello World Email 工作流程 JSON

每天 9 點寄信至 `samchang@onead.com.tw`：

```json
{
  "nodes": [
    {
      "parameters": {
        "triggerTimes": {
          "item": [
            {
              "mode": "everyDay",
              "hour": 9,
              "minute": 0
            }
          ]
        }
      },
      "name": "Cron",
      "type": "n8n-nodes-base.cron",
      "typeVersion": 1,
      "position": [300, 300]
    },
    {
      "parameters": {
        "fromEmail": "your.email@gmail.com",
        "toEmail": "samchang@onead.com.tw",
        "subject": "Hello World",
        "text": "This is an automated Hello World email from n8n."
      },
      "name": "Send Email",
      "type": "n8n-nodes-base.emailSend",
      "typeVersion": 1,
      "position": [500, 300],
      "credentials": {
        "smtp": {
          "name": "Gmail SMTP"
        }
      }
    }
  ],
  "connections": {
    "Cron": {
      "main": [
        [
          {
            "node": "Send Email",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "name": "Daily Hello World Email",
  "active": false
}
```

---

## 🧠 MCP 應用實例（範例簡述）

**範例：Google Sheets 群發郵件通知 + 標記已發送**

### 流程說明：

* Cron：每日定時執行
* Google Sheets > Read：抓取名單資料
* IF：篩選 Notified 欄位為空者
* Send Email：發送通知郵件
* Google Sheets > Update：更新 Notified 欄位為 "Yes"

### Google Sheets 範例欄位：

| Name    | Email                                             | Notified |
| ------- | ------------------------------------------------- | -------- |
| Alice   | [alice@example.com](mailto:alice@example.com)     |          |
| Bob     | [bob@example.com](mailto:bob@example.com)         | Yes      |
| Charlie | [charlie@example.com](mailto:charlie@example.com) |          |

➡ 可用於活動通知、自動提醒、群發行銷等應用情境
