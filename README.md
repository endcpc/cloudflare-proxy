# Cloudflare Proxy

> 基於 Cloudflare Workers 的全功能 HTTP/HTTPS 代理服務

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://sink.proddig.com/cloudflare-proxy-github)

## 特性

- 🌐 **多種訪問方式** - Web 介面、查詢參數、路徑方式、標準 HTTP 代理
- 🔒 **HTTPS 支援** - 完整支援 HTTPS 網站代理
- 🔄 **智能重新導向** - 自動處理 301/302 等重新導向
- 🛠️ **路徑修復** - 自動修復 HTML 中的相對路徑
- 🌍 **CORS 支援** - 完整的跨域資源共享支援
- 📱 **響應式設計** - 完美適配行動端和桌面端
- ⚡ **零成本運行** - Cloudflare Workers 免費版每天 10 萬次請求

## 页面展示

![screenshot](./screenshot.png)

## 安装方式

### 方式一：一鍵部署（推薦）

點擊上方 "Deploy to Cloudflare Workers" 按鈕，按照提示完成部署。

### 方式二：使用 Wrangler CLI

```bash
# 1. 安裝 Wrangler
npm install -g wrangler

# 2. 登入 Cloudflare
wrangler login

# 3. 複製倉庫
git clone https://github.com/Yrobot/cloudflare-proxy.git
cd cloudflare-proxy

# 4. 部署
wrangler deploy
```

### 方式三：使用 Cloudflare Dashboard

1. 登入 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 進入 **Workers & Pages**
3. 點擊 **Create Application** > **Create Worker**
4. 將 `worker.js` 的內容複製粘貼到編輯器
5. 點擊 **Save and Deploy**

### 獲取訪問地址

部署成功後，你會獲得一個 Worker URL：

```
https://your-worker-name.your-subdomain.workers.dev
```

**注意** 這個自帶的網域名 Many 區域是無法直接訪問的，建議綁定一個自己的網域名使用

## 使用方式

### 方式 1: Web 界面

直接訪問你的 Worker URL，在網頁界面輸入目標網址：

```
https://$YOUR-PROXY-DOMAIN/
```

### 方式 2: 查詢參數

在 URL 後添加 `?url=` 參數：

```bash
https://$YOUR-PROXY-DOMAIN/?url=https://example.com
```

### 方式 3: 路徑方式

直接在路徑中指定目標網址：

```bash
# 完整 URL（帶協定）
https://$YOUR-PROXY-DOMAIN/https://example.com

# 簡寫（自動添加 https://）
https://$YOUR-PROXY-DOMAIN/example.com
```

### 方式 4: HTTP 代理

設定系統代理，適用於命令列工具：

```bash
# Linux/macOS
export HTTP_PROXY=https://$YOUR-PROXY-DOMAIN
export HTTPS_PROXY=https://$YOUR-PROXY-DOMAIN

# Windows (PowerShell)
$env:HTTP_PROXY="https://$YOUR-PROXY-DOMAIN"
$env:HTTPS_PROXY="https://$YOUR-PROXY-DOMAIN"

# 使用代理訪問
curl https://api.github.com
```

## 使用場景

### 1. GitHub 文件加速

加速 raw.githubusercontent.com 文件下載：

```bash
# 原始地址（可能很慢）
https://raw.githubusercontent.com/user/repo/main/file.txt

# 使用代理（加速訪問）
https://$YOUR-PROXY-DOMAIN/https://raw.githubusercontent.com/user/repo/main/file.txt
```

### 2. Docker 鏡像加速

配置 Docker 鏡像代理源：

```bash
# 在 /etc/docker/daemon.json 中配置
{
  "registry-mirrors": [
    "https://$YOUR-PROXY-DOMAIN/https://registry-1.docker.io"
  ]
}

# 重啟 Docker
sudo systemctl restart docker
```

### 3. OpenAI API 代理

