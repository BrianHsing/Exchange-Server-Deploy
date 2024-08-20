## 常用的 Exchange 指令

- 憑證匯入<br>
````
Import-ExchangeCertificate -FileData ([System.IO.File]::ReadAllBytes('\\FileServer01\Data\Fabrikam.pfx')) -Password (ConvertTo-SecureString -String 'P@ssw0rd1' -AsPlainText -Force)
````

- 混合式公用資料夾信箱<br>
````
Set-OrganizationConfig -PublicFoldersEnabled Remote -RemotePublicFolderMailboxes Mailbox1
````

- 移轉資料庫路徑<br>
````
Dismount-Database "DBNAME"
Move-DatabasePath  -Identity "DBNAME" -LogFolderPath "D:\MailDB\DBNAME" -EdbFilePath "D\MailDB\DBNAME\DBNAME.edb"
Mount-Database "DBNAME"
````

- 移轉系統 Log<br>
````
Set-FrontEndTransportService EX2016 -ReceiveProtocolLogPath D:\MailLogs\FrontEnd\ProtocolLog\SmtpReceive -SendProtocolLogPath D:\MailLogs\FrontEnd\ProtocolLog\SmtpSend -ConnectivityLogPath D:\MailLogs\FrontEnd\Connectivity -RoutingTableLogPath D:\MailLogs\FrontEnd\Routing -AgentLogPath D:\MailLogs\FrontEnd\AgentLog
````
````
Set-MailboxTransportService EX2016 -ConnectivityLogPath D:\MailLogs\Mailbox\Connectivity -ReceiveProtocolLogPath D:\MailLogs\Mailbox\ProtocolLog\SmtpReceive -SendProtocolLogPath D:\MailLogs\Mailbox\ProtocolLog\SmtpSend -RoutingTableLogPath D:\MailLogs\Mailbox\Routing -PipelineTracingPath D:\MailLogs\Mailbox\PipelineTracing -MailboxSubmissionAgentLogPath D:\MailLogs\Mailbox\AgentLog\Submission -MailboxDeliveryAgentLogPath D:\MailLogs\Mailbox\AgentLog\Delivery -MailboxDeliveryThrottlingLogPath D:\MailLogs\Throttling\Delivery
````
````
Set-TransportService EX2016 -IrmLogPath D:\MailLogs\IRMLogs -ActiveUserStatisticsLogPath D:\MailLogs\Hub\ActiveUsersStats -ServerStatisticsLogPath D:\MailLogs\Hub\ServerStats -PickupDirectoryPath D:\TransportRoles\Pickup -PipelineTracingPath D:\MailLogs\Hub\PipelineTracing -ReplayDirectoryPath D:\TransportRoles\Replay -RoutingTableLogPath D:\MailLogs\Hub\Routing -QueueLogPath D:\MailLogs\Hub\QueueViewer -WlmLogPath D:\MailLogs\WLM -AgentLogPath D:\MailLogs\Hub\AgentLog -TransportHttpLogPath D:\MailLogs\Hub\TransportHttp
````

- 批次 System Mailbox 移轉<br>
````
Get-Mailbox -Database MailDB01 -Arbitration | New-MoveRequest -TargetDatabase DB01 -BatchName SystemMailboxToDB01
Get-MoveRequest -BatchName SystemMailboxToDB01
Get-MigrationUserStatistics -BatchId SystemMailboxToDB01
````

- Discovery Mailbox 移轉<br>
````
Get-Mailbox -RecipientTypeDetails DiscoveryMailbox | New-MoveRequest -TargetDatabase DB01
````

- 建立離線通訊錄<br>
````
New-OfflineAddressBook -Name "OAB2016" -AddressLists "Default Global Address List" -GlobalWebDistributionEnabled $false
````

-設定內部Autodiscover<br>
````
Get-ClientAccessServer | FL id*,auto*
Set-ClientAccessServer -Identity EX2013 -AutoDiscoverServiceInternalURI 
Set-ClientAccessService -Identity EX2016 -AutoDiscoverServiceInternalURI 
#設定內部
Set-ClientAccessService -Identity Internal_Hostname -AutoDiscoverServiceInternalUri https://External_FQDN/Autodiscover/Autodiscover.xml
#範例
Set-ClientAccessService -Identity EX01 -AutoDiscoverServiceInternalUri https://mail.brianhsing.store/Autodiscover/Autodiscover.xml
````


