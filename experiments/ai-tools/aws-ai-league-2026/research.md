# AWS AI League 2026 調査結果

## 結論

Boss突破と経路改善を同じ状態管理基盤に載せるのが、次回の有力な実験方針である。初期マップ、現在位置、鍵の情報、残ライフ、残時間を正しく扱えば、Bossの集計問題・扉・経路選択に横断的に効く。ただしこれは設計仮説であり、12000点への到達を保証しない。

公開情報から本番の完全なBoss出題一覧、採点器、優勝者の最適化アルゴリズムは得られなかった。記事の助言より、手元の問題文・設定・combat logを優先して適用する。

## Confirmed facts：確認した事実と適用範囲

| ID | 確認できたこと | 範囲・競技への示唆 |
|---|---|---|
| F1 | AgentCoreを使う迷路型競技で、モデル・ツール・memory・guardrailsを組み合わせる。[O2](#o2) | 公式の全体像。利用可能なモデルや機能は今回の画面で再確認する。 |
| F2 | Atos開催では正答、coins、制限時間内のtreasure、残ライフ、簡潔さが得点要素。[O3](#o3) | 「経路が短い」だけでは目的を表さない。coins自体の処理と寄り道時間は区別する。 |
| F3 | London参加者はBossを複合能力問題と記述し、鍵を取ってもコード想起に失敗すると扉で5ライフを失う例を示す。[P1](#p1) | Boss専用モデルを作る前に能力の組み合わせと状態受け渡しを調べる。 |
| F4 | NYC優勝者は経路を最適化エンジン化し、Haiku 4.5では専門家へ委譲したと報告。[P2](#p2) | 委譲はモデル・失敗率・遅延の比較対象。常に多エージェントが最適とはいえない。 |
| F5 | 香港優勝者は既存pathfinder、計算Lambda、Web担当、memory、guardrailを使い、ログを見て改善。モデルはその開催ではNova 2 Lite固定と記述。[P3](#p3) | 経路の作り直しを前提にせず、現在の最大損失から着手する。 |
| F6 | 過去大会データでは同じタイルでも得点・役割が変わる。[D1–D3](#d1) | tile IDだけで処理を固定せず、問題文と開催回のルールも使う。 |
| F7 | Community Editionは実大会の近似環境として公開されている。[P4](#p4) | 検証材料には使えるが、公式の採点仕様として引用しない。 |

「confirmed」は、指定出典に記載がある／指定ソースで実装を確認したという意味。本番一般への適用を確認した意味ではない。

## Boss：公開ソースから具体的に分かったこと

NYC過去大会READMEはfinaleのBossを「素数の計算×タイル数」の複合問題と記録している。[D2](#d2)

Community Editionの`challenge_generator.py`では、次の3つの生成処理を確認した。[C1](#c1)

| 生成処理 | 計算する内容 | 実装から得られる検証観点 |
|---|---|---|
| `_boss_template_math_plus_map` | N番目の素数 × 指定タイル数 | 素数の順序と対象tile ID、初期マップの集計を別々に検証する。 |
| `_boss_template_code_plus_web` | 整数累乗の桁和 ＋ wall数 | **関数名とコメントにwebがあるが、現実の処理はWeb取得をしていない。** 名前だけで能力要件を決めない。 |
| `_boss_template_triple_combo` | Fibonacci ＋ 対象challenge数×係数 − coin数 | この実装の対象challengeは`c`で始まりc7/c8以外。鍵・扉も含まれ得る。一般的な「モンスター数」と同一視しない。 |

ここでの正解生成は初期gridから集計し、文字列の数値を返す。3テンプレートが本番の全出題を尽くす証拠はない。解答暗記や期待解フィールドの参照ではなく、未見の問題文を分解して、エージェントに与えられた入力だけから計算する仕組みに使う。

**提案**：既存計算Toolへ「mapの正確な集計」を渡す。最初は新規Boss Lambdaを作らず、既存Toolの組み合わせで試す。Boss失敗を「問題解釈／集計／計算／出力整形／遅延」に分ける。

## 採点・タイル：開催回ごとの差

以下はコミュニティが保存した過去設定であり、今回の競技に固定適用しない。

| 要素 | Londonの記録 [D1](#d1) | 他の記録 | 実装時の扱い |
|---|---|---|---|
| Treasure | 通常1000、finale 1は5000 | 香港practice YAMLは2000 [D3](#d3) | round設定として外出しする。 |
| Coin c7 | 通常250、finale 2は750 | NYC通常250 [D2](#d2) | 低リスクでも寄り道の機会費用を計算。 |
| Spikes c8 | 1ライフ減 | 香港YAMLでは未確認扱い [D3](#d3) | 再訪で再発動するかも要確認。 |
| Life bonus | 残1ライフにつき250 | NYC・香港も同値の記録 | 命の価値はbonusだけでなく、その後の得点機会も含む。 |
| c17 | 通常750、finaleで50 | NYCは50、失敗damage 2 [D2](#d2) | 昔の高得点タイルを新マップで無条件優先しない。 |
| c18 | Healthcare API、JSON形式 | NYCはRisk Analyzer、850点、damage 2 [D2](#d2) | タイル番号だけをキーにした固定回答は壊れる。 |
| Keys/Doors | 色の対応と記憶 | NYCはGrey＝先頭2文字＋末尾2文字、Yellow＝5番目＋7番目の例 [D2](#d2) | 鍵の所有と語の保持、変換の正しさを別々に確認。 |

NYCのRisk Analyzerは競技用の採点表である。一般の医療判断へ転用せず、本番で与えられた表・単位・境界値を計算Toolに渡す。ここでは点数表の複製や医学的正当性の評価はしない。

Community Editionの採点コードで確認した式は次の通り。[C2](#c2)

```text
S = challenge_points + coin_points
    + treasure_reached * treasureBonus
    + lives_remaining * livesBonusMultiplier
    + round(clamp(tokenBonus - T/N * (1-r), 0, tokenBonus))
```

`N=0`はtokenBonusを返す特別扱い。`r`はcustom model数による軽減率で、同コードでは0/50/70/85/92/95%。本番の対象モデル数・適用条件・token集計境界とは照合が必要。

`game_runner.py`の通常challenge処理は、runtimeの`outputTokens`を使い、得られない場合に解答長から推定する。sub-agentを含むruntime側の集計範囲は本番ログで確認する必要がある。短い最終出力だけを見て「全tokenが減った」と判定しない。[C3](#c3)

この計算関数に独立した残秒bonusはない。時間はtreasure到達可否・解ける問題数を制限する。公式記事の「時間効率」という表現から、全開催に共通する残秒×係数の式を捏造しない。

## 公開資料の不一致・未確認事項

| 論点 | 証拠 | 扱い |
|---|---|---|
| NYC finale 3の時間 | P2は2:30、D2は60秒 | 不一致未解消。どちらも今回のデフォルトにしない。 |
| NYC練習時間 | P2の画面説明は5分、D2は180秒 | 表示例・round・転記の可能性はあるが原因未確認。 |
| Supervisorのモデル | P2はHaiku 4.5、P3はNova 2 Lite固定 | 開催差がある。アップグレード可能と仮定しない。 |
| Tokenの扱い | 記事では簡略式、C2/C3ではclamp・軽減・推定など | 本番の明細と一致するか検算してから最適化する。 |
| 元の8500点と12000点の差 | 参照会話だけ。今回の実行ログなし | Bossが3500点を生むとは断定できない。 |
| Workshop本文 | O4のURLは確認できたが本文を取得できなかった | 詳細を公式ルールとして補完しない。手元のworkshop配布物が必要。 |
| 優勝者の完全コード・prompt | P2は戦略説明、P3はprompt非公開 | 再現済みの勝利レシピとして提示しない。 |

## Hypotheses：検証可能な攻略仮説

| ID | 仮説 | 反証・見送る条件 | 次の実験 |
|---|---|---|---|
| H1 | 未挑戦Bossが大きな未取得得点と能力不足を隠している | Boss成功でも得点差が小さい／往復で他の高得点を失う | E1で到達・正答・総得点を別に測る。 |
| H2 | map集計と鍵の明示的状態管理は複数問題に効く | 既存Toolが既に正確で失敗が遅延に集中 | E2で初期mapと現在map、複数鍵を区別。 |
| H3 | 生存・時間制約付きrouteが短距離routeより得点を上げる | マップが小さく既存routeが十分／推論時間増でtime up | E3で同じsolverのまま経路だけ比較。 |
| H4 | 専門家を1つに絞った委譲が誤Tool選択を減らす | 委譲往復の時間損が失敗減より大きい | E4で同じ問題セット・モデルで比較。 |
| H5 | 長い「深く考えて」より出力契約と根拠の受け渡しが効く | 元promptで正答率が高く、短縮後に条件脱落 | E4で数値・JSON・Tool選択を確認。 |
| H6 | Fine-tuningより先に正答・到達率を上げる方が今は有利 | 既に安定していてtokenが主損失、既存学習資産もある | E5は上限改善量と準備時間を見て判断。 |

## 上位参加者の攻略事例

- **Ross Williams、NYC優勝（本人報告）**：最適化Toolと少数の専門家を重視。Sonnet 4を使った以前の構成とHaikuでの構成を変えた。採用するのは「モデルに応じて分担を測る」という考え方であり、未公開アルゴリズムを再現したとは扱わない。[P2](#p2)
- **Max Chui、香港優勝（本人報告）**：予選8063点・13提出。既存pathfinderを保ち、ログによる改善に集中。最終変更が最高結果とは限らなかったと述べる。今回への示唆は、常にbest-known設定を残すこと。[P3](#p3)
- **Mark Ross、London予選3位（本人報告）**：Toolとchallengeの対応、短い回答、実行ログの確認を勧める。学習モデルに時間を使う前に基本能力を確かめる。[P1](#p1)
- **Atos上位者（AWS公式報告）**：custom pathfinding、guardrails、memory、fine-tuningを含む工夫が紹介されるが、誰のどの変更が何点を増やしたかは公表されていない。[O3](#o3)

勝者の共通点を「多くのLambdaを作る」に還元できない。どこで失点したかを観察し、その開催条件で有効な変更を選ぶことが、この調査の分析上の共通項である。

## Sources：公式情報

全資料の確認日は2026-09-09。公開日と大会開催日は分ける。GitHubは初公開日を推測せず、読んだcommitで固定した。

### O1

AWS, [Announcing AWS AI League 2026 Championship](https://aws.amazon.com/about-aws/whats-new/2025/11/ai-league-2026-championship/), **2025-11-30**。公式発表。Agentic／Model Customizationの枠組みと企業内大会を確認。個別Bossのルールの根拠には使わない。

### O2

Marc Karp / Natasya K. Idries, AWS, [AWS AI League: Model customization and agentic showdown](https://aws.amazon.com/blogs/machine-learning/aws-ai-league-model-customization-and-agentic-showdown/), **2025-12-23**。公式解説。構成要素と評価の全体像。2025回顧と2026案内が混在するため個別設定の固定には使わない。

### O3

Rajesh Babu Nuvvula / Mark Ross / Ruchi Bhatia, AWS, [From theory to delivery: How Atos upskilled 400 engineers in agentic AI](https://aws.amazon.com/blogs/machine-learning/from-theory-to-delivery-how-atos-upskilled-400-engineers-in-agentic-ai/), **2026-09-01**。企業大会の公式報告。評価要素と上位構成を確認。実験設計の参考にするが、得点係数は示していない。

### O4

AWS, [Workshop Studio — AI League workshop](https://catalog.us-east-1.prod.workshops.aws/workshops/0c1f072b-ebd1-4d8d-9340-dd47479481c0/en-US), **公開日不明・本文未取得**。P1からの公式リンク。取得できなかった本文に依拠する主張はない。

## Sources：参加者本人・コミュニティ記事

### P1

Mark Ross, AWS AI/ML Community Blog, [AWS AI League 2026: Hints and Tips for the Agentic Challenge](https://blog.awsaicommunity.org/posts/aws-ai-league-2026-agentic-challenge-hints-and-tips/), **2026-07-15**。London予選3位の体験談。Boss複合能力、鍵想起の失敗、明確なTool対応を確認。公式AWSブログとは区別する。

### P2

Ross Williams, AWS AI/ML Community Blog, [AWS AI League 2026: NYC Summit - A Winning Walkthrough](https://blog.awsaicommunity.org/posts/aws-ai-league-2026-nyc-summit-a-winning-walkthrough/), **2026-08-03**。NYC優勝者の一次報告。経路最適化とモデルに応じた委譲が要点。時間の記述はD2と不一致。

### P3

Max Chui, [How I won the AWS AI League in Hong Kong](https://maxchui.dev/blog/winning-the-aws-ai-league-hong-kong), **2026-08-28**（開催 **2026-06-17**）。優勝者本人の体験談。既存Toolとログ改善による最小構成の事例。完全なpromptは公開されていない。

### P4

Mark Ross, AWS AI/ML Community Blog, [Open Source: Practice the AWS AI League Agentic Challenge with the Community Edition](https://blog.awsaicommunity.org/posts/aws-ai-community-open-source-practice-platform/), **2026-07-16**。練習環境と過去大会データの公開案内。近似環境であることを明示する根拠。

## Sources：過去大会データと公開コード

D1–D3はAWS AI Communityの公開リポジトリ。読んだHEADは`a2f585272b504b4746ebe07388ba45212b09035a`、commit日時 **2026-09-07T09:41:54+01:00**。各ファイルの初公開日は未確認。

### D1

[London Summit README](https://github.com/aws-ai-community/ai-league-competition-data/blob/a2f585272b504b4746ebe07388ba45212b09035a/competitions/2026/London-Summit/README.md)。開催表記 **2026年5月**。round別の得点・時間・マップの保存資料。設定をパラメータ化すべき根拠。

### D2

[NYC Summit README](https://github.com/aws-ai-community/ai-league-competition-data/blob/a2f585272b504b4746ebe07388ba45212b09035a/competitions/2026/New-York-City-Summit/README.md)。開催表記 **2026年6月**（コミュニティ記載、独立確認なし）。Boss例・鍵変換・c18変更を確認。記事と時間が異なるためイベント同一性と数値の検証が必要。

### D3

[Hong Kong challenges.yaml](https://github.com/aws-ai-community/ai-league-competition-data/blob/a2f585272b504b4746ebe07388ba45212b09035a/competitions/2026/Hong-Kong-Summit/challenges.yaml)。開催 **2026-06-17**。combat log確認値と未確認値を区別しており、treasure 2000とBoss不在のpractice設定を確認。確認フラグを無視して引用しない。

C1–C4はAWS AI CommunityのCommunity Edition。読んだHEADは`cc5060ee58089a363e57529b4cc61d2866deaa3e`、commit日時 **2026-09-07T12:40:58+01:00**。本番ではなく練習環境の実装証拠。

### C1

[challenge_generator.py](https://github.com/aws-ai-community/ai-league-community-edition/blob/cc5060ee58089a363e57529b4cc61d2866deaa3e/lambda/agentic-api/challenge_generator.py#L272)。Bossの3つの生成処理を確認。関数名・コメントより実行内容を優先した。

### C2

[score_calculator.py](https://github.com/aws-ai-community/ai-league-community-edition/blob/cc5060ee58089a363e57529b4cc61d2866deaa3e/lambda/agentic-api/score_calculator.py#L41)。token bonusの分母・clamp・custom model軽減とscore構成を確認。

### C3

[game_runner.py](https://github.com/aws-ai-community/ai-league-community-edition/blob/cc5060ee58089a363e57529b4cc61d2866deaa3e/lambda/agentic-api/game_runner.py#L959)。実行時の時間判定とtoken取得を確認。consumed tile再訪を無効果にする実装もあるが、本番への同一性は未確認。

### C4

[pathfinder-tool/index.py](https://github.com/aws-ai-community/ai-league-community-edition/blob/cc5060ee58089a363e57529b4cc61d2866deaa3e/lambda/pathfinder-tool/index.py#L158)。BFSはwall以外を通し、`get_coins`は近いcoinを順に回る。この版には鍵・ライフ・問題処理時間を含む制約探索がない。社内の既存Toolが同じコードだとは推定しない。
