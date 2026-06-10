# Experiment Log

## 2026-06-11

RDRA for Mermaid の最初の土台を作った。

今回の成果:

- ビジネスコンテキスト図のテンプレートを作成
- 汎用サンプルとして「受注から請求」を作成
- prefix、icon、diagram rules を Mermaid ファイル冒頭に埋め込み
- prefix と classDef の責務を分離
- 関係線は `---` に統一

## Findings

Mermaid ファイル単体にルールを書いておくと、生成AIに渡したときの編集事故を減らせそう。

GitHub 上で `.mmd` を直接表示する場合、frontmatter、画像ノード、ELK、handDrawn は構文エラーや表示エラーの原因になりやすい。

GitHub表示を優先するファイルでは通常ノードだけを使う方が安定する。

ビジネスコンテキスト図は、業務、関係者、情報、システム、外部システムの5種類だけでも議論の土台になる。

## Open Questions

- 高度な見た目用の Mermaid を別ファイルとして持つか
- MDI アイコンをこのまま固定するか
- Activity、UseCase、Requirement、State、Variation のアイコン候補を精査するか
- `rdra.yml` から Mermaid を生成する前に、手書き Mermaid の書き味をもう少し試すか

## Next Small Step

実際の業務アイデアを1つ選び、`business-context-template.mmd` をコピーして10ノード以内で図にする。

成功条件:

- 3分以内に図の意図が読める
- Git diff で変更箇所が分かる
- AI にノード追加を頼んでも記法が崩れない
