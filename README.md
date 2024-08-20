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

## 常用的 Exchange 指令

- 憑證匯入<br>
````
Import-ExchangeCertificate -FileData ([System.IO.File]::ReadAllBytes('\\FileServer01\Data\Fabrikam.pfx')) -Password (ConvertTo-SecureString -String 'P@ssw0rd1' -AsPlainText -Force)
````

- 混合式公用資料夾信箱<br>
````
Set-OrganizationConfig -PublicFoldersEnabled Remote -RemotePublicFolderMailboxes Mailbox1
````

移轉資料庫路徑
Dismount-Database "DBNAME"
Move-DatabasePath  -Identity "DBNAME" -LogFolderPath "D:\MailDB\DBNAME" -EdbFilePath "D\MailDB\DBNAME\DBNAME.edb"
Mount-Database "DBNAME"

移轉系統Log
Set-FrontEndTransportService EX2016 -ReceiveProtocolLogPath D:\MailLogs\FrontEnd\ProtocolLog\SmtpReceive -SendProtocolLogPath D:\MailLogs\FrontEnd\ProtocolLog\SmtpSend -ConnectivityLogPath D:\MailLogs\FrontEnd\Connectivity -RoutingTableLogPath D:\MailLogs\FrontEnd\Routing -AgentLogPath D:\MailLogs\FrontEnd\AgentLog

Set-MailboxTransportService EX2016 -ConnectivityLogPath D:\MailLogs\Mailbox\Connectivity -ReceiveProtocolLogPath D:\MailLogs\Mailbox\ProtocolLog\SmtpReceive -SendProtocolLogPath D:\MailLogs\Mailbox\ProtocolLog\SmtpSend -RoutingTableLogPath D:\MailLogs\Mailbox\Routing -PipelineTracingPath D:\MailLogs\Mailbox\PipelineTracing -MailboxSubmissionAgentLogPath D:\MailLogs\Mailbox\AgentLog\Submission -MailboxDeliveryAgentLogPath D:\MailLogs\Mailbox\AgentLog\Delivery -MailboxDeliveryThrottlingLogPath D:\MailLogs\Throttling\Delivery

Set-TransportService EX2016 -IrmLogPath D:\MailLogs\IRMLogs -ActiveUserStatisticsLogPath D:\MailLogs\Hub\ActiveUsersStats -ServerStatisticsLogPath D:\MailLogs\Hub\ServerStats -PickupDirectoryPath D:\TransportRoles\Pickup -PipelineTracingPath D:\MailLogs\Hub\PipelineTracing -ReplayDirectoryPath D:\TransportRoles\Replay -RoutingTableLogPath D:\MailLogs\Hub\Routing -QueueLogPath D:\MailLogs\Hub\QueueViewer -WlmLogPath D:\MailLogs\WLM -AgentLogPath D:\MailLogs\Hub\AgentLog -TransportHttpLogPath D:\MailLogs\Hub\TransportHttp

批次System Mailbox移轉
Get-Mailbox -Database MailDB01 -Arbitration | New-MoveRequest -TargetDatabase DB01 -BatchName SystemMailboxToDB01
Get-MoveRequest -BatchName SystemMailboxToDB01
Get-MigrationUserStatistics -BatchId SystemMailboxToDB01

Discovery Mailbox 移轉
Get-Mailbox -RecipientTypeDetails DiscoveryMailbox | New-MoveRequest -TargetDatabase DB01

建立離線通訊錄
New-OfflineAddressBook -Name "OAB2016" -AddressLists "Default Global Address List" -GlobalWebDistributionEnabled $false

設定內部Autodiscover
Get-ClientAccessServer | FL id*,auto*
Set-ClientAccessServer -Identity EX2013 -AutoDiscoverServiceInternalURI 
Set-ClientAccessService -Identity EX2016 -AutoDiscoverServiceInternalURI 
設定內部
Set-ClientAccessService -Identity Internal_Hostname -AutoDiscoverServiceInternalUri https://External_FQDN/Autodiscover/Autodiscover.xml

