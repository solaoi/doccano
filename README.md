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

<img width="725" alt="スクリーンショット 2024-12-01 21 50 31" src="https://github.com/user-attachments/assets/42fa00d2-83ba-415b-9d86-695f56e0bfee">

- [http://127.0.0.1/](http://127.0.0.1/)にアクセスし、`ADMIN_USERNAME` および `ADMIN_PASSWORD` （デフォルトでは `doccano` ）を使用してログインします。

#### 2. プロジェクトの作成

<img width="549" alt="スクリーンショット 2024-12-01 21 53 11" src="https://github.com/user-attachments/assets/52d46953-ae3a-44b1-ab76-23ce6cec10f9">

- プロジェクト作成画面で以下を設定:
  - タスクタイプ: `Text Classification`
  - プロジェクト名と説明を入力
  - `Allow single label` を有効化
- 作成後、詳細画面に遷移します。

#### 3. ラベルのインポート

<img width="1244" alt="スクリーンショット 2024-12-01 21 54 59" src="https://github.com/user-attachments/assets/e91b8123-803d-4696-b2ab-553f3cf208a7">

- サイドバーの `Labels` から `Import Labels` を選択。
- リポジトリ内の `sample/labels.json` を選択し、インポートします。

#### 4. データセットのインポート

<img width="1244" alt="スクリーンショット 2024-12-01 21 56 01" src="https://github.com/user-attachments/assets/3b642629-c871-4eb7-8e57-455d47243df8">

- サイドバーの `Dataset` から `Import Dataset` を選択。
- ファイル形式 `JSONL` を選び、以下のスクリプトで生成した `doccano.jsonl` をインポートします。

```python
import json

from datasets import load_dataset

# 利用したいデータセットをHugging Faceからロード
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

<img width="1483" alt="スクリーンショット 2024-12-01 22 02 13" src="https://github.com/user-attachments/assets/020a5949-db40-4758-bb98-f58b5261d3ee">

- サイドバーの `Start Annotation` をクリックしてアノテーションを開始。
- このFork版では、ショートカットキーを活用し片手で操作ができます。
  - `g` キーで `Good` ラベルを選択
  - `b` キーで `Bad` ラベルを選択
  - `a` キーで上記ラベルを承認（approve）
  - `Space` キーで次のデータへ進む
  - `z` キーで前のデータに戻る

#### 6. 結果のエクスポート

- アノテーションが完了したら、 `Dataset` から `Export Dataset` を使用して結果をエクスポートします。
