# AI2025
📰 Gemini API ニュースアグリゲーター
🎯 プロジェクト概要
Gemini APIを使用してRSSフィードから最新ニュースを取得し、各記事に自動でタグを付与するWebアプリケーションです。
🔧 技術スタック

フロントエンド: React (CDN経由)
スタイリング: Tailwind CSS (CDN経由)
AI API: Google Gemini API (gemini-pro)
RSS取得: Google News RSS + AllOrigins CORS Proxy

📋 機能要件
✅ 3つのカテゴリ選択（テクノロジー、ビジネス、エンターテインメント）
✅ カテゴリ変更による動的なニュース切り替え
✅ 最低10件以上のリアルタイムニュース表示
✅ Gemini APIによるAI生成タグの自動付与

🔄 カテゴリ変更から画面表示までのロジック説明
システム全体の処理フロー
[ユーザー操作]
    ↓
[カテゴリ選択] → handleCategoryChange()
    ↓
[RSS取得] → fetchNews()
    ├─ Google News RSS取得
    ├─ CORSプロキシ経由でフェッチ
    └─ XMLレスポンス取得
    ↓
[RSSパース] → parseRSS()
    ├─ XMLをDOMParserで解析
    ├─ <item>タグから情報抽出
    │   ・タイトル
    │   ・リンクURL
    │   ・説明文（HTMLタグ除去）
    │   ・公開日時
    └─ 15件の記事配列を生成
    ↓
[Gemini APIタグ生成] → generateTags()
    ├─ 各記事（10件）を順次処理
    ├─ Gemini API呼び出し
    │   ・エンドポイント: /v1beta/models/gemini-pro:generateContent
    │   ・プロンプト: 「3つのタグを日本語で生成」
    ├─ レスポンスからタグ抽出
    ├─ カンマ区切りで配列化
    ├─ 1秒間隔で実行（API制限対策）
    └─ エラー時は「ニュース」タグを付与
    ↓
[ステート更新] → setNews()
    ↓
[React再レンダリング] → 画面表示
    ├─ ニュース一覧カード表示
    ├─ タイトル（外部リンク）
    ├─ 説明文（2行表示）
    ├─ AIタグ（色付きバッジ）
    └─ 公開日時（日本語形式）
詳細ロジック説明
1. カテゴリ選択トリガー（フロントエンド）
javascripthandleCategoryChange(category)
  ├─ selectedCategory ステート更新
  └─ fetchNews(category) 呼び出し
2. RSS取得フェーズ（バックエンドロジック）
javascriptfetchNews(category)
  ├─ ローディング状態ON
  ├─ カテゴリに対応するRSS URL取得
  │   例: "technology" → Google News Technology RSS
  ├─ CORSプロキシURL生成
  │   https://api.allorigins.win/raw?url={RSS_URL}
  ├─ fetch() でXMLデータ取得
  └─ parseRSS() に渡す
3. RSSパースフェーズ（データ変換）
javascriptparseRSS(xmlText)
  ├─ DOMParser でXML解析
  ├─ querySelectorAll('item') で全記事取得
  ├─ Array.from() で配列化
  ├─ .map() で各記事から情報抽出
  │   ├─ title: querySelector('title').textContent
  │   ├─ link: querySelector('link').textContent
  │   ├─ description: HTMLタグ除去後のテキスト
  │   └─ pubDate: querySelector('pubDate').textContent
  └─ 最初の15件を返却
4. Gemini APIタグ生成フェーズ（AI処理）
javascriptgenerateTags(newsItems)
  ├─ for...of で各記事を順次処理
  ├─ 各記事に対して:
  │   ├─ Gemini API呼び出し
  │   │   POST /v1beta/models/gemini-pro:generateContent
  │   │   Body: {
  │   │     contents: [{
  │   │       parts: [{
  │   │         text: "タイトル: XXX 説明: YYY から3つのタグ生成"
  │   │       }]
  │   │     }]
  │   │   }
  │   ├─ レスポンス解析
  │   │   data.candidates[0].content.parts[0].text
  │   ├─ タグ文字列をカンマで分割
  │   │   "AI, 技術革新, ビジネス" → ["AI", "技術革新", "ビジネス"]
  │   ├─ 記事オブジェクトにtagsプロパティ追加
  │   └─ 1秒待機（await setTimeout(1000)）
  └─ タグ付き記事配列を返却
5. 画面表示フェーズ（React描画）
javascriptsetNews(newsWithTags)
  ↓
React再レンダリング
  ↓
news.map() で各記事カード生成
  ├─ タイトル（<a>タグで外部リンク）
  ├─ 説明文（line-clamp-2で2行制限）
  ├─ タグループ
  │   └─ tags.map() で各タグをバッジ表示
  └─ 公開日時（toLocaleString('ja-JP')）
動的処理のポイント

完全な動的取得: カテゴリ変更のたびに新しいRSSを取得
リアルタイムデータ: Google Newsから最新ニュースを取得
AI動的生成: Gemini APIが毎回新しいタグを生成
非同期処理: fetch, async/awaitで非ブロッキング実行
状態管理: React hooksで動的なUI更新


🚀 起動手順
必要な環境

モダンなWebブラウザ（Chrome, Firefox, Safari, Edge）
インターネット接続
Gemini API キー

