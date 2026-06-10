# RDRA for Mermaid DSL

RDRA を Mermaid.js + Git + 生成AIで扱うための記法メモ。

## Background

RDRA (Resource Driven Requirements Analysis) を Mermaid.js と Git 上で扱い、エンジニアおよび生成AIとの要件認識合わせを行う。

## Goals

- Git diff でレビュー可能な要件モデルにする
- Mermaid を SSOT に近づける
- AI が安全に編集できる構造化記法にする
- RDRA の図をコードとして管理する
- ビジネスコンテキスト図を起点に他モデルへ展開する

## Scope

最初の対象はビジネスコンテキスト図。

後続で検討する対象:

- ビジネスコンテキスト図
- システムコンテキスト図
- 業務ユースケース図
- ユースケース複合図
- 情報モデル
- 状態モデル

## Mermaid Policy

基本は `flowchart TB` を採用する。

- 上から下: 業務の流れ
- 左から右: 関連範囲の広がり

## Relationship Lines

関係線は `---` のみ使用する。

理由:

- 流れは業務ノードが表現する
- 因果関係を強制しない
- RDRA らしく関係性を表現する

## GitHub Compatible Node Syntax

GitHub 上で表示する `.mmd` は、通常ノードだけで書く。

frontmatter、画像ノード、icon、ELK、handDrawn は使わない。

```text
flowchart TB
  b_order["受注業務"]
```

## Advanced Visual Syntax

Mermaid v11.3.0+ では画像ノードを使える。

ただし GitHub の Mermaid レンダラーでは未対応または不安定な場合があるため、GitHubで直接表示したい `.mmd` では使わない。

将来、GitHub表示ではなく Mermaid Live Editor や専用レンダラー向けの高度版を作る場合は、画像ノードを1行で定義する。

1. `img`
2. `label`
3. `pos`
4. `w`
5. `h`
6. `constraint`

```text
flowchart TB
  b_order@{ img: "https://api.iconify.design/mdi/work.svg", label: "受注業務", pos: "b", w: 60, h: 60, constraint: "on" }
```

## Prefix Rules

ノードIDは `[prefix]_[lowerCamelCase]` 形式にする。

| Prefix | Meaning |
| --- | --- |
| `a_` | actor |
| `b_` | business |
| `bc_` | business use case |
| `ac_` | activity |
| `uc_` | use case |
| `i_` | information |
| `r_` | requirement |
| `v_` | view |
| `e_` | event |
| `t_` | timer |
| `st_` | state |
| `c_` | condition |
| `vr_` | variation |
| `s_` | system |
| `x_` | external system |
| `fr_` | functional requirement |
| `nfr_` | non functional requirement |

## Icon Rules

現時点では MDI を Iconify URL として扱う。

| Type | Icon |
| --- | --- |
| actor | `mdi:person` |
| business | `mdi:work` |
| businessUseCase | `mdi:workflow` |
| activity | `mdi:progress-check` |
| useCase | `mdi:gesture-tap-button` |
| information | `mdi:data` |
| requirement | `mdi:information` |
| view | `mdi:monitor` |
| event | `mdi:flash` |
| timer | `mdi:timer-outline` |
| state | `mdi:state-machine` |
| condition | `mdi:source-branch` |
| variation | `mdi:shape-plus` |
| system | `mdi:server` |
| externalSystem | `mdi:server-network` |
| functionalRequirement | `mdi:cog-outline` |
| nonFunctionalReq | `mdi:shield-check` |

Iconify URL の例:

```text
https://api.iconify.design/mdi/person.svg
```

## Class Rules

prefix は意味、classDef は表現を担当する。

同じ prefix でも、見た目を変更したい場合は classDef を調整する。

```text
flowchart TB
  classDef actor fill:none,stroke:none,color:#222222;
  classDef business fill:none,stroke:none,color:#222222;
  classDef information fill:none,stroke:none,color:#444444;
  classDef requirement fill:none,stroke:none,color:#444444;
  classDef system fill:none,stroke:none,color:#222222;
  classDef external fill:none,stroke:none,color:#666666;
```

## Frontmatter

GitHub互換版の `.mmd` では frontmatter を使わない。

高度版を専用レンダラーで使う場合の候補:

```yaml
---
title: RDRA Diagram
config:
  layout: elk
  look: handDrawn
  theme: neutral
  flowchart:
    curve: basis
    htmlLabels: false
    nodeSpacing: 48
    rankSpacing: 72
    padding: 8
  elk:
    mergeEdges: false
    nodePlacementStrategy: BRANDES_KOEPF
  themeVariables:
    background: "#F7F5F2"
    lineColor: "#666666"
    fontFamily: "Inter, Hiragino Sans, sans-serif"
    fontSize: "14px"
---
```

## AI Editing Rules

AI に編集させるときは、Mermaid 冒頭のコメントブロックを仕様書として扱う。

AI が守ること:

- ノードIDの prefix を維持する
- GitHub互換版では通常ノードだけを使う
- GitHub互換版では frontmatter、画像ノード、icon、ELK、handDrawn を使わない
- 関係線は `---` のみにする
- 新しい意味を追加する場合は prefix 一覧に追加する
- 表現変更は classDef に寄せる
- Mermaid と同じファイル内に、短い意図コメントを残す

## Open Questions

- 高度版 Mermaid を別ファイルとして持つか
- MDI アイコンの最適候補
- prefix から class を自動付与するか
- 将来的に `rdra.yml` から Mermaid を生成するか
