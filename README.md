## モデルのカスタマイズ

### GGUF からのインポート

Ollama では、`Modelfile` を使って GGUF モデルをインポートできます：

1. モデルをインポートするために、ローカルのモデルファイルパスを指定した `FROM` 命令を含む `Modelfile` というファイルを作成します。
   ```
   FROM ./vicuna-33b.Q4_0.gguf
   ```

2. Ollama でモデルを作成します。
   ```
   ollama create example -f Modelfile
   ```

3. モデルを実行します。
   ```
   ollama run example
   ```

### プロンプトのカスタマイズ

`Modelfile` を作成します：
```
FROM llama3.1

# temperature を 1 に設定 [高いほど創造的、低いほど一貫性がある]
PARAMETER temperature 1

# システムメッセージを設定
SYSTEM """
You are Mario from Super Mario Bros. Answer as Mario, the assistant, only.
"""
```

その後、モデルを作成して実行します：
```
ollama create mario -f ./Modelfile
ollama run mario
>>> hi
Hello! It's your friend Mario.
```

`Modelfile` の使い方についての詳細は [Modelfile](https://github.com/ollama/ollama/blob/main/docs/modelfile.md) ドキュメントをご覧ください。

## CLI リファレンス

### モデルの作成

`ollama create` コマンドは、`Modelfile` からモデルを作成するために使用します。
```
ollama create mymodel -f ./Modelfile
```

### モデルの取得

```
ollama pull llama3.1
```

### モデルの削除

```
ollama rm llama3.1
```

### モデルのコピー

```
ollama cp llama3.1 my-model
```

### 複数行の入力

複数行の入力は、テキストを `"""` で囲むことで対応できます：

```
>>> """Hello,
... world!
... """
I'm a basic program that prints the famous "Hello, world!" message to the console.
```

### マルチモーダルモデル

```
ollama run llava "What's in this image? /Users/jmorgan/Desktop/smile.png"
The image features a yellow smiley face, which is likely the central focus of the picture.
```

### プロンプトを引数として渡す

```
$ ollama run llama3.1 "Summarize this file: $(cat README.md)"
 Ollama is a lightweight, extensible framework for building and running language models on the local machine. It provides a simple API for creating, running, and managing models, as well as a library of pre-built models that can be easily used in a variety of applications.
```

### モデル情報の表示

```
ollama show llama3.1
```

### コンピュータ上のモデルの一覧表示

```
ollama list
```

### Ollama の起動

`ollama serve` コマンドは、デスクトップアプリケーションを実行せずに Ollama を起動したいときに使用します。