Set-ClientAccessService -Identity EX01 -AutoDiscoverServiceInternalUri https://mail.brianhsing.store/Autodiscover/Autodiscover.xml
Set-ClientAccessService -Identity EX02 -AutoDiscoverServiceInternalUri https://mail.brianhsing.store/Autodiscover/Autodiscover.xml



內外虛擬目錄URL
$Server = "EX2016"
$HTTPS_FQDN = "EX2016.adtest.com.tw"
Get-OWAVirtualDirectory -Server $Server | Set-OWAVirtualDirectory -ExternalURL $null
Get-ECPVirtualDirectory -Server $Server | Set-ECPVirtualDirectory -ExternalURL $null
Get-OABVirtualDirectory -Server $Server | Set-OABVirtualDirectory -ExternalURL $null
Get-ActiveSyncVirtualDirectory -Server $Server | Set-ActiveSyncVirtualDirectory -ExternalURL $null
Get-WebServicesVirtualDirectory -Server $Server | Set-WebServicesVirtualDirectory -ExternalURL $null
Enable-OutlookAnywhere -Server $Server -ClientAuthenticationMethod Basic -SSLOffloading $False -ExternalHostName $HTTPS_FQDN

更新Addresslist
Get-AddressList | Update-AddressList -vb
Get-GlobalAddressList | Update-GlobalAddressList -vb
Get-OfflineAddressBook |Update-OfflineAddressBook -vb
Get-EmailAddressPolicy | Update-EmailAddressPolicy -vb

查詢產生OAB的Mailbox
Get-Mailbox -Arbitration | where {$_.PersistedCapabilities -like "*oab*"} 

#POP3啟用110 Port
Set-PopSettings -LoginType PlainTextLogin
Get-PopSettings

#POP3啟用995+SSL
Set-PopSettings -ExternalConnectionSettings "mail.qps-taiwan.com:995:SSL","mail.qps-taiwan.com:110:TLS" -X509CertificateName mail.qps-taiwan.com

#服務測試
https://mail.domain/ews/exchange.asmx

設定連結器可Relay到外部Domain
Set-ReceiveConnector "ANONYMOUS RELAY MAIL1" -PermissionGroups AnonymousUsers
Get-ReceiveConnector "ANONYMOUS RELAY MAIL1" | Add-ADPermission -User 'NT AUTHORITY\Anonymous Logon' -ExtendedRights MS-Exch-SMTP-Accept-Any-Recipient

#設定郵件傳送/接收大小
Set-TransportConfig -MaxSendSize
Set-TransportConfig -MaxReceiveSize

Windows Server授權
Slui

Exchange Server授權
Set-ExchangeServer EX2016 -ProductKey QXYKC-7H87P-YKC2Q-XRVQ7-GTJP2
Restart-Service MSExchangeIS



移除離線通訊錄
Remove-OfflineAddressBook -Identity "My OAB"
中斷關聯使用者信箱
Get-Mailbox -Database <MailboxDatabase> | Disable-mailbox
移除系統信箱
Get-Mailbox -Arbitration -Database <MailboxDatabase> | Disable-Mailbox -Arbitration -DisableLastArbitrationMailboxAllowed
移除最後一個系統信箱
Remove-Mailbox <Mailbox ID> -Arbitration -RemoveLastArbitrationMailboxAllowed
移除Exchange
Setup.exe /mode:Uninstall /IAcceptExchangeServerLicenseTerms

移除AD使用者物件Exchange屬性值
get-aduser -filter {msExchMailboxGuid -like "*"} | set-aduser -clear msExchMailboxGuid,msexchhomeservername,legacyexchangedn,mail,mailnickname,msexchmailboxsecuritydescriptor,msexchpoliciesincluded,msexchrecipientdisplaytype,msexchrecipienttypedetails,msexchumdtmfmap,msexchuseraccountcontrol,msexchversion


## 超有用連結

- [Exchange 部署助手](https://setup.cloud.microsoft/exchange/deployment-assistant)<br>
