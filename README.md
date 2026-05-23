# LLM-jp Playground 実験ノートブック

NII LLMC が提供する [LLM-jp Playground](https://llm-jp-playground.apps.llmc.nii.ac.jp) (OpenAI 互換 API) を Jupyter Notebook でインタラクティブに試すサンプル集。

## 🚀 即起動 (mybinder.org)

ブラウザだけで実行できます。`openai` 等のパッケージをインストール済みの JupyterLab 環境が立ち上がります (初回ビルドは 1〜3 分、2 回目以降はキャッシュで数十秒)。

[![Binder - JupyterLab](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/jxta/LLM-jp_playground/main?urlpath=lab)

特定のノートブックを直接開く:

### 🛠️ ベース (技術解説 & パイプライン)

| ノートブック | 起動リンク |
|---|---|
| **01. API 基本** (動的 thinking 判別 / 内部 streaming / LaTeX レンダリング / 4 モデル比較) | [![Open 01](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/jxta/LLM-jp_playground/main?urlpath=lab%2Ftree%2Fnotebooks%2F01_api_basics.ipynb) |
| **02. GVR パイプライン** (Generator → Verifier → Reviser, multi-vote, trace 永続化) | [![Open 02](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/jxta/LLM-jp_playground/main?urlpath=lab%2Ftree%2Fnotebooks%2F02_gvr_pipeline.ipynb) |

### 🎮 遊びノートブック (NII メンバー共有用)

| ノートブック | 起動リンク |
|---|---|
| 🎴 **03. 俳句バトル** (4 モデルが詠む → 匿名化 → 互いに採点 → 自作識別分析) | [![Open 03](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/jxta/LLM-jp_playground/main?urlpath=lab%2Ftree%2Fnotebooks%2F03_haiku_battle.ipynb) |
| 🔗 **04. 連歌リレー** (4 モデルが交互に 5-7-5 / 7-7 を継ぎ、12 句の連歌を編む) | [![Open 04](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/jxta/LLM-jp_playground/main?urlpath=lab%2Ftree%2Fnotebooks%2F04_renga_relay.ipynb) |
| 📞 **05. 伝言ゲーム** (敬語 → ヤンキー → 古文 → 俳句 の文体変身チェーン + 復元テスト) | [![Open 05](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/jxta/LLM-jp_playground/main?urlpath=lab%2Ftree%2Fnotebooks%2F05_telephone_game.ipynb) |
| 🎭 **06. 絵文字マトリックス** (各モデルが自分+他 3 モデルを絵文字 5 つで表現、4×4 マトリックス) | [![Open 06](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/jxta/LLM-jp_playground/main?urlpath=lab%2Ftree%2Fnotebooks%2F06_emoji_matrix.ipynb) |

### 📊 ベンチマーク (本格評価)

| ノートブック | 起動リンク |
|---|---|
| 🏟️ **07. ベンチマーク選手権** (JCommonsenseQA / JMMLU / MGSM-ja / 対話 を実走し総合王者を決定) | [![Open 07](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/jxta/LLM-jp_playground/main?urlpath=lab%2Ftree%2Fnotebooks%2F07_benchmark_olympiad.ipynb) |

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

### 🛠️ ベース

#### `notebooks/01_api_basics.ipynb`

LLM-jp Playground API の基本的な使い方と落とし穴の回避策。

- 利用可能モデル一覧の自動取得 (`/v1/models`)
- **動的プローブで thinking モデル判別** (モデル名のサフィックスは当てにならない)
- 内部 streaming で `reasoning_content` を確実に取得 (non-streaming パスでは落ちるバグの回避)
- 自動ルーティング `smart_chat` (`/no_think` + `enable_thinking=False` 併用)
- LaTeX → MathJax 正規化 + `IPython.display.Markdown` で数式表示
- 4 モデル横並び比較 (LLM-jp-4 8B, 32B-A3B, Qwen3.6-27B, Gemma-4-31B-it)
- パラメータ実験 (temperature, seed) と thinking ON/OFF 比較

#### `notebooks/02_gvr_pipeline.ipynb`

研究タスク向けの **Generator-Verifier-Reviser パイプライン**プロトタイプ。

- `ModelClient` プロバイダ非依存ラッパ (Claude / Gemini も同じインタフェースで差し込み可)
- `VerifierResponse` 構造化出力パーサ
- `GVRPipeline` orchestration (最大改訂回数指定可)
- `multi_verify` 複数 Verifier の多数決投票
- `PipelineTrace` JSON 永続化と再読込
- 数学命題判定 / Julia コードレビュー / Chebyshev bias 論述などの実例
- Claude / Gemini クライアント生成スタブ

### 🎮 遊びノートブック

NII メンバーへの共有用に、API 基本の理解を前提とした楽しい小実験を用意してあります。どれも 10 セル前後の短いノートブックで、`§1. セットアップ` を実行して `LLMJP_API_KEY` を設定すれば動きます。

#### 🎴 `notebooks/03_haiku_battle.ipynb` — 俳句バトル + 匿名相互審査

- 🌸🗻🐉💎 4 モデル全員が同じお題で詠む
- 🎭 A/B/C/D に匿名化シャッフル
- 🧑‍⚖️ 4 モデル自身が審査員として呼び戻され、1〜4 位の順位を付ける
- 🏆 Borda count で総合王者発表
- 🪞 「自分の作品を何位に置いたか」を分析 (😎 自画自賛 / 🥶 自作に最下位)
- **副産物**: Verifier に同モデルを使う妥当性 (自作識別はほぼ不可能) の傍証

#### 🔗 `notebooks/04_renga_relay.ipynb` — 連歌リレー (12 句)

- 4 モデルを順に巡回させ、5-7-5 と 7-7 を交互に **12 句の連歌**を編む
- 各モデルは「**直前の句しか見ない**」という連歌の精神に倣う
- 最後に 1 モデルが全体を講評
- **観察ポイント**: 何句目で連想が飛躍するか / 各モデルの作風 (派手 vs 渋い)
- `03_haiku_battle` の並列+審査と対をなす **逐次協創**の実験

#### 📞 `notebooks/05_telephone_game.ipynb` — 文体の変身チェーン

- 1 文を **敬語 → ヤンキー語 → 古文 → 俳句** の順に変換していく伝言ゲーム
- 各モデルは前のモデルの出力しか見ない (原文は見えない)
- 最後に別モデルが「最終形の俳句から原文を復元」を試みる
- **観察ポイント**: 4 段階を経て意味はどれくらい残るか / 17 音まで圧縮した後の復元率

#### 🎭 `notebooks/06_emoji_matrix.ipynb` — 絵文字パーソナリティ・マトリックス

- 各モデルに「自分 + 他 3 モデル」を **絵文字 5 つずつ** で表現させる
- 4 モデル × 4 描写 = **4×4 マトリックス** を構築 (対角が自画像、🪞 で強調)
- **観察ポイント**: 自己描写の控えめさ / 他者描写の大胆さ / 共通する絵文字 (集合知)
- 結果はスクショ映えするので Slack で共有しやすい
- **既知の現象**: LLM-jp 8b は **自己言及プロンプト** (「これはあなた自身です」) で thinking が暴走し APIError リトライ全失敗 → 部分絵文字でフォールバックされる。観察対象として面白い

### 📊 ベンチマーク

公開された日本語 LLM ベンチマークを実走させ、4 モデルの真の実力を測ります。Nejumi LLM Leaderboard / llm-jp-eval で使われているデータセットの「ミニ実走版」です。

#### 🏟️ `notebooks/07_benchmark_olympiad.ipynb` — 4 モデル日本語LLMベンチマーク選手権

- **🧠 JCommonsenseQA** (JGLUE, Kurihara+ NLP2022) — 5択常識推論 (JGLUE GitHub 原本)
- **📚 JMMLU** (Yin+ 2024) — 4択学術知識、4 科目横断 (`nlp-waseda/JMMLU` GitHub 原本: 世界宗教/初等数学/専門医学/哲学)
- **➗ MGSM-ja** (Shi+ 2022) — 数学CoT 問題、人手翻訳された GSM8K の日本語版 (google-research/url-nlp GitHub 原本)
- **💬 自由対話** — JaMT-Bench スタイル、4 モデル匿名相互審査 + Borda count
- 🏆 **総合王者の表彰式** (ASCII 表彰台、メダル数、競技別内訳)
- ⚡ **速度 vs 精度** 分析 (レイテンシ、1正答あたり秒数)
- 🎬 **ハイライト/ローライト** (全員不正解の問題、1モデルだけ正解できた問題)
- 📈 **Nejumi LLM Leaderboard との位置づけ** + Wilson 95% 信頼区間
- デフォルトは ~10 分の軽量走行。`N_*` を増やすと本格評価 (各タスク 50-100 問で ±10% 以下に収束)

> 🚀 **HuggingFace Hub 不使用**: データは全て GitHub の `raw.githubusercontent.com` (Fastly CDN) から直接取得します。HF Hub LFS の egress 詰まりを起こす環境 (NII 学内ネット等) で確実に動くよう設計。`datasets` パッケージは不要。
>
> ⚠️ **ライセンス**: JMMLU は本実装で **CC BY-SA 4.0 の 4 科目** (世界宗教/初等数学/専門医学/哲学) のみ使用。NC_ND の科目 (日本史/世界史等) は意図的に除外。

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
- **mid-stream APIError**: LLM-jp 8b/32b の thinking モデルで時々発生。03〜07 の `chat()` はリトライ機構付き
- **HuggingFace Hub の egress 不安定問題**: 07 では HF Hub を一切経由せず GitHub raw に切替。NII 学内ネットや帯域絞られている環境でも安定動作
- LLM-jp-4 は英語で思考し日本語で final 出力する興味深い挙動を示す (多言語推論の研究対象)

## 📂 ファイル構成

```
.
├── README.md
├── binder/
│   ├── requirements.txt      # openai 等 (stdlib のみで datasets 不要)
│   └── runtime.txt           # Python 3.11
└── notebooks/
    ├── 01_api_basics.ipynb              # 🛠️ API 基本
    ├── 02_gvr_pipeline.ipynb            # 🛠️ GVR パイプライン
    ├── 03_haiku_battle.ipynb            # 🎴 俳句バトル
    ├── 04_renga_relay.ipynb             # 🔗 連歌リレー
    ├── 05_telephone_game.ipynb          # 📞 伝言ゲーム
    ├── 06_emoji_matrix.ipynb            # 🎭 絵文字マトリックス
    └── 07_benchmark_olympiad.ipynb      # 🏟️ ベンチマーク選手権
```

## 参考文献 (07 ベンチマーク用)

- 栗原健太郎, 河原大輔, 柴田知秀. **JGLUE: 日本語言語理解ベンチマーク**. 言語処理学会第28回年次大会, 2022. https://github.com/yahoojapan/JGLUE
- Z. Yin, et al. **JMMLU: Japanese Massive Multitask Language Understanding Benchmark**. 2024. https://github.com/nlp-waseda/JMMLU
- F. Shi, M. Suzgun, et al. **Language Models are Multilingual Chain-of-Thought Reasoners**. arXiv:2210.03057, 2022. https://github.com/google-research/url-nlp
- L. Zheng, et al. **Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena**. NeurIPS 2023.
- W&B Japan. **Nejumi LLM Leaderboard 4**. https://wandb.ai/wandb-japan/llm-leaderboard
- LLM-jp. **llm-jp-eval**. https://github.com/llm-jp/llm-jp-eval

## ライセンス

- ノートブック内コード: MIT
- LLM-jp Playground 本体の利用: NII LLMC 規約準拠
- ベンチマークデータセット: 各データセットのライセンスに従う (07 ノートブック冒頭に明記)
