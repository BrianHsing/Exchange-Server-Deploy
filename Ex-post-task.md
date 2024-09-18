# Exchange Server 安裝完成後的準備工作

## 輸入產品金鑰

- 在 Exchange 系統管理中心中，左邊功能欄選擇伺服器。在右邊內容選擇伺服器，並點選輸入產品金鑰的連結。<br>
  ![GITHUB](/TaskImages/1.png)<br>
- 將您的金鑰輸入。<br>
  ![GITHUB](/TaskImages/2.png)<br>
- 如果想透過 Exchange Management Shell 輸入也可以，完成後再重啟服務，指令如下。<br>
  ````
  Set-ExchangeServer EX2016 -ProductKey QXYKC-7H87P-XXXXX-XXXXX-XXXXX
  Restart-Service MSExchangeIS
  ````
  ![GITHUB](/TaskImages/3.png)<br>
- 完成後會提示需要重新啟動資訊儲存庫服務，重新啟動資訊儲存庫服務。<br>
  ![GITHUB](/TaskImages/4.png)<br>
- 重新整理 Exchange 系統管理中心後，就可以看到已獲授權的字樣。<br>
  ![GITHUB](/TaskImages/5.png)<br>

## 設定虛擬目錄
Exchange 存取會分內部與外部，內部會依據 Exchange 的作業系統中的電腦名稱作為 FQDN 的一部分，外部預設是空白，需要額外設定。<br>
- 在 Exchange 系統管理中心中，左邊功能欄選擇伺服器。在右邊頁籤選擇虛擬目錄，點選下方圖示設定外部存取網域。<br>
  ![GITHUB](/TaskImages/6.png)<br>
- 搭配外部 URL 使用的伺服器選擇中，點選+號，並且選擇 EX2016，最後輸入 Exchange 對外的網域名稱，完成後按確認。<br>
  ![GITHUB](/TaskImages/7.png)<br>
- 現在我們可以點開一個虛擬目錄 owa，查看是否確實有將外部網域填入。<br>
  ![GITHUB](TaskImages/8.png)<br>
## 設定外部 DNS

| FQDN |  紀錄類型   | 值  |
|  ----  | ----  | ----  |
| mail.briantw.xyz| A Record  | 1.34.xxx.xxx |
| autodiscover.briantw.xyz| CNAME Record  | mail.briantw.xyz |
| briantw.xyz| MX Record  | mail.briantw.xyz |
| briantw.xyz| TXT Record  | v=spf1 ip4:1.34.xxx.xxx ~all |
## 設定憑證
由於在 Exchange 2013 以上的版本，除了自我簽署憑證可以使用介面操作之外，其餘都會需要透過 Exchange Management Shell。<br>
- 開啟 Exchange Management Shell，並且準備好公開憑證，這個公開憑證必須要包含伺服器根憑證與私鑰，否則會無法使用。<br>
- 輸入 `Import-ExchangeCertificate -FileData ([System.IO.File]::ReadAllBytes('憑證PFX路徑')) -Password (ConvertTo-SecureString -String '密碼' -AsPlainText -Force)` 之後，可以看到憑證成功匯入的畫面。<br>
  ![GITHUB](TaskImages/9.png)<br>
- 回到 Exchange 系統管理中心中，左邊功能欄選擇伺服器。在右邊頁籤選擇憑證，可以看到剛剛匯入成功的憑證資訊。<br>
  ![GITHUB](TaskImages/10.png)<br>
- 編輯此憑證，點選服務功能，將SMTP、IIS 功能勾選，完成後儲存，這當中會出現提醒，提醒你要覆寫現有的預設 SMTP 憑證，點選是即可。<br>
   ![GITHUB](TaskImages/10-1.png)<br>
   ![GITHUB](TaskImages/11.png)<br>
- 在 Exchange Management Shell 重新啟動 IIS 服務，指令如下。<br>
  ````
  	net stop WAS
    net start W3SVC
  ````
  ![GITHUB](TaskImages/12.png)<br>
  ![GITHUB](TaskImages/13.png)<br>

- 在瀏覽器輸入 https://mail.briantw.xyz/ecp?ExchClientVer=15 驗證憑證是否有辨認到。<br>


## 郵件流程設定與用戶端存取