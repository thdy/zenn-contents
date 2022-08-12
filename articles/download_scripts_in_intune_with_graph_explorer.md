---
title: "Intuneに登録されたスクリプトの内容をGraph Explorerで確認する"
emoji: "🍟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["intune", "microsoftgraph", "microsoft", "api"]
published: true
---
# 概要
Intuneでは登録したスクリプトファイルをGUI (Microsoft Endpoint Manager) からプレビューしたりダウンロードする機能が備えられていません。

作成したスクリプトはGitHub等で管理することが理想的ですが、どこにも残されておらず中身が確認できなくて困った…という時にはAPIでスクリプトの取得が可能です。

この記事では[Microsoft Graph Explorer](https://docs.microsoft.com/ja-jp/graph/graph-explorer/graph-explorer-overview)というブラウザ上で簡単にMicrosoft Graph APIのリクエストを行えるツールを使ってIntuneスクリプトの内容を確認する手順を記載します。

# 手順
## Intuneに登録されているスクリプトの内容を取得する
まずはMicrosoft Graph Explorerにアクセスします。

https://developer.microsoft.com/en-us/graph/graph-explorer

右上の人物アイコンをクリックしてIntune管理者のアカウントでサインインします。
![](/images/download_scripts_in_intune_with_graph_explorer/1660228383127.png)

サインインできたらまずはスクリプトの一覧を取得するために下記を実行します。
- APIバージョンを `beta` に指定
- APIエンドポイントのURLに `https://graph.microsoft.com/beta/deviceManagement/deviceManagementScripts` をペースト
- `Run query` をクリック
![](/images/download_scripts_in_intune_with_graph_explorer/1660228818006.png)

成功すると下記のようにIntuneに登録されているスクリプトの一覧がレスポンスとして表示されます。
 `displayName` の値を見て目的のスクリプトを見つけたら `id` の値 (※下記サンプルで言う `f3b8a79f-0478-46c7-978b-2491aba58ead` の部分) をコピーしておきます。
```json:Responce preview
{
    "@odata.context": "https://graph.microsoft.com/beta/$metadata#deviceManagement/deviceManagementScripts",
    "value": [
        {
            "enforceSignatureCheck": false,
            "runAs32Bit": true,
            "id": "f3b8a79f-0478-46c7-978b-2491aba58ead",
            "displayName": "getHostInfo.ps1",
            "description": "",
            "scriptContent": null,
            "createdDateTime": "2022-08-11T13:11:02.6840523Z",
            "lastModifiedDateTime": "2022-08-11T13:11:02.6840523Z",
            "runAsAccount": "system",
            "fileName": "getHostInfo.ps1",
            "roleScopeTagIds": [
                "0"
            ]
        }
    ]
}
```

次にIDを指定してスクリプトの内容を取得します。
APIエンドポイントURLの末尾に `/ + 先程コピーしておいたID` を付け足して `Run query` を実行します。
![](/images/download_scripts_in_intune_with_graph_explorer/1660229749714.png)

すると下記のようなレスポンスが返ってきます。
この `scriptContent` の部分が実際のスクリプトの中身ですが、Base64でエンコードされているためこのままでは読めません。
```json:Responce preview
{
    "@odata.context": "https://graph.microsoft.com/beta/$metadata#deviceManagement/deviceManagementScripts/$entity",
    "enforceSignatureCheck": false,
    "runAs32Bit": true,
    "id": "f3b8a79f-0478-46c7-978b-2491aba58ead",
    "displayName": "getHostInfo.ps1",
    "description": "",
    "scriptContent": "JGhvc3ROYW1lID0gaG9zdG5hbWUKJHdpbmRvd3NWZXJzaW9uID0gKEdldC1XbWlPYmplY3QgV2luMzJfT3BlcmF0aW5nU3lzdGVtKS5WZXJzaW9uCgokdXJsID0gImh0dHBzOi8vaG9va3Muc2xhY2suY29tL3dvcmtmbG93cy94eHh4eHh4eHh4eC95eXl5eXl5eXl5eS96enp6enp6enp6enp6enp6enp6enp6enoiCiRib2R5ID0gQ29udmVydFRvLUpzb24gQHsKICAgIGhvc3ROYW1lICAgICAgID0gJGhvc3ROYW1lCiAgICB3aW5kb3dzVmVyc2lvbiA9ICR3aW5kb3dzVmVyc2lvbgp9Ckludm9rZS1SZXN0TWV0aG9kIC11cmkgJHVybCAtTWV0aG9kIFBvc3QgLWJvZHkgJGJvZHkgLUNvbnRlbnRUeXBlICdhcHBsaWNhdGlvbi9qc29uJwo=",
    "createdDateTime": "2022-08-11T13:11:02.6840523Z",
    "lastModifiedDateTime": "2022-08-11T13:11:02.6840523Z",
    "runAsAccount": "system",
    "fileName": "getHostInfo.ps1",
    "roleScopeTagIds": [
        "0"
    ]
}
```

次のセクションではこのエンコードされた文字列を読めるようにデコードします。

## Base64でエンコードされた文字列をデコードする
Base64でデコードするには様々な方法があります。
外部サイトに貼って問題ないようなスクリプトであれば [base64decode.org](https://www.base64decode.org/) などを利用するのがラクですが、今回はローカルで行いたい前提としてVisual Studio Codeの拡張機能である「vscode-base64」を使った例を紹介します。

https://marketplace.visualstudio.com/items?itemName=adamhartford.vscode-base64

前のセクションで取得したBase64エンコードされた文字列を以下の操作でデコードします。
- vscode-base64をインストールしたVisual Studio Codeを起動
- Base64エンコードされた文字列をペーストして全選択
- コマンドパレットを表示して `Base64 Decode` を選択
- デコードされた文字列が表示されます
![](/images/download_scripts_in_intune_with_graph_explorer/base64_decode.gif)

これで無事にIntuneに登録したスクリプトの内容を確認できました。

もっと簡単にスクリプトの内容を確認する方法があるよ、という場合はこの記事のコメントや[Twitter](https://twitter.com/thdy_jp)で教えていただけると嬉しいです。
