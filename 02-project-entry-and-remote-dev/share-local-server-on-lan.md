# Share Local Server on LAN

在同一個區域網路（LAN）內，讓你電腦上正在跑的本機開發網站（例如 IIS Express debug 站台）可以被同事的瀏覽器直接打開，不用另外部署。

它解決的問題：本機除錯用的 web app 預設只綁定 `localhost`，同網路的其他人打你的 IP 會被拒絕或被防火牆擋掉。這篇整理讓對方能連進來的最短路徑。

## 適用情境

- IIS Express / Visual Studio debug session
- 只在辦公室、家用等**信任的私有網路**臨時分享，不是對外公開服務
- 分享結束應立即關閉，不要長期開著

## 1. 找出你的區網 IP

```powershell
ipconfig
```

找到目前使用的介面（乙太網路 / Wi-Fi）底下的 `IPv4 Address`，例如 `192.168.1.100`。這個 IP 就是同事要打的網址：`http://<LAN_IP>:<PORT>/`

## 2. 確認 IIS Express binding 允許外部 host header

IIS Express 預設 binding 常是 `*:<PORT>:localhost`，只認得 host header 是 `localhost` 的請求，同事打 IP 進來會被拒絕（通常是 400 Bad Request）。

編輯專案的 `applicationhost.config`（路徑通常在 `<PROJECT_PATH>\.vs\<SOLUTION_NAME>\config\applicationhost.config`）：

```xml
<!-- 改之前 -->
<binding protocol="http" bindingInformation="*:<PORT>:localhost" />

<!-- 改之後：拿掉 host header 限制 -->
<binding protocol="http" bindingInformation="*:<PORT>:" />
```

## 3. 開防火牆 inbound 規則

```powershell
netsh advfirewall firewall add rule name="<APP_NAME> IIS Express <PORT>" dir=in action=allow protocol=TCP localport=<PORT> profile=private
```

注意 `profile`：

- 一般家用 / 辦公室 Wi-Fi 通常是 `private`
- 如果網卡的 NetworkCategory 是 `DomainAuthenticated`（公司網域環境），要把 `profile` 改成或加上 `domain`

查詢目前介面的網路類別：

```powershell
Get-NetConnectionProfile
```

## 4. 重啟站台讓新設定生效

改完 `applicationhost.config` 後，binding 不會馬上生效，需要：

- 重啟 Visual Studio 的 debug session，或
- 手動關掉 `iisexpress.exe`，讓它下次啟動時重新讀設定

```powershell
Stop-Process -Name iisexpress -Force
```

## 5. 驗證

**本機驗證（模擬同事打 IP 進來）：**

```powershell
Invoke-WebRequest -Uri "http://localhost:<PORT>/" -Headers @{Host="<LAN_IP>:<PORT>"}
```

應該回 `200`，若還是 `400` 代表 binding 沒改成功，或站台沒重啟。

**確認防火牆規則存在：**

```powershell
netsh advfirewall firewall show rule name="<APP_NAME> IIS Express <PORT>"
```

**同事端驗證：**

瀏覽器打開 `http://<LAN_IP>:<PORT>/`

## 6. 常見錯誤

### 同事打開是 400 Bad Request

- Binding 還是綁 `localhost`，沒拿掉 host header 限制（回到步驟 2）
- 站台沒重啟，舊 binding 還在跑（回到步驟 4）

### 連線逾時 / 打不開

- 防火牆規則沒生效或 profile 不對（回到步驟 3）
- 不同 VLAN / 不同子網路，實體上不通：先 `ping <LAN_IP>` 確認通不通
- 公司網路有額外的網段隔離政策，可能要找 IT 開權限

### IP 每次開機都會變

- 區網通常是 DHCP，IP 會換。若常態性分享，可考慮跟 IT 要固定 IP，或用電腦名稱 + mDNS/hosts 對應。

## 7. 安全原則

- 只在信任的私有網路上做，不要對公網（`0.0.0.0`）開放。
- 分享結束後移除防火牆規則：

  ```powershell
  netsh advfirewall firewall delete rule name="<APP_NAME> IIS Express <PORT>"
  ```

- 不要把公司內部專案代號、真實 IP、真實路徑寫進可公開的文件或 repo，一律用 placeholder（`<LAN_IP>`、`<PORT>`、`<APP_NAME>`、`<PROJECT_PATH>`）。
- 這個方法只適合臨時測試 / 展示，不是正式對外服務的做法。正式對外請走反向代理 + TLS，而不是直接開 debug port。
