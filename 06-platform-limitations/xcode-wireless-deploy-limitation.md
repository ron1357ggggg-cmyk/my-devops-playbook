# Xcode Wireless Deploy Limitation

這份文件記錄 Xcode 透過 Wi-Fi 部署 iPhone / iPad App 的限制邊界。

核心結論：

```txt
Tailscale 可以建立 VPN 類似區網連線，
但不等於完整模擬 Xcode 所需的同 Wi-Fi 裝置探索與配對環境。
```

## 1. 問題場景

你可能會以為：

```txt
Mac 和 iPhone 都連上 Tailscale
=> 看起來像同一個區網
=> Xcode 應該可以遠端安裝 App
```

但實務上不一定成立。

## 2. Xcode Wi-Fi 部署通常需要的前置條件

通常需要先完成：

- iPhone 已開啟 Developer Mode
- iPhone 已信任這台 Mac
- Xcode 已成功辨識裝置
- 裝置與 Mac 曾經透過線材配對
- 裝置與 Mac 在可探索的網路環境內
- Xcode Devices and Simulators 裡可看到該裝置
- 已啟用 Connect via network / wireless debugging 類似選項

## 3. 為什麼 Tailscale 不一定可行

Tailscale 解決的是 IP 層連線問題。

但 Xcode 裝置探索與部署可能涉及：

- Bonjour / mDNS
- Apple 裝置配對狀態
- 本機網路探索
- USB 初次信任流程
- Developer Mode 狀態
- Xcode 對裝置的識別與憑證信任

因此：

```txt
能 ping 到 ≠ Xcode 能看到
能 SSH 到 Mac ≠ iPhone 能被 Xcode 部署
Tailscale 通 ≠ Apple 裝置探索完整成立
```

## 4. 建議判斷流程

### Step 1：先用線確認

```txt
Mac 接 iPhone
開 Xcode
確認 Devices and Simulators 看得到裝置
確認可以 build / run
```

### Step 2：開啟無線連線

在 Xcode 裝置管理介面確認是否有啟用 network connection / wireless debugging 類似設定。

### Step 3：拔線測試同 Wi-Fi

```txt
Mac 和 iPhone 在同一個 Wi-Fi
拔線
看 Xcode 是否仍看得到裝置
```

### Step 4：再測 Tailscale

如果同 Wi-Fi 成功，再測 Tailscale。

若 Tailscale 失敗，表示限制點可能在裝置探索與 Apple pairing，而不是單純 IP 連線。

## 5. 最小可行策略

如果目標是穩定部署到 iPhone：

1. 第一次配對與信任用 USB 線。
2. 同 Wi-Fi 環境先測成功。
3. 不要把 Tailscale 當成完整同 Wi-Fi 替代品。
4. 遠端控制 Mac 可以用 SSH / 遠端桌面。
5. 真正裝機測試時，保留實體線或同 Wi-Fi fallback。

## 6. 對 AI 下指令時要講清楚

```txt
請不要假設 Tailscale 可以完整取代 Xcode 同 Wi-Fi 部署。
請先確認：
1. iPhone 是否已與 Mac 配對
2. Developer Mode 是否開啟
3. Xcode 是否能看到裝置
4. 同 Wi-Fi 無線部署是否曾成功
5. 若不成功，請回報限制，不要硬改專案
```

## 7. 常見誤判

| 誤判 | 正確理解 |
|---|---|
| Tailscale = 同 Wi-Fi | Tailscale 是 VPN overlay，不等於所有區網探索都完整可用 |
| 能 ping 就能部署 | Xcode 還需要裝置配對與探索 |
| Claude / Codex 可以直接解 | 這是平台限制，不一定是程式碼問題 |
| 改 project setting 就會好 | 可能根本不是 Xcode project 設定問題 |

## 8. 結論

Xcode Wi-Fi 部署不是單純網路通不通，而是 Apple 裝置信任、配對、探索、開發者模式共同成立。

遇到這類問題時，先判斷平台限制，再決定是否值得讓 AI 修改專案。