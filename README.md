# Notify-MDE-AdvancedHunting-Email
このレポジトリは MDE Advanced Hunting を定期的に実行し、結果をメールで通知するサンプルです。<BR>
<img width="510" alt="image" src="https://github.com/hisashin0728/Notify-MDE-AdvancedHunting-Email/assets/55295601/2d88cde9-3378-442c-a2ed-586555da6501">

# 通知イメージ
<img width="510" alt="image" src="https://github.com/hisashin0728/Notify-MDE-AdvancedHunting-Email/assets/55295601/4d71d1f1-519e-4de7-b6fc-c456c8af4bed">

# 設定
以下の Deploy To Azure ボタンでテンプレートを展開して下さい。<BR>
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhisashin0728%2FNotify-MDE-AdvancedHunting-Email%2Fmain%2Ftemplate.json)

# 設定後必要なアクション
本テンプレートは Office 365 コネクタと Defender ATP コネクタを利用しています。API 接続のため、以下設定を実施して下さい。<BR>
(**注意**) Defender ATP の Powershell スクリプト実施後、反映までに 10 分ほど余裕を見てください。<BR>
※ Azure Portal 側で Entra ID でロール付与されても、実反映に時間がかかるようです。

- Office 365 コネクタ / **ユーザー認証**
 - テンプレート展開後、ロジックアプリの API 接続から、Office 365 コネクタを選択する
 - <img width="459" alt="image" src="https://github.com/hisashin0728/Notify-MDE-AdvancedHunting-Email/assets/55295601/71fff7fb-a88d-4f33-bdde-829c2298376a">
 - 「API 接続の編集」より、``承認する``を押して、送信元の Office 365 ユーザーで認証する
 - <img width="624" alt="image" src="https://github.com/hisashin0728/Notify-MDE-AdvancedHunting-Email/assets/55295601/c845ca37-fa33-4cd0-99f6-bce1234ee765">

- Defender ATP コネクタ / **マネージド ID**
  - テンプレートの展開後、Powershell を用いて以下を実施して下さい。(CloudShell でも可能)
  - Connect-AzureAD で認証
```
PS /home/hisashi> Connect-AzureAD
```
  - Entra ID アプリケーション (ロジックアプリのマネージド ID) に対して、``Ip.Read.All``, ``Url.Read.All``, ``File.Read.All``, ``AdvancedQuery.Read.All`` の権限を付与する
```powershell
$MIGuid = "<Enter your managed identity guid here>"
$MI = Get-AzureADServicePrincipal -ObjectId $MIGuid

$MDEAppId = "fc780465-2017-40d4-a0c5-307022471b92"
$MDEServicePrincipal = Get-AzureADServicePrincipal -Filter "appId eq '$MDEAppId'"

$PermissionName = "Ip.Read.All" 
$AppRole = $MDEServicePrincipal.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
New-AzureAdServiceAppRoleAssignment -ObjectId $MI.ObjectId -PrincipalId $MI.ObjectId -ResourceId $MDEServicePrincipal.ObjectId -Id $AppRole.Id
$PermissionName = "Url.Read.All" 
$AppRole = $MDEServicePrincipal.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
New-AzureAdServiceAppRoleAssignment -ObjectId $MI.ObjectId -PrincipalId $MI.ObjectId -ResourceId $MDEServicePrincipal.ObjectId -Id $AppRole.Id
$PermissionName = "File.Read.All" 
$AppRole = $MDEServicePrincipal.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
New-AzureAdServiceAppRoleAssignment -ObjectId $MI.ObjectId -PrincipalId $MI.ObjectId -ResourceId $MDEServicePrincipal.ObjectId -Id $AppRole.Id
$PermissionName = "AdvancedQuery.Read.All" 
$AppRole = $MDEServicePrincipal.AppRoles | Where-Object {$_.Value -eq $PermissionName -and $_.AllowedMemberTypes -contains "Application"}
New-AzureAdServiceAppRoleAssignment -ObjectId $MI.ObjectId -PrincipalId $MI.ObjectId -ResourceId $MDEServicePrincipal.ObjectId -Id $AppRole.Id
```
  - Entra ID のエンタープライズアプリケーションより、マネージド ID に対して権限が付与されているかを確認する
  - <img width="944" alt="image" src="https://github.com/hisashin0728/Notify-MDE-AdvancedHunting-Email/assets/55295601/916e55a3-7999-4110-9b96-0999c441d5bb">

