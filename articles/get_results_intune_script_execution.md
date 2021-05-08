---
title: "Intune から配布したスクリプトの実行結果を Graph API で取得する"
emoji: "🛠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["intune", "windows"]
published: false
---
# この記事は何か
Intune から Windows 10 デバイスへ配布したスクリプトの実行結果の詳細を Microsoft Graph API を使って取得する tips です。

また、Graph API のクエリパラメータを使って出力結果を見やすくするナレッジも一部含めています。

# 【課題】GUI からはスクリプトの実行ログを収集できない
Intune からスクリプトを配布した場合、Microsoft Endpoint Manager の GUI 上では 不明 / 成功 / 失敗 しか表示されないため、配布実行したスクリプト上で処理ログを記録するようにしたりしていても Intune 上からその結果を確認することができません。

![](https://storage.googleapis.com/zenn-user-upload/pela2a6pfbprxv94qoj0grg4cw12)

公式の Docs でもローカルのログファイルを参照するように書かれています。
https://docs.microsoft.com/ja-jp/mem/intune/apps/intune-management-extension
> クライアント コンピューター上のエージェント ログは、一般的に `\ProgramData\Microsoft\IntuneManagementExtension\Logs` にあります。 CMTrace.exe を使用してこれらのログ ファイルを表示できます。
>

このローカルのログファイルも、今年 2 月にプレビューリリースされた[診断の収集](https://docs.microsoft.com/ja-jp/mem/intune/remote-actions/collect-diagnostics)機能で取得できない事は無いのですが、1 台ずつ zip ファイルでしか取得できないのでそれらを展開して該当のログファイルを開いて…というのはかなりつらいです。

一応レジストリの `HKEY_LOCAL_MACHINE¥SOFTWARE¥Microsoft¥IntuneManagementExtension¥<AzureADユーザーGUID>¥{Script ID}` に実行結果が記録されるので、特定のレジストリを収集できる資産管理ツールの類がある組織ならそれを使うでも良いとは思いますが、Intune オンリーな環境だとレジストリの収集も直接的な機能は実装されていないのが現状です。

# Graph API でスクリプト実行結果を確認する
ということでここからが本題です。
Microsoft Graph API を使ってスクリプトの実行結果を確認します。

## 対象スクリプトのスクリプト ID を取得する
Intune 用 Microsoft Graph API の `deviceManagementScripts` を使用します。

https://docs.microsoft.com/ja-jp/graph/api/intune-shared-devicemanagementscript-list?view=graph-rest-beta

下記の API を実行すると、Intune に登録しているスクリプトの一覧を取得することができます。

```HTTP
GET https://graph.microsoft.com/beta/deviceManagement/deviceManagementScripts
```

curl 等で叩きたい場合は Azure AD 上でアプリ登録してトークン発行する必要がありますが、手っ取り早く実行するには Graph Explorer を使えば GUI 上で認証を通すことができるのでお手軽です。

https://developer.microsoft.com/en-us/graph/graph-explorer

ちなみに今回利用する API の Permission は `DeviceManagementManagedDevice.Read.All` なのであらかじめ許可しておきます。

![](https://storage.googleapis.com/zenn-user-upload/y105kpcjb2y2a1uqfoqsltnaevdh)
![](https://storage.googleapis.com/zenn-user-upload/tv9e66t1srfqonjh7n609ycn1t1w)

以下は実行した結果として返ってくる JSON データの例です。
```JSON
{
  "value": [
        {
            "enforceSignatureCheck": false,
            "runAs32Bit": true,
            "id": "59ea4525-4525-59ea-2545-ea592545ea59",
            "displayName": "Detect-CVE-2021-21551.ps1",
            "description": "Dell PC の脆弱性検出スクリプト",
            "scriptContent": null,
            "createdDateTime": "2021-05-06T23:14:42.7913925Z",
            "lastModifiedDateTime": "2021-05-06T23:28:01.5984015Z",
            "runAsAccount": "system",
            "fileName": "Detect-CVE202121551.ps1",
            "roleScopeTagIds": [
                "0"
            ]
        }
    ]
}
```

ここで把握しておくべきなのは上記で言う `"id": "59ea4525-4525-59ea-2545-ea592545ea59"` の部分です。
スクリプトが複数登録されていれば value セクションが複数返ってくるので、実行結果を取得したいスクリプトの ID を控えておきましょう。

## スクリプトの実行結果を取得する
前述の `deviceManagementScripts` API にスクリプトID を加えて実行することで `deviceManagementScriptDeviceState` という詳細リソースを取得できるようになります。

```HTTP
GET https://graph.microsoft.com/beta/deviceManagement/deviceManagementScripts/{Script ID}/deviceRunStates
```

例えば前項で取得したスクリプト ID である `"id": "59ea4525-4525-59ea-2545-ea592545ea59"` を使って書くと下記のような Request URL になります。

```HTTP
GET https://graph.microsoft.com/beta/deviceManagement/deviceManagementScripts/59ea4525-4525-59ea-2545-ea592545ea59/deviceRunStates
```

これを実行すると下記のような応答が返ってきます。

```JSON
{
  "@odata.type": "#microsoft.graph.deviceManagementScriptDeviceState",
  "id": "59ea4525-4525-59ea-2545-ea592545ea59:fabec251-6d88-441a-93c0-b17a524bb914",
  "runState": "Success",
  "resultMessage": "Dell dbutil_2_3.sys file not present on system",
  "lastStateUpdateDateTime": "2021-05-07T00:15:26Z",
  "errorCode": 0,
  "errorDescription": null
}
```

この `resultMessage` の部分に、スクリプト内で `Write-Output` 等で出力した結果が記録されます。
ただ、これだけだとどのデバイスの実行結果が分からないので `managedDevice` という詳細リソースを取得するためのクエリパラメータとして URL 末尾に `$expand=managedDevice` を付与して実行してみます。

```HTTP
GET https://graph.microsoft.com/beta/deviceManagement/deviceManagementScripts/{Script ID}/deviceRunStates$expand=managedDevice
```

これで実行すると、下記 Docs にある応答例を見てもらうと分かりますがデバイス名やシリアルナンバーはもちろんのこと様々なハードウェア情報が取得できます。

https://docs.microsoft.com/ja-jp/graph/api/resources/intune-devices-manageddevice?view=graph-rest-beta#json-representation

逆にここまで多いと結果の確認が大変なので、必要な情報のみに絞りたいと思います。

下記の例では「スクリプト実行結果」「デバイス名」「ユーザー名」の 3 点のみを取得するため、クエリパラメータとして `$select=resultMessage&$expand=managedDevice($select=deviceName,userPrincipalName)` を付与しています。

```HTTP
GET https://graph.microsoft.com/beta/deviceManagement/deviceManagementScripts/{Script ID}/deviceRunStates?$select=resultMessage&$expand=managedDevice($select=deviceName,userPrincipalName)
```

これで 1 台につき下記の情報のみが返ってくるようになりました。
```JSON
{
    "resultMessage": "Dell dbutil_2_3.sys file not present on system",
    "managedDevice": {
        "deviceName": "DESKTOP-hogePC",
        "userPrincipalName": "hoge@example.com"
    }
}
```

さらに見やすく整形したい場合は [jq コマンド](https://qiita.com/takeshinoda@github/items/2dec7a72930ec1f658af) 等を使うようになると思いますが、この記事上ではいったんここまでとしたいと思います。

何か記載内容についての指摘や提案、質問点などあればコメント欄か私の [Twitter](https://twitter.com/thdy_jp) までご連絡ください。

おしまい