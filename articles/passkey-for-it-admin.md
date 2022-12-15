---
title: "Passkeyとはなにか、組織のIT管理者はどう向き合うか"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["passkey", "fido", "webauthn"]
published: true
---
この記事は [corp-engr 情シスSlack（コーポレートエンジニア x 情シス）#1 Advent Calendar 2022](https://adventar.org/calendars/7756) 16日目の記事です。

ここ最近においてGoogle Chromeのバージョン108やiOS 16とmacOS Venturaで「Passkeyが利用可能になった」という記事[^1][^2]や、ドコモにおいても2023年2月にはdアカウントの認証でPasskeyを使えるようにするというニュース[^3]が出ており、認証技術の界隈においてPasskeyに関する盛り上がりが見られます。

この記事ではPasskeyとはなんなのか、また組織のIT管理者や情報システム部門の目線から見た場合に気になるポイントについてまとめてみたいと思います。

※ 本記事は2022年12月現在の情報を元に書いています  
※ あくまで個人で調べている内容ですので誤りなどあれば指摘・コメントいただけると参考になります

# Passkeyとはなにか
そもそもPasskeyとはなんなのかについて書こうとしましたが、Google Developer Advocateの [@agektmr](https://twitter.com/agektmr) さんのブログがとても詳しくかつ分かりやすく書かれているので興味のある方はまずこちらを読んで頂くと良いかと思います。

https://blog.agektmr.com/2022/12/passkey.html

上記の記事中にも紹介がありますが、Passkeyを使った実際の認証体験がどういったものなのかについては↓の動画を見ていただくとわかりやすいです。

https://www.youtube.com/watch?v=8D84Yosw-Ws

Passkeyの特徴をざっくり挙げるとすれば、
- Apple、Google、MicrosoftといったFIDOを提供しているプラットフォームにおいて、デバイス間でのFIDOクレデンシャル（鍵）を同期できる仕組み
- ユーザーはハードウェアキーを持ち歩く必要がなく、普段使いのスマホで生体認証を使ってフィッシング耐性のある認証を利用できる

といったところでしょうか。

多くの人が所有しているであろうスマートフォンをセキュリティキーとして使える導入のしやすさや、フィッシングにも強いこと、故障や交換でデバイスが変わっても再登録せずに使える点など多くのメリットを享受できるように思えます。

では、組織内でPasskeyを使うとなった場合に懸念事項やデメリットはないのでしょうか。

# 組織でPasskeyを利用する際の懸念事項
Apple IDを使ったPasskeyのクレデンシャルの同期にはiCloudキーチェーンが用いられます。[^4]
一方で、iCloudキーチェーンはManaged Apple IDでは利用できないため[^5]、実際にはPasskeyを使えるのはコンシューマー向けのApple IDのみということになります。

Googleの実装においては、現状AndroidのGoogleパスワードマネージャーを用いてAndroidデバイス間でPasskeyのクレデンシャルを同期する方式[^6]となっているため、Android以外のプラットフォームでは利用できません。加えて、組織のセキュリティポリシー等でGoogleパスワードマネージャーを制限している場合にも利用できないことになると思われます。

また、従来のFIDO認証ではクレデンシャルの複製への耐性があったものの、Passkeyにおいては対応するプラットフォームのアカウントが侵害されるとPasskeyも窃取され、信頼できないクレデンシャルを登録される懸念があります。認証保証レベルとしてAAL3[^7]を必要としている組織においてはPasskeyでは要件を満たせないといったケースが想定できます。

https://twitter.com/_nat/status/1601113684654567425

このように組織でPasskeyを活用するとなった場合に現状ではいくつかの課題がありますが、Passkey対応を謳う3社のプラットフォーマーの中でMicrosoftについては具体的な実装内容がまだ発表されていないのと、AppleやGoogleもまだリリースしたばかりで今後さまざまな対応を進めてゆくと思われるため、各プラットフォームの動向については期待を持ちつつウォッチしたいと思います。

# OktaにおけるPasskeyへの対応
ここまではPasskeyを提供するプラットフォーム側の話でしたが、FIDOにおけるRP(Relying Party)側の対応としてOktaの例を挙げたいと思います。

Oktaは従業員向けのID管理ソリューションですが、従来からFIDO (WebAuthn)をサポートしており現時点でも既にPasskeyをMFA(多要素認証)登録の際のひとつの手段として利用することが可能となっています。[^8]

![](/images/passkey-for-it-admin/1671117304518.png)
*OktaのMFA登録でSecurity Key or Biometricを選択した際の様子*

一方でPasskeyの仕様に依存する課題としてクレデンシャルが特定のデバイスにバインドされなくなったことは「管理対象のデバイスしか認証に使わせたくない企業にとってはリスクである」とOktaは捉えており、この課題へのワークアラウンドとしてOktaアカウントのMFA登録時にPasskeyの新規登録をブロックするオプションをリリースしています。

https://www.okta.com/blog/2022/09/okta-passkey-management-a-new-feature-flag/

PasskeyおよびFIDO認証をサポートするエンタープライズ向けのサービスにおいてPasskeyを使った認証時にデバイスを検証・識別できるような実装が進むまでの間は、厳格なセキュリティポリシーを定めるエンドユーザー組織側においてはOktaが課題と捉えているような点について考慮が必要になることが考えられます。

# Passkeyに対する1Passwordのアプローチ
Passkeyを取り巻くプレーヤーの中で特殊な立ち位置に見えるのが、パスワードマネージャーを提供する1Passwordです。
1Passwordは、1Passwordのブラウザ拡張機能がPasskeyにおける認証器として振る舞うような機能をリリースすることを発表しています。

https://www.future.1password.com/passkeys/

1Passwordはこの記事の中でApple, Google, Microsoftとは違ってクロスプラットフォームでPasskeyを同期できるということから「自分達こそが本来のPasskeyだ」ということを主張しています。

組織でPasskeyを利用する際の懸念事項のセクションでも書いたように現状のプラットフォーム3社(MSはまだ発表していませんが)が提供するPasskeyの制限を考えると、1Passwordのアプローチは企業におけるPasskey活用の選択肢としては比較的導入しやすく現実的なように個人的には感じられます。が、こちらもまだデモの段階なのでGAになるころには取り巻く環境も変わっているかもしれません。

# まとめ
- Passkeyはまだリリースされたばかりでこれから様々な拡張や実装が進められる段階
- 一方でエンドユーザーとしては既に利用が可能な状態であるため、組織としてどういう扱いにするのかは考えておく必要がありそう
- セキュリティと利便性の両立を推進する技術として今後に期待したいですね 😃



[^1]: [最新のGoogle Chrome 108で「Passkey」が利用可能に。Googleアカウントでクレデンシャル同期など － Publickey](https://www.publickey1.jp/blog/22/google_chrome_108passkeygoogle.html)
[^2]: [Apple「passkey」、iOS 16とmacOS Venturaで利用可能に-パスワードのない未来へ | TECH+（テックプラス）](https://news.mynavi.jp/techplus/article/20220916-2455568/)
[^3]: [報道発表資料 : dアカウントのログインにおける新たな認証手段（Web認証・パスキー）の提供を開始 | お知らせ | NTTドコモ](https://www.docomo.ne.jp/info/news_release/2022/10/17_00.html)
[^4]: [パスキーの概要 - Apple Developer](https://developer.apple.com/jp/passkeys/)
[^5]: [管理対象Apple IDのセキュリティ - Apple サポート (日本)](https://support.apple.com/ja-jp/guide/security/sec049674014/web)
[^6]: [Google Developers Japan: Google Password Manager のパスキーのセキュリティ](https://developers-jp.googleblog.com/2022/11/security-of-passkeys.html)
[^7]: NIST SP 800-63-3 で定義されている Authenticator Assurance Level https://pages.nist.gov/800-63-3/sp800-63-3.html
[^8]: [OktaでPasskeyが利用できるようになっていたのでいろいろ確認してみた – CloudNative Inc. BLOGs](https://blog.cloudnative.co.jp/13500/)