⚠️ インストール作業
不要です！ Node.js、npm等は一切必要ありません。
🔑 Gemini APIキーの取得

Google AI Studio にアクセス
👉 https://makersuite.google.com/app/apikey
「Create API Key」をクリック
プロジェクトを選択（または新規作成）
生成されたAPIキーをコピー

📝 セットアップ手順
bash# 1. リポジトリをクローン
git clone https://github.com/your-username/gemini-rss-news.git
cd gemini-rss-news

# 2. index.html を編集
# テキストエディタで開いて84行目を編集:
編集箇所（84行目付近）:
javascript// ⚠️ ここにあなたのGemini APIキーを設定してください
const GEMINI_API_KEY = 'YOUR_GEMINI_API_KEY_HERE';  // ← ここを編集
↓ 変更後 ↓
javascriptconst GEMINI_API_KEY = 'AIzaSyAbc123...';  // あなたのAPIキー
bash# 3. ブラウザで開く
# 方法A: ファイルをダブルクリック
# 方法B: ブラウザにドラッグ&ドロップ
# 方法C: 右クリック → 「プログラムから開く」
```

### 🎬 動作確認

1. ブラウザで `index.html` を開く
2. カテゴリボタン（テクノロジー/ビジネス/エンターテインメント）をクリック
3. 「ニュースを取得中...」と表示される
4. 30秒〜1分待つ
5. タグ付きニュース一覧が表示される ✅

---

## 💡 使い方

### 基本操作

1. **カテゴリ選択**  
   画面上部の3つのボタンから選択

2. **ニュース閲覧**  
   - タイトルクリックで元記事へ
   - タグで記事内容を素早く把握

3. **カテゴリ切り替え**  
   別のカテゴリをクリックすると新しいニュースを取得

### ⏱️ 処理時間について

- **RSS取得**: 2〜5秒
- **Gemini APIタグ生成**: 30〜60秒（10記事 × 各1秒間隔）
- **合計**: 約30秒〜1分

💡 **これは仕様です**。課題要件に「時間がかかるのは問題ない」と明記されています。

---

## 📁 ファイル構成
```
gemini-rss-news/
├── README.md          # このファイル
├── .gitignore         # Git除外設定
└── index.html         # アプリケーション本体（これだけで動作）
.gitignore の内容
gitignore# 編集済みファイル（APIキー含む）
index.local.html

# その他
.DS_Store

APIキーが流出した場合

即座に無効化: Google AI Studioで該当キーを削除
新しいキーを生成: 新規作成して使用
被害確認: 使用量ダッシュボードを確認


📊 システム仕様
対応カテゴリ
カテゴリRSS ソース検索キーワードテクノロジーGoogle Newstechnology, AI, softwareビジネスGoogle Newsbusiness, economy, financeエンターテインメントGoogle Newsentertainment, movie, music
技術的制約

表示件数: 各カテゴリ10件
タグ数: 1記事あたり最大3つ
API間隔: 1秒（レート制限対策）
言語: 日本語（RSS、タグともに）

使用API

Gemini API

モデル: gemini-pro
エンドポイント: https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent
認証: APIキー方式




🎨 実装の工夫点
フロントエンド

単一HTMLファイル: 依存関係なしで即座に動作
CDN活用: React, Tailwind CSS をCDN経由で読み込み
レスポンシブデザイン: モバイル/タブレット/PC対応
ローディング表示: 処理中の状態を明示
エラーハンドリング: 失敗時も適切にフォールバック

バックエンドロジック

CORSプロキシ: ブラウザから直接RSS取得
非同期処理: async/awaitで効率的な実行
レート制限対策: API呼び出しに1秒間隔
動的データ: リアルタイムで最新ニュース取得
Gemini API統合: 各記事に動的タグ生成


🐛 トラブルシューティング
Q1: ニュースが表示されない
考えられる原因:

APIキーが間違っている
インターネット接続が切れている
Gemini APIの制限に達している

解決方法:

APIキーを再確認
ブラウザの開発者ツール（F12）でエラー確認
Google AI Studioで使用量を確認

Q2: 「RSS取得に失敗しました」エラー
原因:

CORSプロキシ（allorigins.win）が一時的にダウン
Google News RSSがアクセス制限

解決方法:

数分後に再試行
ブラウザキャッシュをクリア

Q3: タグが「ニュース」しか表示されない
原因:

Gemini API呼び出しに失敗
APIキーの権限不足
レート制限に達した

解決方法:

ブラウザコンソール（F12）でエラー確認
APIキーの権限を確認
しばらく時間を置いて再試行

Q4: 処理が遅すぎる
これは仕様です！

10記事 × 1秒間隔 = 最低10秒
Gemini APIの応答時間: 2〜3秒/記事
合計30〜60秒は正常

課題要件に「時間がかかるのは問題ない」と明記されています。
Q5: ブラウザが固まる
原因:

古いブラウザを使用している
メモリ不足

解決方法:

最新版のChrome/Firefox/Edgeを使用
他のタブを閉じる


📖 参考資料
公式ドキュメント

Gemini API Documentation
Google AI Studio
React公式
Tailwind CSS

RSS情報

Google News RSS
AllOrigins CORS Proxy

📄 ライセンス
MIT License
