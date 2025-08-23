---
title: "在 n8n 遇到 Connection lost 可能的解決方法"
description: "n8n 上的 Connection lost 的解決方法"
published: 2025-04-30
draft: false
tags: ['n8n', 'Backend']
---
# 緣起 
這幾天在研究 [n8n](https://n8n.io) 該如何使用，原本只有架設在 local ，但發現我們社團的主機有好幾臺在空著就架在了那邊

# 怎麽架的？
我用官方文件的 docker run 來跑， Port 跑在預設的 5678 沒有設Docker Proxy

# 問題來了
問題來了當我架好之後出現了 ![n8n Connection lost 錯誤 Error](/posts/images/n8n-proxy.png "錯誤截圖")

# 原因為何？
按下 F12 看了 Console 之後發現有個 
WebSocket 無法連線，推測是因為 WebSocket 沒有被 Proxy 到

# 解決方法
這裡我附上 Cloudflare Proxy 和 Nginx Proxy 上的解決方法，怕讀者也遇到這種問題
### Cloudflare
在 網路 -> 打開 WebSocket 選項（HTTP/2也要開）

### Nginx
在代理的地方加上這兩行
```
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection $connection_upgrade;
```