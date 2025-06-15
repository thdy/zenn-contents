---
title: "Microsoft Docs 公式 MCP サーバーを使って Microsoft のドキュメントを読みやすくする"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["mcp", "ai", "microsoft", "azure"]
published: false
---
先日、Microsoft から公式の Microsoft Docs MCP サーバーがリリースされました。

https://github.com/MicrosoftDocs/mcp

自分自身これまで Microsoft のドキュメントから必要な情報を探し出したり、書かれている内容を正確に解釈することにとても苦労してきました。
この MCP サーバーによって Microsoft のドキュメントを読むすべての人たちの苦労が軽減することを期待しつつ、実際に VS Code と Cursor の 2 種類のエディタそれぞれでこの MCP サーバーを設定し使ってみたので、その結果について記載します。

# 概要
Microsoft Docs MCP サーバーは、MCP（Model Context Protocol）に対応するクライアントを使って Microsoft の公式ドキュメントから直接正確な情報を検索・取得できるクラウドホスト型サービスです。利用する上で特にアカウントの作成や課金は必要ありません。

特徴として以下のような点があります（リポジトリの README からの意訳）

- Microsoft Learn、Azure や Microsoft 365 のドキュメント、その他 Microsoft の公式な情報源からの情報取得
- セマンティック検索・適切なチャンク化による関連性の高い情報の抽出
- 最新の Microsoft 公式ドキュメントからのリアルタイム検索

# 必要なもの
以下のいずれかのエディタや Claude Desktop など、MCP に対応したクライアントがあれば Microsoft Docs MCP サーバーを利用できます。

- [Visual Studio Code](https://code.visualstudio.com/) (以下 VS Code)
- [Cursor](https://www.cursor.com/)

GitHub のアカウントを持っていなくても MCP サーバーは利用可能なので、非エンジニアな方も気軽にトライしてみてください。

# VS Code から使ってみる
今回の Microsoft Docs MCP サーバーの GitHub リポジトリには、ワンクリックでインストールできるリンクが用意されているのでそれを使います。
![](/images/microsoft-docs-mcp/1749919038707.png)
*https://github.com/MicrosoftDocs/mcp?tab=readme-ov-file#-installation--getting-started*

One-click Installation の列にある `VS Code Install Microsoft Docs MCP` をクリックすると下記のようなポップアップが表示されるので `Visual Studio Code を開く` を選択します。
![](/images/microsoft-docs-mcp/1749918664594.png)

VS Code が開き、下記のような画面が表示されるので `サーバーのインストール` を選択します。
![](/images/microsoft-docs-mcp/1749919607685.png)

インストール手順は以上です。
本当にちゃんと入っているのか気になる方は `settings.json` を開いて下記のような記述があることを確認してください。

```json:settings.json
  "mcp": {
    "servers": {
      "microsoft.docs.mcp": {
        "type": "http",
        "url": "https://learn.microsoft.com/api/mcp"
      }
    }
  }
```

それでは実際に VS Code に搭載されている GitHub Copilot のチャットウィンドウから MCP サーバーを使ってみます。
ここでは明示的に Microsoft Docs の MCP サーバーを使うために `コンテキストの追加` > `ツール` > `microsoft.docs.mcp` を指定して質問を投げています。
![](/images/microsoft-docs-mcp/1749920180421.png)

質問すると、Copilot が MCP サーバーに渡す質問クエリを提示し続行を求めてきたので続行します。
![](/images/microsoft-docs-mcp/1749920327092.png)

いい情報が見つからなかったのか、検索クエリを変えて再度続行を求めてきたので続行します。
![](/images/microsoft-docs-mcp/1749920417309.png)

回答が返ってきました。公式の Learn のページより断然読みやすい…！
![](/images/microsoft-docs-mcp/1749920619661.png)

根拠となるソースの URL は回答の中になかったので聞いてみたら答えてくれました。
![](/images/microsoft-docs-mcp/1749920807532.png)

VS Code の GitHub Copilot で動かしているので、返ってきた回答をそのまま markdown ファイルに書いてもらって同僚に共有したり、既存のドキュメントに注釈として書き込んでもらうような使い方もできそうですね。

# Cursor から使ってみる
基本的には VS Code と同じような内容になりますが、Cursor でも使ってみた結果を記載します。

VS Code と同様に、Microsoft Docs MCP サーバーの GitHub リポジトリに Cursor 用にワンクリックでインストールできるリンクが用意されているのでそれを使います。
![](/images/microsoft-docs-mcp/1749919038707.png)
*https://github.com/MicrosoftDocs/mcp?tab=readme-ov-file#-installation--getting-started*

One-click Installation の列にある `Add to Cursor` をクリックすると下記のようなポップアップが表示されるので `Cursor を開く` を選択します。
![](/images/microsoft-docs-mcp/1749921737839.png)

Cursor に下記のような画面が表示されるので `Install` を選択します。
![](/images/microsoft-docs-mcp/1749921712429.png)

インストールが完了すると Cursor Settings の MCP Tools に `microsoft.docs.mcp` が表示されます。
![](/images/microsoft-docs-mcp/1749946777068.png)

準備が整ったので早速使ってみます。
Cursor では明示的な MCP サーバーの指定方法がなく AI が自動的に判断するため、プロンプトの中で microsoft.docs.mcp を使うことを指示してみます。
![](/images/microsoft-docs-mcp/1749957108093.png)

うまく MCP サーバーを使ってくれたようです。正確な情報を見つけるために検索クエリを何度か変えながら続行を求めてきたので `Run tool` で続行します。
![](/images/microsoft-docs-mcp/1749957130775.png)
![](/images/microsoft-docs-mcp/1749957147622.png)
*クリックして展開すると検索クエリを確認することも可能です*

回答が返ってきました。分かりやすい…
![](/images/microsoft-docs-mcp/1749957161674.png)

調べてもらった内容を Cursor 上から次のアクションに繋げられるのも便利ですね。
![](/images/microsoft-docs-mcp/1749957678697.png)
![](/images/microsoft-docs-mcp/1749957722342.png)
*markdown ファイルにまとめてもらいました*

# 感想
これまで Microsoft のドキュメントを探して読む際に「知りたい情報が記載されたページの見つけやすさ」と「仕様の把握しやすさ」という ２ 点について課題を感じていました。

まず前者のページの特定という点においては、AI によって自然言語からの検索ができたり、直接的なキーワードが分からなくても意図を汲み取ってくれることで飛躍的に探しやすくなったと感じています。

後者の仕様の把握しやすさという点については、リリースされたてということもありこの MCP サーバーからの回答のみで信用しきるかは状況次第かなと考えており、しばらくは実際のドキュメントも合わせて読んだり実環境を触って裏を取ったりすると思いますが、ある程度の情報があれば済むようなシーン（たとえば一度理解した内容にアップデートが入っていないかの確認など）については既にこの MCP サーバーだけで完結できそうな感じがしています。

公式の情報によると今後ドキュメントソースの追加による対応範囲の拡大や、より正確な結果を出すためのクエリ精度の向上が行われるようなので引き続き使っていきたいと思います。

# 関連リソース
- Microsoft が公開しているその他の MCP サーバー: https://github.com/microsoft/mcp