- 內外虛擬目錄URL<br>
````
$Server = "EX2016"
$HTTPS_FQDN = "EX2016.adtest.com.tw"
Get-OWAVirtualDirectory -Server $Server | Set-OWAVirtualDirectory -ExternalURL $null
Get-ECPVirtualDirectory -Server $Server | Set-ECPVirtualDirectory -ExternalURL $null
Get-OABVirtualDirectory -Server $Server | Set-OABVirtualDirectory -ExternalURL $null
Get-ActiveSyncVirtualDirectory -Server $Server | Set-ActiveSyncVirtualDirectory -ExternalURL $null
Get-WebServicesVirtualDirectory -Server $Server | Set-WebServicesVirtualDirectory -ExternalURL $null
Enable-OutlookAnywhere -Server $Server -ClientAuthenticationMethod Basic -SSLOffloading $False -ExternalHostName $HTTPS_FQDN
````

- 更新Addresslist<br>
````
Get-AddressList | Update-AddressList -vb
Get-GlobalAddressList | Update-GlobalAddressList -vb
Get-OfflineAddressBook |Update-OfflineAddressBook -vb
Get-EmailAddressPolicy | Update-EmailAddressPolicy -vb
````

- 查詢產生OAB的Mailbox<br>
````
Get-Mailbox -Arbitration | where {$_.PersistedCapabilities -like "*oab*"} 
````

- POP3啟用110 Port<br>
````
Set-PopSettings -LoginType PlainTextLogin
Get-PopSettings
````

- POP3啟用995+SSL<br>
````
Set-PopSettings -ExternalConnectionSettings "mail.qps-taiwan.com:995:SSL","mail.qps-taiwan.com:110:TLS" -X509CertificateName mail.qps-taiwan.com
````

- 服務測試<br>
````
https://mail.domain/ews/exchange.asmx
````

- 設定連結器可Relay到外部Domain<br>
````
Set-ReceiveConnector "ANONYMOUS RELAY MAIL1" -PermissionGroups AnonymousUsers
Get-ReceiveConnector "ANONYMOUS RELAY MAIL1" | Add-ADPermission -User 'NT AUTHORITY\Anonymous Logon' -ExtendedRights MS-Exch-SMTP-Accept-Any-Recipient
````

- 設定郵件傳送/接收大小<br>
````
Set-TransportConfig -MaxSendSize
Set-TransportConfig -MaxReceiveSize
````

- Windows Server 授權<br>
````
Slui
````

- Exchange Server 授權<br>
````
Set-ExchangeServer EX2016 -ProductKey XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
Restart-Service MSExchangeIS
````

- 移除離線通訊錄<br>
````
Remove-OfflineAddressBook -Identity "My OAB"
````

- 中斷關聯使用者信箱<br>
````
Get-Mailbox -Database <MailboxDatabase> | Disable-mailbox
````
- 移除系統信箱<br>
````
Get-Mailbox -Arbitration -Database <MailboxDatabase> | Disable-Mailbox -Arbitration -DisableLastArbitrationMailboxAllowed
````
- 移除最後一個系統信箱<br>
````
Remove-Mailbox <Mailbox ID> -Arbitration -RemoveLastArbitrationMailboxAllowed
移除Exchange
````
- 移除 Exchange Server<br>
````
Setup.exe /mode:Uninstall /IAcceptExchangeServerLicenseTerms
````

- 移除 AD 使用者物件 Exchange 屬性值<br>
````
get-aduser -filter {msExchMailboxGuid -like "*"} | set-aduser -clear msExchMailboxGuid,msexchhomeservername,legacyexchangedn,mail,mailnickname,msexchmailboxsecuritydescriptor,msexchpoliciesincluded,msexchrecipientdisplaytype,msexchrecipienttypedetails,msexchumdtmfmap,msexchuseraccountcontrol,msexchversion
````