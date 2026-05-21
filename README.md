# `.bus/` — git inbox protocol for S28 multi-pane coordination

**目的**: 統率AI と P1〜P5 pane の間で「貼り付け」を介さず指示/結果をやりとりするための軽量メッセージバス。GitHub 経由 (git push/pull) を transport に使い、自然と監査ログ (git log) が残る。

## ディレクトリ構造

```
.bus/
├── README.md              ← この文書
├── P1/
│   ├── inbox/             ← 統率AI が P1 に渡す指示。ファイル名は NNNN-<slug>.md
│   │   ├── 0001-bootstrap.md
│   │   └── processed/     ← P1 が処理済みファイルを移動する保管庫
│   └── outbox/            ← P1 が統率AI に返す報告
│       └── processed/
├── P2/ ... (同形)
├── P3/ ... 
├── P4/ ...
└── P5/ ...
```

## プロトコル

### 統率AI (= Mac 上の統率 pane の Claude Code) の動作

1. 新規指示は `<repo>/.bus/P{N}/inbox/NNNN-<slug>.md` に書く (N は 4 桁、slug は短い英数字)
2. `git add .bus/P{N}/inbox/NNNN-*.md && git commit -m "bus: P{N} <slug>" && git push`
3. pane の返信は `<repo>/.bus/P{N}/outbox/` を `ls -t` で新着確認

### P{N} pane の動作

1. **起動時**:
   ```
   cd /Users/yoko/work/ai4math-lab
   git pull --rebase origin main
   ls -1 .bus/P{N}/inbox/*.md 2>/dev/null | grep -v processed/ | sort
   ```
2. **未処理 inbox ファイルを番号順に処理** (内容は通常の Claude Code 指示として実行)
3. **処理完了後**:
   - 結果を `.bus/P{N}/outbox/NNNN-<slug>-result.md` に書く (inbox の番号を踏襲)
   - 処理済 inbox を `.bus/P{N}/inbox/processed/` に `git mv` する
   - `git add .bus/P{N}/ && git commit -m "P{N}: <slug> result" && git pull --rebase && git push`
4. **inbox 空 → 統率AI に「P{N} idle」報告のみ outbox に置いて待機**

### 補助スクリプト

`.bus/poll.sh P{N}` で「git pull → 未処理 inbox 表示」を 1 行で実行できる (後で追加予定)。

## 競合回避

- inbox/ は **統率AI のみが書く**。outbox/ は **担当 pane のみが書く**。これで write-side conflict は構造的に起きない
- 同時 push は `git pull --rebase` で吸収。`.bus/P1/...` と `.bus/P2/...` は touch するファイルが完全 disjoint なので merge は trivial
- 万一 conflict が起きたら abort して人間に escalate

## 補足

- ファイル名の数字は **monotonic に増やす**だけで OK (実時刻と一致させない)。処理順保証のため
- 一般指示ファイルは Markdown。コード片は fenced code block で囲む
- 大きな出力 (>200 行) は outbox に置き、本文では「outbox/NNNN-foo.md 参照」と書く
- panel = `P{N}` 表記、ノード = `mdx-0{N}` 表記で統一
# LLM-jp_playground
