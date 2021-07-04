---
title: "OktaをIdPとしてAzure ADとのフェデレーションを構成する"
emoji: "🪢"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["okta", "azuread"]
published: true
---
# はじめに
OktaをIdPとしてAzure AD(Microsoft 365)とフェデレーションを構成する際、[Okta公式のドキュメント](https://help.okta.com/en/prod/Content/Topics/Apps/Office365-Deployment/configure-sso.htm)通りだとうまくいかない部分があったり書いてないことがあったのでそのメモです。
途中でハマったポイントについても書いています。

なおOktaのドキュメントやアプリケーションテンプレート上だとOffice 365と表記されている（恐らくエンドユーザーが主に触るのはOffice 365なのでそれに合わせている）のですが、実際の接続先はAzure ADなので本記事上では個人的にもしっくり来る `Azure AD` のほうで表現しています。

# 前提
今回は下記のシナリオを前提として構成した例になります。
- Oktaをメインディレクトリとする
- オンプレミスのActive Directoryは無し
- Windows PCはAzure AD Join + Intuneにて管理運用している

なおOktaとAzure AD間でフェデレーションを構成するにはAzure ADに追加しているカスタムドメインごとにOkta側のアプリケーション設定をする必要があります。
またOkta側をメインディレクトリとする場合、Azure AD側の `{yourtenant}.onmicrosoft.com` のドメインについてはフェデレーションの対象外（SSOやプロビジョニングは行われない）となります。
（イメージ図）
![](https://storage.googleapis.com/zenn-user-upload/a82246128408545d1a97facb.png)

# Azure AD側の準備
### カスタムドメインをAzure ADに追加する
カスタムドメインが無いと紐付ける先が無いので、もしまだAzure ADでカスタムドメインを運用していなければ追加が必要です
https://docs.microsoft.com/ja-jp/azure/active-directory/fundamentals/add-custom-domain

### プライマリドメインを `{yourtenant}.onmicrosoft.com` のほうにする
これは必須の作業であり、カスタムドメインがプライマリとなっているとフェデレーションを構成できません
現在の設定については `Azure Active Directory > カスタム ドメイン名` の画面で確認できます
![](https://storage.googleapis.com/zenn-user-upload/d62fe7dc581c26fd7d42b99a.png)
プライマリになってなかったら `{yourtenant}.onmicrosoft.com` をクリックして `プライマリにする` を選択します

### プロビジョニングに使用するためのグローバル管理者アカウントを作成する
（これはユーザーアカウントやグループをOkta⇔Azure AD間でプロビジョニングしない場合は設定不要です）
フェデレーションするカスタムドメインではなく `{yourtenant}.onmicrosoft.com` のほうのドメインでユーザーを作成し、グローバル管理者のロールを割り当てておきます。
なお作成したユーザーのMFAが有効になっているとフェデレーションが失敗するのでMFAを無効化しておきます（参考：[Office 365 WS-Fed Integration issue](https://support.okta.com/help/s/question/0D51Y00008QoNQN/office-365-wsfed-integration-issue?language=en_US)）

全体のMFA設定をOFFにするには `Azure Active Directory > プロパティ > セキュリティの既定値群の管理` にて `セキュリティの既定値群の有効化` を `いいえ` にします
![](https://storage.googleapis.com/zenn-user-upload/f4ccbddaa2296dca9ebdd00f.png)

セキュリティの既定値群を無効化するとどんな影響があるのかが気になる方はこちらを参照してください
[Azure Active Directory のセキュリティ デフォルト | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/active-directory/fundamentals/concept-fundamentals-security-defaults)

ユーザーごとのMFAについては `Azure Active Directory > ユーザー > ユーザーごとの MFA` にて有効になっているユーザーが居ないか確認してください

# SSOを設定する
ここからはOktaの管理画面を操作します。
なおOktaにはMicrosoft Office 365を統合するためのアプリケーションテンプレートが用意されているのでこれを使います。

`Applications > Browse App Catalog > Microsoft Office 365 > Add` を選択します。
![](https://storage.googleapis.com/zenn-user-upload/0c7798c5384f5c2018af7b38.png)

## `General Settings`
- Application label
  - 管理用のアプリ名称を設定します
- Microsoft Tenant Name
  - 例として `thdyjp.onmicrosoft.com` というAzure ADテナントを使用する場合ここには `thdyjp` と入力します
      ![](https://storage.googleapis.com/zenn-user-upload/c683ecc190b6342bf3ffe4d1.png)
- Display the following links
  - Oktaダッシュボードに表示するアプリアイコンを選択します
- Seats（Optional）
  - ここに購入済みライセンス数を入れておくと使用率のレポートが出せます
- Application Visibility
  - ダッシュボードにアイコンを表示するかどうかの設定です
- Browser plugin auto-submit
  - 有効にするとユーザーがログインページにアクセスすると自動的にログインします
すべて設定したら `Next` を選択します

## `Sign-On Options・Required`
- Sign on methods
  - `WS-Federation` を選択
- WS-Federation Configration
  - ここは特別な理由がなければ Okta の推奨通り `Automatic (recommended)` を選択
- Office 365 Admin Username
  - Azure AD 側で作成した `{yourtenant}.onmicrosoft.com` のグローバル管理者アカウントを入力
- Office 365 Admin Password
  - 前述のグローバル管理者アカウントのパスワードを入力
- Office 365 Domains
  - `Fetch and Select` を選択するとドメイン選択画面が出るのでフェデレーション対象のカスタムドメインを選択して `Select`
      ![](https://storage.googleapis.com/zenn-user-upload/42e903f6bf0608da47082f01.png)
  - Default Relay State
    - ここは空欄でOK
  - Credentials Details
    - ここはデフォルト設定のまま
  - Advanced API Access
    - プロビジョニングを行う場合はここのチェックを入れて `Authenticate with Microsoft Office 365` を選択
    - グローバル管理者のクレデンシャルを入力すると認可を求められるので `承諾` を選択
      ![](https://storage.googleapis.com/zenn-user-upload/b639d3f95aff489451a7be36.png =300x)

        この時Azure AD側のエンタープライズアプリケーションの中にGraph APIを叩いてAzure ADを操作するためのアプリが作成されます。もしOkta側でAzure ADとのフェデレーション設定をすべて削除したいとなった場合はこのOkta Microsoft Graph Clientも忘れずに削除しましょう。
        ![](https://storage.googleapis.com/zenn-user-upload/76f3d095750b4a651636d54b.png)

- `Done` を選択
ここまででSSOの設定は完了。
続いてプロビジョニングの設定を進めます。

# プロビジョニングを有効化する
## プロビジョニングについて理解する
Oktaにて用意されているAzure AD用のプロビジョニングオプションは4種類あり、それぞれできることが違います。
下記表にまとめていますが、右に行くほどOkta側でできることが増えます。

|サポートされている操作|Licenses and Roles Management Only|Profile Sync|User Sync|Universal Sync|
|----|----|----|----|----|
|**ユーザーのプロビジョニング**|||||
|ライセンスとロールの割り当て|◯|◯|◯|◯|
|ユーザー作成|✕|◯|◯|◯|
|ユーザーの非アクティブ化|◯|◯|◯|◯|
|Azure AD側でのユーザー編集|◯|◯|✕|✕|
|**プロファイルの同期**|||||
|基本的なプロファイル同期|✕|◯|◯|◯|
|一部の拡張属性の同期|✕|✕|◯|◯|
|すべての拡張属性の同期|✕|✕|✕|◯|
|**ADグループとリソースの同期**|||||
|セキュリティグループ|✕|✕|✕|◯|
|連絡先|✕|✕|✕|◯|
|配布リスト|✕|✕|✕|◯|
|リソースメールボックス|✕|✕|✕|◯|

そしてこれらのプロビジョニングオプションにおける個人的注意ポイントを紹介します。

- **一度User SyncおよびUniversal SyncにするとLicenses and Roles Management OnlyやProfile Syncには戻せない**
  戻したい場合は一度Okta側で作成したアプリを削除してイチから作り直すことになります。
  （プロビジョニング設定の変更可否イメージ図）
  ![](https://storage.googleapis.com/zenn-user-upload/50fa8ef5b023911f26f2b939.png =500x)


- **Azure AD Connectを使用してオンプレミスADとHyblid環境を構成しているなど、既にフェデレーションが行われている場合はUser SyncおよびUniversal Syncを使用できない**
  なのでHyblid Azure AD Joinを構成している組織では実質Licenses and Roles Management OnlyかProfile Syncしか選べません。

- **User SyncおよびUniversal SyncにするとAzure AD（Microsoft 365管理センター）側で直接ユーザーを編集できない**
  普段の運用でAzure AD側からも編集できる必要があるかどうかは事前に考慮して選択しましょう。

- **Okta側のライセンスやロールの表記はAzure AD側と若干異なる**
  普段利用するライセンスのグループをOkta側に作成してそこにメンバーを追加する運用にすれば戸惑うのは最初だけで済むかもしれません。

## プロビジョニングを設定する
どのプロビジョニングオプションにするか決めたら、実際に設定します。

`Provisioning > Integration > Congigure API Integration` を選択し、`Enable API Integration` にチェックを入れて `Admin Username` と `Admin Password` にAzure ADのグローバル管理者ロールを持ったアカウントのID/パスワードを入力します。
`Import Groups` はAzure AD側のグループをOktaにインポートするかどうかの設定でここは任意です。
`Test API Credentials` を選択してID/パスワードに誤りが無いことを確認できたら `Save` を選択します。
![](https://storage.googleapis.com/zenn-user-upload/c3f2feb5fddadafadc471c69.png)

これでプロビジョニングを行うために必要な権限がOkta側に与えられ、Provisioningタブに`To App` と `To Okta`が出現します。
![](https://storage.googleapis.com/zenn-user-upload/171a94ccc7fcf913299a38a7.png =250x)

ここからは組織の要件に応じて必要なプロビジョニングの設定をします。
以下は一例として、プロビジョニングオプションに `Profile Sync` を選択して進めます。
![](https://storage.googleapis.com/zenn-user-upload/7cd16ea885456e0611c1b516.png)

Create Users、Update User Attributes、Deactivate Users すべてチェックをつけます。これでOktaでユーザーを作成したり無効化するとAzure AD側にも自動で反映するようになります。
![](https://storage.googleapis.com/zenn-user-upload/f1f07b37a80fa2a8a62c53a8.png)

あとは必要に応じてMappingsで属性の紐付けを行い、Assignmentタブからユーザーをアサインすれば完了です。
実際にプロファイルが同期されているかの確認や、シングルサインオンが行えるかどうかをテストを行ってみましょう。

# ハマったことのメモ
いろいろハマったので合わせて書き残しておきます。

### OktaのパスワードでWindows PCにサインインできるようにするのにハマった
シングルサインオンを構成しているのになぜパスワード同期を有効にするのかというと、Windows PCのパスワードサインインはいわゆるレガシー認証であり認証をOktaにリダイレクトすることができないためです。
Azure AD JoinしているWindows PCのサインインパスワードをOktaと同じものにすることで、ユーザー体験をシンプルにしたかったという目的がありました。

そこでProvisioning設定のSync Passwordにチェックを入れ、`Sync Okta Password`を選択しました。
![](https://storage.googleapis.com/zenn-user-upload/03f40d466d13dd47c345b4db.png)

しかし実際にWindows PC側でパスワードでサインインしようとするとパスワードに誤りがあると表示されるのでOktaサポートに確認したところ、「デフォルトでレガシー認証は許可されないようになっているのでWindowsサインインさせたいなら手動で許可してね」と回答がありました。

どこを設定するかと言うと`Sign on`タブの `Sign On Policy` で `Add Rule` を選択し、`Exchange ActiveSync/Legacy Auth`のところにチェックを入れる、これだけなんですがドキュメントに書いてなくてハマりました。
![](https://storage.googleapis.com/zenn-user-upload/943f8e2cc02440d136cb88ed.png)

### Immutable IDが消えた
Immutable IDというのはOktaとAzure ADのユーザーアカウントを紐付けるために必要な属性値で、通常これは自動的に割り当てられるのですが検証等でユーザーをアサインしたりはずしたり設定を色々触っているとなぜか消えることがあるようです。そしてこれが消えるとSSOもうまく動作しません。
この辺りの詳細はkobasoさんがnoteに書かれているので興味のある方は読んでみてください。

https://note.com/kobaso/n/n6ab079fa0d34

で、実際に消えたらどうすれば良いのかというとPowerShellで適当な値を突っ込むしかないようです。

https://support.okta.com/help/s/article/SSO-from-Okta-to-office365-shows-error-AADSTS51004?language=en_US

なお既にフェデレーションが有効になっていると上記ヘルプページの手順ではImmutable IDが変更できなかったりします。
自身の環境では下記のように対応しました。

```powershell
Connect-MsolService

// ユーザーを列挙してImmutable IDが正しくセットされているか確認
Get-MsolUser -DomainName "thdy.jp" | Select-Object UserprincipalName, ImmutableID

// フェデレーション済みだとImmutable IDを変更できないので一旦UPN変更
Set-MsolUserPrincipalName -UserPrincipalName hoge@thdy.jp -NewUserPrincipalName hoge@thdyjp.onmicrosoft.com

// Immutable IDに指定する文字列は何でも良いので適当な値をセット
Set-MsolUser -UserPrincipalName hoge@thdyjp.onmicrosoft.com -immutableid 2/9JCtHr0EmH+yL07o11vB==

// UPNをもとに戻す
Set-MsolUserPrincipalName -UserPrincipalName hoge@thdyjp.onmicrosoft.com -NewUserPrincipalName hoge@thdy.jp
```

ユーザーをアサインする場合はグループを利用して一括でバシッとアサインするのが良さそうです。
またアサイン後に `Get-MsolUser` のPowerShellコマンド等でImmutable IDが正しくセットされているか確認しましょう。

### Windows Hello for Businessが「追加のセキュリティ確認」の設定を求めてくる
Azure AD JoinなWindows PCをセットアップする際、Windows Hello for Businessを有効していると通常はAzure AD Joinしたタイミングで携帯電話番号やMicrosoft Authenticatorなどの認証の設定を求める「追加のセキュリティ確認」の画面が表示されます。
しかし今回OktaをIdPとしたことでMFAもすべてOkta側で済ませる前提である以上、これを設定したところでエンドユーザーは実際に使用することはありません。
なんとかしてこれを非表示にしたくて調べたところ、「Okta側でMFAを完了した際のクレームをAzure AD側に渡す」というオプション設定がデフォルトでは無効になっていることがわかりました。

この設定は `Settings > Features > O365 Pass Claim For MFA` にあたるのでこれをオンにすることで、Windows Hello for Businessのバックアップ設定は求められなくなりました。
![](https://storage.googleapis.com/zenn-user-upload/4090cafa675f6e865e2be4a7.png)

# さいごに
書いてある内容に対する指摘や修正提案、不明点がある場合はコメント欄もしくは[Twitter](https://twitter.com/thdy_jp)までご連絡ください。

おしまい