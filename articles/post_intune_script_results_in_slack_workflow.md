---
title: "Intuneで配布したスクリプトの実行結果をSlackに通知する"
emoji: "🍩"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["intune", "Slack", "PowerShell"]
published: true
---
Intuneからスクリプトを配布した場合、Microsoft Endpoint ManagerのGUI上では `不明 / 成功 / 失敗` しか表示されないため、スクリプト上で処理ログを記録するようにしたりしていてもIntune上からその内容を確認することができません。

この課題の解決方法のひとつとして本記事ではIntuneから配布したPowerShellスクリプトの実行結果をSlackの任意のチャンネルに通知する手順を紹介します。

# 全体の流れ
- Slackワークフローをつくる
- PowerShellスクリプトを書く
- Intuneからスクリプトを配布する

今回は簡単なサンプルとして、Windows PCのホスト名とOSバージョンを取得するスクリプトの実行結果を通知してみます。

# Slackワークフローをつくる
前提としてSlackワークフロー自体の基本的な使い方についてはこちらの公式ドキュメントを参考にしてください。

https://slack.com/intl/ja-jp/help/articles/360053571454-Slack-%E3%81%A7%E3%83%AF%E3%83%BC%E3%82%AF%E3%83%95%E3%83%AD%E3%83%BC%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B

まず最初にSlackワークフロービルダーを起動して新規ワークフローを作成し、適当な名前を付けて次へ。

![](https://storage.googleapis.com/zenn-user-upload/669272300d4a-20220315.png)

ワークフローを開始する方法として一番下の `Webhook` を選択します。

![](https://storage.googleapis.com/zenn-user-upload/1ecd0749d94b-20220315.png)

スクリプトの値を格納するための変数を追加します。

![](https://storage.googleapis.com/zenn-user-upload/8fc810de9fe4-20220315.png)

今回はホスト名とOSバージョンを通知するためふたつの変数を追加しました。これで保存します。

![](https://storage.googleapis.com/zenn-user-upload/29dfa5bb0997-20220315.png)

次に通知メッセージの処理を追加するために `ステップを追加` を選択します。

![](https://storage.googleapis.com/zenn-user-upload/d63afc0e7f8f-20220315.png)

`メッセージを送信` の追加ボタンを選択します。

![](https://storage.googleapis.com/zenn-user-upload/3071df4af070-20220315.png)

メッセージの送信先のチャンネルを選択し、さらに通知するメッセージのテキストを入力します。
`変数を挿入する` から事前に定義した変数を挿入してメッセージを作成したら保存します。

![](https://storage.googleapis.com/zenn-user-upload/aca2a2d9ec8d-20220315.png)

これでフローができたので `公開する` を選択します。

![](https://storage.googleapis.com/zenn-user-upload/7203f003348e-20220315.png)

ここで表示されるWebhook URLとHTTP本文の例は次のPowerShellスクリプトを書く時に使用するのでコピーしておきます。

![](https://storage.googleapis.com/zenn-user-upload/66b82e7d6db4-20220315.jpg)

これでSlackワークフローの作成は完了です。

# PowerShellスクリプトを書く
下記のスクリプトはサンプルとして、ホスト名とOSバージョンを取得してSlackワークフローにPostする処理を書いています。
```powershell
$hostName = hostname
$windowsVersion = (Get-WmiObject Win32_OperatingSystem).Version

$url = "https://hooks.slack.com/workflows/xxxxxxxxxxx/yyyyyyyyyyy/zzzzzzzzzzzzzzzzzzzzzzzz"
$body = ConvertTo-Json @{
    hostName       = $hostName
    windowsVersion = $windowsVersion
}
Invoke-RestMethod -uri $url -Method Post -body $body -ContentType 'application/json'
```

# Intuneからスクリプトを配布する
作成したPowerShellスクリプトをIntuneから配布します。
ここでは詳しい手順は割愛しますので代わりに公式ドキュメントを置いておきます。

https://docs.microsoft.com/ja-jp/mem/intune/apps/intune-management-extension

無事にスクリプトが実行されたらSlackに下記のようなメッセージが通知されるはずです。
![](https://storage.googleapis.com/zenn-user-upload/bf5510f73318-20220315.png)

# まとめ
上記の例ではスクリプトの取得結果をチャンネルに通知していますが、応用として処理結果をスプレッドシートに記録するステップも作成できます（下記は参考URLです）

https://teachme.jp/90980/manuals/10117361/

またZapierのステップを追加して他の様々な処理やサービスに連携するなど、使い方は様々ですので用途にあった方法を選択するのが良いかと思います。

なお、Slackワークフローは有償ワークスペースでしか利用できないのですが、通知だけであればSlack appのIncoming Webhookを使うことで（多少必要な知識はありますが）同様に実装できるので、無償ワークスペースで通知したい場合はこちらの方法でトライするのも良いかもしれません。

参考記事：
https://zenn.dev/hotaka_noda/articles/4a6f0ccee73a18

https://api.slack.com/tutorials/tracks/posting-messages-with-curl