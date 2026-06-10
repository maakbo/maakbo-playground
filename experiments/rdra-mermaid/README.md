# RDRA for Mermaid

RDRA を Mermaid.js と Git で扱うための小さな実験。

目的は、図をきれいに描くだけではなく、生成AIと人間が同じ要件モデルを安全に編集できる土台を作ること。

## Files

- [rdra-mermaid-dsl.md](./rdra-mermaid-dsl.md)
  - RDRA for Mermaid の記法メモ
- [business-context-template.mmd](./business-context-template.mmd)
  - ビジネスコンテキスト図のテンプレート
- [business-context-example.mmd](./business-context-example.mmd)
  - 汎用的な受注から請求のサンプル
- [experiment-log.md](./experiment-log.md)
  - 試したこと、次に見ること

## Current Scope

まずはビジネスコンテキスト図だけを対象にする。

RDRA 全体への拡張は後続。

現時点の `.mmd` は GitHub 上で表示できることを優先した互換版。
frontmatter、画像ノード、icon、ELK、handDrawn は使わない。

## Design Intent

これは単なる Mermaid ルールではなく、RDRA を Git と AI に最適化した DSL として育てる。

現時点では、次を優先する。

- Git diff でレビューできる
- Mermaid 単体で意図が読める
- AI が安全に編集できる
- きれいに見える
- 完璧よりも、議論の土台としてすぐ使える

## Quick Use

1. `business-context-template.mmd` をコピーする
2. ノードIDを prefix ルールに沿って追加する
3. GitHub互換のため通常ノードで書く
4. 関係線は `---` だけを使う
5. 図の下に短いメモを足す
