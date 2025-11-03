# 社内情報特化型生成AI検索アプリ

## 概要
このアプリケーションは、社内文書やWebページの情報を活用したRAG（Retrieval-Augmented Generation）ベースの検索・問い合わせシステムです。Streamlitを使用したWebインターフェースで、LangChainとOpenAI GPTを活用して、社内の情報に素早くアクセスできます。

## 主な機能

### 1. 社内文書検索
- ユーザーの入力内容と関連性が高い社内文書のありか（ファイルパス、ページ番号）を検索
- 複数の候補文書を優先度順に表示
- PDFなどのページ番号も表示可能

### 2. 社内問い合わせ
- 質問や要望に対して、社内文書の情報をもとにAIが回答を生成
- 回答の根拠となった情報源（ファイル、ページ番号）を明示
- マークダウン形式での詳細な回答表示

### 3. 会話履歴の保持
- セッション内での会話履歴を保持し、文脈を考慮した対話が可能
- 過去のやり取りを参照した連続的な質問に対応

## システム構成

### ファイル構成

```
company_inner_search_app/
├── main.py                    # メインアプリケーション
├── initialize.py              # 初期化処理（RAG環境構築）
├── components.py              # 画面表示関連の関数
├── constants.py               # 定数・設定管理
├── utils.py                   # ユーティリティ関数
├── requirements.txt           # 依存パッケージ（Windows）
├── requirements_mac.txt       # 依存パッケージ（Mac）
├── requirements_windows.txt   # 依存パッケージ（Windows）
├── .env                       # 環境変数（OpenAI APIキー）
├── .streamlit/
│   └── config.toml           # Streamlit設定ファイル
├── data/                      # 参照データソース
│   ├── MTG議事録/
│   ├── サービスについて/
│   ├── 会社について/
│   ├── 顧客について/
│   └── 社員について/
└── logs/                      # アプリケーションログ
    └── application.log
```

### 主要コンポーネント

#### [main.py](main.py)
- アプリケーションのエントリーポイント
- ページ設定、初期化、会話フローの制御
- ユーザー入力の受け付けとLLMレスポンスの表示

#### [initialize.py](initialize.py)
- データソースの読み込み（ローカルファイル、Webページ）
- ベクターストアの構築（Chroma）
- Retrieverの作成
- ロガーの設定

#### [components.py](components.py)
- 画面表示専用の関数群
- タイトル、モード選択、会話ログ表示
- 検索結果・問い合わせ結果の整形表示

#### [constants.py](constants.py)
- アプリケーション全体で使用する定数
- プロンプトテンプレート
- LLM設定（モデル、temperature、chunk設定）
- エラーメッセージ

#### [utils.py](utils.py)
- LLMとの対話処理
- プロンプトチェーンの構築
- 会話履歴の管理

## セットアップ

### 1. 必要な環境
- Python 3.8以上
- OpenAI APIキー

### 2. インストール手順

#### Windowsの場合
```bash
pip install -r requirements_windows.txt
```

#### Macの場合
```bash
pip install -r requirements_mac.txt
```

### 3. 環境変数の設定

`.env`ファイルを作成し、OpenAI APIキーを設定：

```
OPENAI_API_KEY=your_api_key_here
```

### 4. データソースの準備

`data/`フォルダに参照したい文書を配置します。

**サポートされているファイル形式：**
- PDF (`.pdf`)
- Word (`.docx`)
- CSV (`.csv`)
- テキスト (`.txt`)

**フォルダ構成例：**
```
data/
├── MTG議事録/
├── サービスについて/
├── 会社について/
├── 顧客について/
└── 社員について/
```

### 5. アプリケーションの起動

```bash
streamlit run main.py
```

ブラウザで自動的に `http://localhost:8501` が開きます。

## 使い方

### 基本的な操作フロー

1. **モード選択**
   - サイドバーで「社内文書検索」または「社内問い合わせ」を選択

2. **メッセージ入力**
   - 画面下部のチャット入力欄にメッセージを入力
   - 具体的な内容を入力するほど精度が向上

3. **回答の確認**
   - 「社内文書検索」：関連文書のファイルパスとページ番号が表示
   - 「社内問い合わせ」：AIによる回答と情報源が表示

### 入力例

#### 社内文書検索の場合
```
社員の育成方針に関するMTGの議事録
```

#### 社内問い合わせの場合
```
人事部に所属している従業員情報を一覧化して
```

## 技術仕様

### 使用技術スタック

| カテゴリ | 技術 |
|---------|------|
| フレームワーク | Streamlit 1.41.1 |
| LLM | OpenAI GPT-4o-mini |
| LLMフレームワーク | LangChain 0.3.17 |
| ベクターストア | ChromaDB 0.5.0 |
| 埋め込みモデル | OpenAI Embeddings |
| ドキュメント処理 | PyMuPDF, python-docx, beautifulsoup4 |

