<div align="center">
  <img src="https://raw.githubusercontent.com/doccano/doccano/master/docs/images/logo/doccano.png">
</div>

# doccanoのテキスト分類にMarkdown処理を追加したFork版

**doccano**は、オープンソースのアノテーションツールで、テキスト分類、系列ラベリング、系列変換といったタスク向けの機能を提供します。このFork版では、Markdown形式でのテキスト処理を追加しています。

## 利用イメージ

![Demo image](https://raw.githubusercontent.com/doccano/doccano/master/docs/images/demo/demo.gif)

## 公式ドキュメント

[https://doccano.github.io/doccano/](https://doccano.github.io/doccano/)

## 機能

- 複数人でのアノテーション
- 多言語サポート
- モバイルサポート
- ダークテーマ
- REST APIのサポート

## 環境構築方法

### Docker Composeを使用したセットアップ

1. **リポジトリをクローン**

```bash
git clone https://github.com/solaoi/doccano.git
cd doccano
```

Windowsユーザー向けの注意: 改行コード処理の問題を避けるために以下のオプションを使用してください。

```bash
git clone https://github.com/solaoi/doccano.git --config core.autocrlf=input
cd doccano
```

2. `.env` ファイルの作成

```bash
cp ./docker/.env.example .env
```

3. Docker Composeで起動

```bash
docker compose -f docker/docker-compose.prod.yml --env-file .env up
```

起動後、ブラウザで[http://127.0.0.1/](http://127.0.0.1/)にアクセスしてください。

### doccano上でのテキスト分類手順

#### 1. ログイン

- [http://127.0.0.1/](http://127.0.0.1/)にアクセスし、` ADMIN_USERNAME` および `ADMIN_PASSWORD` （デフォルトでは `doccano` ）を使用してログインします。

#### 2. プロジェクトの作成

- プロジェクト作成画面で以下を設定:
  - タスクタイプ: `Text Classification`
  - プロジェクト名と説明を入力
  - `Allow single label` を有効化
- 作成後、詳細画面に遷移します。

#### 3. ラベルのインポート

- サイドバーの `Labels` から `Import Labels` を選択。
- リポジトリ内の `sample/labels.json` を選択し、インポートします。

#### 4. データセットのインポート

- サイドバーの `Dataset` から `Import Dataset` を選択。
- ファイル形式 `JSONL` を選び、以下のスクリプトで生成した `doccano.jsonl` をインポートします。

```python
import json

from datasets import load_dataset

# データセットをHugging Faceからロード
dataset = load_dataset("llm-jp/magpie-sft-v1.0")

# 必要なフィールドを生成する処理（データセットに応じて編集ください）
def formatter(example):
    formatted_conversations = ""
    for conv in example["conversations"]:
        if conv["role"] == "user":
            formatted_conversations += f"**\[user\]**\n{conv['content']}\n"
            question = conv['content']
        elif conv["role"] == "assistant":
            formatted_conversations += f"\n**\[assistant\]**\n{conv['content']}\n"
            answer = conv['content']

    return {
        "original_id": example["id"],
        "question": question,
        "answer": answer,
        # このtext項目がラベル（goodまたはbad）で評価する対象となる
        "text": formatted_conversations,
        "label": [],
    }


# データセットを変換し、不要なフィールドを削除
processed_dataset = dataset.map(formatter, remove_columns=dataset["train"].column_names)

# JSONL形式で保存
output_path = "doccano.jsonl"
processed_dataset["train"].to_json(output_path, orient="records", lines=True)

print(f"JSONL形式のデータセットを以下のパスに保存しました: {output_path}")
```

#### 5. アノテーションの開始

- サイドバーの `Start Annotation` をクリックしてアノテーションを開始。
- ショートカットキー（例: `g` キーで `Good` ラベルを選択）を活用できます。

#### 6. 結果のエクスポート

- アノテーションが完了したら、 `Dataset` から `Export Dataset` を使用して結果をエクスポートします。
