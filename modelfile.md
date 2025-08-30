# Ollama Modelfileの書き方ガイド

> **対象**: Ollama未経験者／Modelfileで独自モデルの構築・カスタマイズをしたい方
> **形式**: README向けMarkdown
> **概要**: Modelfileの基本構造・主要ディレクティブ・作成手順・CLI操作・実践サンプル・つまずき対策を網羅

---

## 目次

* [はじめに](#はじめに)
* [Modelfileの基本構造と文法](#modelfileの基本構造と文法)
* [主なディレクティブの一覧と役割](#主なディレクティブの一覧と役割)

  * [FROM（必須）](#from必須)
  * [PARAMETER](#parameter)
  * [TEMPLATE](#template)
  * [SYSTEM](#system)
  * [ADAPTER](#adapter)
  * [LICENSE](#license)
  * [MESSAGE](#message)
* [独自モデルの構築手順と例](#独自モデルの構築手順と例)
* [既存モデルのカスタマイズ例](#既存モデルのカスタマイズ例)

  * [例1: 既存モデルにシステムメッセージを追加](#例1-既存モデルにシステムメッセージを追加)
  * [例2: テンプレートの変更によるカスタマイズ](#例2-テンプレートの変更によるカスタマイズ)
  * [例3: LoRAアダプタの適用](#例3-loraアダプタの適用)
* [コマンドラインでのビルド方法](#コマンドラインでのビルド方法)
* [実践的なサンプル-marioアシスタントモデル](#実践的なサンプル-marioアシスタントモデル)
* [初心者がつまずきやすいポイントとその解決策](#初心者がつまずきやすいポイントとその解決策)

---

## はじめに

**OllamaのModelfile**は、ローカル環境で大規模言語モデル(LLM)をカスタマイズ・共有するための設定ファイルです。これは既存モデルを基に新しいカスタムモデルを作成するための「設計図」の役割を果たし、いわば**LLM版のDockerfile**と考えることができます。Modelfileを使うことで、大掛かりな再学習を行わずにモデルの挙動やシステムメッセージ、生成パラメータなどを調整できます（大規模な**ファインチューニング**とは異なり、Modelfileはモデルの設定を軽量に調節するアプローチです）。本資料では、**Modelfileの基本構造**や**主要ディレクティブの役割**、**カスタムモデルの作成手順**、**既存モデルのカスタマイズ例**、**コマンドラインでのビルド方法**、**実践的な使用例**、そして**初心者がつまずきやすいポイント**について、初心者向けにわかりやすく解説します。

---

## Modelfileの基本構造と文法

Modelfileはテキストファイルで、1行ごとに`INSTRUCTION 引数`という形式で記述します。`#`から始まる行は**コメント**として無視されます。インストラクション（命令）は**大文字・小文字を区別せず**記述できますが、本ガイドでは読みやすさのため大文字で表記します。また、各インストラクションの順序に厳密な決まりはありませんが、一般的に**FROM行を最初**に置くと読みやすくなります。

```text
# コメント（この行は無視されます）
INSTRUCTION 引数1 引数2 ...
```

主なインストラクションとその役割は以下の通りです：

* **FROM** – 使用する**ベースモデル**を指定します（**必須**の指示）。
* **PARAMETER** – モデル実行時の**パラメータ**（生成設定）を指定します。
* **TEMPLATE** – モデルに渡される**プロンプトのテンプレート**（フォーマット）を定義します。
* **SYSTEM** – プロンプト内で使用される**システムメッセージ**を設定します。
* **ADAPTER** – モデルに適用する**LoRAアダプタ**（微調整結果）を指定します。
* **LICENSE** – モデルの**ライセンス文言**を指定します。
* **MESSAGE** – モデルに組み込む**対話履歴**（例示的なユーザー・アシスタントのやり取り）を追加します。

---

## 主なディレクティブの一覧と役割

### FROM（必須）

ベースにするモデルを指定します。Ollamaのモデルライブラリ上のモデル名（例: `llama2:latest` や `llama3:8b`）や、ローカルに保存したモデルファイルへのパスを指定できます。この行で指定したモデルが、新しく構築するカスタムモデルの土台となります。**タグ**（バージョンや量子化タイプ）もコロン区切りで指定可能です（例: `FROM llama3:8b`）。ベースモデル名を誤るとモデルが見つからずエラーになりますので、正確に記述してください（**モデル名のタイプミス**はよくある間違いです）。

---

### PARAMETER

モデルの**生成パラメータ**を設定します。温度 (`temperature`)、最大文脈長 (`num_ctx`)、トップK (`top_k`)、トップP (`top_p`)、生成停止トークン (`stop`) など、多くの項目を指定できます。これらを調整することで、モデルの**創造性**や**応答の多様性**、**応答の長さ**等を制御できます。例えば、`PARAMETER temperature 1`は生成のランダム性を高め（創造的な回答を促す）設定で、`PARAMETER num_ctx 4096`は文脈ウィンドウサイズを4096トークンに拡大し、より長い会話履歴を保持できるようにします。

主なパラメータの例：

* `temperature` – 応答のランダム性を制御します（高いほど創造的、低いほど一貫性重視）。
* `num_ctx` – **コンテキストウィンドウのサイズ**（モデルが一度に認識できるトークン数）を設定します。
* `top_k` / `top_p` – 次に選択されるトークンの候補数や確率分布のしきい値を制御します。
* `repeat_penalty` – 繰り返し出現する単語にペナルティを与え、同じフレーズの繰り返しを抑制します。
* `stop` – **生成停止シーケンス**を指定します。複数指定したい場合はPARAMETER行を複数用意します。

※そのほかにも`mirostat`関連、`seed`、`num_predict`など多数のパラメータが存在します。

---

### TEMPLATE

モデルに渡す**プロンプトのテンプレート**を定義します。テンプレート記述にはGo言語のテンプレート構文を使用し、以下の特殊変数が使えます:

* `{{ .System }}` – システムメッセージ
* `{{ .Prompt }}` – ユーザー入力
* `{{ .Response }}` – モデルの応答

**例（チャット形式）:**

```text
TEMPLATE """{{ if .System }}<|im_start|>system
{{ .System }}<|im_end|>
{{ end }}{{ if .Prompt }}<|im_start|>user
{{ .Prompt }}<|im_end|>
{{ end }}<|im_start|>assistant
"""
```

TEMPLATEを省略すると各モデルのデフォルトテンプレートが用いられます。

---

### SYSTEM

モデルに与える**システムメッセージ**を設定します。チャットのロール・口調・方針を定義します。

```text
SYSTEM """あなたは知的なアシスタントです。常に敬語で回答してください。"""
```

複数行は\*\*三重引用符 `"""`\*\*で囲みます。自作TEMPLATEを使う場合は`.System`をテンプレートに含めることを忘れずに。

---

### ADAPTER

ベースモデルに適用する**LoRAアダプタ**を指定します。

```text
FROM llama2:7b
ADAPTER /path/to/adapter-dir  # ディレクトリ内の*.safetensorsを適用
```

**FROMのベースモデルはアダプタ作成時と同一**である必要があります。複数行でスタッキングも可能です。

---

### LICENSE

モデルの**ライセンス情報**や使用条件を記述します。

```text
LICENSE """
This model is licensed for non-commercial use only.
"""
```

---

### MESSAGE

モデルに組み込む**対話履歴**を追加します。`system`・`user`・`assistant`のいずれかのロールを指定します。

```text
MESSAGE user 首都圏はどの都市ですか？
MESSAGE assistant 日本の首都圏は東京都を中心としたエリアを指します。
MESSAGE user 大阪は首都圏に含まれますか？
MESSAGE assistant 大阪府は首都圏には含まれません。
```

---

## 独自モデルの構築手順と例

1. **ベースモデルを選ぶ** – 公式ライブラリのモデル名、またはローカルの`.gguf`/`.safetensors`を使用。
2. **Modelfileを作成** – 任意ディレクトリに`Modelfile`を作成し、最低限`FROM`を記述。必要に応じて`PARAMETER`・`SYSTEM`などを追加。
3. **ビルド** – `ollama create <新モデル名> -f Modelfile` を実行。
4. **動作確認** – `ollama run <新モデル名>` で対話実行。

**例：皮肉屋アシスタント（Llama 3 8Bベース）**

```text
# 基本のllama3モデルを継承
FROM llama3:8b

# システムプロンプトを設定
SYSTEM """あなたは非常に皮肉屋なアシスタントです。あなたの答えは技術的に正確であるべきですが、ドライなウィットと非協力的に届けられます。"""

# 創造性を調整する（低温度＝ランダム性が少ない／より焦点を絞った内容）
PARAMETER temperature 0.5
```

```bash
# ビルド
ollama create sarcastic-llama -f ./Modelfile

# 実行
ollama run sarcastic-llama
```

---

## 既存モデルのカスタマイズ例

### 例1: 既存モデルにシステムメッセージを追加

```text
FROM llama2:latest

SYSTEM あなたはAIに関する技術アシスタントです。ユーザーからの質問に対し、正確かつ簡潔に答えてください。

PARAMETER temperature 0.7
PARAMETER num_ctx 4096
```

---

### 例2: テンプレートの変更によるカスタマイズ

Llama2系の`[INST]`形式に合わせ、**停止シーケンス**も設定。

```text
FROM llama2:latest

TEMPLATE """[INST] <<SYS>>
{{ .System }}
<</SYS>>

{{ .Prompt }}
[/INST]{{ .Response }}"""

PARAMETER stop "[INST]"
PARAMETER stop "[/INST]"
PARAMETER temperature 0.7
```

---

### 例3: LoRAアダプタの適用

```text
FROM llama2:7b
ADAPTER ./nihongo-lora/    # ディレクトリ内に .safetensors がある想定
```

`FROM`は**LoRA作成元と一致**させる。

---

## コマンドラインでのビルド方法

* **作成（ビルド）**: `ollama create <モデル名> -f <Modelfile>`

  * 例: `ollama create mymodel -f ./Modelfile`
  * オプション例: `--quantize q4_0`（量子化）

* **実行**: `ollama run <モデル名>`

  * 例: `ollama run mymodel` ／ 一回実行: `ollama run mymodel "こんにちは"`

* **ダウンロード**: `ollama pull <モデル名>`

  * 既存モデルの**差分更新**にも対応

* **一覧**: `ollama list`

* **情報表示**: `ollama show <モデル名>`（`--modelfile`でModelfile表示）

* **削除**: `ollama rm <モデル名>`

* **コピー/リネーム**: `ollama cp <既存> <新>`

* **実行中モデル一覧**: `ollama ps`

* **強制停止**: `ollama stop <モデル名>`（なければCtrl+C）

---

## 実践的なサンプル: Marioアシスタントモデル

【※】公式のLlama 3.2をベースに利用。

**Modelfile:**

```text
FROM llama3.2

# temperatureを1に設定（値を上げ創造性を高める）
PARAMETER temperature 1

# システムメッセージを設定（マリオのキャラクター付与）
SYSTEM """
You are Mario from Super Mario Bros. Answer as Mario, the assistant, only.
"""
```

**ビルド & 実行:**

```bash
ollama create mario -f ./Modelfile
ollama run mario
```

**対話例:**

```text
$ ollama run mario
>>> hi
Hello! It's your friend Mario.
```