### RAG（Retrieval-Augmented Generation）の仕組み

1. **データ読み込み**
   - ローカルファイルとWebページを読み込み
   - サポートされた形式のファイルを自動検出

2. **チャンク分割**
   - 文書を小さなチャンクに分割（chunk_size: 500, chunk_overlap: 50）
   - 改行文字で区切り

3. **ベクトル化**
   - OpenAI Embeddingsで各チャンクをベクトル化
   - ChromaDBに保存

4. **検索・回答生成**
   - ユーザー入力をベクトル化し、類似度の高いチャンクを取得（k=5）
   - 取得したチャンクを参照してLLMが回答を生成

### LLM設定

```python
MODEL = "gpt-4o-mini"
TEMPERATURE = 0.5
chunk_size = 500
chunk_overlap = 50
```

### プロンプト戦略

#### 社内文書検索
- 入力と文脈の関連性を判定
- 関連性が高い場合は空文字、低い場合は「該当資料なし」を返す

#### 社内問い合わせ
- 文脈に基づいた詳細な回答を生成
- マークダウン形式で構造化された回答
- 文脈に情報がない場合は「回答に必要な情報が見つかりませんでした」と返答

## ログ機能

### ログの保存場所
`logs/application.log`

### ログ出力内容
- アプリケーションの起動
- ユーザーメッセージ
- AIメッセージ
- エラー情報
- セッションID（ユーザー識別用）

### ログローテーション
- 1日単位で自動的にログファイルをローテーション
- 日付ごとに分けて保存

## Windows環境対応

Windows環境での文字コード問題に対応するため、以下の処理を実装：

- Unicode正規化（NFC）
- cp932（Shift-JIS）で表現できない文字の除去
- User-Agent設定による403エラー回避

## トラブルシューティング

### よくある問題と解決方法

#### 1. 初期化エラー
**症状：** アプリ起動時に初期化エラーが発生

**解決方法：**
- `.env`ファイルにOpenAI APIキーが正しく設定されているか確認
- `data/`フォルダが存在し、適切なファイルが配置されているか確認

#### 2. Web読み込みエラー
**症状：** Webページの読み込みに失敗

**解決方法：**
- `.env`ファイルに`USER_AGENT`が設定されているか確認（自動設定されます）
- インターネット接続を確認

#### 3. 文字化け（Windows）
**症状：** 日本語が文字化けする

**解決方法：**
- `initialize.py`の`adjust_string`関数が正しく動作しているか確認
- ファイルがUTF-8エンコーディングで保存されているか確認

#### 4. 回答が得られない
**症状：** 「該当資料なし」または「回答に必要な情報が見つかりませんでした」と表示

**解決方法：**
- より具体的な質問内容に変更
- データソースに関連情報が含まれているか確認
- chunk_sizeやkパラメータの調整を検討

## カスタマイズ

### データソースの追加

#### ローカルファイル
`data/`フォルダにファイルを追加するだけで自動的に読み込まれます。

#### Webページ
[constants.py:56-58](constants.py#L56-L58)の`WEB_URL_LOAD_TARGETS`リストにURLを追加：

```python
WEB_URL_LOAD_TARGETS = [
    "https://example.com/page1",
    "https://example.com/page2"
]
```

### LLMモデルの変更

[constants.py:42](constants.py#L42)で変更可能：

```python
MODEL = "gpt-4o-mini"  # または "gpt-4", "gpt-3.5-turbo" など
```

### チャンク設定の調整

[constants.py:44-45](constants.py#L44-L45)で調整可能：

```python
chunk_size = 500        # チャンクのサイズ
chunk_overlap = 50      # チャンク間の重複
```

### 検索結果数の変更

[initialize.py:154](initialize.py#L154)で調整可能：

```python
st.session_state.retriever = db.as_retriever(search_kwargs={"k": 5})
```

## セキュリティ注意事項

- `.env`ファイルは絶対にGitにコミットしないでください
- OpenAI APIキーは厳重に管理してください
- 機密情報を含む文書を扱う場合は、適切なアクセス制御を実施してください

## ライセンス

このプロジェクトの使用には、以下のライセンスが適用される場合があります：
- OpenAI API利用規約
- 各種依存ライブラリのライセンス

## 開発情報

### 動作確認環境
- Python 3.8+
- Windows 10/11, macOS

### 主要な依存パッケージバージョン
- streamlit==1.41.1
- langchain==0.3.17
- langchain-openai==0.3.3
- chromadb==0.5.0
- openai==1.60.2

## お問い合わせ

エラーが繰り返し発生する場合は、管理者にお問い合わせください。