代理 OpenAI API 請求：

```javascript
// 設定代理基礎 URL
const openai = new OpenAI({
  baseURL: "https://$YOUR-PROXY-DOMAIN/https://api.openai.com/v1",
  apiKey: "your-api-key",
});

// 或使用 fetch
fetch("https://$YOUR-PROXY-DOMAIN/https://api.openai.com/v1/chat/completions", {
  method: "POST",
  headers: {
    Authorization: "Bearer your-api-key",
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "gpt-3.5-turbo",
    messages: [{ role: "user", content: "Hello!" }],
  }),
});
```

### 4. 前端 CORS 代理

解決前端跨域問題：

```javascript
// 直接訪問會遇到 CORS 錯誤
fetch("https://api.example.com/data")
  .then((res) => res.json())
  .then((data) => console.log(data));

// 使用代理解決 CORS
fetch("https://$YOUR-PROXY-DOMAIN/https://api.example.com/data")
  .then((res) => res.json())
  .then((data) => console.log(data));
```

### 5. CI/CD 環境

在 GitHub Actions 中使用：

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    env:
      HTTP_PROXY: https://$YOUR-PROXY-DOMAIN
      HTTPS_PROXY: https://$YOUR-PROXY-DOMAIN
    steps:
      - name: Run tests
        run: npm test
```

## 注意與提醒

### 🚨 重要提示

1. **使用自定網域名**

   - Cloudflare 預設的 `*.workers.dev` 網域名在某些地區可能無法訪問
   - **強烈建議**綁定自己的網域名以獲得更好的訪問體驗
   - 在 Worker 設定中點擊 **Triggers** > **Add Custom Domain** 增加自定網域名

2. **關於請求頭**

   - 代理會轉發所有請求頭和內容

3. **訪問限制**
   - Cloudflare Workers 免費版每天 10 萬次請求
   - 單次請求不能超過 100MB
   - CPU 時間限制：免費版 10ms，付費版 50ms

### 🔒 安全設定

#### 增加 API Key 驗證

```javascript
export default {
  async fetch(request) {
    // 檢查 API Key
    const apiKey = request.headers.get("X-API-Key");
    if (apiKey !== "your-secret-key") {
      return new Response("Unauthorized", { status: 401 });
    }

    // 原有邏輯...
  },
};
```

## 免責聲明

本專案僅供學習和研究使用，使用者需遵守以下規定：

1. **合法使用** - 僅用於訪問合法內容，不得用於訪問違法或侵權內容
2. **服務條款** - 使用時需遵守 Cloudflare Workers 服務條款
3. **責任自負** - 使用本代理產生的任何後果由使用者自行承擔
4. **商業用途** - 如需商業使用，請確保符合相關法規
5. **隱私保護** - 建議不要通過代理傳輸個人敏感信息

## 常见问题

### Q: 為什麼有些網站無法訪問？

A: 可能原因：

- 網站有防爬蟲機制，檢測到了代理訪問
- 網站使用了 WebSocket 等 Cloudflare Workers 不支持的協議
- 超過了請求大小或時間限制

### Q: 如何提高訪問速度？

A: 建議：

- 使用自定網域名，選擇離用戶更近的 DNS 伺服器
- 對於靜態資源，考慮使用 Cloudflare 的快取功能

### Q: 可以代理 WebSocket 嗎？

A: 不可以。Cloudflare Workers 目前不支持 WebSocket 連接。

## 贡献

欢迎提交 Issue 和 Pull Request！

## 许可证

[GPL-3 License](LICENSE)

## 相关链接

- [Cloudflare Workers 文档](https://developers.cloudflare.com/workers/)
- [项目 GitHub](https://github.com/Yrobot/cloudflare-proxy)

---

**如果这个项目对你有帮助，请在 [GitHub](https://github.com/Yrobot/cloudflare-proxy) 上给我们一个 ⭐️**
