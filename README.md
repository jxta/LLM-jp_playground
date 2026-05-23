# LLM-jp Playground 実験ノートブック

NII LLMC が提供する [LLM-jp Playground](https://llm-jp-playground.apps.llmc.nii.ac.jp) (OpenAI 互換 API) を Jupyter Notebook でインタラクティブに試すサンプル集。

## 🚀 即起動 (mybinder.org)

ブラウザだけで実行できます。`openai` 等のパッケージをインストール済みの JupyterLab 環境が立ち上がります (初回ビルドは 1〜3 分、2 回目以降はキャッシュで数十秒)。

[![Binder - JupyterLab](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/jxta/LLM-jp_playground/main?urlpath=lab)

特定のノートブックを直接開く:

| ノートブック | 起動リンク |
|---|---|
| **01. API 基本** (動的 thinking 判別 / 内部 streaming / LaTeX レンダリング / 4 モデル比較) | [![Open 01](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/jxta/LLM-jp_playground/main?urlpath=lab%2Ftree%2Fnotebooks%2F01_api_basics.ipynb) |
| **02. GVR パイプライン** (Generator → Verifier → Reviser, multi-vote, trace 永続化) | [![Open 02](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/jxta/LLM-jp_playground/main?urlpath=lab%2Ftree%2Fnotebooks%2F02_gvr_pipeline.ipynb) |
| 🎴 **03. AI 俳句バトル** (4 モデルが詠む → 匿名化 → 互いに採点 → 自作識別分析) | [![Open 03](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/jxta/LLM-jp_playground/main?urlpath=lab%2Ftree%2Fnotebooks%2F03_haiku_battle.ipynb) |

## 🔑 API キー設定 (必須)

LLM-jp Playground は NII 内部リソースのため、利用にはアカウント・トークンが必要です。

Binder 起動後、ノートブック冒頭の `§1. 準備` セルが実行される**前に**、以下のセルを追加して実行してください:

```python
import os
os.environ["LLMJP_API_KEY"] = "ここに自分のトークン"
```

または、`§1` の `API_KEY = os.environ.get("LLMJP_API_KEY", "dummy")` の `"dummy"` 部分を直接トークンに書き換える。

> ⚠️ **セキュリティ**: トークンを含むノートブックを保存・公開しないこと。Binder セッションは揮発的ですが、`File > Download` で持ち帰る前に API キー入力セルを削除/クリアしてください。

## 📚 ノートブック構成

### `notebooks/01_api_basics.ipynb`

LLM-jp Playground API の基本的な使い方と落とし穴の回避策。

- 利用可能モデル一覧の自動取得 (`/v1/models`)
- **動的プローブで thinking モデル判別** (モデル名のサフィックスは当てにならない)
- 内部 streaming で `reasoning_content` を確実に取得 (non-streaming パスでは落ちるバグの回避)
- 自動ルーティング `smart_chat` (`/no_think` + `enable_thinking=False` 併用)
- LaTeX → MathJax 正規化 + `IPython.display.Markdown` で数式表示
- 4 モデル横並び比較 (LLM-jp-4 8B, 32B-A3B, Qwen3.6-27B, Gemma-4-31B-it)
- パラメータ実験 (temperature, seed) と thinking ON/OFF 比較

### `notebooks/02_gvr_pipeline.ipynb`

研究タスク向けの **Generator-Verifier-Reviser パイプライン**プロトタイプ。

- `ModelClient` プロバイダ非依存ラッパ (Claude / Gemini も同じインタフェースで差し込み可)
- `VerifierResponse` 構造化出力パーサ
- `GVRPipeline` orchestration (最大改訂回数指定可)
- `multi_verify` 複数 Verifier の多数決投票
- `PipelineTrace` JSON 永続化と再読込
- 数学命題判定 / Julia コードレビュー / Chebyshev bias 論述などの実例
- Claude / Gemini クライアント生成スタブ

### 🎴 `notebooks/03_haiku_battle.ipynb`

**遊びノートブック** — 4 モデルが同じお題で俳句を 1 句ずつ詠み、匿名化してから互いに採点しあう小さなトーナメント。短く (12 セル) 楽しめます。

- 🌸🗻🐉💎 4 モデル全員が「深夜の研究室、コーヒーと計算機」で詠む
- 🎭 A/B/C/D に匿名化シャッフル
- 🧑‍⚖️ 4 モデル自身が審査員として呼び戻され、1〜4 位の順位を付ける
- 🏆 Borda count で総合王者発表
- 🪞 「自分の作品を何位に置いたか」を分析 (😎 自画自賛 / 🥶 自作に最下位)
- 教育的副産物: Verifier に同モデルを使う妥当性 (自作識別はほぼ不可能) の傍証

## 🛠️ ローカル実行

```bash
git clone https://github.com/jxta/LLM-jp_playground.git
cd LLM-jp_playground
pip install -r binder/requirements.txt
export LLMJP_API_KEY="your-token"
jupyter lab
```

## 🌐 エンドポイント

```
https://llm-jp-playground.apps.llmc.nii.ac.jp/api/v1
```

利用は [NII LLMC](https://llmc.nii.ac.jp/) の規約に従ってください。

## 📝 注意点

- **non-streaming パスでの reasoning ドロップ**: LLM-jp-4 系で `choices[0].message.content` が null になり `reasoning_content` も落ちる現象を Playground 側で確認。`01_api_basics.ipynb` は内部 streaming で回避済み
- **モデルの thinking 挙動はモデル名から判断不能**: `-thinking` サフィックスは訓練系統名であり、ランタイム挙動を保証しない。プローブで実測すること
- LLM-jp-4 は英語で思考し日本語で final 出力する興味深い挙動を示す (多言語推論の研究対象)

## 📂 ファイル構成

```
.
├── README.md
├── binder/
│   ├── requirements.txt      # openai 等
│   └── runtime.txt           # Python 3.11
└── notebooks/
    ├── 01_api_basics.ipynb
    ├── 02_gvr_pipeline.ipynb
    └── 03_haiku_battle.ipynb
```

## ライセンス

- ノートブック内コード: MIT
- LLM-jp Playground 本体の利用: NII LLMC 規約準拠
