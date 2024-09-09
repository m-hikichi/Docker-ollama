# llama.cppを用いたGGUF変換

## 1. 必要なソフトウェアとツールをインストール

以下のソフトウェアとツールがインストールされていることを確認してください：
- Git
- Python 3
- pip（Pythonパッケージマネージャー）

## 2. リポジトリをクローン

コマンドライン（ターミナル）を開いて、次のコマンドを実行します。これにより、必要なコードとファイルがダウンロードされます。

```bash
git clone https://github.com/ggerganov/llama.cpp.git && cd llama.cpp
```

## 3. 必要なPythonライブラリをインストール

プロジェクトで必要なPythonライブラリをインストールします。以下のコマンドを実行します。

```bash
pip install -r requirements.txt
```

## 4. モデルファイルを変換

Hugging Face形式のモデルをGGUF形式に変換します。以下のコマンドを実行してください。

```bash
python3 convert_hf_to_gguf.py /path/to/model --outfile /path/to/output/model.gguf
```
