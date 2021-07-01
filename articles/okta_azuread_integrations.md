---
title: "OktaをIdPとしてAzure ADとのフェデレーションを構成する"
emoji: "⚔️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["okta", "azuread"]
published: false
---
# はじめに
OktaをIdPとしてAzure AD(Microsoft 365)とフェデレーションを構成する際、[Okta公式のドキュメント](https://help.okta.com/en/prod/Content/Topics/Apps/Office365-Deployment/configure-sso.htm)通りだとうまくいかない部分がいくつかあったのでメモです。

# 前提
- Oktaをメインディレクトリとする
- オンプレミスのActive Directoryは無し
- Windows PCはAzure AD Join + Intuneにて管理運用している

なおOktaとAzure AD間でフェデレーションを構成するにはAzure ADに追加しているカスタムドメインごとにOkta側のアプリケーション設定をする必要があります。
また今回はOkta側をメインディレクトリとするためAzure AD側の `{yourtenant}.onmicrosoft.com` のほうはフェデレーションの対象外となります。


# 設定手順
## Azure AD側でやること
- カスタムドメインをAzure ADに追加する（まだ追加してなければ）
  - [カスタム ドメインを追加する - Azure Active Directory | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/active-directory/fundamentals/add-custom-domain)
- プライマリドメインを `{yourtenant}.onmicrosoft.com` のほうにする
  - `Azure Active Directory > カスタム ドメイン名` の画面で確認できます
  - ![](2021-06-17-23-43-21.png)
  - プライマリになってなかったら `{yourtenant}.onmicrosoft.com` をクリックして `プライマリにする` を選択します
- プロビジョニングに使用するためのグローバル管理者アカウントを作成する（プロビジョニングを利用しない場合は不要）
  - フェデレーションするカスタムドメインではなく `{yourtenant}.onmicrosoft.com` のほうのドメインでユーザーを作成し、グローバル管理者のロールを割り当てておきます
  - 作成したユーザーのMFAが有効になっているとフェデレーションが失敗するのでMFAを無効化しておきます（参考：[Office 365 WS-Fed Integration issue](https://support.okta.com/help/s/question/0D51Y00008QoNQN/office-365-wsfed-integration-issue?language=en_US)）

    - 全体のMFA設定をOFFにするには `Azure Active Directory > プロパティ > セキュリティの既定値群の管理` にて `セキュリティの既定値群の有効化` を `いいえ` にします
    ![](2021-06-17-23-53-37.png)
        - 無効化するとどんな影響があるのかが気になる方はこちらを参照してください
          > [Azure Active Directory のセキュリティ デフォルト | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/active-directory/fundamentals/concept-fundamentals-security-defaults)
    - ユーザーごとのMFAについては `Azure Active Directory > ユーザー > ユーザーごとの MFA` にて有効になっているユーザーが居ないか確認してください

## Okta側でやること
`Applications > Browse App Catalog > Microsoft Office 365 > Add` を選択します
![](2021-07-01-23-51-26.png)

### `General Settings`
- Application label
  - 管理用のアプリ名称を設定します
- Microsoft Tenant Name
  - 今回は `thdyjp.onmicrosoft.jp` というテナントを使用するのでここには `thdyjp` と入力します
      ![](2021-07-02-00-16-25.png)
- Display the following links
  - Oktaダッシュボードに表示するアプリアイコンを選択します
- Seats（Optional）
  - ここに購入済みライセンス数を入れておくと使用率のレポートが出せます
- Application Visibility
  - ダッシュボードにアイコンを表示するかどうかの設定です
- Browser plugin auto-submit
  - 有効にするとユーザーがログインページにアクセスすると自動的にログインします
すべて設定したら `Next` を選択します

### `Sign-On Options・Required`
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
      ![](2021-07-02-00-25-57.png)
  - Default Relay State
    - ここは空欄でOK
  - Credentials Details
    - ここはデフォルト設定のまま
  - Advanced API Access
    - プロビジョニングを行うにはここのチェックを入れて `Authenticate with Microsoft Office 365` を選択
    - グローバル管理者のクレデンシャルを入力すると認可を求められるので `承諾` を選択
      ![](2021-07-02-00-31-04.png)
- `Done` を選択
ここまででSSOの設定は完了。
続いてプロビジョニングの設定を進めます。



# アカウントプロビジョニングを有効化する


# アカウント発行からAzure AD Joinまでの流れ



