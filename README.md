# Exchange Server 安裝紀錄

## 前言

Exchange Server 安裝雖然有精靈可以快速地完成，可是在真實環境下，會遭遇相容性版本與原先已知問題的因素，導致無法順利安裝，所以為了讓自己每次能夠安裝時能夠標準化，就花一些時間把安裝步驟寫下來吧。<br>

在安裝之前，可以先了解一下可能會踩的地雷：<br>

- 確認要安裝的 Exchange Server 與 Windows Server 以及 AD 樹系相容性<br>
- 確認要安裝的 Exchange Server 版本與現有 Exchange Server 版本的相容性<br>
- 安裝必要條件，例如 Windows Feature、NET Framework、Visual C++ 可轉散發套件、IIS URL Rewrite 模組、UCMA 4.0、TLS 1.2 啟用等等，這個會依據安裝的 Exchange Server 版本而有所不同<br>
- 檢查要安裝 Exchange Server 的 Window Server 是否可以確實連線到 DC，所以可以透過 nltest /dsetdc 或 nltest /dsgetsite 來檢查。<br>
- Exchange Server 2013 開始已經將 GUI 介面的匯入憑證功能給拔除，所以一律都使用 Powershell 進行匯入，匯入憑證的指令可以參考`Import-ExchangeCertificate -FileData ([System.IO.File]::ReadAllBytes('\\伺服器名稱\檔案資料夾\憑證名稱.pfx')) -Password (ConvertTo-SecureString -String '憑證PFX密碼' -AsPlainText -Force)`，這個憑證必須要有私鑰與根憑證才能正常使用，不然會無法使用<br>
- 內部的 Autodiscover 常常會被忽略，導致憑證放進去的時候，使用者端會出現安全性提示，通常會建議內外設定一致，會比較好管理，會使用這一段指令進行設定`Set-ClientAccessService -Identity EX伺服器名稱 -AutoDiscoverServiceInternalUri https://與外部一致的FQDN/Autodiscover/Autodiscover.xml`<br>
- 後續的步驟大致上就是要建立離線通訊錄、設定內外虛擬目錄URL、設定連接器、指派 Exchange 授權等。<br>

## 安裝 Exchange Server 步驟

- [Exchange Server 2016](/Exchange2016/ex2016.md) <br>
- Exchange Server 2019 <br>

## 超有用連結

- [Exchange 部署助手](https://setup.cloud.microsoft/exchange/deployment-assistant)<br>
